# Optional

# ส่วนที่ 6: Optional (Java 8+)

## Optional คืออะไร?

`Optional<T>` คือ **กล่องที่อาจมีค่าหรือไม่มีค่าก็ได้** ใช้เป็น **return type** ของ method เพื่อบอกผู้เรียกอย่างชัดเจนว่า "ผลลัพธ์อาจไม่มี" แทนการคืน `null`

```
Optional<String>
  ┌─────────────┐        ┌─────────────┐
  │  "สมชาย"    │   หรือ  │   (ว่าง)    │
  └─────────────┘        └─────────────┘
  Optional.of("สมชาย")    Optional.empty()
```

---

## ทำไมต้องมี Optional?

### ปัญหาของ `null`

```java
static String findEmail(String username) {
    if (username.equals("somchai")) {
        return "somchai@example.com";
    }
    return null;   // ผู้เรียกอาจลืมเช็ก
}

String email = findEmail("unknown");
System.out.println(email.toUpperCase()); // 💥 NullPointerException
```

- signature `String findEmail(...)` **ไม่บอก** ว่าอาจคืน `null`
- ผู้เรียกต้องจำเองว่าต้องเช็ก — ลืมเมื่อไรก็ NPE

### ใช้ Optional แทน

```java
import java.util.Optional;

static Optional<String> findEmail(String username) {
    if (username.equals("somchai")) {
        return Optional.of("somchai@example.com");
    }
    return Optional.empty();
}

String email = findEmail("unknown")
        .map(String::toUpperCase)
        .orElse("ไม่พบอีเมล");
System.out.println(email); // ผลลัพธ์: ไม่พบอีเมล
```

> 💡 **เทียบกับ JavaScript:** JS จัดการค่าที่อาจไม่มีด้วย optional chaining `?.` และ nullish coalescing `??` เช่น `user?.email?.toUpperCase() ?? "ไม่พบอีเมล"` — Java ไม่มี operator เหล่านี้ `Optional` คือเครื่องมือที่ทำหน้าที่ใกล้เคียงที่สุด: `.map()` ≈ `?.` และ `.orElse()` ≈ `??`

---

## 1. การสร้าง Optional

| Method                     | ใช้เมื่อ                                  | ถ้าค่าเป็น `null`             |
| -------------------------- | ----------------------------------------- | ----------------------------- |
| `Optional.of(value)`       | มั่นใจว่าค่า **ไม่เป็น null**             | throw `NullPointerException`  |
| `Optional.ofNullable(value)` | ค่า **อาจเป็น null**                     | ได้ `Optional.empty()`        |
| `Optional.empty()`         | ต้องการ Optional ว่าง                     | -                             |

### ตัวอย่าง

```java
import java.util.Optional;

Optional<String> name = Optional.of("สมชาย");
Optional<String> empty = Optional.empty();

String maybeNull = null;
Optional<String> safe = Optional.ofNullable(maybeNull);   // Optional.empty
// Optional<String> boom = Optional.of(maybeNull);        // NullPointerException

System.out.println(name);   // ผลลัพธ์: Optional[สมชาย]
System.out.println(safe);   // ผลลัพธ์: Optional.empty
```

### Optional สำหรับ primitive

```java
import java.util.OptionalInt;
import java.util.OptionalDouble;
import java.util.stream.IntStream;

OptionalInt max = IntStream.of(3, 8, 1).max();
System.out.println(max.getAsInt());             // ผลลัพธ์: 8

OptionalDouble avg = IntStream.empty().average();
System.out.println(avg.orElse(0.0));             // ผลลัพธ์: 0.0
```

---

## 2. ตรวจสอบค่า

```java
Optional<String> name = Optional.of("สมชาย");
Optional<String> empty = Optional.empty();

name.isPresent();    // true
empty.isPresent();   // false
empty.isEmpty();     // true (Java 11+)
```

### `ifPresent` / `ifPresentOrElse`

```java
name.ifPresent(n -> System.out.println("สวัสดี " + n));
// ผลลัพธ์: สวัสดี สมชาย

empty.ifPresentOrElse(                                  // Java 9+
        n -> System.out.println("สวัสดี " + n),
        () -> System.out.println("ไม่พบผู้ใช้")
);
// ผลลัพธ์: ไม่พบผู้ใช้
```

