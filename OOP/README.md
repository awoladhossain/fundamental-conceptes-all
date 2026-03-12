# 🎯 Object-Oriented Programming (OOP)

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [Basic](Basic.md) | OOP Fundamentals | ✅ Done |

## 🎯 Four Pillars of OOP

### 1. Encapsulation
```javascript
class BankAccount {
  #balance = 0;  // Private field

  deposit(amount) {
    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}
```

### 2. Abstraction
```javascript
// Hide complex implementation
class EmailService {
  send(to, subject, body) {
    // Complex SMTP logic hidden
    this.#connect();
    this.#authenticate();
    this.#sendEmail(to, subject, body);
  }
}
```

### 3. Inheritance
```javascript
class Animal {
  speak() { console.log("Sound"); }
}

class Dog extends Animal {
  speak() { console.log("Bark!"); }
}
```

### 4. Polymorphism
```javascript
class Shape {
  area() { throw "Override this"; }
}

class Circle extends Shape {
  constructor(r) { this.r = r; }
  area() { return Math.PI * this.r ** 2; }
}

class Rectangle extends Shape {
  constructor(w, h) { this.w = w; this.h = h; }
  area() { return this.w * this.h; }
}
```

## SOLID Principles

| Principle | Description |
|-----------|-------------|
| **S** - Single Responsibility | One class = One job |
| **O** - Open/Closed | Open for extension, closed for modification |
| **L** - Liskov Substitution | Subclass should be substitutable for parent |
| **I** - Interface Segregation | Many specific interfaces > One general |
| **D** - Dependency Inversion | Depend on abstractions, not concretions |

## 📚 Resources
- [Refactoring Guru - Design Patterns](https://refactoring.guru/)
- [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/9781492077992/)

---
*Last Updated: March 2026*
