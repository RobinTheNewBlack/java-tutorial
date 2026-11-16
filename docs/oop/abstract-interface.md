# Object-Oriented Programming

# ส่วนที่ 3: Abstract Class และ Interface

## Abstraction คืออะไร?

Abstraction คือการ **กำหนด "สิ่งที่ต้องทำ" โดยซ่อน "วิธีทำ"** ผู้ใช้งานรู้แค่ว่าเรียก method อะไรได้ ไม่ต้องรู้รายละเอียดข้างใน Java มีเครื่องมือหลัก 2 ตัวคือ **abstract class** และ **interface**

> 💡 **เทียบกับ JavaScript:** JS ไม่มี `abstract` และ `interface` (ต้องใช้ TypeScript หรือ `throw new Error("not implemented")` เอง) ส่วน Java บังคับตอน compile — ถ้า implement ไม่ครบ **compile ไม่ผ่าน**

---

## 1. Abstract Class

Class ที่ **สร้าง object ตรง ๆ ไม่ได้** ใช้เป็นแม่แบบให้ subclass

### คุณสมบัติ

- ประกาศด้วย `abstract class`
- มี **abstract method** (ไม่มี body) ที่ subclass ต้อง implement
- มี method ปกติ, field, constructor ได้เหมือน class ทั่วไป
- subclass `extends` ได้แค่ 1 class

### ตัวอย่าง

```java
abstract class Shape {
    private final String name;

    Shape(String name) { // abstract class มี constructor ได้
        this.name = name;
    }

    abstract double area();      // ไม่มี body — subclass ต้องเขียนเอง
    abstract double perimeter();

    // method ปกติ — ใช้ abstract method ได้ (Template Method Pattern)
    void printInfo() {
        System.out.printf("%s: พื้นที่ %.2f, เส้นรอบรูป %.2f%n", name, area(), perimeter());
    }
}

class Circle extends Shape {
    private final double radius;

    Circle(double radius) {
        super("วงกลม");
        this.radius = radius;
    }

    @Override
    double area() { return Math.PI * radius * radius; }

    @Override
    double perimeter() { return 2 * Math.PI * radius; }
}

class Rectangle extends Shape {
    private final double width, height;

    Rectangle(double width, double height) {
        super("สี่เหลี่ยม");
        this.width = width;
        this.height = height;
    }

    @Override
    double area() { return width * height; }

    @Override
    double perimeter() { return 2 * (width + height); }
}

public class Main {
    public static void main(String[] args) {
        // Shape s = new Shape("?"); // ❌ Compile error: Shape is abstract; cannot be instantiated

        Shape[] shapes = { new Circle(1), new Rectangle(3, 4) };
        for (Shape s : shapes) {
            s.printInfo();
        }
        // ผลลัพธ์:
        // วงกลม: พื้นที่ 3.14, เส้นรอบรูป 6.28
        // สี่เหลี่ยม: พื้นที่ 12.00, เส้นรอบรูป 14.00
    }
}
```

---

## 2. Interface

Interface คือ **สัญญา (contract)** ที่บอกว่า class ที่ implement ต้องมี method อะไรบ้าง

### คุณสมบัติ

- ประกาศด้วย `interface` และ class ใช้ `implements`
- method ที่ไม่มี body เป็น `public abstract` โดยอัตโนมัติ
- field ทุกตัวเป็น `public static final` (ค่าคงที่) โดยอัตโนมัติ
- **ไม่มี constructor** และไม่มี instance field
- class หนึ่ง **implement ได้หลาย interface**

### ตัวอย่าง

```java
interface Greeter {
    String greet(String name); // public abstract โดยอัตโนมัติ
}

class ThaiGreeter implements Greeter {
    @Override
    public String greet(String name) { // ต้องเป็น public
        return "สวัสดี " + name;
    }
}

class EnglishGreeter implements Greeter {
    @Override
    public String greet(String name) {
        return "Hello " + name;
    }
}

Greeter g = new ThaiGreeter();
System.out.println(g.greet("สมชาย")); // ผลลัพธ์: สวัสดี สมชาย
```

