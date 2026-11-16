# Java Fundamentals

# ส่วนที่ 2: ชนิดข้อมูล (Data Types)

## ชนิดข้อมูลใน Java

Java แบ่งชนิดข้อมูลเป็น 2 กลุ่ม

- **Primitive types** — มี 8 ชนิด เก็บค่าโดยตรง เร็วและประหยัด memory
- **Reference types** — `String`, array, class, interface, record, enum ฯลฯ

> 💡 **เทียบกับ JavaScript:** JS มี `number` ชนิดเดียวสำหรับตัวเลขทุกแบบ (64-bit floating point) แต่ Java แยกเป็นจำนวนเต็ม 4 ขนาดและทศนิยม 2 ขนาด — ต้องเลือกให้เหมาะกับข้อมูล

---

## 1. Primitive Types ทั้ง 8 ชนิด

| Type      | ขนาด    | ช่วงค่า                                          | ค่า default | ตัวอย่าง           |
| --------- | ------- | ------------------------------------------------ | ----------- | ------------------ |
| `byte`    | 8 bit   | -128 ถึง 127                                     | `0`         | `byte b = 100;`    |
| `short`   | 16 bit  | -32,768 ถึง 32,767                               | `0`         | `short s = 3000;`  |
| `int`     | 32 bit  | -2,147,483,648 ถึง 2,147,483,647 (~±2.1 พันล้าน) | `0`         | `int i = 42;`      |
| `long`    | 64 bit  | ~±9.2 × 10¹⁸                                     | `0L`        | `long l = 42L;`    |
| `float`   | 32 bit  | ทศนิยม ~6-7 หลัก                                 | `0.0f`      | `float f = 3.14f;` |
| `double`  | 64 bit  | ทศนิยม ~15-16 หลัก                               | `0.0`       | `double d = 3.14;` |
| `char`    | 16 bit  | ตัวอักษร Unicode 1 ตัว (`'\u0000'` ถึง `'￿'`) | `'\u0000'` | `char c = 'ก';`    |
| `boolean` | -       | `true` / `false`                                 | `false`     | `boolean ok = true;` |

### ตัวอย่าง

```java
public class Main {
    public static void main(String[] args) {
        int population = 66_000_000;
        long worldPopulation = 8_000_000_000L; // เกิน int ต้องใช้ long
        double price = 59.75;
        float rating = 4.5f;
        char initial = 'ส';
        boolean isOpen = true;

        System.out.println(Integer.MAX_VALUE); // ผลลัพธ์: 2147483647
        System.out.println(Long.MAX_VALUE);    // ผลลัพธ์: 9223372036854775807
        System.out.println(initial);           // ผลลัพธ์: ส
    }
}
```

### เลือกใช้อะไรดี?

- จำนวนเต็มทั่วไป → **`int`**
- จำนวนเต็มที่อาจเกิน 2.1 พันล้าน (timestamp, ID ขนาดใหญ่) → **`long`**
- ทศนิยมทั่วไป → **`double`**
- เงิน → **`BigDecimal`** (ไม่ใช่ `double` — ดู [Math](../data-manipulation/math.md))
- `byte` / `short` / `float` — ใช้เฉพาะกรณีพิเศษ (ไฟล์ binary, ประหยัด memory)

---

## 2. Literals

### คุณสมบัติ

- ตัวเลขจำนวนเต็มเป็น `int` โดย default → ต้องเติม **`L`** สำหรับ `long`
- ตัวเลขทศนิยมเป็น `double` โดย default → ต้องเติม **`f`** สำหรับ `float`
- ใช้ **`_`** คั่นหลักได้ (Java 7+) เพื่อให้อ่านง่าย
- `char` ใช้ **single quote** `'A'` ส่วน `String` ใช้ **double quote** `"A"`

### ตัวอย่าง

```java
long big = 3_000_000_000L;     // ไม่มี L จะ error: integer number too large
float pi = 3.14f;              // ไม่มี f จะ error: possible lossy conversion from double to float
double sci = 1.5e3;            // 1500.0 (scientific notation)

int hex = 0xFF;                // 255 (เลขฐาน 16)
int bin = 0b1010;              // 10  (เลขฐาน 2)
int oct = 010;                 // 8   (เลขฐาน 8 — ระวัง! เลข 0 นำหน้า)

char letter = 'A';
char unicode = 'ก';       // 'ก'
String text = "สวัสดี";
```

> 💡 **เทียบกับ JavaScript:** ใน JS `'A'` กับ `"A"` คือ string เหมือนกัน แต่ใน Java `'A'` คือ `char` (ตัวอักษรเดียว) และ `"A"` คือ `String` — สลับกันใช้ไม่ได้

---

## 3. Default Values

ค่า default ใช้กับ **field ของ class** และ **สมาชิกของ array** เท่านั้น — local variable ไม่มีค่า default (ต้องกำหนดเอง)

