# Java Fundamentals

# ส่วนที่ 4: เงื่อนไข (Conditionals)

## Conditional คืออะไร?

Conditional คือการควบคุมให้โปรแกรมทำงานต่างกันตามเงื่อนไข Java มีเครื่องมือหลักคือ `if/else`, `switch` (ทั้งแบบเดิมและแบบใหม่) และ ternary operator

> 💡 **เทียบกับ JavaScript:** เงื่อนไขใน Java **ต้องเป็น `boolean` เท่านั้น** ไม่มี truthy/falsy — `if (name)`, `if (count)`, `if (list.length)` ใช้ไม่ได้ ต้องเขียน `if (name != null)`, `if (count > 0)`, `if (!list.isEmpty())`

---

## 1. `if` / `else if` / `else`

### คุณสมบัติ

- เงื่อนไขในวงเล็บต้องเป็น `boolean`
- ถ้ามี statement เดียว ไม่ต้องใส่ `{}` ก็ได้ แต่ **แนะนำให้ใส่เสมอ**

### ตัวอย่าง

```java
int score = 75;
String grade;

if (score >= 80) {
    grade = "A";
} else if (score >= 70) {
    grade = "B";
} else if (score >= 60) {
    grade = "C";
} else {
    grade = "F";
}

System.out.println("เกรด: " + grade); // ผลลัพธ์: เกรด: B
```

### ไม่มี truthy/falsy

```java
String name = "";
int count = 0;

// if (name) { }      // Error: incompatible types: String cannot be converted to boolean
// if (count) { }     // Error: incompatible types: int cannot be converted to boolean

if (name != null && !name.isEmpty()) {
    System.out.println("มีชื่อ");
}
if (count > 0) {
    System.out.println("มีของ");
}
```

### กับดัก: `=` แทน `==`

```java
int x = 5;
// if (x = 10) { }   // Error: int cannot be converted to boolean — Java ช่วยจับให้

boolean flag = false;
if (flag = true) {   // ⚠️ compile ผ่าน! เพราะผลลัพธ์เป็น boolean
    System.out.println("เข้าเสมอ");
}
// ✅ เขียน if (flag) แทน if (flag == true)
```

### Guard clause (early return)

```java
static String checkAge(int age) {
    if (age < 0) {
        return "อายุไม่ถูกต้อง";
    }
    if (age < 18) {
        return "ยังไม่บรรลุนิติภาวะ";
    }
    return "ผู้ใหญ่";
}
```

---

## 2. Ternary Operator

ใช้แทน `if/else` สั้น ๆ ที่ **คืนค่า**

```java
int age = 16;
String type = age >= 18 ? "ผู้ใหญ่" : "เด็ก";
System.out.println(type); // ผลลัพธ์: เด็ก

int stock = 0;
System.out.println(stock > 0 ? "มีสินค้า" : "สินค้าหมด"); // ผลลัพธ์: สินค้าหมด
```

> ⚠️ หลีกเลี่ยง ternary ซ้อนหลายชั้น — ใช้ `if/else` หรือ `switch` expression แทน

---

## 3. `switch` แบบดั้งเดิม (Statement)

### คุณสมบัติ

- ใช้กับ `int`, `char`, `byte`, `short`, wrapper ของพวกนี้, `String` (Java 7+), และ `enum`
- **Fall-through** — ถ้าไม่ใส่ `break` จะไหลไปทำ case ถัดไป
- `default` คือกรณีที่ไม่ตรงกับ case ใดเลย

### ตัวอย่าง

```java
int day = 3;
String dayName;

switch (day) {
    case 1:
        dayName = "จันทร์";
        break;
    case 2:
        dayName = "อังคาร";
        break;
    case 3:
        dayName = "พุธ";
        break;
    default:
        dayName = "ไม่ทราบ";
        break;
}

System.out.println(dayName); // ผลลัพธ์: พุธ
```

### Fall-through (ลืม `break`)

```java
int level = 1;
switch (level) {
    case 1:
        System.out.println("ระดับ 1");
    case 2:
        System.out.println("ระดับ 2");
    case 3:
        System.out.println("ระดับ 3");
        break;
    default:
        System.out.println("อื่น ๆ");
}
// ผลลัพธ์:
// ระดับ 1
// ระดับ 2
// ระดับ 3
```

