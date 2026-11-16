# Object-Oriented Programming

# ส่วนที่ 1: Class และ Object

## OOP ใน Java

Java เป็นภาษา **OOP เต็มรูปแบบ** — โค้ดทุกบรรทัดอยู่ใน class และ (เกือบ) ทุกอย่างเป็น object ยกเว้น primitive type (`int`, `double`, `boolean` ...)

**4 หลักการของ OOP:**
- **Encapsulation** — ซ่อนข้อมูลภายในและเปิดเผยเฉพาะสิ่งที่จำเป็น
- **Inheritance** — รับสืบทอดคุณสมบัติจาก Class แม่
- **Polymorphism** — Object ต่างชนิดตอบสนองต่อคำสั่งเดียวกันแตกต่างกัน
- **Abstraction** — ซ่อนความซับซ้อน เปิดเผยแค่สิ่งที่จำเป็น

> 💡 **เทียบกับ JavaScript:** `class` ใน JS เป็นแค่ syntax ครอบ prototype และสร้าง object ได้โดยไม่ต้องมี class (`{ name: "สมชาย" }`) ส่วน Java **ต้องมี class ก่อนเสมอ** และ field ทุกตัวต้องประกาศพร้อม type — เพิ่ม property ใหม่ให้ object ทีหลังไม่ได้

---

## 1. Class และ Object พื้นฐาน

`class` คือแม่แบบ (Blueprint) ส่วน **object** (หรือ instance) คือสิ่งที่สร้างจาก class ด้วย `new`

```java
class Dog {
    // Fields (ข้อมูล)
    String name;
    String breed;

    // Method (พฤติกรรม)
    void bark() {
        System.out.println(name + " เห่า: โฮ่ง โฮ่ง!");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog1 = new Dog();
        dog1.name = "บัดดี้";
        dog1.breed = "โกลเดน รีทรีฟเวอร์";
        dog1.bark(); // ผลลัพธ์: บัดดี้ เห่า: โฮ่ง โฮ่ง!

        Dog dog2 = new Dog();
        System.out.println(dog2.name); // ผลลัพธ์: null (ค่า default ของ field)
    }
}
```

### ค่า Default ของ Field

Field ที่ไม่ได้กำหนดค่าจะได้ค่า default อัตโนมัติ (ต่างจากตัวแปร local ที่ต้องกำหนดค่าก่อนใช้)

| Type                         | ค่า Default |
| ---------------------------- | ---------- |
| `int`, `long`, `short`, `byte` | `0`        |
| `double`, `float`            | `0.0`      |
| `boolean`                    | `false`    |
| `char`                       | `'\u0000'` |
| Object ทุกชนิด (`String` ฯลฯ)  | `null`     |

> **หมายเหตุ:** ไฟล์ `.java` หนึ่งไฟล์มี `public class` ได้ **แค่ตัวเดียว** และชื่อไฟล์ต้องตรงกับชื่อ class นั้น (`Main.java` → `public class Main`)

---

## 2. Constructor

Constructor คือ method พิเศษที่ทำงานตอน `new` ใช้ตั้งค่าเริ่มต้น

### คุณสมบัติ

- **ชื่อเหมือน class** และ **ไม่มี return type** (ไม่ใช่แม้แต่ `void`)
- ถ้าไม่เขียนเลย Java จะสร้าง **default constructor** (ไม่มี parameter) ให้
- ถ้าเขียน constructor เองแม้แต่ตัวเดียว default constructor จะ **หายไป**
- **Overload ได้** และเรียกกันเองด้วย `this(...)` (ต้องเป็นบรรทัดแรก)

### ตัวอย่าง

