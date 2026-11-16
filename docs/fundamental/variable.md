# Java Fundamentals

# ส่วนที่ 1: ตัวแปร (Variables)

## ตัวแปรคืออะไร?

ตัวแปร (Variable) คือกล่องสำหรับเก็บข้อมูล ใน Java ทุกตัวแปร **ต้องมีชนิดข้อมูล (type) ที่แน่นอน** และ type นั้นจะ **เปลี่ยนไม่ได้** ตลอดอายุของตัวแปร (Static Typing)

> 💡 **เทียบกับ JavaScript:** JS เป็น dynamic typing — `let x = 5; x = "hello";` ทำได้ แต่ใน Java `int x = 5; x = "hello";` จะ **compile ไม่ผ่าน** ทันที

---

## 1. การประกาศตัวแปร

รูปแบบ: `ชนิดข้อมูล ชื่อตัวแปร = ค่า;`

### คุณสมบัติ

- ต้องระบุ **type** ตอนประกาศ (หรือใช้ `var` ให้ compiler อนุมานให้ — ดูหัวข้อที่ 4)
- **กำหนดค่าใหม่** ได้ แต่ต้องเป็น type เดียวกัน (หรือแปลงได้อัตโนมัติ)
- **ประกาศซ้ำไม่ได้** ใน scope เดียวกัน
- local variable **ต้องกำหนดค่าก่อนใช้** ไม่เช่นนั้น compile error

### ตัวอย่าง

```java
int age = 25;
double height = 175.5;
String name = "สมชาย";
boolean isStudent = true;
char grade = 'A';

age = 26;            // กำหนดค่าใหม่ได้
// age = "ยี่สิบหก";  // Error: incompatible types: String cannot be converted to int
// int age = 30;     // Error: variable age is already defined

System.out.println(name + " อายุ " + age); // ผลลัพธ์: สมชาย อายุ 26
```

### ประกาศก่อน กำหนดค่าทีหลัง

```java
int score;           // ประกาศอย่างเดียว
// System.out.println(score); // Error: variable score might not have been initialized

score = 90;
System.out.println(score); // ผลลัพธ์: 90
```

### ประกาศหลายตัวพร้อมกัน

```java
int x = 1, y = 2, z = 3; // ทำได้ แต่ไม่แนะนำ — อ่านยาก
```

> 💡 **เทียบกับ JavaScript:** Java **ไม่มี `undefined`** — ตัวแปร local ที่ยังไม่ได้กำหนดค่าจะใช้ไม่ได้เลย (compiler ห้าม) และไม่มี hoisting แบบ `var` ของ JS

---

## 2. Primitive vs Reference

Java แบ่งตัวแปรออกเป็น 2 กลุ่มใหญ่

### Primitive types (8 ชนิด)

`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean` — เก็บ **ค่าจริง** ไว้ในตัวแปรโดยตรง

```java
int a = 10;
int b = a;   // คัดลอก "ค่า"
b = 20;

System.out.println(a); // ผลลัพธ์: 10 — a ไม่ได้รับผลกระทบ
System.out.println(b); // ผลลัพธ์: 20
```

### Reference types

ทุกอย่างที่ไม่ใช่ primitive เช่น `String`, array, `ArrayList`, class ที่เราสร้างเอง — ตัวแปรเก็บ **ที่อยู่ (reference)** ของ object

```java
int[] original = {1, 2, 3};
int[] copy = original;   // คัดลอก "reference" — ชี้ไปที่ array เดียวกัน
copy[0] = 99;

System.out.println(original[0]); // ผลลัพธ์: 99 — ของเดิมเปลี่ยนด้วย!
```

### ตารางเปรียบเทียบ

| คุณสมบัติ           | Primitive                    | Reference                          |
| ------------------- | ---------------------------- | ---------------------------------- |
| เก็บอะไร            | ค่าจริง                      | ที่อยู่ของ object                  |
| ค่าเป็น `null` ได้  | ไม่ได้                       | ได้                                |
| มี method           | ไม่มี                        | มี (เช่น `name.length()`)          |
| ตั้งชื่อ type       | ตัวพิมพ์เล็ก (`int`)         | ตัวพิมพ์ใหญ่ (`String`, `Integer`) |
| การเปรียบเทียบ `==` | เทียบค่า                     | เทียบ reference (ว่าเป็นตัวเดียวกัน) |

> 💡 **เทียบกับ JavaScript:** หลักการเหมือน JS — primitive (number, string, boolean) copy by value ส่วน object/array copy by reference ข้อแตกต่างคือ **`String` ใน Java เป็น reference type** (แต่ immutable) ดังนั้นห้ามเทียบด้วย `==` (ดู [Operators](operators.md))

