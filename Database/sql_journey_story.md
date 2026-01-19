# SQL Mastery Through Building an LMS
## From Database Design to Production-Ready System

---

# Part 1: The Foundation - Understanding the Domain

## Meet Jamie: Building "LearnHub" LMS

Jamie is hired to build **LearnHub**, a Learning Management System from scratch. The requirements:

- Students enroll in courses
- Instructors create courses with multiple lessons
- Video content, quizzes, assignments
- Payment integration (one-time purchases, subscriptions)
- Certificates upon completion
- Discussion forums
- Progress tracking
- Multiple pricing tiers

Let's build this system step by step, learning every SQL concept along the way.

---

# Part 2: Database Design - Relations & Normalization

## Chapter 1: Identifying Entities and Relationships

Jamie starts by listing the core entities:

### Core Entities:
- Users (students, instructors, admins)
- Courses
- Lessons
- Enrollments
- Payments
- Certificates
- Quizzes
- Assignments
- Progress tracking
- Reviews

### Relationship Analysis:

#### **User ↔ Course (Many-to-Many)**
- One user can enroll in many courses
- One course can have many students
- **Junction Table Needed**: `enrollments`

#### **Course ↔ Lesson (One-to-Many)**
- One course has many lessons
- One lesson belongs to one course
- **Foreign Key**: `course_id` in lessons table

#### **User ↔ Payment (One-to-Many)**
- One user can make many payments
- One payment belongs to one user
- **Foreign Key**: `user_id` in payments table

#### **Enrollment ↔ Payment (One-to-One or One-to-Many)**
- Depends on business logic
- For LMS: One enrollment can have one initial payment, but may have multiple subscription renewals
- **Design Choice**: Track payment history separately

---

## Chapter 2: First Normal Form (1NF) - Atomic Values

### ❌ Bad Design (Violates 1NF):

```sql
CREATE TABLE courses_bad (
    course_id INT PRIMARY KEY,
    title VARCHAR(200),
    instructor_name VARCHAR(100),
    tags VARCHAR(500),  -- "python,web,backend,django"
    lesson_titles TEXT  -- "Intro|Setup|Variables|Functions"
);
```

**Problems:**
- Can't query "all courses with 'python' tag" easily
- Can't add lesson details (duration, video_url)
- Updating one lesson title requires parsing the entire string

### ✅ Good Design (1NF Compliant):

```sql
CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE course_tags (
    course_id INT,
    tag VARCHAR(50),
    PRIMARY KEY (course_id, tag),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

CREATE TABLE lessons (
    lesson_id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    video_url VARCHAR(500),
    duration_minutes INT,
    order_index INT,
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**Benefits:**
- Each cell contains one value
- Easy to query, update, delete individual items
- Can add attributes to lessons independently

---

## Chapter 3: Second Normal Form (2NF) - No Partial Dependencies

### Understanding Composite Keys in LMS

Jamie creates a quiz results table:

### ❌ Violates 2NF:

```sql
CREATE TABLE quiz_results_bad (
    student_id INT,
    quiz_id INT,
    student_name VARCHAR(100),  -- Depends only on student_id
    quiz_title VARCHAR(200),    -- Depends only on quiz_id
    score INT,
    attempt_date TIMESTAMP,
    PRIMARY KEY (student_id, quiz_id, attempt_date)
);
```

**Problem:** 
- Primary key is (student_id, quiz_id, attempt_date)
- But `student_name` depends only on `student_id` (partial dependency)
- But `quiz_title` depends only on `quiz_id` (partial dependency)

### ✅ 2NF Compliant:

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    user_type ENUM('student', 'instructor', 'admin'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE quizzes (
    quiz_id INT PRIMARY KEY AUTO_INCREMENT,
    lesson_id INT,
    title VARCHAR(200),
    passing_score INT,
    FOREIGN KEY (lesson_id) REFERENCES lessons(lesson_id)
);

CREATE TABLE quiz_attempts (
    attempt_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    quiz_id INT NOT NULL,
    score INT,
    attempt_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (student_id) REFERENCES users(user_id),
    FOREIGN KEY (quiz_id) REFERENCES quizzes(quiz_id),
    INDEX idx_student_quiz (student_id, quiz_id)
);
```

---

## Chapter 4: Third Normal Form (3NF) - No Transitive Dependencies

### The Enrollment Challenge

### ❌ Violates 3NF:

```sql
CREATE TABLE enrollments_bad (
    enrollment_id INT PRIMARY KEY,
    student_id INT,
    course_id INT,
    course_title VARCHAR(200),      -- Depends on course_id (transitive)
    course_price DECIMAL(10,2),     -- Depends on course_id (transitive)
    instructor_id INT,              -- Depends on course_id (transitive)
    instructor_name VARCHAR(100),   -- Depends on instructor_id (transitive)
    enrollment_date TIMESTAMP
);
```

**Problem:** 
- `instructor_name` depends on `instructor_id`, which depends on `enrollment_id`
- That's a transitive dependency: A → B → C

### ✅ 3NF Compliant:

```sql
CREATE TABLE courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    instructor_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10,2),
    thumbnail_url VARCHAR(500),
    level ENUM('beginner', 'intermediate', 'advanced'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (instructor_id) REFERENCES users(user_id),
    INDEX idx_instructor (instructor_id)
);

CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP NULL,
    progress_percentage DECIMAL(5,2) DEFAULT 0,
    FOREIGN KEY (student_id) REFERENCES users(user_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id),
    UNIQUE KEY unique_enrollment (student_id, course_id),
    INDEX idx_student (student_id),
    INDEX idx_course (course_id)
);
```

---

# Part 3: JOINs - Retrieving Connected Data

## Chapter 5: Basic JOINs for Common LMS Queries

### Query 1: Get All Courses with Instructor Details (INNER JOIN)

```sql
SELECT 
    c.course_id,
    c.title AS course_title,
    c.price,
    u.name AS instructor_name,
    u.email AS instructor_email
FROM courses c
INNER JOIN users u ON c.instructor_id = u.user_id
WHERE u.user_type = 'instructor';
```

**Use Case:** Display course catalog with instructor info

**Why INNER JOIN?** We only want courses that have valid instructors.

---

### Query 2: Get All Students and Their Enrollments (LEFT JOIN)