```java
class Product {
    String name;
    double price;
    int stock;

    // Constructor หลัก
    Product(String name, double price, int stock) {
        this.name = name;   // this.name = field, name = parameter
        this.price = price;
        this.stock = stock;
    }

    // Overload — เรียกต่อไปยัง constructor หลักด้วย this(...)
    Product(String name, double price) {
        this(name, price, 0); // stock default = 0
    }

    void show() {
        System.out.println(name + " ราคา " + price + " บาท (คงเหลือ " + stock + ")");
    }
}

public class Main {
    public static void main(String[] args) {
        new Product("ข้าวมันไก่", 50, 20).show(); // ผลลัพธ์: ข้าวมันไก่ ราคา 50.0 บาท (คงเหลือ 20)
        new Product("ชาไทย", 35).show();          // ผลลัพธ์: ชาไทย ราคา 35.0 บาท (คงเหลือ 0)
    }
}
```

> 💡 **เทียบกับ JavaScript:** JS มี `constructor()` ได้ตัวเดียวและใช้ default parameter แทน ส่วน Java ใช้ **constructor overloading** + `this(...)` แทน default parameter

---

## 3. `this` Keyword

`this` อ้างถึง **object ปัจจุบัน** ใช้ได้ 3 แบบ

```java
class Counter {
    private int count;

    Counter(int count) {
        this.count = count;   // 1. แยก field ออกจาก parameter ที่ชื่อซ้ำ
    }

    Counter() {
        this(0);              // 2. เรียก constructor อื่นใน class เดียวกัน
    }

    Counter increment() {
        count++;
        return this;          // 3. คืนตัวเองเพื่อทำ method chaining
    }

    int getCount() { return count; }
}

Counter c = new Counter().increment().increment().increment();
System.out.println(c.getCount()); // ผลลัพธ์: 3
```

> 💡 **เทียบกับ JavaScript:** `this` ใน Java **ไม่มีปัญหาเปลี่ยนค่าตามวิธีเรียก** แบบ JS (ไม่ต้อง `bind`) — `this` ชี้ไปที่ object เจ้าของ method เสมอ และไม่ต้องเขียน `this.` นำหน้า field ถ้าไม่มีชื่อซ้ำ

---

## 4. Access Modifiers

ควบคุมว่าใครเข้าถึง class, field, method ได้บ้าง

| Modifier                     | Class เดียวกัน | Package เดียวกัน | Subclass (ต่าง package) | ทุกที่ |
| ---------------------------- | :-----------: | :-------------: | :--------------------: | :---: |
| `public`                     | ✅            | ✅              | ✅                     | ✅    |
| `protected`                  | ✅            | ✅              | ✅                     | ❌    |
| *(ไม่ใส่)* — package-private | ✅            | ✅              | ❌                     | ❌    |
| `private`                    | ✅            | ❌              | ❌                     | ❌    |

> 💡 **เทียบกับ JavaScript:** JS มีแค่ public กับ `#private` ส่วน Java มี 4 ระดับ และ **ค่า default (ไม่ใส่อะไร) คือ package-private ไม่ใช่ public**

---

## 5. Encapsulation — Getter และ Setter

แนวทางมาตรฐานของ Java: ให้ field เป็น `private` แล้วเปิดให้เข้าถึงผ่าน method `getXxx()` / `setXxx()` ซึ่งตรวจสอบความถูกต้องได้

```java
class BankAccount {
    private final String owner; // final — กำหนดครั้งเดียวใน constructor
    private double balance;

    BankAccount(String owner, double initialBalance) {
        this.owner = owner;
        this.balance = initialBalance;
    }

    public String getOwner() {
        return owner;
    }

    public double getBalance() {
        return balance;
    }

    // ไม่มี setBalance — บังคับให้เปลี่ยนยอดผ่าน deposit/withdraw เท่านั้น
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("จำนวนเงินต้องมากกว่า 0");
        }
        balance += amount;
    }

    public boolean withdraw(double amount) {
        if (amount > balance) {
            System.out.println("ยอดเงินไม่เพียงพอ");
            return false;
        }
        balance -= amount;
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount("สมชาย", 1000);
        account.deposit(500);
        account.withdraw(200);
        System.out.println(account.getOwner() + ": " + account.getBalance()); // ผลลัพธ์: สมชาย: 1300.0

        account.withdraw(5000); // ผลลัพธ์: ยอดเงินไม่เพียงพอ
        // account.balance = 999999; // ❌ Compile error: balance has private access
    }
}
```

