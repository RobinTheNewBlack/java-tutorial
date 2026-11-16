# Method

# ส่วนที่ 2: Lambda และ Functional Interface

## Lambda คืออะไร?

Lambda Expression (Java 8+) คือการเขียน "ฟังก์ชันไม่มีชื่อ" แบบสั้น ๆ เพื่อส่งเป็น argument หรือเก็บในตัวแปร — แนวคิดเดียวกับ **arrow function** ใน JavaScript

แต่ Java ไม่มี "function type" แบบ JS — lambda ทุกตัวต้องถูกแปลงเป็น object ของ **Functional Interface** (interface ที่มี abstract method เพียงตัวเดียว)

> 💡 **เทียบกับ JavaScript:** ใน JS function เป็น first-class value `const f = x => x * 2` ได้เลย ส่วน Java ต้องบอกว่า lambda นี้เป็น **type อะไร** เช่น `Function<Integer, Integer> f = x -> x * 2;` และเรียกใช้ผ่านชื่อ method ของ interface (`f.apply(5)`) ไม่ใช่ `f(5)`

---

## 1. Syntax ของ Lambda

```java
(parameters) -> expression
(parameters) -> { statements; }
```

### ตัวอย่าง

```java
import java.util.function.*;

public class Main {
    public static void main(String[] args) {
        // ไม่มี parameter
        Runnable sayHi = () -> System.out.println("สวัสดี!");
        sayHi.run(); // ผลลัพธ์: สวัสดี!

        // parameter เดียว — ไม่ต้องใส่วงเล็บ
        Function<Integer, Integer> doubleIt = n -> n * 2;
        System.out.println(doubleIt.apply(4)); // ผลลัพธ์: 8

        // หลาย parameter — ต้องใส่วงเล็บ
        BinaryOperator<Integer> add = (a, b) -> a + b;
        System.out.println(add.apply(3, 5)); // ผลลัพธ์: 8

        // หลายบรรทัด — ต้องมี { } และ return
        Function<String, String> greet = name -> {
            String msg = "สวัสดี, " + name;
            return msg + "!";
        };
        System.out.println(greet.apply("สมชาย")); // ผลลัพธ์: สวัสดี, สมชาย!

        // ระบุ type ของ parameter เองก็ได้ (หรือใช้ var ใน Java 11+)
        BinaryOperator<Integer> multiply = (Integer a, Integer b) -> a * b;
        System.out.println(multiply.apply(4, 5)); // ผลลัพธ์: 20
    }
}
```

| JavaScript                    | Java                           |
| ----------------------------- | ------------------------------ |
| `() => console.log("hi")`     | `() -> System.out.println("hi")` |
| `x => x * 2`                  | `x -> x * 2`                   |
| `(a, b) => a + b`             | `(a, b) -> a + b`              |
| `x => { return x * 2; }`      | `x -> { return x * 2; }`       |
| `f(5)`                        | `f.apply(5)` / `f.test(5)` / ... |

> **ข้อสังเกต:** Java ใช้ `->` (ขีดเดียว) ส่วน JS ใช้ `=>` (เท่ากับ)

---

## 2. Functional Interface

Functional Interface คือ interface ที่มี **abstract method เพียงตัวเดียว** (มี `default`/`static` method ได้ไม่จำกัด) — lambda จะถูกใช้เป็น implementation ของ method ตัวนั้น

### สร้างเอง ด้วย `@FunctionalInterface`

```java
@FunctionalInterface
interface Discount {
    double apply(double price); // abstract method ตัวเดียว
}

public class Main {
    public static void main(String[] args) {
        Discount memberDiscount = price -> price * 0.9;
        Discount vipDiscount = price -> price * 0.7;

        System.out.println(memberDiscount.apply(1000)); // ผลลัพธ์: 900.0
        System.out.println(vipDiscount.apply(1000));    // ผลลัพธ์: 700.0
    }
}
```

> **หมายเหตุ:** `@FunctionalInterface` ไม่บังคับ แต่ควรใส่ — ถ้ามีคนเผลอเพิ่ม abstract method ตัวที่สอง compiler จะแจ้ง error ทันที

---

## 3. Functional Interface สำเร็จรูป (`java.util.function`)

ส่วนใหญ่ไม่ต้องสร้างเอง Java เตรียมไว้ให้แล้ว

