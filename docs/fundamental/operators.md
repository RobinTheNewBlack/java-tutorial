# Java Fundamentals

# ส่วนที่ 3: ตัวดำเนินการ (Operators)

## Operators คืออะไร?

Operator คือสัญลักษณ์ที่ใช้ดำเนินการกับค่า (operand) เช่น บวก ลบ เปรียบเทียบ ส่วนใหญ่หน้าตาเหมือน JavaScript แต่มี **พฤติกรรมต่างกันในหลายจุดสำคัญ** โดยเฉพาะการหารและการเปรียบเทียบ

---

## 1. Arithmetic Operators (คณิตศาสตร์)

| Operator | ความหมาย   | ตัวอย่าง  | ผลลัพธ์ |
| -------- | ---------- | --------- | ------- |
| `+`      | บวก        | `7 + 2`   | `9`     |
| `-`      | ลบ         | `7 - 2`   | `5`     |
| `*`      | คูณ        | `7 * 2`   | `14`    |
| `/`      | หาร        | `7 / 2`   | `3` ⚠️  |
| `%`      | หารเอาเศษ  | `7 % 2`   | `1`     |

### Integer Division — หารจำนวนเต็มได้จำนวนเต็ม

ถ้าทั้งสองฝั่งเป็นจำนวนเต็ม ผลลัพธ์จะ **ตัดทศนิยมทิ้ง** (ปัดเข้าหา 0)

```java
System.out.println(7 / 2);      // ผลลัพธ์: 3
System.out.println(-7 / 2);     // ผลลัพธ์: -3 (ปัดเข้าหา 0 ไม่ใช่ -4)
System.out.println(7 / 2.0);    // ผลลัพธ์: 3.5 (มี double อย่างน้อย 1 ฝั่ง)
System.out.println(7.0 / 2);    // ผลลัพธ์: 3.5
System.out.println((double) 7 / 2); // ผลลัพธ์: 3.5 (cast ก่อนหาร)
System.out.println((double) (7 / 2)); // ผลลัพธ์: 3.0 ⚠️ หารเสร็จแล้วค่อย cast
```

### ตัวอย่างที่มักพลาด — คำนวณค่าเฉลี่ย

```java
int total = 250;
int count = 3;

double wrong = total / count;           // 83.0 ❌ หารแบบ int ก่อนแล้วค่อยแปลง
double right = (double) total / count;  // 83.33333333333333 ✅
```

### หารด้วยศูนย์

```java
// System.out.println(10 / 0);   // ArithmeticException: / by zero
System.out.println(10.0 / 0);    // ผลลัพธ์: Infinity
System.out.println(0.0 / 0);     // ผลลัพธ์: NaN
```

### Modulo `%`

```java
System.out.println(10 % 3);    // ผลลัพธ์: 1
System.out.println(-10 % 3);   // ผลลัพธ์: -1 (เครื่องหมายตามตัวตั้ง)
System.out.println(5.5 % 2);   // ผลลัพธ์: 1.5 (ใช้กับ double ได้)
System.out.println(Math.floorMod(-10, 3)); // ผลลัพธ์: 2 (ได้ค่าบวกเสมอเมื่อตัวหารเป็นบวก)
```

> 💡 **เทียบกับ JavaScript:** ใน JS `7 / 2` ได้ `3.5` เสมอ เพราะมี `number` ชนิดเดียว แต่ใน Java `int / int` = `int` และ `10 / 0` ของจำนวนเต็มจะ **throw exception** แทนที่จะได้ `Infinity`

### Overflow — ล้นแบบเงียบ ๆ

```java
int max = Integer.MAX_VALUE;
System.out.println(max + 1);   // ผลลัพธ์: -2147483648 (วนกลับไปค่าต่ำสุด ไม่มี error!)
// ใช้ Math.addExact(max, 1) ถ้าอยากให้ throw ArithmeticException
```

---

## 2. Assignment Operators (กำหนดค่า)