### ใช้ fall-through อย่างตั้งใจ — รวมหลาย case

```java
String day = "เสาร์";
switch (day) {
    case "เสาร์":
    case "อาทิตย์":
        System.out.println("วันหยุด");
        break;
    default:
        System.out.println("วันทำงาน");
}
// ผลลัพธ์: วันหยุด
```

> 💡 **เทียบกับ JavaScript:** `switch` แบบดั้งเดิมทำงานเหมือน JS เกือบทุกอย่าง (รวมถึง fall-through) ต่างกันที่ Java เทียบ `String` ด้วย `.equals()` ให้อัตโนมัติ และ case ต้องเป็น **ค่าคงที่** (ใส่ expression อย่าง `case x > 5:` ไม่ได้)

---

## 4. `switch` Expression (Java 14+)

`switch` แบบใหม่ใช้ลูกศร `->` **ไม่มี fall-through** และ **คืนค่าได้**

### คุณสมบัติ

- ใช้ `->` แทน `:` — ไม่ต้องใส่ `break`
- รวมหลายค่าด้วย `,` เช่น `case 6, 7 ->`
- เมื่อใช้เป็น expression (คืนค่า) ต้อง **ครอบคลุมทุกกรณี (exhaustive)** — ส่วนใหญ่ต้องมี `default`
- ถ้าต้องการหลายบรรทัด ใช้ block `{ }` และ **`yield`** เพื่อคืนค่า

### ตัวอย่าง

```java
int day = 6;

String type = switch (day) {
    case 1, 2, 3, 4, 5 -> "วันทำงาน";
    case 6, 7 -> "วันหยุด";
    default -> "ไม่ถูกต้อง";
};

System.out.println(type); // ผลลัพธ์: วันหยุด
```

### `yield` — คืนค่าจาก block

```java
String grade = "B";

int bonus = switch (grade) {
    case "A" -> 1000;
    case "B" -> {
        System.out.println("คำนวณโบนัสเกรด B");
        int base = 500;
        yield base + 100;   // yield คืนค่าออกจาก switch
    }
    default -> 0;
};

System.out.println(bonus); // ผลลัพธ์: 600
```

### ใช้เป็น statement (ไม่คืนค่า)

```java
String command = "start";
switch (command) {
    case "start" -> System.out.println("เริ่มทำงาน");
    case "stop" -> System.out.println("หยุดทำงาน");
    default -> System.out.println("ไม่รู้จักคำสั่ง");
}
// ผลลัพธ์: เริ่มทำงาน
```

### กับ `enum` — ไม่ต้องมี `default` ถ้าครบทุกค่า

```java
enum Size { SMALL, MEDIUM, LARGE }

Size size = Size.MEDIUM;
int price = switch (size) {   // case ใช้ชื่อค่าเลย ไม่ต้องใส่ Size.
    case SMALL -> 45;
    case MEDIUM -> 55;
    case LARGE -> 65;
};                            // ครบทุกค่าแล้ว compiler ไม่บังคับ default
System.out.println(price);    // ผลลัพธ์: 55
```

---

## 5. Pattern Matching for `switch` (Java 21)

ตั้งแต่ Java 21 `switch` สามารถ **ตรวจ type** และ **แยกข้อมูลออกจาก record** ได้ในตัว

### Type patterns

```java
static String describe(Object obj) {
    return switch (obj) {
        case null -> "ค่าเป็น null";                    // จัดการ null ได้ (Java 21)
        case Integer i -> "จำนวนเต็ม: " + i;
        case String s -> "ข้อความยาว " + s.length() + " ตัวอักษร";
        case Double d -> "ทศนิยม: " + d;
        default -> "ไม่รู้จัก type: " + obj.getClass().getSimpleName();
    };
}

// describe(42)        → "จำนวนเต็ม: 42"
// describe("สมชาย")   → "ข้อความยาว 5 ตัวอักษร"
// describe(null)      → "ค่าเป็น null"
```

### Guard ด้วย `when`

```java
static String classify(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "จำนวนลบ";
        case Integer i when i == 0 -> "ศูนย์";
        case Integer i -> "จำนวนบวก";
        case String s when s.isBlank() -> "ข้อความว่าง";
        case String s -> "ข้อความ: " + s;
        default -> "อื่น ๆ";
    };
}

// classify(-5)  → "จำนวนลบ"
// classify("")  → "ข้อความว่าง"
```

