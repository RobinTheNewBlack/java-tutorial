# String Manipulation

# ส่วนที่ 2: String ใน Java

## String คืออะไร?

`String` คือ class สำหรับเก็บข้อความ (ลำดับของ `char`) เป็น **reference type** แต่มี syntax พิเศษให้สร้างด้วย `"..."` ได้

---

## 1. Immutability — String แก้ไขไม่ได้

### คุณสมบัติ

- เมื่อสร้างแล้ว **เนื้อหาเปลี่ยนไม่ได้**
- ทุก method ที่ดูเหมือน "แก้ไข" (เช่น `toUpperCase`, `replace`) จะ **คืน String ใหม่**
- ปลอดภัยเมื่อใช้ร่วมกันหลาย thread และใช้เป็น key ของ `HashMap` ได้

### ตัวอย่าง

```java
String name = "somchai";
name.toUpperCase();                 // คืน String ใหม่ แต่ไม่ได้เก็บไว้
System.out.println(name);           // ผลลัพธ์: somchai ⚠️ ไม่เปลี่ยน

name = name.toUpperCase();          // ✅ ต้องกำหนดค่ากลับ
System.out.println(name);           // ผลลัพธ์: SOMCHAI
```

> 💡 **เทียบกับ JavaScript:** เหมือนกับ JS — string เป็น immutable และ method คืนค่าใหม่เสมอ

---

## 2. String Pool และการเปรียบเทียบ

Java เก็บ string literal ไว้ใน **String pool** — literal ที่มีเนื้อหาเหมือนกันจะชี้ไปที่ object เดียวกัน

```java
String a = "สมชาย";              // อยู่ใน pool
String b = "สมชาย";              // ใช้ตัวเดียวกับ a
String c = new String("สมชาย");  // สร้าง object ใหม่นอก pool

System.out.println(a == b);           // ผลลัพธ์: true  (บังเอิญเป็น object เดียวกัน)
System.out.println(a == c);           // ผลลัพธ์: false (คนละ object)
System.out.println(a.equals(c));      // ผลลัพธ์: true  ✅
System.out.println(a == c.intern());  // ผลลัพธ์: true  (intern() คืนตัวใน pool)
```

### วิธีเปรียบเทียบที่ถูกต้อง

```java
String input = "Admin";

input.equals("admin");              // false — case-sensitive
input.equalsIgnoreCase("admin");    // true
"admin".equals(input);              // ปลอดภัยจาก NPE ถ้า input เป็น null
input.compareTo("Bob");             // ค่าติดลบ — "Admin" มาก่อน "Bob" ตามลำดับ Unicode
```

> 💡 **เทียบกับ JavaScript:** ใน JS `"a" === "a"` เทียบเนื้อหาเสมอ แต่ใน Java **ห้ามใช้ `==` กับ String** — ใช้ `.equals()` เท่านั้น เพราะ `==` เทียบ reference (ผลลัพธ์อาจถูกบ้างผิดบ้างตาม pool)

---

## 3. Method พื้นฐาน

### ความยาวและการเข้าถึงตัวอักษร

```java
String text = "Hello Java";

text.length();          // 10 — เป็น method (มีวงเล็บ) ต่างจาก array.length
text.charAt(0);         // 'H' — คืน char
text.charAt(text.length() - 1); // 'a' — ตัวสุดท้าย
text.isEmpty();         // false — length() == 0
"   ".isBlank();        // true — ว่างหรือมีแต่ whitespace (Java 11+)
// text.charAt(20);     // StringIndexOutOfBoundsException
```

### ค้นหา

```java
String email = "somchai@example.com";

email.indexOf("@");            // 7
email.indexOf("e");            // 8 (ตัวแรกที่เจอ)
email.lastIndexOf(".");        // 15
email.indexOf("zzz");          // -1 — ไม่เจอ
email.contains("example");     // true
email.startsWith("som");       // true
email.endsWith(".com");        // true
email.matches("[\\w.]+@[\\w.]+"); // true — ตรวจด้วย regex (ต้อง match ทั้ง string)
```

