# Object-Oriented Programming

# ส่วนที่ 2: Inheritance และ Polymorphism

## Inheritance คืออะไร?

Inheritance (การสืบทอด) คือการสร้าง class ใหม่ (**subclass / child**) ที่รับ field และ method มาจาก class เดิม (**superclass / parent**) แล้วเพิ่มหรือปรับพฤติกรรมเองได้ ใช้ keyword `extends`

### คุณสมบัติ

- Java สืบทอด class ได้ **แค่ 1 class** (single inheritance) — แต่ implement interface ได้หลายตัว
- ทุก class สืบทอดจาก **`java.lang.Object`** โดยอัตโนมัติ
- subclass **ไม่ได้รับ constructor** และเข้าถึง `private` member ของ parent ตรง ๆ ไม่ได้

> 💡 **เทียบกับ JavaScript:** `extends` และ `super` ใช้คล้าย JS มาก แต่ JS สืบทอดผ่าน **prototype chain** (แก้ไข/เพิ่ม method ตอน runtime ได้) ส่วน Java ตรวจสอบทุกอย่างตอน **compile time**

---

## 1. `extends` พื้นฐาน

```java
class Animal {
    protected String name; // protected — subclass เข้าถึงได้

    Animal(String name) {
        this.name = name;
    }

    void eat() {
        System.out.println(name + " กำลังกินอาหาร");
    }

    void makeSound() {
        System.out.println(name + " ส่งเสียง...");
    }
}

class Dog extends Animal {
    Dog(String name) {
        super(name); // เรียก constructor ของ Animal
    }

    void fetch() {
        System.out.println(name + " วิ่งไปคาบลูกบอล");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog("บัดดี้");
        dog.eat();   // ผลลัพธ์: บัดดี้ กำลังกินอาหาร   (สืบทอดมาจาก Animal)
        dog.fetch(); // ผลลัพธ์: บัดดี้ วิ่งไปคาบลูกบอล (method ของ Dog เอง)
    }
}
```

---

## 2. `super` และ Constructor Chaining

`super` ใช้ได้ 2 แบบ
1. `super(...)` — เรียก constructor ของ parent (**ต้องเป็นบรรทัดแรก** ของ constructor)
2. `super.method()` — เรียก method เวอร์ชันของ parent

### Constructor Chaining

ทุก constructor จะเรียก constructor ของ parent ก่อนเสมอ ถ้าไม่เขียน `super(...)` Java จะใส่ `super()` (ไม่มี parameter) ให้อัตโนมัติ — ถ้า parent ไม่มี constructor แบบไม่มี parameter จะ **compile error**

```java
class Person {
    Person() {
        System.out.println("1. สร้าง Person");
    }
}

class Employee extends Person {
    Employee() {
        // super(); ← Java ใส่ให้อัตโนมัติ
        System.out.println("2. สร้าง Employee");
    }
}

class Manager extends Employee {
    Manager() {
        System.out.println("3. สร้าง Manager");
    }
}

new Manager();
// ผลลัพธ์:
// 1. สร้าง Person
// 2. สร้าง Employee
// 3. สร้าง Manager
```

> **หมายเหตุ:** Constructor ทำงานจาก **บนลงล่าง** (parent ก่อน child เสมอ) ทำให้ child มั่นใจได้ว่า parent พร้อมใช้งานแล้ว

---

## 3. Method Overriding และ `@Override`

Subclass เขียน method ที่มี **signature เดียวกัน** กับ parent เพื่อเปลี่ยนพฤติกรรม

### กฎของ Overriding

- ชื่อ, parameter ต้องตรงกัน; return type ต้องเหมือนหรือเป็น subtype (covariant)
- access modifier **กว้างเท่าเดิมหรือกว้างกว่า** ได้ แต่แคบลงไม่ได้ (`public` → `private` ไม่ได้)
- override method `static`, `private`, `final` ไม่ได้

### ตัวอย่าง