```sql
SELECT 
    u.user_id,
    u.name AS student_name,
    u.email,
    e.enrollment_id,
    c.title AS course_title,
    e.progress_percentage
FROM users u
LEFT JOIN enrollments e ON u.user_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id
WHERE u.user_type = 'student';
```

**Use Case:** Admin dashboard showing all students, including those who haven't enrolled yet

**Why LEFT JOIN?** We want ALL students, even those with zero enrollments.

**Result:**
```
user_id | student_name | email           | enrollment_id | course_title     | progress
1       | Alice        | alice@email.com | 101          | SQL Mastery      | 45.50
1       | Alice        | alice@email.com | 102          | Python Basics    | 100.00
2       | Bob          | bob@email.com   | NULL         | NULL             | NULL
```

Bob appears even though he hasn't enrolled in anything.

---

### Query 3: Student Progress Dashboard (Multiple JOINs)

```sql
SELECT 
    u.name AS student_name,
    c.title AS course_title,
    COUNT(DISTINCT l.lesson_id) AS total_lessons,
    COUNT(DISTINCT lp.lesson_id) AS completed_lessons,
    ROUND(COUNT(DISTINCT lp.lesson_id) * 100.0 / COUNT(DISTINCT l.lesson_id), 2) AS completion_percentage
FROM users u
INNER JOIN enrollments e ON u.user_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
INNER JOIN lessons l ON c.course_id = l.course_id
LEFT JOIN lesson_progress lp ON l.lesson_id = lp.lesson_id 
    AND lp.student_id = u.user_id 
    AND lp.completed = TRUE
WHERE u.user_id = 1
GROUP BY u.user_id, c.course_id, u.name, c.title;
```

**Breaking it down:**
1. Start with user
2. INNER JOIN enrollments (only enrolled courses)
3. INNER JOIN courses (get course details)
4. INNER JOIN lessons (all lessons in enrolled courses)
5. LEFT JOIN lesson_progress (completed lessons - NULL if not started)

**New Table Introduced:**

```sql
CREATE TABLE lesson_progress (
    progress_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    lesson_id INT NOT NULL,
    completed BOOLEAN DEFAULT FALSE,
    completed_at TIMESTAMP NULL,
    time_spent_minutes INT DEFAULT 0,
    FOREIGN KEY (student_id) REFERENCES users(user_id),
    FOREIGN KEY (lesson_id) REFERENCES lessons(lesson_id),
    UNIQUE KEY unique_progress (student_id, lesson_id)
);
```

---

### Query 4: Instructor Revenue Report (Complex JOIN with Aggregation)

```sql
SELECT 
    i.name AS instructor_name,
    c.title AS course_title,
    COUNT(DISTINCT e.enrollment_id) AS total_enrollments,
    c.price,
    SUM(p.amount) AS total_revenue,
    AVG(r.rating) AS average_rating,
    COUNT(DISTINCT r.review_id) AS review_count
FROM users i
INNER JOIN courses c ON i.user_id = c.instructor_id
LEFT JOIN enrollments e ON c.course_id = e.course_id
LEFT JOIN payments p ON e.enrollment_id = p.enrollment_id AND p.status = 'completed'
LEFT JOIN reviews r ON c.course_id = r.course_id
WHERE i.user_type = 'instructor'
GROUP BY i.user_id, c.course_id, i.name, c.title, c.price
ORDER BY total_revenue DESC;
```

**Use Case:** Instructor earnings dashboard

**Why mix INNER and LEFT?**
- INNER JOIN courses: Only show instructors who have courses
- LEFT JOIN enrollments/payments: Show courses even with zero sales
- LEFT JOIN reviews: Show courses even without reviews

---

# Part 4: Transactions & ACID - Payment Processing

## Chapter 6: The Payment Flow Problem

Jamie's first implementation:

```sql
-- ❌ DANGEROUS: Without transaction
INSERT INTO enrollments (student_id, course_id) VALUES (1, 101);
INSERT INTO payments (user_id, amount, status) VALUES (1, 99.99, 'completed');
UPDATE users SET account_balance = account_balance - 99.99 WHERE user_id = 1;
```

**What could go wrong?**
1. Server crashes after enrollment but before payment → Free course!
2. Enrollment succeeds, payment fails → Student charged but not enrolled
3. Two students buying the last seat simultaneously → Overselling

---

## Chapter 7: ACID Properties in Action

### A - Atomicity: The Enrollment Transaction

```sql
START TRANSACTION;

-- Step 1: Create enrollment
INSERT INTO enrollments (student_id, course_id, enrolled_at)
VALUES (1, 101, NOW());

-- Get the enrollment_id we just created
SET @enrollment_id = LAST_INSERT_ID();

-- Step 2: Record payment
INSERT INTO payments (
    user_id, 
    enrollment_id, 
    amount, 
    payment_method,
    status,
    transaction_id
)
VALUES (
    1, 
    @enrollment_id, 
    99.99, 
    'credit_card',
    'completed',
    'txn_abc123'
);

-- Step 3: Update course enrollment count
UPDATE courses 
SET enrolled_students = enrolled_students + 1 
WHERE course_id = 101;

-- Step 4: Grant access
INSERT INTO course_access (student_id, course_id, access_expires_at)
VALUES (1, 101, DATE_ADD(NOW(), INTERVAL 1 YEAR));

-- All or nothing!
COMMIT;
```

**If ANY step fails, ROLLBACK happens automatically.**

**Payment Table:**

```sql
CREATE TABLE payments (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    enrollment_id INT NULL,
    subscription_id INT NULL,
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    payment_method ENUM('credit_card', 'paypal', 'stripe', 'bank_transfer'),
    status ENUM('pending', 'completed', 'failed', 'refunded'),
    transaction_id VARCHAR(100) UNIQUE,
    payment_gateway VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    processed_at TIMESTAMP NULL,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (enrollment_id) REFERENCES enrollments(enrollment_id),
    INDEX idx_user (user_id),
    INDEX idx_status (status),
    INDEX idx_transaction (transaction_id)
);
```

---

### C - Consistency: Enforcing Business Rules

**Scenario:** A course has limited seats (max 50 students)