> 💡 **เทียบกับ JavaScript:** คล้าย `interface` ใน TypeScript แต่ TypeScript ใช้ **structural typing** (มี method ครบก็ถือว่าเป็น type นั้น) ส่วน Java ใช้ **nominal typing** — class ต้องเขียน `implements Greeter` อย่างชัดเจน ถึงจะใช้เป็น `Greeter` ได้

---

## 3. Default, Static และ Private Method ใน Interface

| ชนิด              | ตั้งแต่ Java | มี body | ใช้ทำอะไร                                               |
| ---------------- | ---------- | ------ | ------------------------------------------------------ |
| abstract method  | 1.0        | ไม่มี   | สัญญาที่ class ต้อง implement                              |
| `default` method | 8          | มี      | ให้ implementation เริ่มต้น (override ได้) เพิ่ม method ใหม่โดยไม่ทำให้ class เดิมพัง |
| `static` method  | 8          | มี      | utility ที่เกี่ยวกับ interface เรียกผ่าน `InterfaceName.method()` |
| `private` method | 9          | มี      | helper ภายใน interface ใช้ร่วมกันระหว่าง default method       |

```java
interface Notifier {
    void send(String to, String message); // abstract

    default void sendAll(java.util.List<String> recipients, String message) {
        for (String to : recipients) {
            send(to, format(message));
        }
    }

    static Notifier console() { // factory method
        return (to, message) -> System.out.println("[ถึง " + to + "] " + message);
    }

    private String format(String message) { // helper (Java 9+)
        return "📢 " + message;
    }
}

public class Main {
    public static void main(String[] args) {
        Notifier notifier = Notifier.console();
        notifier.sendAll(java.util.List.of("สมชาย", "สมหญิง"), "ประชุม 10 โมง");
        // ผลลัพธ์:
        // [ถึง สมชาย] 📢 ประชุม 10 โมง
        // [ถึง สมหญิง] 📢 ประชุม 10 โมง
    }
}
```

> **หมายเหตุ:** `Notifier` มี abstract method ตัวเดียว จึงเป็น functional interface — สร้างด้วย lambda ได้

---

## 4. Implement หลาย Interface

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    @Override
    public void fly() { System.out.println("เป็ดบิน"); }

    @Override
    public void swim() { System.out.println("เป็ดว่ายน้ำ"); }
}

Duck duck = new Duck();
Flyable f = duck;   // ใช้ในฐานะ Flyable
Swimmable s = duck; // ใช้ในฐานะ Swimmable
f.fly();  // ผลลัพธ์: เป็ดบิน
s.swim(); // ผลลัพธ์: เป็ดว่ายน้ำ
```

### ปัญหา Default Method ชนกัน (Diamond Problem)

ถ้า 2 interface มี `default` method ชื่อเดียวกัน class ต้อง override และเลือกเองด้วย `InterfaceName.super.method()`

```java
interface A { default String hello() { return "A"; } }
interface B { default String hello() { return "B"; } }

class C implements A, B {
    @Override
    public String hello() {
        return A.super.hello() + B.super.hello(); // ระบุชัดเจนว่าใช้ของใคร
    }
}