```java
class Animal {
    protected final String name;

    Animal(String name) { this.name = name; }

    String makeSound() {
        return "...";
    }

    String describe() {
        return name + " ร้องว่า " + makeSound();
    }
}

class Cat extends Animal {
    Cat(String name) { super(name); }

    @Override
    String makeSound() {
        return "เหมียว";
    }
}

class Dog extends Animal {
    Dog(String name) { super(name); }

    @Override
    String makeSound() {
        return "โฮ่ง";
    }

    @Override
    String describe() {
        return super.describe() + " (และกระดิกหาง)"; // ต่อยอดจาก parent
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(new Cat("ส้มโอ").describe()); // ผลลัพธ์: ส้มโอ ร้องว่า เหมียว
        System.out.println(new Dog("บัดดี้").describe()); // ผลลัพธ์: บัดดี้ ร้องว่า โฮ่ง (และกระดิกหาง)
    }
}
```

> **ทำไมต้องใส่ `@Override`?** ถ้าพิมพ์ชื่อผิด (เช่น `makeSoud`) หรือ parameter ไม่ตรง compiler จะแจ้ง error ทันที — ถ้าไม่ใส่ Java จะคิดว่าเป็น method ใหม่ และ bug จะเงียบ

### Overloading vs Overriding

| หัวข้อ           | Overloading                 | Overriding                       |
| --------------- | --------------------------- | -------------------------------- |
| เกิดที่           | Class เดียวกัน (หรือ subclass) | Subclass                         |
| Parameter       | ต้องต่างกัน                    | ต้องเหมือนกัน                       |
| ตัดสินใจตอน      | Compile time                | Runtime                          |
| Annotation      | -                           | `@Override`                      |

---

## 4. Polymorphism — Compile-time Type vs Runtime Type

ตัวแปรชนิด parent สามารถเก็บ object ของ child ได้ — และเมื่อเรียก method ที่ถูก override Java จะใช้เวอร์ชันของ **object จริง** (dynamic dispatch)

```java
Animal a = new Dog("บัดดี้");
//  ↑ compile-time type     ↑ runtime type
```

- **Compile-time type** (`Animal`) — กำหนดว่า **เรียก method อะไรได้บ้าง**
- **Runtime type** (`Dog`) — กำหนดว่า **ใช้ implementation ของใคร**

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Animal> animals = List.of(
                new Cat("ส้มโอ"),
                new Dog("บัดดี้"),
                new Cat("มะลิ")
        );

        for (Animal animal : animals) {
            System.out.println(animal.makeSound()); // แต่ละตัวทำงานต่างกัน
        }
        // ผลลัพธ์:
        // เหมียว
        // โฮ่ง
        // เหมียว

        Animal a = new Dog("บัดดี้");
        // a.fetch(); // ❌ Compile error — Animal ไม่มี fetch() ถึงแม้ object จริงจะเป็น Dog
    }
}
```

> 💡 **เทียบกับ JavaScript:** JS ใช้ duck typing — มี method ชื่อนั้นก็เรียกได้ ส่วน Java ต้องผ่านการตรวจของ compiler ตาม **type ที่ประกาศ** ก่อน

---

## 5. Upcasting และ Downcasting

| แบบ          | ทิศทาง         | ต้อง cast เอง? | ปลอดภัย?                        |
| ----------- | ------------- | ------------- | ------------------------------- |
| Upcasting   | Child → Parent | ไม่ต้อง (อัตโนมัติ) | ปลอดภัยเสมอ                     |
| Downcasting | Parent → Child | ต้อง `(Dog)`   | อาจเกิด `ClassCastException`     |

```java
Animal animal = new Dog("บัดดี้"); // Upcasting — อัตโนมัติ

Dog dog = (Dog) animal;            // Downcasting — ต้องระบุเอง
System.out.println(dog.makeSound()); // ผลลัพธ์: โฮ่ง

Animal cat = new Cat("ส้มโอ");
// Dog wrong = (Dog) cat; // ❌ Runtime: ClassCastException: Cat cannot be cast to Dog
```

---

## 6. `instanceof` และ Pattern Matching

ตรวจ type ก่อน downcast เพื่อความปลอดภัย

### แบบเดิม (ก่อน Java 16)

```java
if (animal instanceof Dog) {
    Dog d = (Dog) animal; // ต้อง cast ซ้ำ
    System.out.println(d.makeSound());
}
```

### Pattern Matching for `instanceof` (Java 16+)

ตรวจและประกาศตัวแปรในคำสั่งเดียว

```java
if (animal instanceof Dog d) {
    System.out.println(d.makeSound()); // ใช้ d ได้เลย ไม่ต้อง cast
}