```sql
-- Add constraint to courses table
ALTER TABLE courses ADD COLUMN max_students INT DEFAULT NULL;
ALTER TABLE courses ADD COLUMN enrolled_students INT DEFAULT 0;

-- Enrollment with consistency check
START TRANSACTION;

-- Check seat availability
SELECT max_students, enrolled_students INTO @max, @enrolled
FROM courses
WHERE course_id = 101
FOR UPDATE;  -- Lock this row!

-- Verify seats available
IF @max IS NULL OR @enrolled < @max THEN
    -- Seats available, proceed
    INSERT INTO enrollments (student_id, course_id)
    VALUES (1, 101);
    
    UPDATE courses 
    SET enrolled_students = enrolled_students + 1
    WHERE course_id = 101;
    
    COMMIT;
ELSE
    -- Course full
    ROLLBACK;
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Course is full';
END IF;
```

**Database Constraints for Consistency:**

```sql
-- Ensure progress percentage is valid
ALTER TABLE enrollments 
ADD CONSTRAINT chk_progress 
CHECK (progress_percentage BETWEEN 0 AND 100);

-- Ensure quiz scores are valid
ALTER TABLE quiz_attempts
ADD CONSTRAINT chk_score
CHECK (score >= 0);

-- Prevent negative prices
ALTER TABLE courses
ADD CONSTRAINT chk_price
CHECK (price >= 0);

-- Ensure payment amounts are positive
ALTER TABLE payments
ADD CONSTRAINT chk_amount
CHECK (amount > 0);
```

---

### I - Isolation: Handling Concurrent Enrollments

**Problem:** Two students try to buy the last seat simultaneously.

```
Time | Student A                          | Student B
-----|------------------------------------|---------------------------------
1    | START TRANSACTION                  |
2    | SELECT enrolled_students           | START TRANSACTION
     | (sees 49, thinks seat available)   |
3    |                                    | SELECT enrolled_students
     |                                    | (sees 49, thinks seat available)
4    | UPDATE courses SET enrolled=50     |
5    | COMMIT                             |
6    |                                    | UPDATE courses SET enrolled=51
7    |                                    | COMMIT (OVERSOLD!)
```

**Solution: Proper Isolation Level**

```sql
-- Set appropriate isolation level
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

START TRANSACTION;

-- FOR UPDATE locks the row
SELECT max_students, enrolled_students 
FROM courses 
WHERE course_id = 101
FOR UPDATE;

-- Other transactions wait here until this transaction completes

-- Check and enroll
IF (@enrolled < @max) THEN
    -- Enroll student
    -- Update count
END IF;

COMMIT;  -- Lock released
```

**Isolation Levels Comparison for LMS:**

| Level | Use Case | Trade-off |
|-------|----------|-----------|
| READ UNCOMMITTED | Never use | Can see dirty data |
| READ COMMITTED | Viewing course catalog | Good for reads, possible phantom reads |
| REPEATABLE READ | Payment processing | Prevents most issues, MySQL default |
| SERIALIZABLE | Limited seat enrollments | Safest, slowest |

---

### D - Durability: Payment Confirmation

```sql
START TRANSACTION;

-- Process payment
INSERT INTO payments (user_id, amount, status, transaction_id)
VALUES (1, 99.99, 'completed', 'txn_xyz789');

-- Create enrollment
INSERT INTO enrollments (student_id, course_id)
VALUES (1, 101);

COMMIT;  -- At this point, data is written to disk

-- Even if server crashes here, the enrollment and payment persist
```

**Behind the scenes:**
1. Changes written to transaction log (write-ahead logging)
2. COMMIT ensures log is flushed to disk
3. On recovery, database replays transaction log

---

# Part 5: Advanced Scenarios & Edge Cases

## Chapter 8: Refund Handling

**Business Logic:**
- Student can request refund within 30 days
- Must not have completed >20% of course
- Refund voids the enrollment

```sql
DELIMITER //

CREATE PROCEDURE process_refund(
    IN p_enrollment_id INT,
    OUT p_result VARCHAR(100)
)
BEGIN
    DECLARE v_payment_id INT;
    DECLARE v_amount DECIMAL(10,2);
    DECLARE v_enrolled_date TIMESTAMP;
    DECLARE v_progress DECIMAL(5,2);
    DECLARE v_days_since_enrollment INT;
    
    -- Start transaction
    START TRANSACTION;
    
    -- Get enrollment details
    SELECT 
        enrolled_at,
        progress_percentage
    INTO v_enrolled_date, v_progress
    FROM enrollments
    WHERE enrollment_id = p_enrollment_id
    FOR UPDATE;  -- Lock this enrollment
    
    -- Calculate days since enrollment
    SET v_days_since_enrollment = DATEDIFF(NOW(), v_enrolled_date);
    
    -- Check refund eligibility
    IF v_days_since_enrollment > 30 THEN
        SET p_result = 'ERROR: Refund period expired';
        ROLLBACK;
    ELSEIF v_progress > 20 THEN
        SET p_result = 'ERROR: Course progress exceeds 20%';
        ROLLBACK;
    ELSE
        -- Find original payment
        SELECT payment_id, amount
        INTO v_payment_id, v_amount
        FROM payments
        WHERE enrollment_id = p_enrollment_id
        AND status = 'completed'
        LIMIT 1;
        
        -- Create refund payment record
        INSERT INTO payments (
            user_id,
            enrollment_id,
            amount,
            payment_method,
            status,
            transaction_id
        )
        SELECT 
            user_id,
            enrollment_id,
            -amount,  -- Negative amount for refund
            payment_method,
            'refunded',
            CONCAT('refund_', transaction_id)
        FROM payments
        WHERE payment_id = v_payment_id;
        
        -- Update original payment status
        UPDATE payments
        SET status = 'refunded'
        WHERE payment_id = v_payment_id;
        
        -- Remove enrollment (soft delete)
        UPDATE enrollments
        SET completed_at = NOW(),
            progress_percentage = 0
        WHERE enrollment_id = p_enrollment_id;
        
        -- Update course enrollment count
        UPDATE courses c
        INNER JOIN enrollments e ON c.course_id = e.course_id
        SET c.enrolled_students = c.enrolled_students - 1
        WHERE e.enrollment_id = p_enrollment_id;
        
        COMMIT;
        SET p_result = CONCAT('SUCCESS: Refunded $', v_amount);
    END IF;
END//

DELIMITER ;

-- Usage:
CALL process_refund(101, @result);
SELECT @result;
```

