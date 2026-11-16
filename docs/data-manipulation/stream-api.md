# Stream API

# ส่วนที่ 4: Stream API (Java 8+)

## Stream คืออะไร?

Stream คือ **ลำดับของข้อมูลที่ไหลผ่าน pipeline** ของการประมวลผล เช่น กรอง → แปลง → สรุปผล เขียนแบบ declarative (บอกว่า "ต้องการอะไร" ไม่ใช่ "ทำทีละขั้นอย่างไร")

```
แหล่งข้อมูล  ──►  intermediate ops  ──►  intermediate ops  ──►  terminal op  ──►  ผลลัพธ์
(List, array)     (filter)               (map)                 (collect)
```

### คุณสมบัติ

- **ไม่แก้ไขข้อมูลต้นทาง** — สร้างผลลัพธ์ใหม่เสมอ
- **Lazy** — intermediate operation จะยังไม่ทำงานจนกว่าจะมี terminal operation
- **ใช้ได้ครั้งเดียว** — stream ที่ถูก terminal แล้วใช้ซ้ำไม่ได้ (`IllegalStateException`)

### ตัวอย่าง

```java
import java.util.List;

List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6);

List<Integer> result = numbers.stream()
        .filter(n -> n % 2 == 0)   // [2, 4, 6]
        .map(n -> n * 10)          // [20, 40, 60]
        .toList();                 // Java 16+

System.out.println(result);        // ผลลัพธ์: [20, 40, 60]
System.out.println(numbers);       // ผลลัพธ์: [1, 2, 3, 4, 5, 6] — ต้นทางไม่เปลี่ยน
```

> 💡 **เทียบกับ JavaScript:** ใน JS เรียก `arr.filter(...).map(...)` ได้ตรง ๆ และแต่ละขั้นสร้าง array ใหม่ทันที ส่วน Java ต้อง **`.stream()` ก่อน** แล้วปิดท้ายด้วย terminal operation (`toList()`, `collect(...)`) และทำงานแบบ lazy ทีละสมาชิก

---

## 1. การสร้าง Stream

```java
import java.util.*;
import java.util.stream.*;

// จาก Collection
List<String> names = List.of("สมชาย", "สมหญิง");
Stream<String> s1 = names.stream();

// จาก array
String[] arr = {"ก", "ข", "ค"};
Stream<String> s2 = Arrays.stream(arr);

// จากค่าโดยตรง
Stream<String> s3 = Stream.of("A", "B", "C");

// ช่วงตัวเลข
IntStream r1 = IntStream.range(1, 5);         // 1, 2, 3, 4
IntStream r2 = IntStream.rangeClosed(1, 5);   // 1, 2, 3, 4, 5

// สร้างแบบไม่สิ้นสุด — ต้องใช้คู่กับ limit
Stream<Integer> evens = Stream.iterate(0, n -> n + 2).limit(5);        // 0, 2, 4, 6, 8
Stream<Integer> lessThan10 = Stream.iterate(1, n -> n < 10, n -> n * 2); // 1, 2, 4, 8 (Java 9+)
Stream<Double> randoms = Stream.generate(Math::random).limit(3);

// จาก String
IntStream chars = "hello".chars();
Stream<String> lines = "a\nb\nc".lines();     // Java 11+
```

---

## 2. Intermediate Operations (คืน Stream ใหม่)

### `filter` — กรอง

```java
List<Integer> scores = List.of(45, 82, 67, 90, 38);
List<Integer> passed = scores.stream()
        .filter(s -> s >= 50)
        .toList();
// [82, 67, 90]
```

### `map` — แปลงค่า

```java
List<String> names = List.of("somchai", "somying");
List<String> upper = names.stream()
        .map(String::toUpperCase)        // method reference
        .toList();
// [SOMCHAI, SOMYING]

List<Integer> lengths = names.stream()
        .map(String::length)
        .toList();
// [7, 7]
```

### `flatMap` — แปลงแล้วทำให้แบน

```java
List<List<String>> teams = List.of(
        List.of("สมชาย", "สมหญิง"),
        List.of("สมศรี"),
        List.of("อนันต์", "มานี")
);

List<String> everyone = teams.stream()
        .flatMap(List::stream)   // List<List<String>> → Stream<String>
        .toList();
// [สมชาย, สมหญิง, สมศรี, อนันต์, มานี]

List<String> words = Stream.of("hello world", "java stream")
        .flatMap(line -> Arrays.stream(line.split(" ")))
        .toList();
// [hello, world, java, stream]
```

### `sorted` — เรียงลำดับ