// ใช้ร่วมกับ && ได้
if (animal instanceof Dog d && d.name.startsWith("บ")) {
    System.out.println("หมาชื่อขึ้นต้นด้วย บ");
}
```

### Pattern Matching for `switch` (Java 21+)

```java
static String describe(Object obj) {
    return switch (obj) {
        case Dog d -> "หมาชื่อ " + d.name;
        case Cat c -> "แมวชื่อ " + c.name;
        case String s when s.isBlank() -> "ข้อความว่าง";
        case String s -> "ข้อความ: " + s;
        case null -> "ไม่มีค่า";
        default -> "อื่น ๆ";
    };
}
```

> 💡 **เทียบกับ JavaScript:** `instanceof` ใน JS ใช้คล้ายกัน แต่ JS ไม่ต้อง cast เพราะไม่มี static type (TypeScript จะ narrow type ให้อัตโนมัติ คล้าย pattern matching ของ Java)

---

## 7. Method สำคัญของ `Object` Class

ทุก class สืบทอด method เหล่านี้จาก `Object`

| Method                  | หน้าที่                                    | ควร override? |
| ----------------------- | ----------------------------------------- | ------------- |
| `toString()`            | แสดง object เป็น String                    | ✅ ควร         |
| `equals(Object o)`      | เปรียบเทียบความเท่ากันของค่า                   | ✅ ถ้าต้องเทียบค่า |
| `hashCode()`            | ค่า hash ใช้ใน `HashMap`/`HashSet`          | ✅ คู่กับ equals |
| `getClass()`            | คืน `Class` ของ runtime type              | ❌ (final)     |
| `clone()`               | สำเนา object                               | ⚠️ ไม่แนะนำ ใช้ copy constructor แทน |
| `wait()/notify()`       | ใช้กับ thread (ระดับต่ำ)                       | ❌ (final)     |

```java
Animal a = new Dog("บัดดี้");
System.out.println(a.getClass().getSimpleName()); // ผลลัพธ์: Dog
```

---

## 8. `final` Method และ `final` Class

```java
class Payment {
    // ห้าม subclass เปลี่ยนตรรกะการตรวจสอบนี้
    final boolean validate(double amount) {
        return amount > 0;
    }
}

class QrPayment extends Payment {
    // @Override
    // boolean validate(double amount) { return true; } // ❌ Compile error: cannot override final method
}

final class ThaiBaht { } // ห้ามสืบทอดทั้ง class
```

---

## 9. Composition over Inheritance

Inheritance ผูก child กับ parent แน่นมาก (tight coupling) — ถ้า parent เปลี่ยน child อาจพังได้ แนวคิดที่นิยมกว่าคือ **"has-a" (composition)** แทน **"is-a" (inheritance)**

### ❌ ใช้ Inheritance ผิดที่

```java
// Car "เป็น" Engine? — ไม่ใช่!
class Engine {
    void start() { System.out.println("เครื่องยนต์ติด"); }
}

class Car extends Engine { } // ผิดความหมาย และเปลี่ยนเป็นรถไฟฟ้าไม่ได้
```

### ✅ ใช้ Composition

```java
interface Engine {
    void start();
}

class PetrolEngine implements Engine {
    public void start() { System.out.println("เครื่องยนต์เบนซินติด: บรื้นน"); }
}

class ElectricEngine implements Engine {
    public void start() { System.out.println("มอเตอร์ไฟฟ้าพร้อม: วี้ดด"); }
}

class Car {
    private final Engine engine; // Car "มี" Engine

    Car(Engine engine) {          // รับเข้ามาจากภายนอก (Dependency Injection)
        this.engine = engine;
    }

    void drive() {
        engine.start();
        System.out.println("รถเริ่มวิ่ง");
    }
}

