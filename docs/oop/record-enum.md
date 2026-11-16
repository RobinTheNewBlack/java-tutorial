# Object-Oriented Programming

# ส่วนที่ 4: Record และ Enum

## ภาพรวม

- **Record** (Java 16+) — class สำหรับเก็บข้อมูลแบบ **immutable** เขียนสั้นมาก compiler สร้าง constructor, getter, `equals`, `hashCode`, `toString` ให้เอง
- **Enum** (Java 5+) — type ที่มีค่าได้เฉพาะชุดที่กำหนดไว้ เช่น สถานะคำสั่งซื้อ, วันในสัปดาห์

ทั้งสองถูกใช้หนักมากใน **Spring Boot** — record สำหรับ DTO (request/response), enum สำหรับสถานะต่าง ๆ ใน entity

---

## 1. Record พื้นฐาน (Java 16+)

### ปัญหาเดิม — class เก็บข้อมูลต้องเขียนยาว

```java
// ก่อนมี record: ~40 บรรทัดสำหรับข้อมูล 2 field
public final class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) { this.name = name; this.age = age; }
    public String getName() { return name; }
    public int getAge() { return age; }
    @Override public boolean equals(Object o) { /* ... */ return false; }
    @Override public int hashCode() { /* ... */ return 0; }
    @Override public String toString() { /* ... */ return ""; }
}
```

### ใช้ Record — บรรทัดเดียว

```java
public record Person(String name, int age) {}
```

### สิ่งที่ compiler สร้างให้

| สิ่งที่ได้                         | ตัวอย่าง                                  |
| -------------------------------- | ---------------------------------------- |
| `private final` field ทุกตัว      | `name`, `age`                            |
| Canonical constructor            | `new Person("สมชาย", 25)`                |
| Accessor (ไม่มี `get` นำหน้า)       | `p.name()`, `p.age()`                    |
| `equals()` / `hashCode()`        | เทียบตามค่าทุก field                       |
| `toString()`                     | `Person[name=สมชาย, age=25]`             |

### ตัวอย่าง

```java
public class Main {
    record Person(String name, int age) {}

    public static void main(String[] args) {
        Person p1 = new Person("สมชาย", 25);
        Person p2 = new Person("สมชาย", 25);

        System.out.println(p1.name());     // ผลลัพธ์: สมชาย
        System.out.println(p1);            // ผลลัพธ์: Person[name=สมชาย, age=25]
        System.out.println(p1.equals(p2)); // ผลลัพธ์: true  (เทียบค่า)
        System.out.println(p1 == p2);      // ผลลัพธ์: false (คนละ object)
    }
}
```

> 💡 **เทียบกับ JavaScript:** ใกล้เคียงกับ object literal `{ name: "สมชาย", age: 25 }` ที่ถูก `Object.freeze()` แต่มี type ชัดเจน และเทียบค่าด้วย `equals` ได้ (JS ต้องเทียบทีละ field หรือใช้ library)

---

## 2. Immutability — Record เปลี่ยนค่าไม่ได้

Record ไม่มี setter และ field เป็น `final` ถ้าต้องการค่าใหม่ ให้สร้าง object ใหม่ (แบบ "wither")

```java
record Person(String name, int age) {
    Person withAge(int newAge) {
        return new Person(name, newAge); // คืน object ใหม่
    }
}

Person p = new Person("สมหญิง", 30);
// p.age = 31; // ❌ Compile error
Person older = p.withAge(31);
System.out.println(p);     // ผลลัพธ์: Person[name=สมหญิง, age=30]
System.out.println(older); // ผลลัพธ์: Person[name=สมหญิง, age=31]
```

> 💡 **เทียบกับ JavaScript:** เหมือนแนวคิด `{ ...p, age: 31 }` ใน JS/React ที่ไม่แก้ object เดิม

> **ข้อควรระวัง:** Record เป็น **shallow immutable** — ถ้า field เป็น `List` ที่แก้ได้ ข้อมูลข้างในยังถูกแก้ได้ ควร copy ด้วย `List.copyOf()` ใน constructor

---

## 3. Compact Constructor — ตรวจสอบข้อมูล

Compact constructor คือ constructor ที่ **ไม่ต้องเขียน parameter และไม่ต้อง assign field** ใช้สำหรับ validate หรือ normalize ค่า