```java
import java.util.Comparator;

List<String> sorted = Stream.of("ทุเรียน", "กล้วย", "มะม่วง")
        .sorted()                                   // natural order
        .toList();
// [กล้วย, ทุเรียน, มะม่วง]

List<String> byLength = Stream.of("ทุเรียน", "กล้วย", "มะม่วง")
        .sorted(Comparator.comparing(String::length).reversed())
        .toList();
// [ทุเรียน, มะม่วง, กล้วย]  (7, 6, 5 ตัวอักษร)
```

### `distinct`, `limit`, `skip`

```java
List<Integer> nums = List.of(5, 3, 5, 1, 3, 8, 9);

nums.stream().distinct().toList();     // [5, 3, 1, 8, 9]
nums.stream().limit(3).toList();       // [5, 3, 5]
nums.stream().skip(4).toList();        // [3, 8, 9]

// pagination: หน้า 2 (หน้าละ 3 รายการ)
int page = 2, size = 3;
nums.stream().skip((long) (page - 1) * size).limit(size).toList(); // [1, 3, 8]
```

### `peek` — แอบดูค่าระหว่างทาง (ใช้ debug)

```java
List<Integer> r = Stream.of(1, 2, 3)
        .peek(n -> System.out.println("ก่อน: " + n))
        .map(n -> n * n)
        .peek(n -> System.out.println("หลัง: " + n))
        .toList();
```

### `mapToInt` / `boxed` — แปลงระหว่าง Stream กับ primitive stream

```java
List<String> items = List.of("กาแฟ", "ชา");
int totalChars = items.stream().mapToInt(String::length).sum(); // 6

List<Integer> boxed = IntStream.range(0, 3).boxed().toList();   // [0, 1, 2]
```

---

## 3. Terminal Operations (ปิด Stream และให้ผลลัพธ์)

### `toList` / `collect`

```java
import java.util.stream.Collectors;

List<String> a = Stream.of("x", "y").toList();                         // immutable (Java 16+)
List<String> b = Stream.of("x", "y").collect(Collectors.toList());     // mutable (Java 8+)
Set<String> c = Stream.of("x", "y", "x").collect(Collectors.toSet());  // [x, y]
```

### `forEach`

```java
Stream.of("สมชาย", "สมหญิง").forEach(System.out::println);
```

### `reduce` — รวมเป็นค่าเดียว

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5);

int sum = nums.stream().reduce(0, (acc, n) -> acc + n);     // 15
int sum2 = nums.stream().reduce(0, Integer::sum);           // 15
int product = nums.stream().reduce(1, (a, b) -> a * b);     // 120

Optional<Integer> max = nums.stream().reduce(Integer::max); // Optional[5] — ไม่มีค่าเริ่มต้นจึงคืน Optional
```

### `count`, `min`, `max`

```java
long count = Stream.of("a", "b", "c").count();                       // 3
Optional<String> shortest = Stream.of("ทุเรียน", "ส้ม", "มะม่วง")
        .min(Comparator.comparing(String::length));                  // Optional[ส้ม]
```

### `anyMatch`, `allMatch`, `noneMatch`

```java
List<Integer> ages = List.of(15, 22, 35);

ages.stream().anyMatch(a -> a >= 18);   // true — มีอย่างน้อย 1 คน
ages.stream().allMatch(a -> a >= 18);   // false — ไม่ใช่ทุกคน
ages.stream().noneMatch(a -> a > 60);   // true — ไม่มีใครเกิน 60
```

### `findFirst`, `findAny` — คืน `Optional`

```java
Optional<String> first = Stream.of("สมชาย", "อนันต์", "สมศรี")
        .filter(n -> n.startsWith("อ"))
        .findFirst();

System.out.println(first.orElse("ไม่พบ")); // ผลลัพธ์: อนันต์
```

ดูวิธีจัดการ `Optional` ใน [Optional](optional.md)

### Primitive stream: `sum`, `average`, `summaryStatistics`

```java
int[] scores = {80, 92, 75};
IntSummaryStatistics stats = Arrays.stream(scores).summaryStatistics();

stats.getSum();      // 247
stats.getAverage();  // 82.33333333333333
stats.getMax();      // 92
stats.getMin();      // 75
stats.getCount();    // 3

double avg = IntStream.of(1, 2, 3, 4).average().orElse(0); // 2.5
```

---

## 4. Collectors

ใช้กับ `.collect(...)` เพื่อรวบรวมผลลัพธ์ในรูปแบบต่าง ๆ

```java
import java.util.*;
import java.util.stream.*;