| Interface             | Method              | รับ → คืน            | ใช้เมื่อ                            |
| --------------------- | ------------------- | ------------------- | --------------------------------- |
| `Function<T, R>`      | `R apply(T t)`      | T → R               | แปลงค่า (เหมือน callback ของ `map`)  |
| `BiFunction<T, U, R>` | `R apply(T t, U u)` | (T, U) → R          | แปลงค่าจาก 2 input                  |
| `Supplier<T>`         | `T get()`           | () → T              | สร้าง/จัดหาค่า (lazy)                |
| `Consumer<T>`         | `void accept(T t)`  | T → void            | ทำอะไรบางอย่างกับค่า (เหมือน `forEach`) |
| `Predicate<T>`        | `boolean test(T t)` | T → boolean         | ตรวจเงื่อนไข (เหมือน `filter`)        |
| `UnaryOperator<T>`    | `T apply(T t)`      | T → T               | `Function` ที่ input/output type เดียวกัน |
| `BinaryOperator<T>`   | `T apply(T a, T b)` | (T, T) → T          | รวมค่าสองตัว (เหมือน `reduce`)        |
| `Runnable`            | `void run()`        | () → void           | งานที่ไม่รับและไม่คืนค่า              |

### ตัวอย่าง

```java
import java.util.function.*;

public class Main {
    public static void main(String[] args) {
        Function<String, Integer> length = s -> s.length();
        System.out.println(length.apply("สมชาย")); // ผลลัพธ์: 5

        BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
        System.out.println(repeat.apply("ฮ่า", 3)); // ผลลัพธ์: ฮ่าฮ่าฮ่า

        Supplier<Double> random = () -> Math.random();
        System.out.println(random.get() < 1.0);   // ผลลัพธ์: true

        Consumer<String> printer = s -> System.out.println("ชื่อ: " + s);
        printer.accept("สมหญิง");                 // ผลลัพธ์: ชื่อ: สมหญิง

        Predicate<Integer> isAdult = age -> age >= 18;
        System.out.println(isAdult.test(20));     // ผลลัพธ์: true

        UnaryOperator<String> upper = s -> s.toUpperCase();
        System.out.println(upper.apply("java"));  // ผลลัพธ์: JAVA

        BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
        System.out.println(max.apply(7, 3));      // ผลลัพธ์: 7
    }
}
```

### Primitive Specializations

มี interface สำหรับ primitive เพื่อเลี่ยง boxing เช่น `IntPredicate`, `IntFunction<R>`, `ToIntFunction<T>`, `IntUnaryOperator`, `IntBinaryOperator`

```java
IntBinaryOperator sum = (a, b) -> a + b;
System.out.println(sum.applyAsInt(2, 3)); // ผลลัพธ์: 5
```

---

## 4. Method Reference (`::`)

ถ้า lambda แค่ "ส่งต่อ" ไปเรียก method ที่มีอยู่แล้ว เขียนสั้นลงได้ด้วย method reference มี 4 แบบ

| แบบ                                     | Syntax                      | เทียบ Lambda                        |
| --------------------------------------- | --------------------------- | ---------------------------------- |
| 1. Static method                        | `ClassName::staticMethod`   | `x -> ClassName.staticMethod(x)`   |
| 2. Instance method ของ object ที่มีอยู่แล้ว   | `object::method`            | `x -> object.method(x)`            |
| 3. Instance method ของ object ใดก็ได้ (type) | `ClassName::instanceMethod` | `(obj) -> obj.instanceMethod()`    |
| 4. Constructor                          | `ClassName::new`            | `x -> new ClassName(x)`            |

### ตัวอย่าง

```java
import java.util.*;
import java.util.function.*;

public class Main {
    public static void main(String[] args) {
        // 1. Static method
        Function<String, Integer> parse = Integer::parseInt;
        System.out.println(parse.apply("42") + 1); // ผลลัพธ์: 43

        // 2. Instance method ของ object ที่มีอยู่แล้ว
        String prefix = "คุณ";
        Predicate<String> isKhun = prefix::equals;  // s -> prefix.equals(s)
        Consumer<String> print = System.out::println;
        print.accept("สวัสดี");                    // ผลลัพธ์: สวัสดี
        System.out.println(isKhun.test("คุณ"));    // ผลลัพธ์: true

        // 3. Instance method ของ object ใดก็ได้ — parameter ตัวแรกกลายเป็น "ตัวที่ถูกเรียก"
        Function<String, String> toUpper = String::toUpperCase; // s -> s.toUpperCase()
        System.out.println(toUpper.apply("java")); // ผลลัพธ์: JAVA

        BiFunction<String, String, Boolean> eq = String::equalsIgnoreCase; // (a, b) -> a.equalsIgnoreCase(b)
        System.out.println(eq.apply("Java", "JAVA")); // ผลลัพธ์: true

        // 4. Constructor
        Supplier<List<String>> listFactory = ArrayList::new;
        List<String> names = listFactory.get();
        names.add("สมชาย");
        System.out.println(names); // ผลลัพธ์: [สมชาย]

        Function<String, StringBuilder> sbFactory = StringBuilder::new;
        System.out.println(sbFactory.apply("Java").reverse()); // ผลลัพธ์: avaJ
    }
}
```