| Operator | ตัวอย่าง  | เทียบเท่า     |
| -------- | --------- | ------------- |
| `=`      | `x = 5`   | -             |
| `+=`     | `x += 3`  | `x = x + 3`   |
| `-=`     | `x -= 3`  | `x = x - 3`   |
| `*=`     | `x *= 3`  | `x = x * 3`   |
| `/=`     | `x /= 3`  | `x = x / 3`   |
| `%=`     | `x %= 3`  | `x = x % 3`   |

```java
int score = 10;
score += 5;   // 15
score *= 2;   // 30
score /= 4;   // 7 (integer division)

String msg = "สวัสดี";
msg += " สมชาย";
System.out.println(msg); // ผลลัพธ์: สวัสดี สมชาย
```

### Compound assignment มี cast ซ่อนอยู่

```java
byte b = 10;
// b = b + 5;   // Error: b + 5 เป็น int ต้อง cast
b += 5;         // ได้ — เทียบเท่า b = (byte) (b + 5)
```

---

## 3. Increment / Decrement

```java
int a = 5;
int b = a++;   // post-increment: ใช้ค่าก่อน แล้วค่อยเพิ่ม → b = 5, a = 6
int c = ++a;   // pre-increment: เพิ่มก่อน แล้วค่อยใช้ → a = 7, c = 7

System.out.println(a + " " + b + " " + c); // ผลลัพธ์: 7 5 7
```

---

## 4. Comparison Operators (เปรียบเทียบ)

| Operator | ความหมาย             |
| -------- | -------------------- |
| `==`     | เท่ากับ              |
| `!=`     | ไม่เท่ากับ           |
| `>`      | มากกว่า              |
| `<`      | น้อยกว่า             |
| `>=`     | มากกว่าหรือเท่ากับ   |
| `<=`     | น้อยกว่าหรือเท่ากับ  |

### `==` vs `.equals()` — สำคัญที่สุดในหัวข้อนี้

- **Primitive** → `==` เทียบ **ค่า**
- **Reference (object)** → `==` เทียบว่าเป็น **object ตัวเดียวกันหรือไม่** ส่วน `.equals()` เทียบ **เนื้อหา**

```java
int x = 10, y = 10;
System.out.println(x == y);            // ผลลัพธ์: true (primitive)

String s1 = "สมชาย";
String s2 = "สมชาย";
String s3 = new String("สมชาย");

System.out.println(s1 == s2);          // ผลลัพธ์: true  (String pool — บังเอิญเป็นตัวเดียวกัน)
System.out.println(s1 == s3);          // ผลลัพธ์: false (คนละ object)
System.out.println(s1.equals(s3));     // ผลลัพธ์: true  ✅
System.out.println("สมชาย".equalsIgnoreCase("สมชาย")); // ผลลัพธ์: true
```

### เทียบอย่างปลอดภัยเมื่ออาจเป็น `null`

```java
import java.util.Objects;

String input = null;
// input.equals("admin");                 // NullPointerException
System.out.println("admin".equals(input)); // ผลลัพธ์: false ✅ เอา literal ไว้หน้า
System.out.println(Objects.equals(input, null)); // ผลลัพธ์: true ✅ null-safe
```

> 💡 **เทียบกับ JavaScript:** Java **ไม่มี `===`** และไม่มี type coercion — `5 == "5"` จะ compile ไม่ผ่านเลย ส่วนการเทียบ object/String ต้องใช้ `.equals()` เพราะ `==` ของ Java คล้าย `===` ของ JS ที่ใช้กับ object (เทียบ reference)

---

## 5. Logical Operators (ตรรกะ)

| Operator | ความหมาย | ตัวอย่าง             |
| -------- | -------- | -------------------- |
| `&&`     | AND      | `a > 0 && b > 0`     |
| `\|\|`   | OR       | `a > 0 \|\| b > 0`   |
| `!`      | NOT      | `!isActive`          |

### Short-circuit evaluation

`&&` และ `||` จะ **หยุดประเมินทันที** เมื่อรู้ผลแล้ว — ใช้ป้องกัน `NullPointerException` ได้