### ตัดข้อความ — `substring`

```java
String s = "Hello Java";

s.substring(6);        // "Java" — จาก index 6 ถึงท้าย
s.substring(0, 5);     // "Hello" — index 0 ถึง 4 (end ไม่รวม)
// s.substring(-3);    // StringIndexOutOfBoundsException (ไม่มี index ติดลบ)
s.substring(s.length() - 4); // "Java" — 4 ตัวท้าย
```

### แปลงตัวพิมพ์ / ตัดช่องว่าง

```java
"Java".toUpperCase();          // "JAVA"
"Java".toLowerCase();          // "java"

"  สวัสดี  ".trim();           // "สวัสดี" — ตัด whitespace ASCII (≤ U+0020)
"  สวัสดี  ".strip();          // "สวัสดี" — ตัด whitespace ทุกแบบรวม Unicode (Java 11+)
"  สวัสดี  ".stripLeading();   // "สวัสดี  "
"  สวัสดี  ".stripTrailing();  // "  สวัสดี"
```

### แทนที่

```java
String msg = "แมว แมว หมา";

msg.replace("แมว", "นก");          // "นก นก หมา" — แทนทุกตัว (ไม่ใช้ regex)
msg.replace('ว', 'ง');             // แทน char ทุกตัว
msg.replaceFirst("แมว", "นก");     // "นก แมว หมา" — แทนตัวแรก (regex)
"a1b22c333".replaceAll("\\d+", "#"); // "a#b#c#" — ใช้ regex
```

> 💡 **เทียบกับ JavaScript:** `replace` ของ Java **แทนทุกตัว** (เหมือน `replaceAll` ของ JS) ส่วน `replaceAll` ของ Java รับ **regex เป็น String** (ต้อง escape `\\`)

### แยกข้อความ — `split`

```java
String csv = "สมชาย,สมหญิง,สมศรี";
String[] names = csv.split(",");
System.out.println(names.length);     // ผลลัพธ์: 3
System.out.println(names[1]);         // ผลลัพธ์: สมหญิง

"a  b   c".split("\\s+");             // ["a", "b", "c"] — split รับ regex
"1.2.3".split("\\.");                 // ["1", "2", "3"] — . ต้อง escape
"a,b,,".split(",");                   // ["a", "b"] — ตัด string ว่างท้ายทิ้ง
"a,b,,".split(",", -1);               // ["a", "b", "", ""] — เก็บไว้ทั้งหมด
```

### ทำซ้ำ / รวม

```java
"ha".repeat(3);                               // "hahaha" (Java 11+)
"-".repeat(20);                               // "--------------------"

String.join(", ", "ก", "ข", "ค");             // "ก, ข, ค"
String.join(" | ", java.util.List.of("A", "B")); // "A | B"
"สวัสดี".concat(" Java");                     // "สวัสดี Java"
```

### แปลงเป็นอย่างอื่น

```java
"hello".toCharArray();         // char[] {'h','e','l','l','o'}
"hello".chars().count();       // 5 — IntStream ของรหัสตัวอักษร
"hello".chars()
       .filter(ch -> ch == 'l')
       .count();               // 2

"line1\nline2".lines().toList(); // ["line1", "line2"] (Java 11+, toList Java 16+)

String.valueOf(123);           // "123"
String.valueOf(true);          // "true"
Integer.parseInt("42");        // 42
```

---

## 4. จัดรูปแบบข้อความ — `format` / `formatted`

```java
String name = "สมชาย";
int age = 25;
double salary = 35000.5;

String s1 = String.format("ชื่อ %s อายุ %d ปี", name, age);
String s2 = "เงินเดือน %,.2f บาท".formatted(salary);   // Java 15+

System.out.println(s1); // ผลลัพธ์: ชื่อ สมชาย อายุ 25 ปี
System.out.println(s2); // ผลลัพธ์: เงินเดือน 35,000.50 บาท
```

