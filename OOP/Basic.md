# Object Oriented Programming (OOP)

Object-Oriented Programming (OOP) is a programming paradigm based on the concept of **objects** that contain data and methods. It helps in building scalable, reusable, and maintainable software.

---

## 📌 Core Concepts

### 1. Classes and Objects

- **Class:** A blueprint or template from which objects are created.
- **Object:** An instance of a class.

**Example (Java):**

```java
class Car {
    String brand;
    int year;

    void drive() {
        System.out.println("Driving " + brand);
    }
}

public class Main {
    public static void main(String[] args) {
        Car car1 = new Car();
        car1.brand = "Toyota";
        car1.year = 2022;
        car1.drive();
    }
}
```

### 2. Abstraction

- Hiding implementation details and showing only essential features.
- Achieved using abstract classes and interfaces.

```java
abstract class Animal {
    abstract void sound();
}

class Dog extends Animal {
    void sound() {
        System.out.println("Woof Woof");
    }
}
```

### 3. Encapsulation

- Wrapping data (variables) and methods into a single unit (class).
- Achieved using private fields and public getters/setters.

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

### 4. Inheritance

- Mechanism to acquire properties and behaviors of another class.
- Promotes code reusability.

```java
class Vehicle {
    void start() {
        System.out.println("Vehicle started");
    }
}

class Car extends Vehicle {
    void honk() {
        System.out.println("Car honks");
    }
}
```

### 5. Polymorphism

- Ability of an object to take many forms.
- Two types:
  > Compile-time (Method Overloading)
  > Runtime (Method Overriding)

```java
class Shape {
    void draw() {
        System.out.println("Drawing Shape");
    }
}

class Circle extends Shape {
    void draw() {
        System.out.println("Drawing Circle");
    }
}
```

### 6. Composition

- Instead of inheritance, classes can be composed of other classes.
- Promotes HAS-A relationship

```java
class Engine {
    void start() {
        System.out.println("Engine starts");
    }
}

class Car {
    Engine engine = new Engine();
    void drive() {
        engine.start();
        System.out.println("Car drives");
    }
}
```

### 7. Association, Aggregation, and Composition

- Association: Relationship between two classes.
- Aggregation: Weak relationship (child can exist without parent).
- Composition: Strong relationship (child cannot exist without parent).

### 8. Interfaces

- Defines a contract that classes must follow.
- Supports multiple inheritance in Java.

```java
interface Flyable {
    void fly();
}

class Bird implements Flyable {
    public void fly() {
        System.out.println("Bird flies");
    }
}
```

### 9. Abstract Classes vs Interfaces

| Feature              | Abstract Class                    | Interface                                                        |
| -------------------- | --------------------------------- | ---------------------------------------------------------------- |
| Methods              | Can have both abstract & concrete | Only abstract (Java 7), default/static methods allowed (Java 8+) |
| Multiple Inheritance | ❌ No                             | ✅ Yes (via multiple interfaces)                                 |
| Variables            | Can have instance variables       | Only constants (static final)                                    |

### 10. Method Overloading

Definition: Multiple methods in the same class share the same name but differ in parameter lists (type, number, or order). Compile-time polymorphism.

Example (Java):

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }

    double add(double a, double b) {
        return a + b;
    }

    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println(calc.add(2, 3));        // 5
        System.out.println(calc.add(1, 2, 3));     // 6
        System.out.println(calc.add(1.5, 2.5));    // 4.0
    }
}
```

### 11. Method Overriding

Definition: A subclass provides a specific implementation of a method that is already defined in its superclass. Runtime polymorphism (dynamic dispatch).

Example (Java):

```java
class Animal {
    void sound() {
        System.out.println("Some generic animal sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Woof Woof");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal a1 = new Animal();
        Animal a2 = new Dog();   // reference type Animal, runtime object Dog

        a1.sound(); // Some generic animal sound
        a2.sound(); // Woof Woof  (Dog's overridden method called)
    }
}
```

Notes:

- Overloading: same class, different signatures.
- Overriding: subclass redefines superclass method, same signature; use @Override for clarity.

### 12. Constructor, this, and super (Easy to memorize)

- Constructor: special method that initializes an object.
  - Same name as class, no return type, called with new.
  - If none provided, Java gives a default no-arg constructor.
- this: refers to the current object.
  - Use to disambiguate fields (this.field) or call another constructor (this(...)).
- super: refers to the superclass.
  - Use to call superclass constructor (super(...)) or access superclass members (super.method()).
  - Must be the first statement in a constructor when used.

Quick examples (Java):

```java
// Constructor + this example
class Person {
    String name;
    int age;

    // No-arg constructor calls parameterized constructor
    Person() {
        this("Unknown", 0); // calls Person(String,int)
    }

    // Parameterized constructor
    Person(String name, int age) {
        this.name = name; // this disambiguates field vs parameter
        this.age = age;
    }