---

## 3. ดึงค่าออกมา

### `orElse` — ค่า default

```java
String a = Optional.of("สมชาย").orElse("ไม่ระบุ");  // "สมชาย"
String b = Optional.<String>empty().orElse("ไม่ระบุ"); // "ไม่ระบุ"
```

### `orElseGet` — ค่า default ที่คำนวณเมื่อจำเป็น (lazy)

```java
static String loadDefault() {
    System.out.println("กำลังโหลดค่า default...");
    return "Guest";
}

Optional<String> user = Optional.of("สมชาย");

user.orElse(loadDefault());          // ⚠️ loadDefault() ถูกเรียกเสมอ แม้มีค่าอยู่แล้ว
user.orElseGet(() -> loadDefault()); // ✅ ถูกเรียกเฉพาะเมื่อว่าง
user.orElseGet(Main::loadDefault);   // ✅ method reference
```

> ใช้ `orElse` กับค่าคงที่ / ค่าที่มีอยู่แล้ว และใช้ `orElseGet` เมื่อค่า default ต้องคำนวณหรือมีต้นทุน (query DB, สร้าง object)

### `orElseThrow` — throw exception ถ้าว่าง

```java
Optional<String> empty = Optional.empty();

String v1 = Optional.of("สมชาย").orElseThrow();    // "สมชาย" (Java 10+)
// empty.orElseThrow();                             // NoSuchElementException

String v2 = empty.orElseThrow(
        () -> new IllegalArgumentException("ไม่พบผู้ใช้")
);                                                  // IllegalArgumentException: ไม่พบผู้ใช้
```

### `get()` — หลีกเลี่ยง

```java
Optional<String> empty = Optional.empty();
// empty.get();  // NoSuchElementException — ไม่ต่างอะไรกับ NPE
```

`get()` ทำงานเหมือน `orElseThrow()` แต่ชื่อไม่สื่อว่าอาจ throw — **ใช้ `orElseThrow()` แทนถ้าต้องการพฤติกรรมนี้**

---

## 4. แปลงและกรองค่า

### `map` — แปลงค่าภายใน (ถ้ามี)

```java
Optional<String> name = Optional.of("somchai");

Optional<String> upper = name.map(String::toUpperCase);   // Optional[SOMCHAI]
Optional<Integer> length = name.map(String::length);      // Optional[7]

Optional<String> empty = Optional.empty();
empty.map(String::toUpperCase);                           // Optional.empty — ไม่ error
```

### `filter` — เก็บค่าไว้เฉพาะเมื่อผ่านเงื่อนไข

```java
Optional<Integer> age = Optional.of(15);

age.filter(a -> a >= 18)
   .ifPresentOrElse(
       a -> System.out.println("เข้าได้"),
       () -> System.out.println("อายุไม่ถึง")
   );
// ผลลัพธ์: อายุไม่ถึง
```

### `flatMap` — เมื่อ function คืน Optional อยู่แล้ว

```java
import java.util.Optional;

public class Main {
    record Address(String city) {}
    record User(String name, Optional<Address> address) {}  // ตัวอย่างเพื่อสาธิต flatMap เท่านั้น

    public static void main(String[] args) {
        Optional<User> user = Optional.of(new User("สมชาย", Optional.of(new Address("เชียงใหม่"))));

        Optional<Optional<Address>> nested = user.map(User::address); // ซ้อนกัน ❌
        Optional<Address> flat = user.flatMap(User::address);          // แบน ✅

        String city = user.flatMap(User::address)
                .map(Address::city)
                .orElse("ไม่ทราบ");
        System.out.println(city); // ผลลัพธ์: เชียงใหม่
    }
}
```

### Chain แบบเดียวกับ `?.` ใน JS

```java
import java.util.Optional;

public class Main {
    record Address(String city) {}
    record Customer(String name, Address address) {}  // address อาจเป็น null

    static String cityOf(Customer c) {
        return Optional.ofNullable(c)
                .map(Customer::address)       // ≈ c?.address
                .map(Address::city)           // ≈ ?.city
                .orElse("ไม่ทราบ");           // ≈ ?? "ไม่ทราบ"
    }

    public static void main(String[] args) {
        System.out.println(cityOf(new Customer("สมชาย", new Address("ขอนแก่น")))); // ผลลัพธ์: ขอนแก่น
        System.out.println(cityOf(new Customer("สมหญิง", null)));                 // ผลลัพธ์: ไม่ทราบ
        System.out.println(cityOf(null));                                           // ผลลัพธ์: ไม่ทราบ
    }
}
```