```java
import java.util.List;

public class Main {
    record Product(String name, double price, List<String> tags) {
        Product { // compact constructor — ไม่มี (...)
            if (name == null || name.isBlank()) {
                throw new IllegalArgumentException("ชื่อสินค้าห้ามว่าง");
            }
            if (price < 0) {
                throw new IllegalArgumentException("ราคาต้องไม่ติดลบ");
            }
            name = name.trim();       // normalize ค่าก่อน assign อัตโนมัติ
            tags = List.copyOf(tags); // defensive copy → immutable จริง
        }

        // overload constructor ได้ แต่ต้องเรียก canonical constructor
        Product(String name, double price) {
            this(name, price, List.of());
        }
    }

    public static void main(String[] args) {
        Product p = new Product("  ทุเรียน  ", 250);
        System.out.println(p); // ผลลัพธ์: Product[name=ทุเรียน, price=250.0, tags=[]]

        try {
            new Product("มังคุด", -10);
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage()); // ผลลัพธ์: ราคาต้องไม่ติดลบ
        }
    }
}
```

### ข้อจำกัดของ Record

- **extends class อื่นไม่ได้** (สืบทอดจาก `java.lang.Record` อยู่แล้ว) แต่ **implement interface ได้**
- เป็น `final` — ไม่มีใครสืบทอดได้
- เพิ่ม instance field นอกเหนือจาก component ไม่ได้ (เพิ่ม `static` field และ method ได้)

---

## 4. Record เป็น DTO ใน Spring Boot

DTO (Data Transfer Object) คือ object สำหรับรับ/ส่งข้อมูลผ่าน API — record เหมาะมากเพราะไม่ต้องเปลี่ยนค่าและเขียนสั้น

```java
// ตัวอย่างแนวคิด (ต้องมี Spring Boot + Jakarta Validation)
public record CreateUserRequest(
        @NotBlank String name,
        @Email String email,
        @Min(18) int age
) {}

public record UserResponse(Long id, String name, String email) {}

@RestController
@RequestMapping("/api/users")
class UserController {
    @PostMapping
    UserResponse create(@Valid @RequestBody CreateUserRequest request) {
        // Jackson แปลง JSON → record และ record → JSON ให้อัตโนมัติ
        return new UserResponse(1L, request.name(), request.email());
    }
}
```

> **หมายเหตุ:** **ไม่ควรใช้ record เป็น JPA `@Entity`** เพราะ JPA ต้องการ no-arg constructor และ field ที่แก้ได้ — ใช้ record เป็น DTO แล้วใช้ class ปกติเป็น Entity

---

## 5. Enum พื้นฐาน

```java
enum OrderStatus {
    PENDING, PAID, SHIPPED, DELIVERED, CANCELLED
}

public class Main {
    public static void main(String[] args) {
        OrderStatus status = OrderStatus.PAID;

        System.out.println(status);           // ผลลัพธ์: PAID
        System.out.println(status.name());    // ผลลัพธ์: PAID
        System.out.println(status.ordinal()); // ผลลัพธ์: 1 (ลำดับ เริ่มที่ 0)
        System.out.println(status == OrderStatus.PAID); // ผลลัพธ์: true — เทียบด้วย == ได้เลย

        // values() — ค่าทั้งหมด
        for (OrderStatus s : OrderStatus.values()) {
            System.out.print(s + " ");
        }
        // ผลลัพธ์: PENDING PAID SHIPPED DELIVERED CANCELLED

        // valueOf() — แปลง String → enum (ต้องตรงตัวพิมพ์)
        OrderStatus parsed = OrderStatus.valueOf("SHIPPED");
        System.out.println(parsed); // ผลลัพธ์: SHIPPED
        // OrderStatus.valueOf("shipped"); // ❌ IllegalArgumentException
    }
}
```

> 💡 **เทียบกับ JavaScript:** JS ไม่มี enum (ใช้ `Object.freeze({ PAID: "PAID" })` หรือ string union ใน TypeScript) — enum ของ Java เป็น **class เต็มรูปแบบ** มี field, constructor, method ได้ และ compiler ตรวจว่าค่าถูกต้องเสมอ

---

## 6. Enum ที่มี Field, Constructor และ Method