---

## Chapter 9: Subscription-Based Access

**Business Model:**
- Monthly subscription: $29/month for all courses
- Annual subscription: $290/year (save 17%)
- Auto-renewal
- Grace period handling

```sql
CREATE TABLE subscriptions (
    subscription_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    plan_type ENUM('monthly', 'annual'),
    status ENUM('active', 'cancelled', 'expired', 'past_due'),
    start_date DATE NOT NULL,
    current_period_end DATE NOT NULL,
    next_billing_date DATE,
    cancel_at_period_end BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX idx_user (user_id),
    INDEX idx_status (status),
    INDEX idx_billing_date (next_billing_date)
);

CREATE TABLE subscription_payments (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    subscription_id INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    billing_period_start DATE,
    billing_period_end DATE,
    status ENUM('pending', 'completed', 'failed'),
    retry_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (subscription_id) REFERENCES subscriptions(subscription_id)
);
```

**Subscription Renewal Transaction:**

```sql
DELIMITER //

CREATE PROCEDURE renew_subscription(
    IN p_subscription_id INT
)
BEGIN
    DECLARE v_user_id INT;
    DECLARE v_plan_type VARCHAR(20);
    DECLARE v_amount DECIMAL(10,2);
    DECLARE v_current_end DATE;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        -- If payment fails, mark subscription as past_due
        UPDATE subscriptions
        SET status = 'past_due'
        WHERE subscription_id = p_subscription_id;
        
        ROLLBACK;
    END;
    
    START TRANSACTION;
    
    -- Get subscription details
    SELECT user_id, plan_type, current_period_end
    INTO v_user_id, v_plan_type, v_current_end
    FROM subscriptions
    WHERE subscription_id = p_subscription_id
    AND status = 'active'
    FOR UPDATE;
    
    -- Calculate amount
    SET v_amount = CASE 
        WHEN v_plan_type = 'monthly' THEN 29.00
        WHEN v_plan_type = 'annual' THEN 290.00
    END;
    
    -- Create payment record
    INSERT INTO subscription_payments (
        subscription_id,
        amount,
        billing_period_start,
        billing_period_end,
        status
    )
    VALUES (
        p_subscription_id,
        v_amount,
        v_current_end,
        DATE_ADD(v_current_end, INTERVAL IF(v_plan_type='monthly', 1, 12) MONTH),
        'completed'
    );
    
    -- Update subscription
    UPDATE subscriptions
    SET 
        current_period_end = DATE_ADD(v_current_end, INTERVAL IF(v_plan_type='monthly', 1, 12) MONTH),
        next_billing_date = DATE_ADD(v_current_end, INTERVAL IF(v_plan_type='monthly', 1, 12) MONTH),
        status = 'active'
    WHERE subscription_id = p_subscription_id;
    
    COMMIT;
END//

DELIMITER ;
```

**Scheduled Job (runs daily):**

```sql
-- Find subscriptions due for renewal
SELECT subscription_id
FROM subscriptions
WHERE status = 'active'
AND next_billing_date = CURDATE()
AND cancel_at_period_end = FALSE;

-- For each subscription, call renew_subscription()
```

---

## Chapter 10: Data Integrity Edge Cases

### Edge Case 1: Student Deletes Account Mid-Course

**Problem:** What happens to enrollment, progress, payments?

**Solution: Soft Delete Pattern**

```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;
ALTER TABLE users ADD COLUMN deletion_reason TEXT NULL;

-- Don't actually delete user
UPDATE users 
SET deleted_at = NOW(),
    deletion_reason = 'User requested account deletion',
    email = CONCAT('deleted_', user_id, '@deleted.com')  -- Anonymize
WHERE user_id = 1;

-- Keep enrollments/payments for records
-- But prevent login
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;

-- All queries use the view
SELECT * FROM active_users WHERE email = 'student@email.com';
```

---

### Edge Case 2: Instructor Deletes Course with Active Students

**Problem:** Students paid for lifetime access, but course is deleted.

**Solution: Cascade Rules + Archive**

```sql
-- Archive table
CREATE TABLE archived_courses (
    archive_id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT,
    course_data JSON,  -- Store entire course as JSON
    archived_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    archived_by INT,
    reason TEXT
);

DELIMITER //

CREATE PROCEDURE archive_course(
    IN p_course_id INT,
    IN p_reason TEXT
)
BEGIN
    START TRANSACTION;
    
    -- Archive course with all lessons
    INSERT INTO archived_courses (course_id, course_data, reason)
    SELECT 
        c.course_id,
        JSON_OBJECT(
            'title', c.title,
            'description', c.description,
            'lessons', (
                SELECT JSON_ARRAYAGG(
                    JSON_OBJECT(
                        'title', l.title,
                        'content', l.content,
                        'video_url', l.video_url
                    )
                )
                FROM lessons l
                WHERE l.course_id = c.course_id
            )
        ),
        p_reason
    FROM courses c
    WHERE c.course_id = p_course_id;
    
    -- Keep enrollments active (students still have access to archived content)
    -- Just mark course as archived
    UPDATE courses
    SET status = 'archived',
        updated_at = NOW()
    WHERE course_id = p_course_id;
    
    COMMIT;
END//

DELIMITER ;
```

---

### Edge Case 3: Duplicate Payment (User Clicks "Pay" Twice)

**Problem:** Payment gateway processes twice before we can update DB.

**Solution: Idempotency Key**

```sql
ALTER TABLE payments ADD COLUMN idempotency_key VARCHAR(100) UNIQUE;

-- Client generates unique key
-- e.g., 'enroll_user1_course101_20240119_abc123'

START TRANSACTION;

-- Check if payment with this key already exists
SELECT payment_id INTO @existing_payment
FROM payments
WHERE idempotency_key = 'enroll_user1_course101_20240119_abc123';

IF @existing_payment IS NOT NULL THEN
    -- Payment already processed, return existing enrollment
    SELECT * FROM enrollments WHERE enrollment_id = (
        SELECT enrollment_id FROM payments WHERE payment_id = @existing_payment
    );
    ROLLBACK;
ELSE
    -- Process new payment
    INSERT INTO payments (user_id, amount, idempotency_key, ...)
    VALUES (1, 99.99, 'enroll_user1_course101_20240119_abc123', ...);
    
    -- Create enrollment
    -- ...
    
    COMMIT;
END IF;
```