> ⚠️ case ที่เจาะจงกว่า (มี `when`) ต้องอยู่ **ก่อน** case ทั่วไป ไม่เช่นนั้น compile error (dominated pattern)

### Record patterns + sealed interface

ใช้คู่กับ `sealed` (Java 17) และ `record` (Java 16) — compiler รู้ว่ามี type ครบแค่ไหน จึงไม่ต้องมี `default`

```java
public class Main {
    sealed interface Shape permits Circle, Rectangle, Triangle {}
    record Circle(double radius) implements Shape {}
    record Rectangle(double width, double height) implements Shape {}
    record Triangle(double base, double height) implements Shape {}

    static double area(Shape shape) {
        return switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Rectangle(double w, double h) -> w * h;          // แยก field ออกมาเลย
            case Triangle(var b, var h) -> 0.5 * b * h;           // ใช้ var ได้
        };                                                        // ไม่ต้องมี default — ครบทุก permits
    }

    public static void main(String[] args) {
        System.out.println(area(new Rectangle(4, 5)));  // ผลลัพธ์: 20.0
        System.out.println(area(new Triangle(6, 2)));   // ผลลัพธ์: 6.0
        System.out.printf("%.2f%n", area(new Circle(1))); // ผลลัพธ์: 3.14
    }
}
```

> 💡 **เทียบกับ JavaScript:** JS ยังไม่มี pattern matching (มีแค่ proposal) — ใกล้เคียงสุดคือ discriminated union ของ TypeScript (`switch (shape.kind)`) แต่ Java ตรวจความครบถ้วนและแยก field ให้ตั้งแต่ compile time

ดูรายละเอียด `record` และ `sealed` เพิ่มเติมใน [Record & Enum](../oop/record-enum.md) และ [Abstract Class & Interface](../oop/abstract-interface.md)

---

## ตารางเปรียบเทียบ

| คุณสมบัติ                | `if/else`        | `switch` แบบเดิม  | `switch` expression (14+) | Pattern `switch` (21) |
| ------------------------ | ---------------- | ----------------- | ------------------------- | --------------------- |
| เงื่อนไขซับซ้อน (`>`, `&&`) | ได้           | ไม่ได้            | ไม่ได้                    | ได้ด้วย `when`        |
| คืนค่าได้                | ไม่ได้           | ไม่ได้            | ได้                       | ได้                   |
| Fall-through             | -                | มี (ต้อง `break`) | ไม่มี                     | ไม่มี                 |
| ตรวจ type                | `instanceof`     | ไม่ได้            | ไม่ได้                    | ได้                   |
| จัดการ `null`            | ได้              | NPE               | NPE                       | `case null`           |
| บังคับครบทุกกรณี         | ไม่              | ไม่               | ใช่ (เมื่อคืนค่า)         | ใช่                   |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใส่ `{}` เสมอ** แม้ `if` จะมี statement เดียว
2. **ใช้ `switch` expression แบบ `->`** (Java 14+) แทนแบบเดิม — ไม่มีปัญหาลืม `break`
3. **ใช้ guard clause / early return** ลดการซ้อน `if` หลายชั้น
4. **ใช้ pattern matching `switch`** (Java 21) กับ `sealed` + `record` แทน `if-instanceof` ยาว ๆ
5. **หลีกเลี่ยง `default`** เมื่อ switch กับ `enum`/`sealed` ที่ครบทุกกรณีแล้ว — เพื่อให้ compiler เตือนเมื่อมีค่าใหม่เพิ่มเข้ามา
6. **อย่าเขียน `if (flag == true)`** — เขียน `if (flag)` ก็พอ

```java
public class Main {
    enum OrderStatus { PENDING, PAID, SHIPPED, CANCELLED }

    static String message(OrderStatus status) {
        return switch (status) {
            case PENDING -> "รอชำระเงิน";
            case PAID -> "ชำระเงินแล้ว";
            case SHIPPED -> "จัดส่งแล้ว";
            case CANCELLED -> "ยกเลิก";
        };
    }

    public static void main(String[] args) {
        System.out.println(message(OrderStatus.PAID)); // ผลลัพธ์: ชำระเงินแล้ว
    }
}
```

---