```java
enum Planet {
    MERCURY("ดาวพุธ", 3.303e+23),
    EARTH("โลก", 5.976e+24),
    JUPITER("ดาวพฤหัสบดี", 1.9e+27); // ← ต้องมี ; เมื่อมีโค้ดต่อ

    private final String thaiName;
    private final double mass;

    Planet(String thaiName, double mass) { // constructor เป็น private เสมอ
        this.thaiName = thaiName;
        this.mass = mass;
    }

    public String getThaiName() { return thaiName; }

    public boolean isHeavierThan(Planet other) {
        return this.mass > other.mass;
    }
}

System.out.println(Planet.EARTH.getThaiName());                   // ผลลัพธ์: โลก
System.out.println(Planet.JUPITER.isHeavierThan(Planet.EARTH));   // ผลลัพธ์: true
```

### ตัวอย่างใช้งานจริง: สถานะพร้อม label ภาษาไทย

```java
import java.util.Arrays;

enum OrderStatus {
    PENDING("รอชำระเงิน"),
    PAID("ชำระแล้ว"),
    SHIPPED("กำลังจัดส่ง"),
    DELIVERED("ส่งสำเร็จ"),
    CANCELLED("ยกเลิก");

    private final String label;

    OrderStatus(String label) { this.label = label; }

    public String getLabel() { return label; }

    public boolean canCancel() {
        return this == PENDING || this == PAID;
    }

    // ค้นหาจาก label — ปลอดภัยกว่า valueOf
    public static OrderStatus fromLabel(String label) {
        return Arrays.stream(values())
                .filter(s -> s.label.equals(label))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("ไม่พบสถานะ: " + label));
    }
}

System.out.println(OrderStatus.SHIPPED.getLabel());   // ผลลัพธ์: กำลังจัดส่ง
System.out.println(OrderStatus.SHIPPED.canCancel());  // ผลลัพธ์: false
System.out.println(OrderStatus.fromLabel("ชำระแล้ว")); // ผลลัพธ์: PAID
```

---

## 7. Switch กับ Enum

### Switch Expression (Java 14+)

ใน `case` ไม่ต้องเขียนชื่อ enum นำหน้า

```java
static String nextStep(OrderStatus status) {
    return switch (status) {
        case PENDING -> "รอลูกค้าชำระเงิน";
        case PAID -> "เตรียมแพ็กสินค้า";
        case SHIPPED -> "ติดตามพัสดุ";
        case DELIVERED, CANCELLED -> "จบรายการ";
    }; // ครบทุกค่าแล้ว ไม่ต้องมี default
}

System.out.println(nextStep(OrderStatus.PAID)); // ผลลัพธ์: เตรียมแพ็กสินค้า
```

> **ข้อดี:** ถ้าเพิ่มค่าใหม่ใน enum (เช่น `REFUNDED`) แต่ลืมเพิ่มใน switch expression → **compile error** ช่วยกันลืมได้

---

## 8. Enum ที่มี Abstract Method

แต่ละค่าของ enum มี implementation เป็นของตัวเองได้ — เหมาะกับ Strategy Pattern แบบง่าย

```java
enum Operation {
    PLUS("+") {
        @Override
        double apply(double a, double b) { return a + b; }
    },
    MINUS("-") {
        @Override
        double apply(double a, double b) { return a - b; }
    },
    TIMES("*") {
        @Override
        double apply(double a, double b) { return a * b; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    abstract double apply(double a, double b); // ทุกค่าต้อง implement

    @Override
    public String toString() { return symbol; }
}

for (Operation op : Operation.values()) {
    System.out.println("6 " + op + " 3 = " + op.apply(6, 3));
}
// ผลลัพธ์:
// 6 + 3 = 9.0
// 6 - 3 = 3.0
// 6 * 3 = 18.0
```

---

## 9. EnumMap และ EnumSet

Collection ที่ออกแบบมาเฉพาะ enum — **เร็วและประหยัด memory** กว่า `HashMap`/`HashSet` มาก (ภายในใช้ array/bit vector)