public class Main {
    record Employee(String name, String dept, int salary) {}

    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee("สมชาย", "IT", 50000),
                new Employee("สมหญิง", "HR", 40000),
                new Employee("สมศรี", "IT", 60000),
                new Employee("อนันต์", "Sales", 35000)
        );

        // joining — ต่อ string
        String names = employees.stream()
                .map(Employee::name)
                .collect(Collectors.joining(", ", "[", "]"));
        System.out.println(names); // ผลลัพธ์: [สมชาย, สมหญิง, สมศรี, อนันต์]

        // groupingBy — จัดกลุ่ม
        Map<String, List<Employee>> byDept = employees.stream()
                .collect(Collectors.groupingBy(Employee::dept));
        System.out.println(byDept.get("IT").size()); // ผลลัพธ์: 2

        // groupingBy + counting — นับจำนวนต่อกลุ่ม
        Map<String, Long> countByDept = employees.stream()
                .collect(Collectors.groupingBy(Employee::dept, TreeMap::new, Collectors.counting()));
        System.out.println(countByDept); // ผลลัพธ์: {HR=1, IT=2, Sales=1}

        // groupingBy + summingInt — รวมเงินเดือนต่อแผนก
        Map<String, Integer> salaryByDept = employees.stream()
                .collect(Collectors.groupingBy(Employee::dept, Collectors.summingInt(Employee::salary)));
        System.out.println(salaryByDept.get("IT")); // ผลลัพธ์: 110000

        // groupingBy + mapping — เก็บเฉพาะชื่อ
        Map<String, List<String>> namesByDept = employees.stream()
                .collect(Collectors.groupingBy(Employee::dept,
                        Collectors.mapping(Employee::name, Collectors.toList())));
        System.out.println(namesByDept.get("IT")); // ผลลัพธ์: [สมชาย, สมศรี]

        // partitioningBy — แบ่งเป็น 2 กลุ่ม true / false
        Map<Boolean, List<String>> highSalary = employees.stream()
                .collect(Collectors.partitioningBy(e -> e.salary() >= 45000,
                        Collectors.mapping(Employee::name, Collectors.toList())));
        System.out.println(highSalary.get(true));  // ผลลัพธ์: [สมชาย, สมศรี]
        System.out.println(highSalary.get(false)); // ผลลัพธ์: [สมหญิง, อนันต์]

        // toMap — แปลงเป็น Map
        Map<String, Integer> salaryByName = employees.stream()
                .collect(Collectors.toMap(Employee::name, Employee::salary));
        System.out.println(salaryByName.get("สมศรี")); // ผลลัพธ์: 60000

        // averagingInt
        double avg = employees.stream().collect(Collectors.averagingInt(Employee::salary));
        System.out.println(avg); // ผลลัพธ์: 46250.0
    }
}
```

### ⚠️ `toMap` กับ key ซ้ำ

```java
// key ซ้ำ → IllegalStateException: Duplicate key
Map<String, Integer> m = Stream.of("กาแฟ", "ชา", "กาแฟ")
        .collect(Collectors.toMap(s -> s, s -> 1, Integer::sum)); // ✅ ใส่ merge function
// {กาแฟ=2, ชา=1}
```

| Collector                     | ผลลัพธ์                   | เทียบ JS                                |
| ----------------------------- | ------------------------- | --------------------------------------- |
| `toList()` / `toSet()`        | `List` / `Set`            | -                                       |
| `joining(", ")`               | `String`                  | `arr.join(", ")`                        |
| `groupingBy(f)`               | `Map<K, List<T>>`         | `Object.groupBy(arr, f)`                |
| `groupingBy(f, counting())`   | `Map<K, Long>`            | `reduce` นับเอง                         |
| `partitioningBy(pred)`        | `Map<Boolean, List<T>>`   | `filter` 2 ครั้ง                         |
| `toMap(k, v)`                 | `Map<K, V>`               | `Object.fromEntries(arr.map(...))`      |
| `summingInt(f)`               | `Integer`                 | `reduce((a, x) => a + x.f, 0)`          |

---

## 5. Lazy Evaluation

intermediate operation จะไม่ทำงานจนกว่าจะมี terminal operation และประมวลผล **ทีละสมาชิกตลอด pipeline** (ไม่ใช่ทีละขั้นทั้ง list)

```java
Stream<String> pipeline = Stream.of("ก", "ข", "ค")
        .filter(s -> {
            System.out.println("filter: " + s);
            return true;
        });
// ยังไม่มีอะไรพิมพ์ออกมา — เพราะยังไม่มี terminal operation

Optional<String> first = Stream.of("ก", "ข", "ค")
        .filter(s -> {
            System.out.println("filter: " + s);
            return !s.equals("ก");
        })
        .map(s -> {
            System.out.println("map: " + s);
            return s + "!";
        })
        .findFirst();