---

## Chapter 11: Denormalization for Performance

After 1 year, LearnHub has:
- 100,000 students
- 5,000 courses
- 2 million enrollments
- Homepage showing "popular courses" is SLOW

### The Slow Query:

```sql
-- Takes 5+ seconds!
SELECT 
    c.course_id,
    c.title,
    u.name AS instructor_name,
    COUNT(DISTINCT e.enrollment_id) AS total_enrollments,
    AVG(r.rating) AS avg_rating,
    COUNT(DISTINCT r.review_id) AS review_count
FROM courses c
INNER JOIN users u ON c.instructor_id = u.user_id
LEFT JOIN enrollments e ON c.course_id = e.course_id
LEFT JOIN reviews r ON c.course_id = r.course_id
GROUP BY c.course_id
ORDER BY total_enrollments DESC
LIMIT 10;
```

**Why slow?**
- Joins 4 tables
- Aggregates 2 million enrollments
- Runs on EVERY homepage visit
- No indexes on aggregation columns

---

### Denormalization Strategy 1: Computed Columns

```sql
ALTER TABLE courses 
ADD COLUMN total_enrollments INT DEFAULT 0,
ADD COLUMN avg_rating DECIMAL(3,2) DEFAULT 0,
ADD COLUMN review_count INT DEFAULT 0,
ADD COLUMN last_enrollment_date TIMESTAMP NULL;

-- Update these with triggers
DELIMITER //

CREATE TRIGGER after_enrollment_insert
AFTER INSERT ON enrollments
FOR EACH ROW
BEGIN
    UPDATE courses
    SET total_enrollments = total_enrollments + 1,
        last_enrollment_date = NEW.enrolled_at
    WHERE course_id = NEW.course_id;
END//

CREATE TRIGGER after_review_insert
AFTER INSERT ON reviews
FOR EACH ROW
BEGIN
    UPDATE courses
    SET avg_rating = (
            SELECT AVG(rating) 
            FROM reviews 
            WHERE course_id = NEW.course_id
        ),
        review_count = review_count + 1
    WHERE course_id = NEW.course_id;
END//

DELIMITER ;

-- Now the query is FAST:
SELECT 
    c.course_id,
    c.title,
    u.name AS instructor_name,
    c.total_enrollments,
    c.avg_rating,
    c.review_count
FROM courses c
INNER JOIN users u ON c.instructor_id = u.user_id
WHERE c.status = 'active'
ORDER BY c.total_enrollments DESC
LIMIT 10;
```

**Trade-offs:**
- ✅ Query went from 5 seconds to 50ms
- ✅ Simple to maintain
- ❌ Extra storage space
- ❌ Writes are slightly slower (trigger overhead)

---

### Denormalization Strategy 2: Materialized View (Summary Table)

For analytics dashboard showing daily revenue:

```sql
CREATE TABLE daily_revenue_summary (
    summary_date DATE PRIMARY KEY,
    total_revenue DECIMAL(12,2),
    total_transactions INT,
    new_enrollments INT,
    new_subscriptions INT,
    refund_amount DECIMAL(12,2),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Populate daily (scheduled job at midnight)
INSERT INTO daily_revenue_summary (summary_date, total_revenue, total_transactions, new_enrollments)
SELECT 
    DATE(created_at) as summary_date,
    SUM(amount) as total_revenue,
    COUNT(*) as total_transactions,
    COUNT(DISTINCT enrollment_id) as new_enrollments
FROM payments
WHERE status = 'completed'
AND DATE(created_at) = CURDATE() - INTERVAL 1 DAY
GROUP BY DATE(created_at)
ON DUPLICATE KEY UPDATE
    total_revenue = VALUES(total_revenue),
    total_transactions = VALUES(total_transactions),
    new_enrollments = VALUES(new_enrollments);

-- Dashboard query is now instant
SELECT * FROM daily_revenue_summary
WHERE summary_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
ORDER BY summary_date DESC;
```

---

# Part 6: Real-World Production Scenarios

## Chapter 12: Handling Peak Load (Flash Sale)

**Scenario:** LearnHub runs a 24-hour sale - all courses 50% off. Traffic spikes 100x.

### Problem: Database Overload

```sql
-- This query runs thousands of times per second:
SELECT * FROM courses WHERE course_id = 101;
```

### Solution: Multi-Layer Caching

**Layer 1: Application Cache (Redis)**

```javascript
// Pseudocode
async function getCourse(courseId) {
    // Check Redis first
    const cached = await redis.get(`course:${courseId}`);
    if (cached) {
        return JSON.parse(cached);
    }
    
    // Cache miss - query database
    const course = await db.query(
        'SELECT * FROM courses WHERE course_id = ?', 
        [courseId]
    );
    
    // Cache for 1 hour
    await redis.setex(
        `course:${courseId}`, 
        3600, 
        JSON.stringify(course)
    );
    
    return course;
}
```

**Layer 2: Database Query Cache**

```sql
-- MySQL query cache (deprecated in MySQL 8.0, use ProxySQL instead)
-- Or use read replicas for SELECT queries

-- Write to master
INSERT INTO enrollments (student_id, course_id) VALUES (1, 101);

-- Read from replica
SELECT * FROM courses WHERE course_id = 101;
```

**Layer 3: Connection Pooling**

```javascript
// Don't create new connection per request
const pool = mysql.createPool({
    host: 'localhost',
    user: 'app_user',
    password: 'password',
    database: 'learnhub',
    connectionLimit: 100  // Reuse connections
});
```

---

## Chapter 13: Ensuring Payment Data Integrity

### Critical Payment Rules:

1. **No double charging**
2. **No lost payments**
3. **Accurate financial records**
4. **Audit trail for all transactions**

### Solution: Payment State Machine

