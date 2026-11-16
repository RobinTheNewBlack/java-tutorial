# Method

# ส่วนที่ 1: เมธอด (Methods)

## Method คืออะไร?

Method คือบล็อกโค้ดที่เขียนไว้ครั้งเดียวแล้วเรียกใช้ซ้ำได้หลายครั้ง เหมือน function ใน JavaScript แต่ใน Java **ทุก method ต้องอยู่ภายใน class เสมอ** ไม่มี function ลอย ๆ แบบ JS

> 💡 **เทียบกับ JavaScript:** JS มี function declaration, function expression และ arrow function ที่อยู่นอก class ได้ ส่วน Java มีแค่ **method** ที่ต้องอยู่ใน class (หรือ interface/record/enum) และต้องระบุ **type ของ parameter และค่าที่ return** เสมอ

---

## 1. โครงสร้างของ Method (Method Anatomy)

```java
public static int add(int a, int b) {
    return a + b;
}
```

| ส่วนประกอบ      | ตัวอย่าง           | ความหมาย                                                |
| --------------- | ------------------ | ------------------------------------------------------- |
| Access modifier | `public`           | ใครเรียกใช้ได้บ้าง (`public`, `protected`, `private`, ไม่ใส่) |
| Modifier อื่น ๆ   | `static`           | เป็นของ class ไม่ต้องสร้าง object (ยังมี `final`, `abstract` ฯลฯ) |
| Return type     | `int`              | ชนิดของค่าที่ส่งกลับ ถ้าไม่ส่งอะไรกลับใช้ `void`               |
| ชื่อ method      | `add`              | ใช้ camelCase ขึ้นต้นด้วยกริยา เช่น `calculateTotal`        |
| Parameter list  | `(int a, int b)`   | ต้องระบุ type ทุกตัว                                       |
| Body            | `{ return a + b; }` | โค้ดที่ทำงาน                                              |

### ตัวอย่าง

```java
public class Main {
    public static void main(String[] args) {
        greet("สมชาย");                 // ผลลัพธ์: สวัสดี, สมชาย!
        int sum = add(3, 5);
        System.out.println(sum);         // ผลลัพธ์: 8
        System.out.println(isAdult(17)); // ผลลัพธ์: false
    }

    // ไม่ return ค่า → ใช้ void
    static void greet(String name) {
        System.out.println("สวัสดี, " + name + "!");
    }

    // return int
    static int add(int a, int b) {
        return a + b;
    }

    // return boolean
    static boolean isAdult(int age) {
        return age >= 18;
    }
}
```

> **หมายเหตุ:** ถ้าประกาศ return type ที่ไม่ใช่ `void` แต่บางเส้นทางไม่มี `return` → **compile error** `missing return statement` ทันที ต่างจาก JS ที่จะคืน `undefined` เงียบ ๆ

---

## 2. Static Method vs Instance Method

### คุณสมบัติ

- **Static method** — เป็นของ **class** เรียกผ่านชื่อ class ได้เลย (`Math.max(1, 2)`) ไม่ต้อง `new` เข้าถึง field ของ instance ไม่ได้
- **Instance method** — เป็นของ **object** ต้องสร้าง object ก่อน เข้าถึง field ของ object ผ่าน `this` ได้

### ตัวอย่าง

```java
class Calculator {
    private int memory = 0; // instance field

    // static — ไม่ขึ้นกับ object ใด
    static int square(int x) {
        return x * x;
    }

    // instance — ใช้ข้อมูลของ object
    void addToMemory(int x) {
        memory += x;
    }

    int getMemory() {
        return memory;
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Calculator.square(4)); // ผลลัพธ์: 16

        Calculator calc = new Calculator();
        calc.addToMemory(10);
        calc.addToMemory(5);
        System.out.println(calc.getMemory());     // ผลลัพธ์: 15
    }
}
```

> **ข้อควรระวัง:** ภายใน `static` method (เช่น `main`) เรียก instance method ตรง ๆ ไม่ได้ ต้องสร้าง object ก่อน — นี่คือเหตุผลที่ตัวอย่างใน `main` มักเรียก method ที่เป็น `static`

| หัวข้อ               | Static Method               | Instance Method              |
| ------------------- | --------------------------- | ---------------------------- |
| เรียกผ่าน            | `ClassName.method()`        | `object.method()`            |
| ใช้ `this` ได้        | ไม่ได้                       | ได้                           |
| เข้าถึง instance field | ไม่ได้                       | ได้                           |
| ใช้เมื่อ               | utility, คำนวณจาก input ล้วน ๆ | พฤติกรรมที่ขึ้นกับสถานะของ object |