> **Tip:** ในโปรเจกต์จริงมักใช้ **Lombok** (`@Getter`, `@Setter`) ลดโค้ดซ้ำ หรือใช้ **`record`** สำหรับ data class ที่ไม่เปลี่ยนค่า (ดูบท [Record & Enum](record-enum.md))

---

## 6. Static Members

สมาชิก `static` เป็นของ **class** ไม่ใช่ของ object — ทุก object ใช้ร่วมกันตัวเดียว

```java
class Student {
    private static int totalStudents = 0; // ใช้ร่วมกันทุก object
    static final String SCHOOL = "โรงเรียนสาธิต"; // ค่าคงที่ (UPPER_SNAKE_CASE)

    private final int id;
    private final String name;

    Student(String name) {
        totalStudents++;
        this.id = totalStudents;
        this.name = name;
    }

    static int getTotalStudents() { // static method
        return totalStudents;
    }

    String info() {
        return "#" + id + " " + name + " (" + SCHOOL + ")";
    }
}

public class Main {
    public static void main(String[] args) {
        Student s1 = new Student("สมชาย");
        Student s2 = new Student("สมหญิง");

        System.out.println(s2.info());                  // ผลลัพธ์: #2 สมหญิง (โรงเรียนสาธิต)
        System.out.println(Student.getTotalStudents()); // ผลลัพธ์: 2
    }
}
```

### Static Block

ใช้ initialize ค่า static ที่ซับซ้อน ทำงานครั้งเดียวตอนโหลด class

```java
class Config {
    static final Map<String, String> DEFAULTS;

    static {
        DEFAULTS = new HashMap<>();
        DEFAULTS.put("lang", "th");
        DEFAULTS.put("zone", "Asia/Bangkok");
    }
}
```

---

## 7. `final` — Class และ Field

| ใช้กับ          | ความหมาย                                                  |
| -------------- | --------------------------------------------------------- |
| `final` ตัวแปร  | กำหนดค่าได้ครั้งเดียว (เหมือน `const` ใน JS)                   |
| `final` field  | ต้องกำหนดค่าตอนประกาศหรือใน constructor แล้วเปลี่ยนไม่ได้        |
| `final` method | subclass override ไม่ได้                                    |
| `final` class  | ห้ามสืบทอด (เช่น `String`, `Integer` เป็น final class)         |

```java
final class Point { // ห้ามมีคน extends
    private final int x;
    private final int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    Point move(int dx, int dy) {
        return new Point(x + dx, y + dy); // immutable — คืน object ใหม่แทนการแก้ค่าเดิม
    }
}

// class Point3D extends Point {} // ❌ Compile error: cannot inherit from final Point
```

> **ข้อควรระวัง:** `final` กับ object ล็อกแค่ **reference** — ข้อมูลภายในยังเปลี่ยนได้ (เหมือน `const` กับ object ใน JS)
> ```java
> final List<String> names = new ArrayList<>();
> names.add("สมชาย");        // ได้
> // names = new ArrayList<>(); // ❌ Compile error
> ```

---

## 8. toString, equals และ hashCode

ทุก class สืบทอดจาก `Object` ซึ่งมี 3 method สำคัญที่ควร override

### `toString()` — แสดงผลเป็นข้อความ

```java
class Book {
    private final String isbn;
    private final String title;

    Book(String isbn, String title) {
        this.isbn = isbn;
        this.title = title;
    }

    @Override
    public String toString() {
        return "Book{isbn='" + isbn + "', title='" + title + "'}";
    }
}

System.out.println(new Book("978-1", "สามก๊ก"));
// ไม่ override: Book@1b6d3586 (ชื่อ class + hash)
// override แล้ว: Book{isbn='978-1', title='สามก๊ก'}
```

### `equals()` — เปรียบเทียบ "ค่า" ของ object