public class Main {
    public static void main(String[] args) {
        new Car(new ElectricEngine()).drive();
        // ผลลัพธ์:
        // มอเตอร์ไฟฟ้าพร้อม: วี้ดด
        // รถเริ่มวิ่ง
    }
}
```

> **Tip:** รูปแบบ "รับ dependency ผ่าน constructor" นี้คือพื้นฐานของ **Dependency Injection** ใน Spring Framework

| ใช้ Inheritance เมื่อ                         | ใช้ Composition เมื่อ                        |
| ------------------------------------------- | ------------------------------------------ |
| เป็นความสัมพันธ์ "is-a" ชัดเจน (Dog is an Animal) | เป็น "has-a" (Car has an Engine)           |
| ต้องการ polymorphism ผ่าน class hierarchy     | ต้องการสลับพฤติกรรมได้ตอน runtime             |
| ลำดับชั้นตื้น (1–2 ชั้น)                         | ต้องการลด coupling และทดสอบง่าย              |

---

## 10. Sealed Classes (Java 17+)

`sealed` จำกัดว่า **class ไหนบ้างที่สืบทอดได้** — ทำให้ compiler รู้ subtype ครบทุกตัว เหมาะกับการใช้คู่กับ pattern matching `switch`

### คุณสมบัติ

- ระบุ subclass ที่อนุญาตด้วย `permits`
- subclass ทุกตัวต้องเป็น `final`, `sealed` หรือ `non-sealed`
- ใช้กับ interface ได้ด้วย

### ตัวอย่าง

```java
public class Main {
    sealed interface Shape permits Circle, Rectangle, Square {}

    record Circle(double radius) implements Shape {}
    record Rectangle(double width, double height) implements Shape {}
    record Square(double side) implements Shape {}

    static double area(Shape shape) {
        // ไม่ต้องมี default — compiler รู้ว่าครบทุกกรณีแล้ว (Java 21)
        return switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Rectangle r -> r.width() * r.height();
            case Square s -> s.side() * s.side();
        };
    }

    public static void main(String[] args) {
        System.out.println(area(new Rectangle(3, 4))); // ผลลัพธ์: 12.0
        System.out.println(area(new Square(5)));       // ผลลัพธ์: 25.0
    }
}
```

> **ข้อดี:** ถ้าวันหนึ่งเพิ่ม `Triangle` เข้าไปใน `permits` แต่ลืมแก้ `switch` → **compile error** ทันที แทนที่จะเป็น bug ตอน runtime

> 💡 **เทียบกับ JavaScript:** คล้าย **discriminated union** ใน TypeScript (`type Shape = Circle | Rectangle | Square`) ที่ใช้ `switch (shape.kind)` แล้ว compiler ตรวจว่าครบทุกกรณี

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใส่ `@Override` ทุกครั้ง** ที่ override method
2. **Favor composition over inheritance** — ใช้ inheritance เฉพาะความสัมพันธ์ "is-a" ที่ชัดเจน
3. **ลำดับชั้นไม่ควรลึก** — เกิน 2–3 ชั้นจะเข้าใจและแก้ไขยาก
4. **ประกาศตัวแปรด้วย type ที่กว้างที่สุดที่เพียงพอ** — เช่น `List<String> list = new ArrayList<>();` แทน `ArrayList<String>`
5. **ใช้ pattern matching `instanceof`** แทนการ cast เอง และพยายามใช้ polymorphism แทนการเช็ก `instanceof` เยอะ ๆ
6. **ใช้ `sealed`** เมื่อรู้ชุด subtype ที่แน่นอน เพื่อให้ compiler ช่วยตรวจ
7. **อย่าเรียก method ที่ override ได้ใน constructor** — subclass อาจยังไม่ได้ initialize field

```java
// แนวทางที่แนะนำ: ใช้ polymorphism แทน if-else เช็ก type
// ❌
if (animal instanceof Dog) { /* ... */ } else if (animal instanceof Cat) { /* ... */ }

// ✅
animal.makeSound(); // แต่ละ class จัดการเอง
```

---