System.out.println(new C().hello()); // ผลลัพธ์: AB
```

---

## 5. Abstract Class vs Interface — เลือกใช้อะไรดี?

| หัวข้อ                      | Abstract Class                       | Interface                                  |
| -------------------------- | ------------------------------------ | ------------------------------------------ |
| สืบทอด/implement ได้กี่ตัว     | 1 (`extends`)                        | หลายตัว (`implements`)                      |
| Instance field (state)     | มีได้                                 | ไม่ได้ (มีแค่ค่าคงที่)                          |
| Constructor                | มีได้                                 | ไม่มี                                        |
| Access modifier ของ method | ได้ทุกแบบ                             | `public` (หรือ `private` สำหรับ helper)        |
| ความสัมพันธ์                  | "is-a" — เป็นชนิดเดียวกัน แชร์โค้ด/state | "can-do" — ความสามารถ/สัญญา                 |
| ตัวอย่าง                     | `AbstractList`, `HttpServlet`        | `Comparable`, `Runnable`, `List`, `JpaRepository` |

**หลักการเลือก:**
- เริ่มจาก **interface** ก่อนเสมอ (ยืดหยุ่นกว่า, mock/test ง่าย)
- ใช้ **abstract class** เมื่อ subclass ต้องการแชร์ **state (field)** หรือโค้ดจำนวนมาก
- ทำทั้งสองอย่างก็ได้: interface เป็นสัญญา + abstract class เป็น base implementation (เช่น `List` + `AbstractList`)

---

## 6. Anonymous Class

สร้าง class ที่ไม่มีชื่อพร้อม object ในคราวเดียว ใช้เมื่อต้องการ implement ใช้ครั้งเดียว

```java
Greeter formal = new Greeter() {
    @Override
    public String greet(String name) {
        return "เรียน คุณ" + name;
    }
};
System.out.println(formal.greet("สมศรี")); // ผลลัพธ์: เรียน คุณสมศรี

// ถ้า interface มี abstract method ตัวเดียว ใช้ lambda สั้นกว่ามาก
Greeter casual = name -> "หวัดดี " + name;
System.out.println(casual.greet("สมศรี")); // ผลลัพธ์: หวัดดี สมศรี
```

> **หมายเหตุ:** Anonymous class ยังจำเป็นเมื่อ implement interface ที่มีหลาย method หรือ extend abstract class — แต่ถ้าเป็น functional interface ให้ใช้ lambda

---

## 7. Nested Class (Inner / Static Nested)

Class ที่ประกาศอยู่ภายใน class อื่น

| ชนิด                | ประกาศ                     | เข้าถึง instance ของ outer | สร้างอย่างไร                         |
| ------------------ | ------------------------- | ----------------------- | ---------------------------------- |
| Static nested class | `static class Inner {}`  | ไม่ได้                   | `new Outer.Inner()`                |
| Inner class         | `class Inner {}`          | ได้                      | `outer.new Inner()`                |
| Local class         | ประกาศใน method           | ได้                      | ภายใน method นั้น                   |
| Anonymous class     | `new Interface() { ... }` | ได้                      | ทันทีที่ประกาศ                        |

```java
class Order {
    private final String customer;

    Order(String customer) { this.customer = customer; }

    // Static nested — ไม่ผูกกับ Order object ใด (ใช้บ่อยสุด เช่น Builder, DTO ย่อย)
    static class Item {
        final String name;
        final int qty;
        Item(String name, int qty) { this.name = name; this.qty = qty; }
    }

    // Inner class — เข้าถึง field ของ Order ได้
    class Receipt {
        String print() { return "ใบเสร็จของ " + customer; }
    }
}

Order.Item item = new Order.Item("ชาไทย", 2);
Order order = new Order("สมชาย");
Order.Receipt receipt = order.new Receipt();
System.out.println(receipt.print()); // ผลลัพธ์: ใบเสร็จของ สมชาย
```

> **Tip:** ถ้าไม่จำเป็นต้องใช้ข้อมูลของ outer object ให้ใช้ **`static` nested class** เสมอ — inner class ถือ reference ไปยัง outer ทำให้เปลือง memory และอาจ leak ได้

---

## 8. ตัวอย่างจริง: ระบบชำระเงิน (PaymentMethod)

ตัวอย่างนี้แสดงการ **เขียนโค้ดผ่าน interface** — `CheckoutService` ไม่รู้เลยว่าชำระด้วยอะไร ทำให้เพิ่มวิธีชำระใหม่ได้โดยไม่ต้องแก้ service

```java
import java.util.List;

interface PaymentMethod {
    boolean pay(double amount);
    String name();

    default String receipt(double amount) {
        return String.format("ชำระ %.2f บาท ด้วย %s", amount, name());
    }
}