```java
String name = null;

if (name != null && name.length() > 3) {   // ฝั่งขวาไม่ถูกเรียกเพราะฝั่งซ้าย false
    System.out.println("ชื่อยาว");
}

boolean isAdmin = true;
if (isAdmin || checkPermission()) {        // checkPermission() ไม่ถูกเรียก
    System.out.println("เข้าได้");
}
```

> `&` และ `|` (ตัวเดียว) ใช้กับ boolean ได้เช่นกัน แต่ **ไม่ short-circuit** — ประเมินทั้งสองฝั่งเสมอ

> 💡 **เทียบกับ JavaScript:** ใน JS `a || "default"` คืนค่า `a` หรือ `"default"` ได้ แต่ใน Java `&&`/`||` **รับและคืนเฉพาะ `boolean`** — `"" || "x"` compile ไม่ผ่าน และไม่มี truthy/falsy (`if (count)` ใช้ไม่ได้ ต้องเขียน `if (count != 0)`)

---

## 6. Ternary Operator

```java
int age = 20;
String status = age >= 18 ? "ผู้ใหญ่" : "เด็ก";
System.out.println(status); // ผลลัพธ์: ผู้ใหญ่

// ใช้แทน ?? ของ JS
String nickname = null;
String display = nickname != null ? nickname : "ไม่ระบุ";
System.out.println(display); // ผลลัพธ์: ไม่ระบุ
```

> ทั้งสองฝั่งของ `:` ต้องเป็น type ที่เข้ากันได้

---

## 7. Bitwise Operators (โดยสังเขป)

| Operator | ความหมาย                            | ตัวอย่าง     | ผลลัพธ์ |
| -------- | ----------------------------------- | ------------ | ------- |
| `&`      | AND                                 | `5 & 3`      | `1`     |
| `\|`     | OR                                  | `5 \| 3`     | `7`     |
| `^`      | XOR                                 | `5 ^ 3`      | `6`     |
| `~`      | NOT                                 | `~5`         | `-6`    |
| `<<`     | shift ซ้าย                          | `1 << 3`     | `8`     |
| `>>`     | shift ขวา (รักษาเครื่องหมาย)        | `-16 >> 2`   | `-4`    |
| `>>>`    | shift ขวา (เติม 0)                  | `-16 >>> 28` | `15`    |

```java
int flags = 0b0101;
boolean hasBit0 = (flags & 1) != 0;   // ต้องเทียบกับ 0 เอง (ไม่มี truthy)
System.out.println(hasBit0);          // ผลลัพธ์: true
```

---

## 8. `instanceof` และ Pattern Matching

`instanceof` ใช้ตรวจว่า object เป็น type ที่ระบุหรือไม่

### แบบเดิม (ต้อง cast เอง)

```java
Object value = "สวัสดี";

if (value instanceof String) {
    String s = (String) value;     // cast ซ้ำ
    System.out.println(s.length()); // ผลลัพธ์: 6
}
```

### Pattern Matching for `instanceof` (Java 16+)

ประกาศตัวแปรพร้อม cast ในตัว

```java
Object value = "สวัสดี";

if (value instanceof String s) {          // ถ้าเป็น String → ได้ตัวแปร s เลย
    System.out.println(s.length());       // ผลลัพธ์: 6
}

if (value instanceof String s && s.length() > 3) { // ใช้ s ต่อใน && ได้
    System.out.println("ข้อความยาว: " + s);
}

Object num = 42;
if (!(num instanceof Integer i)) {
    return;
}
System.out.println(i * 2); // ผลลัพธ์: 84 — i ใช้ได้หลัง if เพราะ return ไปแล้ว
```

> 💡 **เทียบกับ JavaScript:** คล้าย `typeof x === "string"` / `x instanceof Foo` แต่ Java ได้ตัวแปรที่ "แคบลง" มาใช้ทันที คล้าย type narrowing ของ TypeScript

---

## 9. Operator Precedence (ลำดับความสำคัญ)

เรียงจาก **สูง → ต่ำ** (ทำก่อน → ทำทีหลัง)