    void info() {
        System.out.println(name + " - " + age);
    }
}

class Employee extends Person {
    String company;

    // super(...) calls superclass constructor; must be first statement
    Employee(String name, int age, String company) {
        super(name, age); // initialize Person part
        this.company = company;
    }

    void info() {
        super.info(); // call superclass method
        System.out.println("Company: " + company);
    }

    public static void main(String[] args) {
        Person p = new Person("Alice", 30);
        p.info(); // Alice - 30

        Employee e = new Employee("Bob", 25, "Acme");
        e.info();
        // Output:
        // Bob - 25
        // Company: Acme
    }
}
```

Memorize cheat-sheet:

- Constructor = init object, same name, no return.
- this = current object / call other constructors / disambiguate.
- super = parent object / call parent constructor / access parent members.

### 13. Static vs Dynamic Binding

- Static (compile-time) binding: method call resolved at compile time. Happens with method overloading, and with private, final, and static methods.
- Dynamic (runtime) binding: method call resolved at runtime based on actual object. Happens with overridden instance methods.

Example — static binding (overloading):

```java
class OverloadExample {
    void show(int x) { System.out.println("int: " + x); }
    void show(String s) { System.out.println("String: " + s); }

    public static void main(String[] args) {
        OverloadExample o = new OverloadExample();
        o.show(10);       // compile-time selects show(int)
        o.show("hello");  // compile-time selects show(String)
    }
}
```

Example — dynamic binding (overriding):

```java
class Animal {
    void sound() { System.out.println("Animal sound"); }
}
class Dog extends Animal {
    @Override
    void sound() { System.out.println("Woof"); }
}
public class BindDemo {
    public static void main(String[] args) {
        Animal a = new Dog(); // reference type Animal, object Dog
        a.sound();            // Dog.sound() called at runtime (dynamic binding)
    }
}
```

Cheat: Overloading = static/compile-time. Overriding = dynamic/runtime.

---

### 14. Object Cloning

- Java clone mechanism: Object.clone() creates a field-by-field copy.
- To allow cloning: implement Cloneable and override clone() (call super.clone()).
- Default clone is shallow: object references are copied, not cloned. For deep copy, manually clone referenced objects.

Shallow clone example:

```java
class Address {
    String city;
    Address(String city) { this.city = city; }
}

class Person implements Cloneable {
    String name;
    Address addr;
    Person(String name, Address addr) { this.name = name; this.addr = addr; }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone(); // shallow copy
    }

    public static void main(String[] args) throws Exception {
        Address a1 = new Address("NY");
        Person p1 = new Person("Alice", a1);
        Person p2 = (Person) p1.clone(); // shallow: p1.addr == p2.addr

        p2.addr.city = "LA";
        System.out.println(p1.addr.city); // prints "LA" — shared reference
    }
}
```

Deep clone example (clone referenced objects too):

```java
class Address implements Cloneable {
    String city;
    Address(String city) { this.city = city; }
    @Override protected Address clone() throws CloneNotSupportedException { return (Address) super.clone(); }
}

class Person implements Cloneable {
    String name;
    Address addr;
    Person(String name, Address addr) { this.name = name; this.addr = addr; }

    @Override
    protected Person clone() throws CloneNotSupportedException {
        Person copy = (Person) super.clone();
        copy.addr = this.addr.clone(); // deep clone of address
        return copy;
    }
}
```

Tip: Prefer copy constructors or serialization for complex deep copies.

---

### 15. Inner Classes

Types:

- Member inner class (non-static): associated with an instance, can access outer instance members.
- Static nested class: like a static member, cannot access instance members directly.
- Local class: defined inside a method.
- Anonymous class: inline single-use class (often for listeners or Runnable).

Member inner class example:

```java
class Outer {
    int x = 10;
    class Inner {
        void show() { System.out.println("x = " + x); } // can access outer instance
    }
    public static void main(String[] args) {
        Outer o = new Outer();
        Outer.Inner i = o.new Inner();
        i.show(); // x = 10
    }
}
```

Static nested class example:

```java
class Outer {
    static class Nested {
        static void info() { System.out.println("static nested"); }
    }
    public static void main(String[] args) {
        Outer.Nested.info();
    }
}
```

Anonymous class example (Runnable):

```java
public class AnonDemo {
    public static void main(String[] args) {
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous Runnable");
            }
        });
        t.start();
    }
}
```

Memorize:

- Member inner -> needs outer instance.
- Static nested -> behaves like static member.
- Local -> inside method.
- Anonymous -> single-use inline class.

### 16. Generics with OOP

- Purpose: Enable classes and methods to operate on typed objects while providing compile-time type safety and avoiding casts.
- Use when a class or method should work with many types (collections, wrappers, utilities).

Simple generic class example:

```java
class Box<T> {
    private T value;
    Box(T value) { this.value = value; }
    T get() { return value; }
    void set(T value) { this.value = value; }