class CreditCard implements PaymentMethod {
    private final String cardNumber;
    private final double creditLimit;

    CreditCard(String cardNumber, double creditLimit) {
        this.cardNumber = cardNumber;
        this.creditLimit = creditLimit;
    }

    @Override
    public boolean pay(double amount) {
        if (amount > creditLimit) {
            System.out.println("วงเงินไม่พอ");
            return false;
        }
        System.out.println("ตัดบัตร ****" + cardNumber.substring(cardNumber.length() - 4));
        return true;
    }

    @Override
    public String name() { return "บัตรเครดิต"; }
}

class Cash implements PaymentMethod {
    private final double received;

    Cash(double received) { this.received = received; }

    @Override
    public boolean pay(double amount) {
        if (received < amount) {
            System.out.println("เงินสดไม่พอ");
            return false;
        }
        System.out.printf("รับเงิน %.2f ทอน %.2f%n", received, received - amount);
        return true;
    }

    @Override
    public String name() { return "เงินสด"; }
}

class CheckoutService {
    private final PaymentMethod paymentMethod; // พึ่งพา interface ไม่ใช่ class จริง

    CheckoutService(PaymentMethod paymentMethod) { // Constructor Injection
        this.paymentMethod = paymentMethod;
    }

    void checkout(double total) {
        if (paymentMethod.pay(total)) {
            System.out.println(paymentMethod.receipt(total));
        }
    }
}

public class Main {
    public static void main(String[] args) {
        List<PaymentMethod> methods = List.of(
                new CreditCard("4111222233334444", 50_000),
                new Cash(1_000)
        );

        for (PaymentMethod method : methods) {
            new CheckoutService(method).checkout(850);
        }
        // ผลลัพธ์:
        // ตัดบัตร ****4444
        // ชำระ 850.00 บาท ด้วย บัตรเครดิต
        // รับเงิน 1000.00 ทอน 150.00
        // ชำระ 850.00 บาท ด้วย เงินสด
    }
}
```

### เชื่อมโยงกับ Spring Framework

ใน Spring Boot รูปแบบนี้คือสิ่งที่ทำทุกวัน — เราประกาศ interface แล้วให้ Spring **inject** implementation ให้อัตโนมัติ

```java
// ตัวอย่างแนวคิด (ต้องมี Spring ใน classpath)
@Service
class CheckoutService {
    private final PaymentMethod paymentMethod;

    CheckoutService(PaymentMethod paymentMethod) { // Spring ส่ง @Component ที่ implement PaymentMethod มาให้
        this.paymentMethod = paymentMethod;
    }
}
```

ข้อดี: เปลี่ยน implementation ได้โดยไม่แก้ service, และตอนเขียน unit test ส่ง **mock** ของ `PaymentMethod` เข้าไปแทนได้ง่าย

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **Program to an interface, not an implementation** — ประกาศตัวแปร/parameter เป็น interface (`List`, `PaymentMethod`) ไม่ใช่ class จริง
2. **เริ่มจาก interface** แล้วค่อยเพิ่ม abstract class เมื่อมีโค้ดซ้ำจริง ๆ
3. **Interface ควรเล็กและเฉพาะเจาะจง** (Interface Segregation) — ดีกว่ามี interface ใหญ่ที่ class ต้อง implement method ที่ไม่ได้ใช้
4. **ใช้ `default` method อย่างระมัดระวัง** — เหมาะกับการเพิ่ม method ใหม่โดยไม่ทำให้ implementation เดิมพัง ไม่ใช่ที่เก็บ logic หลัก
5. **ใช้ lambda แทน anonymous class** สำหรับ functional interface
6. **Nested class ให้เป็น `static`** ถ้าไม่ต้องใช้ข้อมูลของ outer object

```java
// แนวทางที่แนะนำ
List<String> names = new ArrayList<>();          // ✅ ประกาศเป็น interface
PaymentMethod method = new Cash(500);            // ✅
// ArrayList<String> names = new ArrayList<>();  // ❌ ผูกกับ implementation
```

---