```java
import java.util.*;

enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

public class Main {
    public static void main(String[] args) {
        // EnumSet — set ของค่า enum
        EnumSet<Day> weekend = EnumSet.of(Day.SAT, Day.SUN);
        EnumSet<Day> weekdays = EnumSet.complementOf(weekend);
        EnumSet<Day> midWeek = EnumSet.range(Day.TUE, Day.THU);

        System.out.println(weekdays); // ผลลัพธ์: [MON, TUE, WED, THU, FRI]
        System.out.println(midWeek);  // ผลลัพธ์: [TUE, WED, THU]
        System.out.println(weekend.contains(Day.SUN)); // ผลลัพธ์: true

        // EnumMap — map ที่ key เป็น enum (เรียงตามลำดับ enum เสมอ)
        EnumMap<Day, String> schedule = new EnumMap<>(Day.class);
        schedule.put(Day.WED, "เรียน Java");
        schedule.put(Day.MON, "ประชุมทีม");
        System.out.println(schedule); // ผลลัพธ์: {MON=ประชุมทีม, WED=เรียน Java}
    }
}
```

---

## 10. Record vs Class vs Lombok `@Data`

| หัวข้อ                   | `record`                  | Class ปกติ               | Lombok `@Data`                 |
| ----------------------- | ------------------------- | ------------------------ | ------------------------------ |
| ต้องใช้ Library            | ไม่ (Java 16+)             | ไม่                       | ต้องเพิ่ม Lombok                 |
| ความยาวโค้ด               | สั้นมาก                     | ยาว                       | สั้น                             |
| Immutable               | ✅ เสมอ                    | แล้วแต่เขียน               | ❌ (มี setter) — ใช้ `@Value` ถ้าต้องการ immutable |
| Getter                  | `name()`                  | `getName()`              | `getName()`                    |
| Setter                  | ไม่มี                       | เขียนเอง                   | มีให้                            |
| `equals/hashCode/toString` | สร้างให้                  | เขียนเอง                   | สร้างให้                          |
| สืบทอด class อื่น          | ไม่ได้                      | ได้                        | ได้                             |
| ใช้เป็น JPA Entity         | ❌ ไม่เหมาะ                 | ✅                        | ⚠️ ได้แต่ระวัง `equals/hashCode` |
| เหมาะกับ                 | DTO, value object, ผลลัพธ์ query | Entity, object ที่มีพฤติกรรม | Entity/DTO ในโปรเจกต์ที่ใช้ Lombok อยู่แล้ว |

```java
// Lombok (ตัวอย่างแนวคิด — ต้องเพิ่ม dependency)
@Data                 // getter + setter + equals + hashCode + toString
@NoArgsConstructor
@AllArgsConstructor
public class UserEntity {
    private Long id;
    private String name;
}
```

> **Tip:** ในโปรเจกต์ Spring Boot สมัยใหม่ นิยมใช้ **record สำหรับ DTO** และ **class (อาจใช้ Lombok `@Getter`/`@Setter`) สำหรับ JPA Entity**

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ record สำหรับข้อมูลที่ไม่เปลี่ยนค่า** — DTO, ผลลัพธ์จาก method ที่ต้องคืนหลายค่า, key ใน `Map`
2. **Validate ใน compact constructor** — object ที่สร้างได้ต้องถูกต้องเสมอ
3. **Defensive copy** สำหรับ field ที่เป็น collection (`List.copyOf`)
4. **ใช้ enum แทน String/int constant** — `OrderStatus.PAID` ปลอดภัยกว่า `"paid"` หรือ `1`
5. **เทียบ enum ด้วย `==`** ได้เลย (ปลอดภัยกับ `null` และเร็วกว่า `equals`)
6. **ไม่ควรพึ่ง `ordinal()`** ในการเก็บลง database — ถ้าสลับลำดับ enum ข้อมูลจะผิด (ใน JPA ใช้ `@Enumerated(EnumType.STRING)`)
7. **ใช้ `EnumMap`/`EnumSet`** เมื่อ key/สมาชิกเป็น enum

```java
// แนวทางที่แนะนำ
record Money(long satang, String currency) {
    Money {
        if (satang < 0) throw new IllegalArgumentException("จำนวนเงินติดลบไม่ได้");
        if (currency == null) currency = "THB";
    }

    Money plus(Money other) {
        return new Money(satang + other.satang, currency);
    }
}

System.out.println(new Money(5000, null).plus(new Money(2500, "THB")));
// ผลลัพธ์: Money[satang=7500, currency=THB]
```

---