| Format    | ความหมาย                | ตัวอย่าง               | ผลลัพธ์        |
| --------- | ----------------------- | ---------------------- | -------------- |
| `%s`      | String (ทุก object)     | `%s`, "Java"           | `Java`         |
| `%d`      | จำนวนเต็ม               | `%d`, 42               | `42`           |
| `%f`      | ทศนิยม (6 ตำแหน่ง)      | `%f`, 3.14             | `3.140000`     |
| `%.2f`    | ทศนิยม 2 ตำแหน่ง        | `%.2f`, 3.14159        | `3.14`         |
| `%,d`     | คั่นหลักพัน             | `%,d`, 1000000         | `1,000,000`    |
| `%5d`     | ชิดขวา กว้าง 5          | `%5d`, 42              | `   42`        |
| `%-5s`    | ชิดซ้าย กว้าง 5         | `%-5s`, "ab"           | `ab   `        |
| `%05d`    | เติม 0 ข้างหน้า         | `%05d`, 42             | `00042`        |
| `%b`      | boolean                 | `%b`, true             | `true`         |
| `%n`      | ขึ้นบรรทัดใหม่          | -                      | -              |

> 💡 **เทียบกับ JavaScript:** Java **ไม่มี template literal** (`` `ชื่อ ${name}` ``) — ใช้ `+`, `String.format`, `.formatted()` หรือ text block + `.formatted()` แทน

---

## 5. `StringBuilder` — สร้าง String แบบแก้ไขได้

เนื่องจาก String เป็น immutable การต่อ string ใน loop จะสร้าง object ใหม่ทุกรอบ → ช้า ใช้ `StringBuilder` แทน

### ตัวอย่าง

```java
// ❌ ช้า — สร้าง String ใหม่ทุกรอบ
String result = "";
for (int i = 0; i < 5; i++) {
    result += i + ",";
}

// ✅ เร็ว — แก้ไขใน buffer เดียว
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 5; i++) {
    sb.append(i).append(",");
}
sb.setLength(sb.length() - 1);   // ลบ , ตัวสุดท้าย
System.out.println(sb.toString()); // ผลลัพธ์: 0,1,2,3,4
```

### Method ของ StringBuilder

```java
StringBuilder sb = new StringBuilder("Java");

sb.append(" 21");           // "Java 21"
sb.insert(0, "Hello ");     // "Hello Java 21"
sb.replace(0, 5, "Hi");     // "Hi Java 21"
sb.deleteCharAt(0);         // "i Java 21"
sb.delete(0, 2);            // "Java 21"
sb.reverse();               // "12 avaJ"
System.out.println(sb);     // ผลลัพธ์: 12 avaJ

// ตรวจ palindrome
String word = "level";
boolean isPalindrome = new StringBuilder(word).reverse().toString().equals(word);
System.out.println(isPalindrome); // ผลลัพธ์: true
```

> ต่อ string ธรรมดาด้วย `+` นอก loop ไม่ต้องกังวล — compiler ปรับให้มีประสิทธิภาพอยู่แล้ว

---

## 6. Text Blocks (Java 15+)

เขียนข้อความหลายบรรทัดด้วย `"""` — เหมาะกับ JSON, SQL, HTML

```java
String json = """
        {
            "name": "สมชาย",
            "age": 25
        }
        """;
System.out.print(json);
// ผลลัพธ์:
// {
//     "name": "สมชาย",
//     "age": 25
// }
```

### คุณสมบัติ

- `"""` เปิดต้องตามด้วย **ขึ้นบรรทัดใหม่** ทันที
- indentation ร่วมกันของทุกบรรทัด (รวมบรรทัด `"""` ปิด) จะถูกตัดออกอัตโนมัติ
- ไม่ต้อง escape `"` ภายใน
- ใช้ `\` ท้ายบรรทัดเพื่อ **ไม่ขึ้นบรรทัดใหม่**

```java
String sql = """
        SELECT id, name
        FROM users
        WHERE name = '%s'
        """.formatted("สมชาย");

String oneLine = """
        บรรทัดนี้ยาวมาก \
        แต่จะต่อกันเป็นบรรทัดเดียว""";