    public static void main(String[] args) {
        Box<String> b = new Box<>("hello");
        String s = b.get(); // no cast needed
        System.out.println(s);
    }
}
```

Generic method and bounded type parameter:

```java
class Utils {
    // Generic method
    static <T> void printArray(T[] arr) {
        for (T e : arr) System.out.println(e);
    }

    // Bounded type parameter (only Number or its subclasses)
    static <T extends Number> double sum(T a, T b) {
        return a.doubleValue() + b.doubleValue();
    }
}
```

Cheat:

- T, E, K, V are common type parameter names.
- Use bounds (extends / super) to restrict types.
- Generics are compile-time only (type erasure at runtime).

---

### 17. Dependency Injection (DI)

- Definition: Technique where an object's dependencies are provided from outside rather than created inside. Promotes testability, loose coupling, and easier maintenance.
- Types: Constructor injection (recommended), setter injection, field injection.

Simple Java example (constructor injection):

```java
interface MessageService {
    void send(String msg);
}

class EmailService implements MessageService {
    public void send(String msg) { System.out.println("Email: " + msg); }
}

class UserController {
    private final MessageService service;

    // Dependency provided via constructor (injected)
    UserController(MessageService service) {
        this.service = service;
    }

    void notifyUser(String msg) {
        service.send(msg);
    }

    public static void main(String[] args) {
        MessageService svc = new EmailService();                 // wiring
        UserController ctrl = new UserController(svc);          // inject
        ctrl.notifyUser("Welcome!");
    }
}
```

Notes:

- Inversion of Control (IoC): DI is a form of IoC — the container/framework (or caller) wires dependencies.
- Use DI frameworks (Spring, Guice) for large apps; prefer constructor injection for immutability and easier testing.
- Benefits: easier unit testing (mock dependencies), single responsibility, pluggable implementations.

---

### 18. UML Diagrams for OOP Design

- UML (Unified Modeling Language): visual language to model software structure and behavior.
- Common diagram types for OOP:
  - Class Diagram: shows classes, attributes, methods, and relationships.
  - Sequence Diagram: shows object interactions over time.
  - Package Diagram: groups related classes.

Class diagram basics:

- Class box: three compartments — Name | Attributes | Methods
- Visibility: + public, - private, # protected
- Relationships:
  - Inheritance: solid line with closed hollow arrow (A —|> B)
  - Association: solid line (A — B)
  - Aggregation: hollow diamond (whole —<>— part) (weak)
  - Composition: filled diamond (whole —◆— part) (strong)
  - Dependency: dashed arrow (uses)

Example (textual UML for simple model):

- Car

  - -model: String
  - -engine: Engine
  - +drive(): void

- Engine
  - -type: String
  - +start(): void

Relationships:

- Car _composes_ Engine (filled diamond) — Engine cannot exist logically without Car.

Quick tips to draw useful UML:

- Start with key domain classes and their responsibilities.
- Show only relevant attributes/methods (avoid clutter).
- Use interfaces for contracts and show "implements" with dashed arrow.
- For behavior, add sequence diagrams for critical flows (e.g., login, purchase).

Cheat:

- Class diagram = static structure.
- Sequence diagram = runtime interaction.
- Keep diagrams focused (single responsibility per diagram).

### 19. Design Patterns (Quick & Memorize)

Design patterns solve common design problems. Grouped by purpose.

#### Creational Patterns

- Singleton — ensure one instance.

```java
// Simple thread-safe lazy singleton
class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
```

- Factory — create objects via a factory, hide concrete classes.

```java
interface Shape { void draw(); }
class Circle implements Shape { public void draw(){ System.out.println("Circle"); } }
class Square implements Shape { public void draw(){ System.out.println("Square"); } }

class ShapeFactory {
    static Shape create(String type) {
        switch(type) {
            case "circle": return new Circle();
            case "square": return new Square();
            default: throw new IllegalArgumentException("Unknown");
        }
    }
}
```

- Builder — construct complex object step-by-step (fluent).

```java
class Person {
    private final String name; private final int age; private final String city;
    private Person(Builder b){ name=b.name; age=b.age; city=b.city; }
    static class Builder {
        String name; int age; String city;
        Builder name(String n){ this.name=n; return this; }
        Builder age(int a){ this.age=a; return this; }
        Builder city(String c){ this.city=c; return this; }
        Person build(){ return new Person(this); }
    }
}
// Usage: Person p = new Person.Builder().name("A").age(30).city("X").build();
```

#### Structural Patterns

- Adapter — make incompatible interfaces work together.

```java
interface Power { int getVoltage(); }
class USPower { int voltage() { return 120; } }