---

## 3. Pass-by-Value — Java ส่งค่าเสมอ

Java **ส่งแบบ pass-by-value เสมอ** ไม่มี pass-by-reference
- ถ้าเป็น **primitive** (`int`, `double`, `boolean` ...) → ส่ง **สำเนาของค่า**
- ถ้าเป็น **object** → ส่ง **สำเนาของ reference** (ชี้ไปที่ object เดียวกัน) จึงแก้ไข *ข้างใน* object ได้ แต่เปลี่ยนให้ตัวแปรต้นทางชี้ไปที่ object ใหม่ไม่ได้

### Primitive — แก้ใน method ไม่กระทบตัวแปรเดิม

```java
static void addTen(int x) {
    x = x + 10; // แก้แค่สำเนา
}

int score = 50;
addTen(score);
System.out.println(score); // ผลลัพธ์: 50 (ไม่เปลี่ยน)
```

### Object — แก้ข้อมูลข้างในได้

```java
static void addItem(List<String> list) {
    list.add("มะม่วง"); // แก้ object ที่ reference ชี้อยู่
}

List<String> fruits = new ArrayList<>();
addItem(fruits);
System.out.println(fruits); // ผลลัพธ์: [มะม่วง]
```

### ตัวอย่างคลาสสิก: Swap ที่ไม่ทำงาน

```java
public class Main {
    static void swap(int a, int b) {
        int temp = a;
        a = b;
        b = temp;
        System.out.println("ใน swap: a=" + a + ", b=" + b);
    }

    static void swapBox(StringBuilder a, StringBuilder b) {
        StringBuilder temp = a;
        a = b;       // เปลี่ยนแค่สำเนา reference
        b = temp;
    }

    public static void main(String[] args) {
        int x = 1, y = 2;
        swap(x, y);                                  // ผลลัพธ์: ใน swap: a=2, b=1
        System.out.println("x=" + x + ", y=" + y);   // ผลลัพธ์: x=1, y=2 (ไม่สลับ!)

        StringBuilder s1 = new StringBuilder("สมชาย");
        StringBuilder s2 = new StringBuilder("สมหญิง");
        swapBox(s1, s2);
        System.out.println(s1 + ", " + s2);          // ผลลัพธ์: สมชาย, สมหญิง (ก็ไม่สลับ!)
    }
}
```

**วิธีสลับที่ใช้ได้จริง** — ส่ง array/object ที่ห่อค่าไว้ แล้วสลับข้อมูลข้างใน

```java
static void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

int[] nums = {1, 2};
swap(nums, 0, 1);
System.out.println(Arrays.toString(nums)); // ผลลัพธ์: [2, 1]
```

> 💡 **เทียบกับ JavaScript:** พฤติกรรมนี้ **เหมือน JS ทุกประการ** — JS ก็ส่ง primitive เป็นสำเนา และส่ง object เป็นสำเนาของ reference ("call by sharing") ถ้าเข้าใจ JS แล้วก็ใช้ความเข้าใจเดิมได้เลย

---

## 4. Method Overloading

ประกาศ method **ชื่อเดียวกัน** ได้หลายตัว ถ้า **parameter list ต่างกัน** (จำนวนหรือ type) — compiler จะเลือกตัวที่ตรงที่สุดให้ตอน compile

### คุณสมบัติ

- ต้องต่างกันที่ **จำนวน**, **ชนิด** หรือ **ลำดับ** ของ parameter
- ต่างกันแค่ **return type** อย่างเดียว → **ไม่ถือเป็น overload** (compile error)

### ตัวอย่าง

```java
public class Main {
    static int area(int side) {
        return side * side; // สี่เหลี่ยมจัตุรัส
    }

    static int area(int width, int height) {
        return width * height; // สี่เหลี่ยมผืนผ้า
    }

    static double area(double radius) {
        return Math.PI * radius * radius; // วงกลม
    }

    public static void main(String[] args) {
        System.out.println(area(4));       // ผลลัพธ์: 16
        System.out.println(area(3, 5));    // ผลลัพธ์: 15
        System.out.println(area(1.0));     // ผลลัพธ์: 3.141592653589793
    }
}
```

> 💡 **เทียบกับ JavaScript:** JS ไม่มี overloading — ถ้าประกาศ function ชื่อซ้ำ ตัวหลังจะทับตัวแรก ต้องเช็ก `arguments.length` หรือ `typeof` เอง Java ให้ compiler จัดการแทน

---