---

## 3. `final` — ค่าที่เปลี่ยนไม่ได้

`final` ทำให้ตัวแปร **กำหนดค่าได้ครั้งเดียว** เทียบเท่า `const` ของ JS

### คุณสมบัติ

- กำหนดค่าได้ **ครั้งเดียว** (จะกำหนดตอนประกาศหรือทีหลังก็ได้ แต่ต้องครั้งเดียว)
- สำหรับ reference type — **reference คงที่** แต่ **ข้อมูลภายใน object ยังเปลี่ยนได้**

### ตัวอย่าง

```java
final double VAT_RATE = 0.07;
// VAT_RATE = 0.1; // Error: cannot assign a value to final variable VAT_RATE

final String status;
if (Math.random() > 0.5) {
    status = "ผ่าน";
} else {
    status = "ไม่ผ่าน";
}
// status = "อื่น ๆ"; // Error: variable status might already have been assigned
```

### `final` กับ Object

```java
import java.util.ArrayList;
import java.util.List;

final List<String> colors = new ArrayList<>();
colors.add("แดง");        // ได้ — แก้ไขข้อมูลภายในได้
colors.add("เขียว");
// colors = new ArrayList<>(); // Error — เปลี่ยน reference ไม่ได้

System.out.println(colors); // ผลลัพธ์: [แดง, เขียว]
```

---

## 4. `var` — Local Variable Type Inference (Java 10+)

`var` ให้ compiler **อนุมาน type** จากค่าที่กำหนด — ช่วยลดความยาวของโค้ด แต่ type ยังคง **ตายตัว** เหมือนเดิม

### คุณสมบัติ

- ใช้ได้กับ **local variable เท่านั้น** (ภายใน method, `for` loop, try-with-resources)
- **ต้องกำหนดค่าทันที** ตอนประกาศ (compiler ต้องรู้ type)
- ใช้กับ field, parameter ของ method, return type **ไม่ได้**
- `var x = null;` **ไม่ได้** (อนุมาน type ไม่ได้)

### ตัวอย่าง

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

var name = "สมชาย";            // String
var age = 25;                   // int
var price = 99.5;               // double
var names = new ArrayList<String>();  // ArrayList<String>

// ช่วยได้มากเมื่อ type ยาว
Map<String, List<Integer>> scoresOld = new HashMap<>();
var scores = new HashMap<String, List<Integer>>(); // สั้นกว่า

// age = "ยี่สิบห้า"; // Error — age เป็น int ตายตัวแล้ว
// var x;           // Error: cannot infer type for local variable x
// var y = null;    // Error: variable initializer is 'null'
```

### `var` ใน for loop

```java
var fruits = List.of("แอปเปิ้ล", "กล้วย", "ส้ม");
for (var fruit : fruits) {
    System.out.println(fruit);
}
```

> 💡 **เทียบกับ JavaScript:** `var` ของ Java **ไม่เกี่ยวอะไรกับ `var` ของ JS เลย** — มันคือ `let` ที่ compiler เดา type ให้ ไม่มี hoisting ไม่มี function scope และเปลี่ยน type ภายหลังไม่ได้ ถ้าอยากได้แบบ `const` ให้ใช้ `final var`

---

## 5. Scope ของตัวแปร

ตัวแปรใน Java เป็น **block-scoped** เสมอ — เข้าถึงได้เฉพาะภายใน `{}` ที่ประกาศ

### ตัวอย่าง

```java
public class Main {
    static int counter = 0; // class-level (field) — ใช้ได้ทั้ง class

    public static void main(String[] args) {
        int total = 100; // local variable ของ main

        if (total > 50) {
            String message = "มากกว่า 50"; // อยู่ใน block ของ if เท่านั้น
            System.out.println(message);
        }
        // System.out.println(message); // Error: cannot find symbol

        for (int i = 0; i < 3; i++) {
            counter++;
        }
        // System.out.println(i); // Error: i อยู่ใน for เท่านั้น

        System.out.println(counter); // ผลลัพธ์: 3
    }
}
```

### Shadowing ไม่ได้ใน local scope

```java
int x = 10;
if (true) {
    // int x = 20; // Error: variable x is already defined in method main
}
```

> 💡 **เทียบกับ JavaScript:** JS ยอมให้ประกาศ `let x` ซ้อนใน block ด้านใน (shadowing) ได้ แต่ Java **ไม่ยอม** ให้ local variable ซ้อนชื่อกับ local variable ด้านนอก

---

## 6. ค่าคงที่ระดับ class — `static final`

ค่าคงที่ที่ใช้ร่วมกันทั้งโปรแกรม ประกาศเป็น **field** ด้วย `static final` และตั้งชื่อเป็น `UPPER_SNAKE_CASE`

```java
public class Main {
    public static final double VAT_RATE = 0.07;
    private static final int MAX_LOGIN_ATTEMPTS = 3;
    static final String APP_NAME = "ร้านกาแฟสมชาย";