```sql
CREATE TABLE payment_events (
    event_id INT PRIMARY KEY AUTO_INCREMENT,
    payment_id INT NOT NULL,
    event_type ENUM(
        'initiated',
        'payment_gateway_called',
        'payment_gateway_success',
        'payment_gateway_failed',
        'enrollment_created',
        'completion_confirmed',
        'failed'
    ),
    event_data JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (payment_id) REFERENCES payments(payment_id),
    INDEX idx_payment (payment_id),
    INDEX idx_created (created_at)
);

DELIMITER //

CREATE PROCEDURE process_course_purchase(
    IN p_user_id INT,
    IN p_course_id INT,
    IN p_payment_method VARCHAR(50),
    IN p_transaction_id VARCHAR(100),
    OUT p_enrollment_id INT,
    OUT p_status VARCHAR(50)
)
BEGIN
    DECLARE v_payment_id INT;
    DECLARE v_amount DECIMAL(10,2);
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        -- Log failure
        INSERT INTO payment_events (payment_id, event_type, event_data)
        VALUES (v_payment_id, 'failed', JSON_OBJECT('error', 'Database error'));
        
        ROLLBACK;
        SET p_status = 'FAILED';
    END;
    
    START TRANSACTION;
    
    -- Get course price
    SELECT price INTO v_amount
    FROM courses
    WHERE course_id = p_course_id
    FOR UPDATE;  -- Lock course row
    
    -- Create payment record (pending state)
    INSERT INTO payments (
        user_id, 
        amount, 
        payment_method, 
        status, 
        transaction_id,
        created_at
    )
    VALUES (
        p_user_id, 
        v_amount, 
        p_payment_method, 
        'pending',
        p_transaction_id,
        NOW()
    );
    
    SET v_payment_id = LAST_INSERT_ID();
    
    -- Log: Payment initiated
    INSERT INTO payment_events (payment_id, event_type, event_data)
    VALUES (v_payment_id, 'initiated', JSON_OBJECT(
        'user_id', p_user_id,
        'course_id', p_course_id,
        'amount', v_amount
    ));
    
    -- Simulate payment gateway call (in reality, this is an external API)
    -- For this example, assume success
    
    -- Log: Gateway success
    INSERT INTO payment_events (payment_id, event_type, event_data)
    VALUES (v_payment_id, 'payment_gateway_success', JSON_OBJECT(
        'transaction_id', p_transaction_id
    ));
    
    -- Update payment status
    UPDATE payments
    SET status = 'completed',
        processed_at = NOW()
    WHERE payment_id = v_payment_id;
    
    -- Create enrollment
    INSERT INTO enrollments (student_id, course_id, enrolled_at)
    VALUES (p_user_id, p_course_id, NOW());
    
    SET p_enrollment_id = LAST_INSERT_ID();
    
    -- Link payment to enrollment
    UPDATE payments
    SET enrollment_id = p_enrollment_id
    WHERE payment_id = v_payment_id;
    
    -- Log: Enrollment created
    INSERT INTO payment_events (payment_id, event_type, event_data)
    VALUES (v_payment_id, 'enrollment_created', JSON_OBJECT(
        'enrollment_id', p_enrollment_id
    ));
    
    -- Update course stats
    UPDATE courses
    SET total_enrollments = total_enrollments + 1
    WHERE course_id = p_course_id;
    
    -- Log: Completion
    INSERT INTO payment_events (payment_id, event_type, event_data)
    VALUES (v_payment_id, 'completion_confirmed', JSON_OBJECT(
        'enrollment_id', p_enrollment_id,
        'payment_id', v_payment_id
    ));
    
    COMMIT;
    
    SET p_status = 'SUCCESS';
END//

DELIMITER ;

-- Usage:
CALL process_course_purchase(
    1,           -- user_id
    101,         -- course_id
    'stripe',    -- payment_method
    'txn_xyz',   -- transaction_id
    @enrollment_id,
    @status
);

SELECT @enrollment_id, @status;

-- Audit trail
SELECT * FROM payment_events WHERE payment_id = 1 ORDER BY created_at;
```

**Output:**
```
event_id | payment_id | event_type              | event_data                          | created_at
1        | 1          | initiated               | {"user_id":1,"course_id":101,...}   | 2024-01-19 10:00:00
2        | 1          | payment_gateway_success | {"transaction_id":"txn_xyz"}        | 2024-01-19 10:00:01
3        | 1          | enrollment_created      | {"enrollment_id":501}               | 2024-01-19 10:00:02
4        | 1          | completion_confirmed    | {"enrollment_id":501,"payment...}   | 2024-01-19 10:00:03
```

---

## Chapter 14: Preventing Race Conditions

### Scenario: Limited Time Offer with Limited Seats

```
Flash Sale: First 100 students get 90% off
```

**Problem:** 1,000 students click "Enroll" simultaneously.

### Solution: Pessimistic Locking

```sql
DELIMITER //

CREATE PROCEDURE enroll_in_flash_sale(
    IN p_user_id INT,
    IN p_sale_id INT,
    OUT p_result VARCHAR(100)
)
BEGIN
    DECLARE v_seats_available INT;
    DECLARE v_max_seats INT;
    DECLARE v_course_id INT;
    DECLARE v_discount_price DECIMAL(10,2);
    
    START TRANSACTION;
    
    -- Lock the flash sale row
    SELECT 
        course_id,
        max_seats,
        seats_taken,
        discount_price
    INTO 
        v_course_id,
        v_max_seats,
        v_seats_available,
        v_discount_price
    FROM flash_sales
    WHERE sale_id = p_sale_id
    AND start_time <= NOW()
    AND end_time >= NOW()
    FOR UPDATE;  -- CRITICAL: This locks the row
    
    -- Check if seats available
    IF v_seats_available >= v_max_seats THEN
        SET p_result = 'SOLD_OUT';
        ROLLBACK;
    ELSE
        -- Check if user already enrolled
        IF EXISTS (
            SELECT 1 FROM enrollments 
            WHERE student_id = p_user_id 
            AND course_id = v_course_id
        ) THEN
            SET p_result = 'ALREADY_ENROLLED';
            ROLLBACK;
        ELSE
            -- Create enrollment with special price
            INSERT INTO enrollments (student_id, course_id)
            VALUES (p_user_id, v_course_id);
            
            INSERT INTO payments (user_id, enrollment_id, amount, status)
            VALUES (
                p_user_id, 
                LAST_INSERT_ID(), 
                v_discount_price, 
                'completed'
            );
            
            -- Increment seats taken
            UPDATE flash_sales
            SET seats_taken = seats_taken + 1
            WHERE sale_id = p_sale_id;
            
            COMMIT;
            SET p_result = 'SUCCESS';
        END IF;
    END IF;
END//

DELIMITER ;
```