> ถ้า `map` คืน `null` ผลลัพธ์จะกลายเป็น `Optional.empty()` อัตโนมัติ — นี่คือสิ่งที่ทำให้ chain ข้างบนปลอดภัย

### `or` และ `stream` (Java 9+)

```java
import java.util.List;
import java.util.Optional;

Optional<String> primary = Optional.empty();
Optional<String> backup = Optional.of("backup@example.com");

Optional<String> email = primary.or(() -> backup);   // ลองตัวสำรองถ้าตัวแรกว่าง
System.out.println(email.orElseThrow()); // ผลลัพธ์: backup@example.com

// stream() — รวม Optional หลายตัวโดยตัดตัวว่างทิ้ง
List<Optional<String>> list = List.of(Optional.of("ก"), Optional.empty(), Optional.of("ข"));
List<String> values = list.stream().flatMap(Optional::stream).toList();
System.out.println(values); // ผลลัพธ์: [ก, ข]
```

---

## 5. Optional กับ Stream

terminal operation หลายตัวของ Stream คืน `Optional`

```java
import java.util.List;

List<String> names = List.of("สมชาย", "สมหญิง", "อนันต์");

String found = names.stream()
        .filter(n -> n.startsWith("อ"))
        .findFirst()                       // Optional<String>
        .orElse("ไม่พบ");
System.out.println(found);                 // ผลลัพธ์: อนันต์

int longest = names.stream()
        .map(String::length)
        .max(Integer::compare)             // Optional<Integer>
        .orElse(0);
System.out.println(longest);               // ผลลัพธ์: 6
```

---

## 6. Anti-patterns — สิ่งที่ไม่ควรทำ

### ❌ ใช้ `isPresent()` + `get()` แบบ if/else ธรรมดา

```java
Optional<String> name = findName();

// ❌ ไม่ต่างจากการเช็ก null
if (name.isPresent()) {
    System.out.println(name.get());
} else {
    System.out.println("ไม่พบ");
}

// ✅
System.out.println(name.orElse("ไม่พบ"));
// หรือ
name.ifPresentOrElse(System.out::println, () -> System.out.println("ไม่พบ"));
```

### ❌ ใช้ Optional เป็น field ของ class

```java
// ❌ Optional ไม่ได้ออกแบบมาเป็น field (ไม่ Serializable, เปลือง memory)
class User {
    private Optional<String> nickname;
}

// ✅ ใช้ field ปกติ แล้วคืน Optional จาก getter
class User2 {
    private String nickname;   // อาจเป็น null

    public Optional<String> getNickname() {
        return Optional.ofNullable(nickname);
    }
}
```

### ❌ ใช้ Optional เป็น parameter ของ method

```java
// ❌ ผู้เรียกต้องห่อ Optional ทุกครั้ง และยังส่ง null มาได้อยู่ดี
void greet(Optional<String> name) { }

// ✅ ใช้ overloading หรือรับค่าที่อาจเป็น null แล้วจัดการภายใน
void greet(String name) { }
void greet() { greet("Guest"); }
```

### ❌ คืน `null` จาก method ที่ return type เป็น Optional

```java
Optional<String> find() {
    return null;              // ❌ ทำลายจุดประสงค์ของ Optional ทั้งหมด
}

Optional<String> find2() {
    return Optional.empty();  // ✅
}
```

### ❌ ห่อ Collection ด้วย Optional

```java
Optional<List<String>> findTags();  // ❌
List<String> findTags2();           // ✅ คืน List ว่าง (List.of()) แทน
```

### ❌ ใช้ `Optional.of()` กับค่าที่อาจเป็น null

```java
String input = null;
// Optional.of(input);          // ❌ NullPointerException
Optional.ofNullable(input);     // ✅
```

---

## ตารางสรุป Method