| ลำดับ | Operators                                   | ทิศทาง        |
| ----- | ------------------------------------------- | ------------- |
| 1     | `x++` `x--` `.` `[]` `()` (method call)     | ซ้าย → ขวา    |
| 2     | `++x` `--x` `+x` `-x` `!` `~` `(type)`      | ขวา → ซ้าย    |
| 3     | `*` `/` `%`                                 | ซ้าย → ขวา    |
| 4     | `+` `-`                                     | ซ้าย → ขวา    |
| 5     | `<<` `>>` `>>>`                             | ซ้าย → ขวา    |
| 6     | `<` `>` `<=` `>=` `instanceof`              | ซ้าย → ขวา    |
| 7     | `==` `!=`                                   | ซ้าย → ขวา    |
| 8     | `&`                                         | ซ้าย → ขวา    |
| 9     | `^`                                         | ซ้าย → ขวา    |
| 10    | `\|`                                        | ซ้าย → ขวา    |
| 11    | `&&`                                        | ซ้าย → ขวา    |
| 12    | `\|\|`                                      | ซ้าย → ขวา    |
| 13    | `? :`                                       | ขวา → ซ้าย    |
| 14    | `=` `+=` `-=` `*=` `/=` `%=` ฯลฯ            | ขวา → ซ้าย    |

```java
int result = 2 + 3 * 4;        // 14 (คูณก่อน)
int result2 = (2 + 3) * 4;     // 20
boolean ok = 5 > 3 && 2 > 1;   // true (เปรียบเทียบก่อน &&)
System.out.println("ผลรวม: " + 1 + 2);   // ผลลัพธ์: ผลรวม: 12 ⚠️
System.out.println("ผลรวม: " + (1 + 2)); // ผลลัพธ์: ผลรวม: 3 ✅
```

---

## ตารางเปรียบเทียบกับ JavaScript

| เรื่อง                    | JavaScript                     | Java                                      |
| ------------------------- | ------------------------------ | ----------------------------------------- |
| `7 / 2`                   | `3.5`                          | `3` (int / int)                           |
| `10 / 0`                  | `Infinity`                     | `ArithmeticException` (int)               |
| เทียบค่า                  | `===`                          | `==` (primitive), `.equals()` (object)    |
| `5 == "5"`                | `true`                         | compile error                             |
| `a \|\| b`                | คืนค่า a หรือ b                | คืน `boolean` เท่านั้น                    |
| `a ?? b`                  | มี                             | ไม่มี — ใช้ ternary / `Objects.requireNonNullElse` |
| `a?.b`                    | มี                             | ไม่มี — ใช้ `Optional` หรือเช็ก null      |
| `**` (ยกกำลัง)            | `2 ** 3`                       | ไม่มี — ใช้ `Math.pow(2, 3)`              |
| `typeof`                  | มี                             | ไม่มี — ใช้ `instanceof`                  |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ระวัง integer division** — cast เป็น `double` ก่อนหารเมื่อต้องการทศนิยม
2. **เทียบ object ด้วย `.equals()`** เสมอ ใช้ `==` เฉพาะ primitive (และ enum)
3. **เขียน literal ไว้ด้านหน้า** — `"admin".equals(role)` ป้องกัน NPE
4. **ใช้วงเล็บ** เมื่อ expression ซับซ้อน แม้จะรู้ลำดับ precedence ก็ตาม
5. **ใช้ pattern matching `instanceof`** (Java 16+) แทนการ cast เอง
6. **ใช้ `Math.addExact` / `multiplyExact`** เมื่อกลัว overflow

```java
public class Main {
    public static void main(String[] args) {
        int totalScore = 250;
        int students = 3;
        String role = null;

        double avg = (double) totalScore / students;
        boolean isAdmin = "admin".equals(role);

        System.out.printf("เฉลี่ย %.2f, admin = %b%n", avg, isAdmin);
        // ผลลัพธ์: เฉลี่ย 83.33, admin = false
    }
}
```

---