`==` กับ object เปรียบเทียบ **reference** (เป็น object ตัวเดียวกันไหม) ถ้าต้องการเปรียบเทียบ **ค่า** ต้องใช้ `equals()`

```java
String a = new String("สมชาย");
String b = new String("สมชาย");
System.out.println(a == b);      // ผลลัพธ์: false (คนละ object)
System.out.println(a.equals(b)); // ผลลัพธ์: true  (ค่าเท่ากัน)
```

### ทำไมต้อง override `equals` และ `hashCode` คู่กันเสมอ?

**กฎ (contract):** ถ้า `a.equals(b)` เป็น `true` แล้ว `a.hashCode()` ต้องเท่ากับ `b.hashCode()`

`HashMap` / `HashSet` ใช้ `hashCode()` หา "ช่อง" ก่อน แล้วค่อยใช้ `equals()` เทียบ — ถ้า override แค่ `equals` object ที่เท่ากันจะไปอยู่คนละช่อง ทำให้ `HashSet` มีค่าซ้ำ หรือ `map.get()` หาไม่เจอ

```java
import java.util.*;

class Book {
    private final String isbn;
    private final String title;

    Book(String isbn, String title) {
        this.isbn = isbn;
        this.title = title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                    // object เดียวกัน
        if (!(o instanceof Book other)) return false;  // pattern matching (Java 16+)
        return isbn.equals(other.isbn);                // ถือว่าเท่ากันถ้า isbn ตรงกัน
    }

    @Override
    public int hashCode() {
        return Objects.hash(isbn); // ใช้ field ชุดเดียวกับ equals
    }
}

public class Main {
    public static void main(String[] args) {
        Set<Book> books = new HashSet<>();
        books.add(new Book("978-1", "สามก๊ก"));
        books.add(new Book("978-1", "สามก๊ก (พิมพ์ใหม่)"));
        System.out.println(books.size()); // ผลลัพธ์: 1 (ถ้าไม่ override hashCode จะได้ 2)
    }
}
```

> 💡 **เทียบกับ JavaScript:** JS ไม่มี `equals` มาตรฐาน — `{a:1} === {a:1}` เป็น `false` เสมอ และ `Set`/`Map` ใช้ reference เป็น key Java ให้เรากำหนดความหมายของ "เท่ากัน" เองได้

> **Tip:** IDE (IntelliJ: `Alt+Insert` → `equals() and hashCode()`) สร้างให้อัตโนมัติ หรือใช้ `record` ที่สร้างทั้ง 3 method ให้เลย

---

## 9. Object Lifecycle และ Garbage Collection

1. **สร้าง** — `new` จองหน่วยความจำบน **heap** แล้วเรียก constructor
2. **ใช้งาน** — เข้าถึงผ่าน reference
3. **ไม่มีใครอ้างถึง** — เมื่อไม่มีตัวแปรใดชี้มาที่ object แล้ว object นั้นมีสิทธิ์ถูกเก็บกวาด
4. **Garbage Collector (GC)** — JVM คืนหน่วยความจำให้อัตโนมัติ (เราไม่ต้อง `free` เอง)

```java
Dog dog = new Dog(); // สร้าง object บน heap
dog = null;          // ไม่มีใครอ้างถึงแล้ว → GC เก็บได้เมื่อไรก็ได้
```

> **หมายเหตุ:** ไม่ควรพึ่ง `finalize()` (deprecated ตั้งแต่ Java 9) หรือ `System.gc()` — ถ้าต้องปิด resource (ไฟล์, connection) ให้ใช้ **try-with-resources** (ดูบท [Exception](../advanced/exception.md))

> 💡 **เทียบกับ JavaScript:** เหมือนกัน — ทั้ง JS และ Java มี GC อัตโนมัติ

---

## 10. Package และ Import

**Package** คือการจัดกลุ่ม class เหมือนโฟลเดอร์ ชื่อ package มักกลับด้านชื่อโดเมน เช่น `com.example.shop`