---

## 5. Effectively Final — การจับตัวแปรภายนอก (Capture)

Lambda อ่านตัวแปร local จากภายนอกได้ แต่ตัวแปรนั้นต้องเป็น **`final` หรือ effectively final** (ไม่ถูกแก้ค่าหลังประกาศ)

```java
String greeting = "สวัสดี"; // ไม่ถูกแก้ค่าอีก → effectively final
Consumer<String> greeter = name -> System.out.println(greeting + ", " + name);
greeter.accept("สมชาย"); // ผลลัพธ์: สวัสดี, สมชาย

int count = 0;
// Runnable r = () -> count++;  // ❌ Compile error: variable used in lambda should be final or effectively final
```

### วิธีแก้เมื่อต้องการ "นับ" จากใน lambda

```java
import java.util.concurrent.atomic.AtomicInteger;

AtomicInteger counter = new AtomicInteger(0);
List.of("ก", "ข", "ค").forEach(s -> counter.incrementAndGet());
System.out.println(counter.get()); // ผลลัพธ์: 3

// แต่ที่ดีกว่าคือใช้ Stream API ไม่ต้องมี state ภายนอก
long total = List.of("ก", "ข", "ค").stream().count();
System.out.println(total); // ผลลัพธ์: 3
```

> 💡 **เทียบกับ JavaScript:** closure ใน JS **แก้ค่าตัวแปรภายนอกได้** (`let count = 0; const inc = () => count++;`) แต่ Java ห้ามแก้ตัวแปร local ที่ถูก capture — ตั้งใจออกแบบให้ปลอดภัยเวลาใช้หลาย thread (field ของ object ยังแก้ได้ เพราะไม่ใช่ local variable)

> **หมายเหตุ:** `this` ใน lambda หมายถึง object ที่ล้อมรอบ (เหมือน arrow function ใน JS ที่ใช้ `this` ของ scope ภายนอก)

---

## 6. Composing — การต่อ Function เข้าด้วยกัน

### `Function.andThen` / `compose`

```java
Function<Integer, Integer> plus2 = x -> x + 2;
Function<Integer, Integer> times3 = x -> x * 3;

// andThen: ทำตัวแรกก่อน แล้วค่อยทำตัวที่สอง → times3(plus2(x))
System.out.println(plus2.andThen(times3).apply(5)); // ผลลัพธ์: 21

// compose: ทำตัวในวงเล็บก่อน → plus2(times3(x))
System.out.println(plus2.compose(times3).apply(5)); // ผลลัพธ์: 17
```

### `Predicate.and` / `or` / `negate` / `not`

```java
Predicate<Integer> isAdult = age -> age >= 18;
Predicate<Integer> isSenior = age -> age >= 60;

Predicate<Integer> workingAge = isAdult.and(isSenior.negate());
System.out.println(workingAge.test(30)); // ผลลัพธ์: true
System.out.println(workingAge.test(65)); // ผลลัพธ์: false

// Predicate.not (Java 11+) — สะดวกกับ method reference
List<String> names = List.of("สมชาย", "", "สมหญิง", " ");
List<String> nonBlank = names.stream()
        .filter(Predicate.not(String::isBlank))
        .toList(); // toList() ต้องใช้ Java 16+
System.out.println(nonBlank); // ผลลัพธ์: [สมชาย, สมหญิง]
```

### `Consumer.andThen`

```java
Consumer<String> log = s -> System.out.println("LOG: " + s);
Consumer<String> save = s -> System.out.println("บันทึก: " + s);

log.andThen(save).accept("สมศรี");
// ผลลัพธ์:
// LOG: สมศรี
// บันทึก: สมศรี
```

---

## 7. Comparator — ใช้ Lambda เรียงข้อมูล

`Comparator<T>` เป็น functional interface (method `compare(a, b)`) และมี helper อย่าง `comparing`, `thenComparing`, `reversed`