System.out.println(oneLine); // ผลลัพธ์: บรรทัดนี้ยาวมาก แต่จะต่อกันเป็นบรรทัดเดียว
```

> 💡 **เทียบกับ JavaScript:** คล้าย template literal แบบหลายบรรทัด (`` ` ` ``) แต่ **ไม่มี `${}`** — ใช้ `.formatted()` แทน

---

## 7. ตัวอักษรไทยและ Unicode

`char` ใน Java มีขนาด 16 bit (UTF-16) — ตัวอักษรไทยแต่ละตัว (รวมสระและวรรณยุกต์) นับเป็น 1 `char`

```java
String thai = "น้ำ";
System.out.println(thai.length());   // ผลลัพธ์: 3 (น + ้ + ำ)

String emoji = "😀";
System.out.println(emoji.length());                         // ผลลัพธ์: 2 (surrogate pair)
System.out.println(emoji.codePointCount(0, emoji.length())); // ผลลัพธ์: 1
```

---

## ตารางเปรียบเทียบกับ JavaScript

| JavaScript                       | Java                                      |
| -------------------------------- | ----------------------------------------- |
| `s.length`                       | `s.length()`                              |
| `s[0]` / `s.charAt(0)`           | `s.charAt(0)` (คืน `char`)                |
| `s.at(-1)`                       | `s.charAt(s.length() - 1)`                |
| `s === t`                        | `s.equals(t)`                             |
| `s.includes(x)`                  | `s.contains(x)`                           |
| `s.indexOf(x)`                   | `s.indexOf(x)`                            |
| `s.slice(1, 3)` / `s.substring`  | `s.substring(1, 3)`                       |
| `s.slice(-3)`                    | `s.substring(s.length() - 3)`             |
| `s.toUpperCase()`                | `s.toUpperCase()`                         |
| `s.trim()`                       | `s.strip()` (หรือ `trim()`)               |
| `s.replace(a, b)` (ตัวแรก)       | `s.replaceFirst(a, b)` (regex)            |
| `s.replaceAll(a, b)`             | `s.replace(a, b)`                         |
| `s.split(",")`                   | `s.split(",")` (regex, คืน `String[]`)    |
| `s.repeat(3)`                    | `s.repeat(3)`                             |
| `s.padStart(5, "0")`             | `String.format("%5s", s).replace(' ', '0')` / `"%05d"` สำหรับตัวเลข |
| `arr.join(", ")`                 | `String.join(", ", list)`                 |
| `` `Hi ${name}` ``               | `"Hi %s".formatted(name)`                 |
| `s.length === 0`                 | `s.isEmpty()`                             |
| `s.trim() === ""`                | `s.isBlank()`                             |
| `[...s]`                         | `s.toCharArray()` / `s.chars()`           |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **เทียบด้วย `.equals()`** เสมอ ห้ามใช้ `==` กับ String
2. **ใช้ `StringBuilder`** เมื่อต่อ string ใน loop
3. **ใช้ `strip()` และ `isBlank()`** (Java 11+) แทน `trim()` และ `trim().isEmpty()`
4. **ระวัง `split` / `replaceAll` / `matches`** รับ regex — ต้อง escape อักขระพิเศษ (`.`, `|`, `$`)
5. **ใช้ text block** สำหรับข้อความหลายบรรทัด (Java 15+)
6. **เขียน literal ไว้หน้า** — `"yes".equalsIgnoreCase(answer)` ป้องกัน NPE

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        String raw = "  สมชาย , สมหญิง ,สมศรี  ";

        List<String> names = java.util.Arrays.stream(raw.split(","))
                .map(String::strip)
                .filter(s -> !s.isBlank())
                .toList();

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < names.size(); i++) {
            sb.append(i + 1).append(". ").append(names.get(i)).append("\n");
        }
        System.out.print(sb);
        // ผลลัพธ์:
        // 1. สมชาย
        // 2. สมหญิง
        // 3. สมศรี
    }
}
```

---