```
src/main/java/
└── com/example/shop/
    ├── Main.java
    ├── model/
    │   └── Product.java
    └── service/
        └── OrderService.java
```

```java
// ไฟล์ com/example/shop/model/Product.java
package com.example.shop.model; // บรรทัดแรกของไฟล์ ต้องตรงกับโฟลเดอร์

public class Product {
    private final String name;

    public Product(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

```java
// ไฟล์ com/example/shop/Main.java
package com.example.shop;

import com.example.shop.model.Product; // import class เดียว
import java.util.*;                     // import ทั้ง package (ไม่รวม sub-package)
import static java.lang.Math.max;       // static import — เรียก max() ได้เลย

public class Main {
    public static void main(String[] args) {
        Product p = new Product("ทุเรียน");
        List<String> list = new ArrayList<>();
        list.add(p.getName());
        System.out.println(list + " " + max(3, 7)); // ผลลัพธ์: [ทุเรียน] 7
    }
}
```

> **หมายเหตุ:** `java.lang` (`String`, `Math`, `Integer`, `System` ...) ถูก import ให้อัตโนมัติ

> 💡 **เทียบกับ JavaScript:** `import` ของ Java **ไม่ได้โหลดโค้ด** เหมือน ES module แค่บอก compiler ว่าชื่อสั้น `Product` หมายถึง `com.example.shop.model.Product` และ Java ไม่มี `export` — ใช้ access modifier (`public`) คุมแทน

---

## ตารางเปรียบเทียบ Java Class vs JavaScript Class

| หัวข้อ                | Java                                | JavaScript                     |
| -------------------- | ----------------------------------- | ------------------------------ |
| ประกาศ field          | ต้องประกาศพร้อม type                  | กำหนดใน constructor ได้เลย       |
| เพิ่ม property ทีหลัง   | ไม่ได้                                | ได้                             |
| Private              | `private`                           | `#field`                       |
| Constructor หลายแบบ   | Overloading                         | ได้ตัวเดียว                       |
| Static               | `static`                            | `static`                       |
| Getter/Setter        | method `getX()` / `setX()`          | `get x()` / `set x(v)`         |
| เปรียบเทียบค่า          | override `equals` + `hashCode`      | ไม่มีมาตรฐาน                     |
| แสดงผล               | override `toString()`               | override `toString()`          |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **Field เป็น `private` เสมอ** — เปิดให้เข้าถึงผ่าน method เท่าที่จำเป็น
2. **ใช้ `final` กับ field ที่ไม่ควรเปลี่ยน** — immutable object ปลอดภัยและเข้าใจง่ายกว่า
3. **ไม่ต้องสร้าง setter ทุก field** — เปิดเฉพาะที่มีเหตุผลทางธุรกิจ (เช่น `deposit()` แทน `setBalance()`)
4. **Override `equals` แล้วต้อง override `hashCode` ด้วยเสมอ** และใช้ field ชุดเดียวกัน
5. **Override `toString()`** ช่วยให้ debug/log ง่ายขึ้นมาก
6. **ตั้งชื่อ**: class เป็น `PascalCase`, method/field เป็น `camelCase`, ค่าคงที่เป็น `UPPER_SNAKE_CASE`, package เป็น `lowercase`
7. **ถ้าเป็น data class ล้วน ๆ ให้พิจารณาใช้ `record`** แทน

```java
// แนวทางที่แนะนำ
public final class Customer { // final — ปลอดภัยที่จะเรียก setName() ใน constructor
    private static final int MAX_NAME_LENGTH = 50;

    private final String id;
    private String name;

    public Customer(String id, String name) {
        this.id = id;
        setName(name);
    }

    public String getId() { return id; }
    public String getName() { return name; }

    public void setName(String name) {
        if (name == null || name.isBlank() || name.length() > MAX_NAME_LENGTH) {
            throw new IllegalArgumentException("ชื่อไม่ถูกต้อง");
        }
        this.name = name;
    }

    @Override
    public String toString() {
        return "Customer{id='" + id + "', name='" + name + "'}";
    }
}
```

---