**What `FOR UPDATE` does:**
- Locks the row until transaction completes
- Other transactions wait in line
- Prevents two students from seeing "49 seats taken" and both enrolling

---

## Chapter 15: Database Partitioning for Scale

After 3 years, LearnHub has:
- 50 million payment records
- Queries on recent payments are slow
- Historical data rarely accessed

### Solution: Table Partitioning

```sql
-- Partition payments table by date
ALTER TABLE payments
PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Query only searches relevant partition
SELECT * FROM payments
WHERE created_at >= '2024-01-01'
AND created_at < '2025-01-01';
-- Only scans p2024 partition!

-- Archive old partitions to separate storage
ALTER TABLE payments 
DROP PARTITION p2022;  -- Removes old data instantly
```

---

# Part 7: Advanced Interview Scenarios

## Chapter 16: Common Interview Questions & Answers

### Q1: "Design the database schema for course prerequisites"

**Requirement:** Some courses require completing other courses first.

```sql
-- Self-referencing table
CREATE TABLE course_prerequisites (
    course_id INT NOT NULL,
    prerequisite_course_id INT NOT NULL,
    PRIMARY KEY (course_id, prerequisite_course_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id),
    FOREIGN KEY (prerequisite_course_id) REFERENCES courses(course_id),
    CONSTRAINT prevent_self_prerequisite CHECK (course_id != prerequisite_course_id)
);

-- Example: Advanced SQL requires SQL Basics
INSERT INTO course_prerequisites VALUES (201, 101);

-- Query: Can user enroll in course 201?
SELECT 
    CASE 
        WHEN COUNT(*) = 0 THEN 'CAN_ENROLL'
        ELSE 'MISSING_PREREQUISITES'
    END AS enrollment_status
FROM course_prerequisites cp
WHERE cp.course_id = 201
AND NOT EXISTS (
    SELECT 1 
    FROM enrollments e
    WHERE e.student_id = 1
    AND e.course_id = cp.prerequisite_course_id
    AND e.completed_at IS NOT NULL
);
```

---

### Q2: "How would you prevent instructors from seeing student payment info?"

**Answer: Row-Level Security with Views**

```sql
-- Create view that only shows relevant data
CREATE VIEW instructor_enrollments AS
SELECT 
    e.enrollment_id,
    e.student_id,
    u.name AS student_name,
    u.email AS student_email,
    e.enrolled_at,
    e.progress_percentage,
    c.course_id,
    c.instructor_id
FROM enrollments e
INNER JOIN users u ON e.student_id = u.user_id
INNER JOIN courses c ON e.course_id = c.course_id;

-- Grant instructors access to view, not base table
GRANT SELECT ON instructor_enrollments TO instructor_role;
REVOKE SELECT ON payments FROM instructor_role;

-- Instructor can only see their own course enrollments
SELECT * FROM instructor_enrollments
WHERE instructor_id = @current_instructor_id;
```

---

### Q3: "What happens if a payment succeeds but enrollment creation fails?"

**Answer: This is why we use transactions!**

```sql
START TRANSACTION;

-- Payment processed by external gateway (already succeeded)
INSERT INTO payments (user_id, amount, status, transaction_id)
VALUES (1, 99.99, 'completed', 'external_txn_123');

-- Enrollment creation fails (disk full, constraint violation, etc.)
INSERT INTO enrollments (student_id, course_id)
VALUES (1, 101);  -- FAILS!

-- Transaction rolls back, payment record is removed
ROLLBACK;
```

**Problem:** Payment already processed externally!

**Real Solution: Two-Phase Commit**

```sql
-- Phase 1: Create payment in 'pending' state
INSERT INTO payments (user_id, amount, status, transaction_id)
VALUES (1, 99.99, 'pending', 'external_txn_123');

-- Call external payment gateway
-- If gateway succeeds:

START TRANSACTION;

-- Update payment to completed
UPDATE payments SET status = 'completed' WHERE payment_id = 1;

-- Create enrollment
INSERT INTO enrollments (student_id, course_id) VALUES (1, 101);

COMMIT;

-- If anything fails after gateway success, we have a payment record
-- to reconcile/refund later
```

---

### Q4: "Design a system to detect and prevent payment fraud"

```sql
CREATE TABLE fraud_detection_rules (
    rule_id INT PRIMARY KEY AUTO_INCREMENT,
    rule_name VARCHAR(100),
    rule_type ENUM('velocity', 'amount', 'location', 'pattern'),
    rule_condition JSON,
    action ENUM('flag', 'block', 'require_verification'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE fraud_alerts (
    alert_id INT PRIMARY KEY AUTO_INCREMENT,
    payment_id INT,
    user_id INT,
    rule_id INT,
    alert_level ENUM('low', 'medium', 'high', 'critical'),
    alert_data JSON,
    status ENUM('pending', 'reviewed', 'false_positive', 'confirmed_fraud'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    reviewed_at TIMESTAMP NULL,
    reviewed_by INT NULL,
    FOREIGN KEY (payment_id) REFERENCES payments(payment_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Fraud check before processing payment
DELIMITER //

CREATE PROCEDURE check_fraud(
    IN p_user_id INT,
    IN p_amount DECIMAL(10,2),
    OUT p_fraud_score INT,
    OUT p_should_block BOOLEAN
)
BEGIN
    DECLARE v_payment_count_24h INT;
    DECLARE v_total_spent_24h DECIMAL(10,2);
    DECLARE v_avg_transaction DECIMAL(10,2);
    
    SET p_fraud_score = 0;
    SET p_should_block = FALSE;
    
    -- Rule 1: More than 5 payments in 24 hours (velocity check)
    SELECT COUNT(*), COALESCE(SUM(amount), 0)
    INTO v_payment_count_24h, v_total_spent_24h
    FROM payments
    WHERE user_id = p_user_id
    AND created_at >= NOW() - INTERVAL 24 HOUR
    AND status != 'failed';
    
    IF v_payment_count_24h > 5 THEN
        SET p_fraud_score = p_fraud_score + 50;
        
        INSERT INTO fraud_alerts (user_id, rule_id, alert_level, alert_data)
        VALUES (
            p_user_id, 
            1, 
            'high', 
            JSON_OBJECT('payment_count', v_payment_count_24h, 'period', '24h')
        );
    END IF;
    
    -- Rule 2: Single transaction >$500 (high amount)
    IF p_amount > 500 THEN
        SET p_fraud_score = p_fraud_score + 30;
    END IF;
    
    -- Rule 3: Transaction amount >> user's average
    SELECT AVG(amount) INTO v_avg_transaction
    FROM payments
    WHERE user_id = p_user_id
    AND status = 'completed';
    
    IF p_amount > v_avg_transaction * 5 THEN
        SET p_fraud_score = p_fraud_score + 40;
    END IF;
    
    -- Decision
    IF p_fraud_score >= 100 THEN
        SET p_should_block = TRUE;
    END IF;
END//

DELIMITER ;

-- Usage in payment flow:
CALL check_fraud(1, 999.99, @fraud_score, @should_block);

IF @should_block THEN
    -- Block payment, require manual review
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Payment flagged for fraud review';
ELSE
    -- Process payment
END IF;
```