```java
public class Main {
    static int count;        // 0
    static double amount;    // 0.0
    static boolean active;   // false
    static String name;      // null (reference type ทุกตัว default เป็น null)

    public static void main(String[] args) {
        System.out.println(count);  // ผลลัพธ์: 0
        System.out.println(active); // ผลลัพธ์: false
        System.out.println(name);   // ผลลัพธ์: null

        int[] scores = new int[3];
        System.out.println(scores[0]); // ผลลัพธ์: 0

        int local;
        // System.out.println(local); // Error: variable local might not have been initialized
    }
}
```

---

## 4. Wrapper Classes

primitive แต่ละตัวมี **wrapper class** ที่เป็น object คู่กัน ใช้เมื่อต้องการ object เช่น เก็บใน Collection (`List<int>` ใช้ไม่ได้ ต้องเป็น `List<Integer>`)

| Primitive | Wrapper     |
| --------- | ----------- |
| `byte`    | `Byte`      |
| `short`   | `Short`     |
| `int`     | `Integer`   |
| `long`    | `Long`      |
| `float`   | `Float`     |
| `double`  | `Double`    |
| `char`    | `Character` |
| `boolean` | `Boolean`   |

### Method ที่มีประโยชน์

```java
int n = Integer.parseInt("123");          // String → int
double d = Double.parseDouble("3.14");    // String → double
String s = Integer.toString(42);          // int → String
String s2 = String.valueOf(42);           // int → String (ใช้ได้กับทุก type)

System.out.println(Integer.MAX_VALUE);    // ผลลัพธ์: 2147483647
System.out.println(Integer.toBinaryString(10)); // ผลลัพธ์: 1010
System.out.println(Character.isDigit('5'));     // ผลลัพธ์: true
System.out.println(Character.isLetter('ก'));    // ผลลัพธ์: true

// Integer.parseInt("abc"); // NumberFormatException
```

> 💡 **เทียบกับ JavaScript:** `Integer.parseInt("12abc")` จะ **throw `NumberFormatException`** ขณะที่ `parseInt("12abc")` ของ JS คืน `12` และ `Number("abc")` คืน `NaN` — Java ไม่มี `NaN` สำหรับจำนวนเต็ม

---

## 5. Autoboxing & Unboxing

Java แปลงระหว่าง primitive ↔ wrapper ให้อัตโนมัติ

- **Autoboxing** — `int` → `Integer`
- **Unboxing** — `Integer` → `int`

```java
import java.util.ArrayList;
import java.util.List;

Integer boxed = 10;        // autoboxing (Integer.valueOf(10))
int unboxed = boxed;       // unboxing (boxed.intValue())

List<Integer> numbers = new ArrayList<>();
numbers.add(5);            // autoboxing int → Integer
int first = numbers.get(0); // unboxing Integer → int
```

### ⚠️ กับดัก 1: `null` unboxing → NullPointerException

```java
Integer maybe = null;
// int value = maybe; // NullPointerException ตอนรัน!
```

### ⚠️ กับดัก 2: Integer cache — `==` กับ `Integer` ที่มีค่ามากกว่า 127

Java cache object `Integer` ค่า **-128 ถึง 127** ไว้ ทำให้ `==` (ที่เทียบ reference) ให้ผลไม่สม่ำเสมอ

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);      // ผลลัพธ์: true  (ได้ object เดียวกันจาก cache)

Integer c = 128;
Integer d = 128;
System.out.println(c == d);      // ผลลัพธ์: false (คนละ object!)
System.out.println(c.equals(d)); // ผลลัพธ์: true  ✅ ใช้ equals เสมอ

int e = 128;
System.out.println(c == e);      // ผลลัพธ์: true  (c ถูก unbox เป็น int ก่อนเทียบ)
```

> 💡 **เทียบกับ JavaScript:** ใน JS `128 === 128` เป็น `true` เสมอ แต่ใน Java ถ้าเป็น **wrapper object** (`Integer`, `Long`) ต้องเทียบด้วย `.equals()` หรือ `Objects.equals(a, b)` เท่านั้น

---

## 6. String เบื้องต้น

`String` เป็น **reference type** (class) ไม่ใช่ primitive แต่มี syntax พิเศษให้สร้างด้วย `"..."` ได้

```java
String firstName = "สมชาย";
String lastName = "ใจดี";
String fullName = firstName + " " + lastName;   // ต่อ string ด้วย +

System.out.println(fullName);           // ผลลัพธ์: สมชาย ใจดี
System.out.println(fullName.length());  // ผลลัพธ์: 10
System.out.println("อายุ " + 25);       // ผลลัพธ์: อายุ 25 (int ถูกแปลงเป็น String)
System.out.println(1 + 2 + "3");        // ผลลัพธ์: 33 (คำนวณซ้ายไปขวา: 3 + "3")
System.out.println("1" + 2 + 3);        // ผลลัพธ์: 123
```

### คุณสมบัติ

- **Immutable** — แก้ไขไม่ได้ ทุก method คืน String ใหม่
- เทียบเนื้อหาด้วย **`.equals()`** ไม่ใช่ `==`
- รายละเอียดเพิ่มเติมดูที่ [String Methods](../data-manipulation/string-methods.md)

---

## 7. Type Casting (การแปลงชนิดข้อมูล)

### Widening (ขยาย) — อัตโนมัติ

แปลงจาก type เล็กไปใหญ่ **ไม่สูญเสียข้อมูล** Java ทำให้อัตโนมัติ

```
byte → short → int → long → float → double
                ↑
              char