    public static void main(String[] args) {
        double price = 100;
        double total = price + (price * VAT_RATE);
        System.out.println(APP_NAME + ": " + total); // ผลลัพธ์: ร้านกาแฟสมชาย: 107.0
    }
}
```

| keyword  | ความหมาย                                              |
| -------- | ----------------------------------------------------- |
| `static` | เป็นของ class ไม่ใช่ของ object — มีสำเนาเดียว         |
| `final`  | กำหนดค่าได้ครั้งเดียว                                 |
| `public` / `private` | ใครเข้าถึงได้ (ดูเพิ่มใน [Class & Object](../oop/class-object.md)) |

---

## 7. Naming Conventions

| ประเภท         | รูปแบบ             | ตัวอย่าง                       |
| -------------- | ------------------ | ------------------------------ |
| ตัวแปร / field | `camelCase`        | `firstName`, `totalPrice`      |
| ค่าคงที่       | `UPPER_SNAKE_CASE` | `MAX_SIZE`, `VAT_RATE`         |
| boolean        | ขึ้นต้นด้วย is/has/can | `isActive`, `hasPermission` |

### กฎการตั้งชื่อ

- ขึ้นต้นด้วยตัวอักษร, `_` หรือ `$` (ห้ามขึ้นต้นด้วยตัวเลข)
- **case-sensitive** — `name` กับ `Name` คนละตัว
- ห้ามใช้ keyword เช่น `int`, `class`, `new`, `final`
- `_` ตัวเดียวเป็น keyword (ใช้เป็นชื่อตัวแปรไม่ได้ตั้งแต่ Java 9)

```java
int userAge = 25;        // ✅
int _count = 0;          // ✅ แต่ไม่นิยม
// int 2ndPlace = 2;     // ❌ ขึ้นต้นด้วยตัวเลข
// int class = 1;        // ❌ keyword
```

---

## ตารางเปรียบเทียบกับ JavaScript

| คุณสมบัติ              | JS `var`          | JS `let`   | JS `const` | Java `Type x` | Java `var x`  | Java `final` |
| ---------------------- | ----------------- | ---------- | ---------- | ------------- | ------------- | ------------ |
| Scope                  | Function          | Block      | Block      | Block         | Block         | Block        |
| Hoisting               | ได้ (undefined)   | TDZ        | TDZ        | ไม่มี         | ไม่มี         | ไม่มี        |
| ประกาศซ้ำได้           | ได้               | ไม่ได้     | ไม่ได้     | ไม่ได้        | ไม่ได้        | ไม่ได้       |
| กำหนดค่าใหม่ได้        | ได้               | ได้        | ไม่ได้     | ได้           | ได้           | ไม่ได้       |
| เปลี่ยน type ได้       | ได้               | ได้        | -          | ไม่ได้        | ไม่ได้        | -            |
| ต้องกำหนดค่าตอนประกาศ  | ไม่บังคับ         | ไม่บังคับ  | บังคับ     | ไม่บังคับ     | บังคับ        | ไม่บังคับ*   |

\* `final` local variable กำหนดค่าทีหลังได้ แต่ได้แค่ครั้งเดียว

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ประกาศตัวแปรใกล้จุดที่ใช้** และให้ scope แคบที่สุด
2. **ใช้ `final`** กับค่าที่ไม่ควรเปลี่ยน — ช่วยให้อ่านง่ายและป้องกัน bug
3. **ใช้ `var` เมื่อ type ชัดเจนจากฝั่งขวา** เช่น `var list = new ArrayList<String>();` แต่หลีกเลี่ยงเมื่อ type ไม่ชัด เช่น `var result = process();`
4. **ค่าคงที่ใช้ `static final` + `UPPER_SNAKE_CASE`** แทนการเขียน magic number กระจายในโค้ด
5. **ตั้งชื่อให้สื่อความหมาย** — `customerCount` ดีกว่า `c`

```java
public class Main {
    private static final int MAX_RETRIES = 3; // ค่าคงที่

    public static void main(String[] args) {
        int currentRetry = 0;                 // ค่าที่จะเปลี่ยนแปลง
        final var customerName = "สมชาย";     // ค่าที่ไม่เปลี่ยน

        while (currentRetry < MAX_RETRIES) {
            currentRetry++;
        }
        System.out.println(customerName + " retry " + currentRetry + " ครั้ง");
        // ผลลัพธ์: สมชาย retry 3 ครั้ง
    }
}
```

---