class USPowerAdapter implements Power {
    private final USPower up = new USPower();
    public int getVoltage(){ return up.voltage(); }
}
```

- Composite — treat individual objects and compositions uniformly.

```java
interface Component { void show(); }
class Leaf implements Component { private String name; Leaf(String n){name=n;} public void show(){ System.out.println(name); } }
class Composite implements Component {
    private List<Component> children = new ArrayList<>();
    void add(Component c){ children.add(c); }
    public void show(){ for(Component c: children) c.show(); }
}
```

- Decorator — add responsibilities to objects dynamically.

```java
interface Coffee { double cost(); String desc(); }
class SimpleCoffee implements Coffee { public double cost(){return 2; } public String desc(){return "Coffee";} }
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee c; CoffeeDecorator(Coffee c){this.c=c;}
    public double cost(){ return c.cost(); } public String desc(){ return c.desc(); }
}
class Milk extends CoffeeDecorator { Milk(Coffee c){super(c);} public double cost(){return super.cost()+0.5;} public String desc(){return super.desc()+" + milk";} }
```

#### Behavioral Patterns

- Observer — publish/subscribe: observers notified of changes.

```java
interface Observer { void update(String msg); }
class Subject {
    private List<Observer> obs = new ArrayList<>();
    void add(Observer o){ obs.add(o); }
    void notifyAllObservers(String m){ for(Observer o: obs) o.update(m); }
}
class ConcreteObserver implements Observer { private String name; ConcreteObserver(String n){name=n;} public void update(String msg){ System.out.println(name + " got: " + msg); } }
```

- Strategy — swap algorithms at runtime.

```java
interface PaymentStrategy { void pay(int amount); }
class CreditCard implements PaymentStrategy { public void pay(int a){ System.out.println("Paid " + a + " by card"); } }
class PayPal implements PaymentStrategy { public void pay(int a){ System.out.println("Paid " + a + " by PayPal"); } }
class Checkout {
    private PaymentStrategy strategy;
    Checkout(PaymentStrategy s){ this.strategy = s; }
    void doPay(int amt){ strategy.pay(amt); }
}
```

- Command — encapsulate requests as objects (undo/queue).

```java
interface Command { void execute(); }
class Light { void on(){ System.out.println("Light on"); } void off(){ System.out.println("Light off"); } }
class LightOnCommand implements Command {
    private Light light;
    LightOnCommand(Light l){ this.light = l; }
    public void execute(){ light.on(); }
}
class Remote {
    private Command slot;
    void setCommand(Command c){ this.slot = c; }
    void press(){ if(slot!=null) slot.execute(); }
}
```

Cheat-sheet:

- Creational = create objects (Singleton, Factory, Builder).
- Structural = compose classes/objects (Adapter, Composite, Decorator).
- Behavioral = communication/algorithms (Observer, Strategy, Command).

### 20. SOLID Principles

- Purpose: Five design principles to make code more maintainable, flexible, and testable.

1. Single Responsibility Principle (SRP)
   - A class should have one reason to change — one responsibility.

```java
class ReportGenerator {
    String generate() { /* build report data */ return "report"; }
}
class ReportPrinter {
    void print(String r) { System.out.println(r); }
}
```

2. Open/Closed Principle (OCP)
   - Software entities should be open for extension, closed for modification.

```java
interface Shape { double area(); }
class Circle implements Shape { double radius; public double area(){ return Math.PI*radius*radius; } }
class Rectangle implements Shape { double w,h; public double area(){ return w*h; } }
// New shapes added by implementing Shape, no change to existing code.
```

3. Liskov Substitution Principle (LSP)
   - Subtypes must be substitutable for their base types.
   - Violation example: subclass that throws unsupported operations (e.g., Ostrich extends Bird but cannot fly). Correct by splitting behaviors.

```java
interface FlyingBird { void fly(); }
class Sparrow implements FlyingBird { public void fly(){ System.out.println("fly"); } }
class Ostrich { /* does not implement FlyingBird */ }
```

4. Interface Segregation Principle (ISP)
   - Prefer many small, specific interfaces over a large, general one.

```java
interface Work { void doWork(); }
interface Eat { void eat(); }
class Worker implements Work, Eat { public void doWork(){} public void eat(){} }
```

5. Dependency Inversion Principle (DIP)
   - High-level modules should not depend on low-level modules; both should depend on abstractions.

```java
interface MessageService { void send(String msg); }
class EmailService implements MessageService { public void send(String m){ System.out.println("Email:"+m); } }
class Notification {
    private final MessageService svc;
    Notification(MessageService svc){ this.svc = svc; } // depends on abstraction
    void notify(String m){ svc.send(m); }
}
```

Cheat-sheet:

- SRP = one job per class.
- OCP = extend, don't modify.
- LSP = derived types behave like base types.
- ISP = small interfaces.
- DIP = depend on abstractions.