---

# Part 8: Performance Optimization

## Chapter 17: Indexing Strategy

```sql
-- Bad: No indexes
CREATE TABLE lesson_progress (
    progress_id INT PRIMARY KEY,
    student_id INT,
    lesson_id INT,
    completed BOOLEAN
);

-- Query is slow:
SELECT * FROM lesson_progress 
WHERE student_id = 1;  -- Full table scan!

-- Good: Strategic indexes
CREATE TABLE lesson_progress (
    progress_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    lesson_id INT NOT NULL,
    completed BOOLEAN DEFAULT FALSE,
    completed_at TIMESTAMP NULL,
    
    -- Composite index for common query pattern
    INDEX idx_student_lesson (student_id, lesson_id),
    
    -- Index for "find all completed lessons"
    INDEX idx_completed (completed, completed_at),
    
    FOREIGN KEY (student_id) REFERENCES users(user_id),
    FOREIGN KEY (lesson_id) REFERENCES lessons(lesson_id)
);

-- Use EXPLAIN to verify index usage:
EXPLAIN SELECT * FROM lesson_progress 
WHERE student_id = 1 AND lesson_id = 50;

-- Should show: "Using index"
```

**Index Guidelines:**
- Index foreign keys
- Index columns in WHERE clauses
- Index columns in ORDER BY
- Composite indexes for multi-column queries
- Don't over-index (slows down writes)

---

## Chapter 18: Query Optimization

### Slow Query:

```sql
-- Find students who completed all lessons in a course
SELECT u.user_id, u.name
FROM users u
WHERE user_type = 'student'
AND (
    SELECT COUNT(DISTINCT l.lesson_id)
    FROM lessons l
    WHERE l.course_id = 101
) = (
    SELECT COUNT(DISTINCT lp.lesson_id)
    FROM lesson_progress lp
    INNER JOIN lessons l ON lp.lesson_id = l.lesson_id
    WHERE lp.student_id = u.user_id
    AND l.course_id = 101
    AND lp.completed = TRUE
);
```

### Optimized Version:

```sql
-- Use a single JOIN instead of subqueries
SELECT u.user_id, u.name
FROM users u
INNER JOIN (
    SELECT lp.student_id
    FROM lesson_progress lp
    INNER JOIN lessons l ON lp.lesson_id = l.lesson_id
    WHERE l.course_id = 101
    AND lp.completed = TRUE
    GROUP BY lp.student_id
    HAVING COUNT(DISTINCT lp.lesson_id) = (
        SELECT COUNT(*) FROM lessons WHERE course_id = 101
    )
) completed_students ON u.user_id = completed_students.student_id
WHERE u.user_type = 'student';
```

---

# Summary: Complete Concept Checklist

## ✅ Relations
- One-to-Many (user → orders)
- Many-to-Many (courses ↔ students via enrollments)
- One-to-One (user → user_profile)
- Self-referencing (course prerequisites)

## ✅ Normalization
- 1NF: Atomic values, no arrays
- 2NF: No partial dependencies on composite keys
- 3NF: No transitive dependencies
- When to apply: Always start normalized

## ✅ Denormalization
- Computed columns (total_enrollments in courses)
- Summary tables (daily_revenue_summary)
- When to apply: Read-heavy systems, performance critical
- Trade-offs: Storage + complexity for speed

## ✅ JOINs
- INNER JOIN: Only matching records
- LEFT JOIN: All from left table
- Multiple JOINs: Building complex reports
- JOIN with aggregation: COUNT, SUM, AVG

## ✅ Transactions
- START TRANSACTION, COMMIT, ROLLBACK
- SAVEPOINT for partial rollback
- When to use: Multi-step operations that must succeed/fail together
- Error handling patterns

## ✅ ACID
- **Atomicity**: Payment + enrollment both succeed or both fail
- **Consistency**: Constraints prevent invalid states
- **Isolation**: FOR UPDATE prevents race conditions
- **Durability**: COMMIT guarantees persistence

## ✅ Advanced Topics
- Stored procedures for complex business logic
- Triggers for automatic updates
- Views for security and simplification
- Partitioning for massive tables
- Indexing strategy
- Fraud detection
- Audit trails
- Soft deletes
- Idempotency

## ✅ Production Patterns
- Payment state machines
- Refund handling
- Subscription renewals
- Rate limiting with database
- Caching strategies
- Read replicas
- Connection pooling

---

# Interview Preparation Checklist

When asked about SQL in interviews, use the LMS as your reference:

**"Explain ACID"** → Use the payment + enrollment example

**"When to use transactions?"** → Flash sale with limited seats

**"Difference between normalization and denormalization?"** → Courses table evolution

**"Design a payment system"** → Walk through payment_events table

**"Handle concurrent requests?"** → FOR UPDATE in flash sale

**"Optimize a slow query?"** → Course catalog with JOINs → denormalized columns

**"Prevent data corruption?"** → Foreign keys, constraints, transactions

**"Scale to millions of users?"** → Partitioning, indexes, caching, read replicas

---

*You now have a production-grade understanding of SQL from basics to advanced, all contextualized in a real Learning Management System. Practice by building this schema yourself!*