```java
import java.util.*;

public class Main {
    record Person(String name, int age) {} // record ต้องใช้ Java 16+

    public static void main(String[] args) {
        List<Person> people = new ArrayList<>(List.of(
                new Person("สมหญิง", 30),
                new Person("สมชาย", 25),
                new Person("สมศรี", 30)
        ));

        // แบบ lambda ล้วน (เหมือน JS: (a, b) => a.age - b.age)
        people.sort((a, b) -> Integer.compare(a.age(), b.age()));

        // แบบ Comparator.comparing — อ่านง่ายกว่า
        people.sort(Comparator.comparing(Person::age));
        System.out.println(people.get(0).name()); // ผลลัพธ์: สมชาย

        // เรียงอายุมาก → น้อย แล้วตามชื่อ
        people.sort(Comparator.comparing(Person::age).reversed()
                .thenComparing(Person::name));
        people.forEach(p -> System.out.println(p.name() + " " + p.age()));
        // ผลลัพธ์:
        // สมศรี 30
        // สมหญิง 30
        // สมชาย 25
    }
}
```

> **Tip:** ใช้ `Comparator.comparingInt(Person::age)` เพื่อเลี่ยง boxing และ **อย่าใช้ `a - b`** เปรียบเทียบตัวเลข เพราะอาจ overflow ได้ — ใช้ `Integer.compare(a, b)` แทน

---

## 8. ใช้ Lambda รับ Behavior เป็น Parameter

เทคนิคนี้คือหัวใจของ Stream API และ Spring — ส่ง "พฤติกรรม" เข้าไปใน method

```java
import java.util.*;
import java.util.function.*;

public class Main {
    static <T> List<T> filter(List<T> list, Predicate<T> condition) {
        List<T> result = new ArrayList<>();
        for (T item : list) {
            if (condition.test(item)) result.add(item);
        }
        return result;
    }

    public static void main(String[] args) {
        List<Integer> scores = List.of(45, 82, 67, 91, 30);

        System.out.println(filter(scores, s -> s >= 50)); // ผลลัพธ์: [82, 67, 91]
        System.out.println(filter(scores, s -> s >= 80)); // ผลลัพธ์: [82, 91]
    }
}
```

---

## ตารางเปรียบเทียบ Java Lambda vs JavaScript Arrow Function

| หัวข้อ                       | Java Lambda                                 | JavaScript Arrow Function          |
| --------------------------- | ------------------------------------------- | ---------------------------------- |
| Syntax                      | `(a, b) -> a + b`                           | `(a, b) => a + b`                  |
| Type                        | ต้องมี target type (Functional Interface)    | function เป็น value อยู่แล้ว            |
| การเรียกใช้                    | `f.apply(x)`, `p.test(x)`, `r.run()`        | `f(x)`                             |
| แก้ค่าตัวแปรภายนอก (closure)    | ไม่ได้ (ต้อง effectively final)               | ได้                                 |
| `this`                      | ของ class ที่ล้อมรอบ                           | ของ scope ภายนอก (lexical)         |
| Method reference            | `String::toUpperCase`                       | ไม่มี (ต้องเขียน `s => s.toUpperCase()`) |
| Throw checked exception     | ไม่ได้ ถ้า interface ไม่ประกาศ `throws`          | ได้เสมอ                             |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ interface สำเร็จรูปใน `java.util.function` ก่อน** — สร้าง functional interface เองเมื่อต้องการชื่อที่สื่อความหมายเฉพาะ domain (เช่น `Discount`, `Validator`)
2. **ใส่ `@FunctionalInterface`** ทุกครั้งที่สร้างเอง
3. **Lambda ควรสั้น** — ถ้าเกิน 3–4 บรรทัด ให้แยกเป็น method แล้วใช้ method reference
4. **ใช้ method reference เมื่ออ่านง่ายกว่า** — `map(String::trim)` ดีกว่า `map(s -> s.trim())`
5. **หลีกเลี่ยง side effect** ใน lambda (เช่นแก้ค่า list ภายนอก) — ให้ใช้ Stream `collect`/`toList()` แทน
6. **ใช้ `Comparator.comparing(...)`** แทนการเขียน `compare` เอง

```java
// แนวทางที่แนะนำ
List<String> result = List.of(" สมชาย ", "สมหญิง", "  ").stream()
        .map(String::trim)                    // method reference
        .filter(Predicate.not(String::isEmpty))
        .sorted(Comparator.comparing(String::length))
        .toList();

System.out.println(result); // ผลลัพธ์: [สมชาย, สมหญิง]
```

---