// ผลลัพธ์:
// filter: ก
// filter: ข
// map: ข
// ("ค" ไม่ถูกประมวลผลเลย — short-circuit เพราะ findFirst ได้คำตอบแล้ว)
```

> 💡 **เทียบกับ JavaScript:** `arr.filter(...).map(...).find(...)` ใน JS จะ filter **ทั้ง array** ก่อน แล้ว map **ทั้ง array** แล้วค่อย find แต่ Java Stream ทำทีละตัวและหยุดทันทีที่ได้คำตอบ — คล้าย iterator helpers / generator ของ JS มากกว่า

### Stream ใช้ซ้ำไม่ได้

```java
Stream<Integer> s = Stream.of(1, 2, 3);
s.count();
// s.count(); // IllegalStateException: stream has already been operated upon or closed
```

---

## ตารางเปรียบเทียบกับ JavaScript Array Methods

| JavaScript                              | Java Stream                                              |
| --------------------------------------- | -------------------------------------------------------- |
| `arr.map(x => x * 2)`                   | `list.stream().map(x -> x * 2).toList()`                 |
| `arr.filter(x => x > 0)`                | `list.stream().filter(x -> x > 0).toList()`              |
| `arr.reduce((a, x) => a + x, 0)`        | `list.stream().reduce(0, Integer::sum)`                  |
| `arr.find(x => x > 0)`                  | `list.stream().filter(x -> x > 0).findFirst()` → `Optional` |
| `arr.findIndex(...)`                    | `IntStream.range(0, list.size()).filter(i -> ...).findFirst()` |
| `arr.some(x => x > 0)`                  | `list.stream().anyMatch(x -> x > 0)`                     |
| `arr.every(x => x > 0)`                 | `list.stream().allMatch(x -> x > 0)`                     |
| `arr.flat()`                            | `list.stream().flatMap(List::stream).toList()`           |
| `arr.flatMap(x => [...])`               | `list.stream().flatMap(x -> Stream.of(...))`             |
| `arr.includes(x)`                       | `list.contains(x)` / `stream().anyMatch(x::equals)`      |
| `arr.slice(2, 5)`                       | `stream().skip(2).limit(3)`                              |
| `[...new Set(arr)]`                     | `stream().distinct().toList()`                           |
| `arr.toSorted()`                        | `stream().sorted().toList()`                             |
| `arr.join(", ")`                        | `stream().collect(Collectors.joining(", "))`             |
| `arr.length` (หลัง filter)              | `stream().filter(...).count()`                           |
| `Math.max(...arr)`                      | `stream().max(Comparator.naturalOrder())` / `mapToInt(..).max()` |
| `Object.groupBy(arr, fn)`               | `collect(Collectors.groupingBy(fn))`                     |
| `arr.forEach(fn)`                       | `list.forEach(fn)` / `stream().forEach(fn)`              |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ `toList()`** (Java 16+) แทน `collect(Collectors.toList())` เมื่อไม่ต้องแก้ไขผลลัพธ์
2. **อย่ามี side effect ใน `map` / `filter`** — เช่น แก้ตัวแปรภายนอกหรือ `add` ลง list อื่น
3. **ใช้ method reference** (`String::length`, `Employee::name`) เมื่ออ่านง่ายกว่า lambda
4. **ใช้ primitive stream** (`mapToInt`, `IntStream`) สำหรับงานตัวเลข — หลีกเลี่ยง boxing
5. **`peek` ใช้สำหรับ debug เท่านั้น**
6. **ไม่ต้องใช้ stream ทุกที่** — loop ธรรมดาอาจอ่านง่ายกว่าเมื่อ logic ซับซ้อนหรือมี `break`
7. **ระวัง `toMap` กับ key ซ้ำ** — ใส่ merge function เสมอถ้าไม่แน่ใจ

```java
import java.util.*;
import java.util.stream.*;

public class Main {
    record Order(String customer, String product, int amount) {}

    public static void main(String[] args) {
        List<Order> orders = List.of(
                new Order("สมชาย", "กาแฟ", 120),
                new Order("สมหญิง", "ชา", 80),
                new Order("สมชาย", "เค้ก", 150),
                new Order("สมศรี", "กาแฟ", 60)
        );

        // ยอดซื้อรวมของลูกค้าแต่ละคน เรียงจากมากไปน้อย แสดง 2 อันดับแรก
        String top2 = orders.stream()
                .collect(Collectors.groupingBy(Order::customer, Collectors.summingInt(Order::amount)))
                .entrySet().stream()
                .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
                .limit(2)
                .map(e -> e.getKey() + "=" + e.getValue())
                .collect(Collectors.joining(", "));

        System.out.println(top2); // ผลลัพธ์: สมชาย=270, สมหญิง=80
    }
}
```

---