## 5. Varargs — รับ argument ไม่จำกัดจำนวน

ใช้ `Type... name` ภายใน method จะได้เป็น **array**

```java
static int sum(int... numbers) {
    int total = 0;
    for (int n : numbers) {
        total += n;
    }
    return total;
}

System.out.println(sum());           // ผลลัพธ์: 0
System.out.println(sum(1, 2, 3));    // ผลลัพธ์: 6
System.out.println(sum(new int[]{10, 20})); // ผลลัพธ์: 30 — ส่ง array ตรง ๆ ก็ได้
```

### กฎของ Varargs

- มีได้ **แค่ตัวเดียว** ต่อ method และต้องเป็น **parameter ตัวสุดท้าย**

```java
static void printAll(String title, String... items) {
    System.out.println(title + ": " + String.join(", ", items));
}

printAll("ผลไม้", "มะม่วง", "ทุเรียน", "เงาะ"); // ผลลัพธ์: ผลไม้: มะม่วง, ทุเรียน, เงาะ
```

> 💡 **เทียบกับ JavaScript:** เหมือน rest parameter `(...numbers)` ใน JS แต่ Java ใช้ `...` **หลัง type** และได้ array ที่มี type ชัดเจน

---

## 6. Recursion — เมธอดเรียกตัวเอง

Recursion ต้องมี **base case** (จุดหยุด) เสมอ ไม่งั้นจะเกิด `StackOverflowError`

```java
static long factorial(int n) {
    if (n <= 1) {
        return 1; // base case
    }
    return n * factorial(n - 1);
}

static int fibonacci(int n) {
    if (n < 2) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

System.out.println(factorial(5));  // ผลลัพธ์: 120
System.out.println(fibonacci(10)); // ผลลัพธ์: 55
```

> **หมายเหตุ:** Java **ไม่มี tail-call optimization** ถ้า recursion ลึกมาก (หลักหมื่นชั้นขึ้นไป) ควรเปลี่ยนเป็น loop

---

## 7. Return Early (Guard Clause)

ตรวจเงื่อนไขที่ไม่ผ่านแล้ว `return` (หรือ `throw`) ออกไปก่อน ช่วยลดการซ้อน `if` หลายชั้น

```java
// ❌ ซ้อนลึก อ่านยาก
static String checkLogin(String username, String password) {
    if (username != null) {
        if (password != null) {
            if (password.length() >= 8) {
                return "เข้าสู่ระบบสำเร็จ";
            } else {
                return "รหัสผ่านสั้นเกินไป";
            }
        } else {
            return "กรุณากรอกรหัสผ่าน";
        }
    } else {
        return "กรุณากรอกชื่อผู้ใช้";
    }
}

// ✅ Return early — อ่านจากบนลงล่างได้เลย
static String checkLoginClean(String username, String password) {
    if (username == null) return "กรุณากรอกชื่อผู้ใช้";
    if (password == null) return "กรุณากรอกรหัสผ่าน";
    if (password.length() < 8) return "รหัสผ่านสั้นเกินไป";

    return "เข้าสู่ระบบสำเร็จ";
}

System.out.println(checkLoginClean("สมชาย", "1234")); // ผลลัพธ์: รหัสผ่านสั้นเกินไป
```

---

## 8. Java ไม่มี Default Parameter

> 💡 **เทียบกับ JavaScript:** ใน JS เขียน `function greet(name = "ผู้เยี่ยมชม")` ได้ แต่ **Java ไม่รองรับ default parameter** และไม่มี named argument ด้วย ต้องใช้วิธีอื่นแทน

### วิธีที่ 1: Overloading (นิยมที่สุด)

ให้ method ที่ parameter น้อยกว่า เรียกต่อไปยังตัวที่ครบ พร้อมส่งค่า default

```java
static void greet() {
    greet("ผู้เยี่ยมชม"); // ส่งค่า default
}

static void greet(String name) {
    greet(name, "สวัสดี");
}

static void greet(String name, String greeting) {
    System.out.println(greeting + ", " + name + "!");
}

greet();                        // ผลลัพธ์: สวัสดี, ผู้เยี่ยมชม!
greet("สมชาย");                 // ผลลัพธ์: สวัสดี, สมชาย!
greet("สมหญิง", "อรุณสวัสดิ์");   // ผลลัพธ์: อรุณสวัสดิ์, สมหญิง!
```

### วิธีที่ 2: Builder Pattern (เมื่อมี option เยอะ)

เหมาะกับกรณีที่มีค่า optional หลายตัว — เทียบได้กับการส่ง options object `{ ... }` ใน JS