```

```java
int i = 100;
long l = i;        // int → long อัตโนมัติ
double d = l;      // long → double อัตโนมัติ
System.out.println(d); // ผลลัพธ์: 100.0
```

### Narrowing (บีบ) — ต้อง cast เอง

แปลงจาก type ใหญ่ไปเล็ก **อาจสูญเสียข้อมูล** ต้องเขียน `(type)` เอง

```java
double price = 99.99;
int rounded = (int) price;      // ตัดทศนิยมทิ้ง (ไม่ปัดเศษ)
System.out.println(rounded);    // ผลลัพธ์: 99

int big = 300;
byte small = (byte) big;        // ล้นช่วง -128..127
System.out.println(small);      // ผลลัพธ์: 44 (300 - 256)

long huge = 3_000_000_000L;
int overflow = (int) huge;
System.out.println(overflow);   // ผลลัพธ์: -1294967296
```

### char ↔ int

```java
char c = 'A';
int code = c;                  // widening: char → int
System.out.println(code);      // ผลลัพธ์: 65

char next = (char) (c + 1);    // ต้อง cast กลับ
System.out.println(next);      // ผลลัพธ์: B
```

### ตารางสรุป

| แบบ       | ทิศทาง          | ต้อง cast? | เสียข้อมูล?       | ตัวอย่าง                |
| --------- | --------------- | ---------- | ----------------- | ----------------------- |
| Widening  | เล็ก → ใหญ่     | ไม่        | ไม่ (ยกเว้น long→float/double อาจเสียความละเอียด) | `long x = 5;` |
| Narrowing | ใหญ่ → เล็ก     | ต้อง       | อาจเสีย           | `int x = (int) 9.9;`    |

---

## 8. `null`

`null` หมายถึง **ตัวแปร reference ที่ไม่ได้ชี้ไปที่ object ใด** ใช้ได้กับ reference type เท่านั้น

```java
String name = null;            // ได้
// int age = null;             // Error: primitive เป็น null ไม่ได้
Integer maybeAge = null;       // ได้ — wrapper เป็น reference

// name.length();              // NullPointerException ตอนรัน

if (name != null) {
    System.out.println(name.length());
}
```

> 💡 **เทียบกับ JavaScript:** Java มีแค่ `null` **ไม่มี `undefined`** และไม่มี optional chaining `?.` หรือ `??` — ต้องเช็ก `!= null` เอง หรือใช้ [`Optional`](../data-manipulation/optional.md) / `Objects.requireNonNullElse(x, "default")`

---

## ตารางเปรียบเทียบกับ JavaScript

| JavaScript                  | Java                                          |
| --------------------------- | --------------------------------------------- |
| `number`                    | `int`, `long`, `double` (+ `byte`, `short`, `float`) |
| `bigint`                    | `java.math.BigInteger`                        |
| `string`                    | `String` (reference) / `char` (ตัวอักษรเดียว) |
| `boolean`                   | `boolean`                                     |
| `null`                      | `null`                                        |
| `undefined`                 | ไม่มี                                         |
| `NaN`                       | มีเฉพาะ `double`/`float` (`Double.NaN`)       |
| `typeof x`                  | รู้ตั้งแต่ compile time / `x.getClass()`      |
| truthy / falsy              | ไม่มี — `if` รับเฉพาะ `boolean`                |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ `int` และ `double` เป็นหลัก** — ใช้ `long` เมื่อค่าอาจเกิน 2.1 พันล้าน
2. **ห้ามใช้ `double` กับเงิน** — ใช้ `BigDecimal`
3. **เทียบ wrapper (`Integer`, `Long`) และ `String` ด้วย `.equals()`** ไม่ใช่ `==`
4. **ระวัง unboxing `null`** — ถ้าค่าเป็น `null` ได้ ให้เช็กก่อนหรือใช้ primitive ตั้งแต่ต้น
5. **ใช้ `_` คั่นหลัก** ในตัวเลขยาว ๆ เช่น `1_000_000`
6. **ระวัง narrowing cast** — ข้อมูลอาจล้นโดยไม่มี error

```java
public class Main {
    public static void main(String[] args) {
        long totalViews = 3_500_000_000L;   // ใช้ long เพราะเกิน int
        double average = 4.75;
        Integer a = 1000, b = 1000;

        System.out.println(a.equals(b));    // ผลลัพธ์: true ✅
        System.out.println((int) average);  // ผลลัพธ์: 4 (ตัดทศนิยม)
        System.out.println(totalViews);     // ผลลัพธ์: 3500000000
    }
}
```

---