| Method                       | คืนค่า          | ความหมาย                                     |
| ---------------------------- | --------------- | -------------------------------------------- |
| `isPresent()` / `isEmpty()`  | `boolean`       | มีค่า / ว่าง                                 |
| `ifPresent(fn)`              | `void`          | ทำงานถ้ามีค่า                                |
| `ifPresentOrElse(fn, run)`   | `void`          | ทำงานถ้ามีค่า ไม่งั้นทำอีกอย่าง (9+)         |
| `orElse(v)`                  | `T`             | ค่าภายใน หรือ `v`                            |
| `orElseGet(supplier)`        | `T`             | ค่าภายใน หรือคำนวณ default (lazy)            |
| `orElseThrow()`              | `T`             | ค่าภายใน หรือ `NoSuchElementException` (10+) |
| `orElseThrow(supplier)`      | `T`             | ค่าภายใน หรือ throw exception ที่กำหนด       |
| `map(fn)`                    | `Optional<R>`   | แปลงค่าภายใน                                 |
| `flatMap(fn)`                | `Optional<R>`   | แปลงด้วย function ที่คืน Optional            |
| `filter(pred)`               | `Optional<T>`   | เก็บค่าถ้าผ่านเงื่อนไข                       |
| `or(supplier)`               | `Optional<T>`   | ใช้ Optional อื่นถ้าว่าง (9+)                |
| `stream()`                   | `Stream<T>`     | stream 0 หรือ 1 สมาชิก (9+)                  |

---

## ตารางเปรียบเทียบกับ JavaScript

| JavaScript                         | Java                                                    |
| ---------------------------------- | ------------------------------------------------------- |
| `value ?? "default"`               | `Optional.ofNullable(value).orElse("default")` / `Objects.requireNonNullElse(value, "default")` |
| `obj?.a?.b`                        | `Optional.ofNullable(obj).map(O::a).map(A::b)`          |
| `obj?.a?.b ?? "x"`                 | `...map(A::b).orElse("x")`                              |
| `if (value != null) fn(value)`     | `opt.ifPresent(fn)`                                     |
| `value ?? throwError()`            | `opt.orElseThrow(() -> new XxxException(...))`          |
| `arr.find(...)` → `undefined`      | `stream().filter(...).findFirst()` → `Optional.empty()` |
| `value || compute()`               | `opt.orElseGet(() -> compute())`                        |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ Optional เป็น return type เท่านั้น** — สำหรับ method ที่ "อาจไม่มีผลลัพธ์" เช่น `findById`
2. **ห้ามคืน `null` จาก method ที่คืน Optional** — ใช้ `Optional.empty()`
3. **หลีกเลี่ยง `get()`** — ใช้ `orElse`, `orElseGet`, `orElseThrow`, `ifPresent` แทน
4. **ใช้ `orElseGet`** เมื่อค่า default มีต้นทุนในการสร้าง
5. **ไม่ใช้ Optional เป็น field, parameter หรือห่อ Collection**
6. **ใช้ `map` / `flatMap` / `filter` ต่อกันเป็น chain** แทน `if (isPresent())` ซ้อนกัน
7. **กรณีง่าย ๆ** ใช้ `Objects.requireNonNullElse(value, defaultValue)` (Java 9+) ก็พอ ไม่ต้องสร้าง Optional

```java
import java.util.List;
import java.util.Optional;

public class Main {
    record Product(String id, String name, int price) {}

    static final List<Product> PRODUCTS = List.of(
            new Product("P1", "กาแฟเย็น", 55),
            new Product("P2", "ชาไทย", 45)
    );

    static Optional<Product> findById(String id) {
        return PRODUCTS.stream()
                .filter(p -> p.id().equals(id))
                .findFirst();
    }

    public static void main(String[] args) {
        String p1 = findById("P1")
                .map(p -> p.name() + " ราคา " + p.price() + " บาท")
                .orElse("ไม่พบสินค้า");

        String p9 = findById("P9")
                .map(Product::name)
                .orElse("ไม่พบสินค้า");

        System.out.println(p1); // ผลลัพธ์: กาแฟเย็น ราคา 55 บาท
        System.out.println(p9); // ผลลัพธ์: ไม่พบสินค้า
    }
}
```

---