```java
class EmailRequest {
    private final String to;
    private final String subject;
    private final boolean urgent;

    private EmailRequest(Builder b) {
        this.to = b.to;
        this.subject = b.subject;
        this.urgent = b.urgent;
    }

    @Override
    public String toString() {
        return "ถึง " + to + " | เรื่อง: " + subject + " | ด่วน: " + urgent;
    }

    static class Builder {
        private final String to;              // บังคับ
        private String subject = "(ไม่มีหัวข้อ)"; // ค่า default
        private boolean urgent = false;       // ค่า default

        Builder(String to) { this.to = to; }

        Builder subject(String subject) { this.subject = subject; return this; }
        Builder urgent(boolean urgent) { this.urgent = urgent; return this; }

        EmailRequest build() { return new EmailRequest(this); }
    }
}

EmailRequest email = new EmailRequest.Builder("somchai@example.com")
        .subject("ประชุมพรุ่งนี้")
        .build();

System.out.println(email);
// ผลลัพธ์: ถึง somchai@example.com | เรื่อง: ประชุมพรุ่งนี้ | ด่วน: false
```

> **Tip:** ในโปรเจกต์จริง (เช่น Spring Boot) มักใช้ Lombok `@Builder` สร้าง Builder ให้อัตโนมัติ

### วิธีอื่น ๆ (ใช้เท่าที่จำเป็น)

- ส่ง `null` แล้วเช็กใน method — ไม่แนะนำ อ่านยากและเสี่ยง `NullPointerException`
- รับ `Optional<T>` เป็น parameter — **ไม่แนะนำ** (`Optional` ออกแบบมาเป็น return type)

---

## 9. Method Reference (Preview)

ถ้า lambda แค่ส่งต่อไปเรียก method ที่มีอยู่แล้ว เขียนสั้นลงได้ด้วย `::`

```java
List<String> names = List.of("สมชาย", "สมหญิง", "สมศรี");

// แบบ lambda
names.forEach(name -> System.out.println(name));

// แบบ method reference — ความหมายเดียวกัน
names.forEach(System.out::println);
```

รายละเอียดเต็ม ๆ ทั้ง 4 แบบอยู่ในบทถัดไป → [Lambda & Functional Interface](lambda-functional-interface.md)

---

## ตารางเปรียบเทียบ Java Method vs JavaScript Function

| หัวข้อ                | Java Method                       | JavaScript Function               |
| -------------------- | --------------------------------- | --------------------------------- |
| อยู่นอก class ได้      | ไม่ได้                             | ได้                                |
| ระบุ type             | บังคับ (param + return)            | ไม่มี (ยกเว้น TypeScript)            |
| Overloading          | รองรับ                             | ไม่รองรับ (ตัวหลังทับตัวแรก)            |
| Default parameter    | ไม่รองรับ (ใช้ overloading/builder)  | รองรับ `name = "..."`               |
| Rest / Varargs       | `int... nums`                     | `...nums`                         |
| ส่ง argument ผิดจำนวน  | Compile error                     | ได้ (ค่าที่ขาดเป็น `undefined`)         |
| Pass-by-value        | ใช่ (object ส่งสำเนา reference)     | ใช่ (call by sharing)               |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ตั้งชื่อเป็นกริยา + camelCase** — `calculateTotal()`, `findUserById()`, `isValid()` (boolean ขึ้นต้นด้วย `is`/`has`/`can`)
2. **หนึ่ง method ทำหนึ่งอย่าง** — ถ้า method ยาวเกิน ~20–30 บรรทัด ลองแยกเป็น method ย่อย
3. **ใช้ Return Early** — ลดการซ้อน `if` ทำให้อ่านง่าย
4. **parameter ไม่ควรเกิน 3–4 ตัว** — ถ้าเยอะกว่านั้นให้รวมเป็น object/record หรือใช้ Builder
5. **ใช้ `static` เมื่อ method ไม่ได้ใช้ state ของ object** — เช่น utility method
6. **อย่าแก้ไข parameter ที่รับเข้ามา** โดยไม่จำเป็น — ทำให้ผู้เรียกงงว่า object ของตัวเองถูกเปลี่ยน

```java
// แนวทางที่แนะนำ
static double calculateDiscountedPrice(double price, double discountPercent) {
    if (price <= 0) return 0;                       // return early
    if (discountPercent <= 0) return price;

    double discount = price * discountPercent / 100;
    return price - discount;
}

System.out.println(calculateDiscountedPrice(1000, 15)); // ผลลัพธ์: 850.0
```

---
