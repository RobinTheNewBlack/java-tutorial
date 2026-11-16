# Advanced

# ส่วนที่ 2: Generics

## Generics คืออะไร?

Generics (Java 5+) คือการเขียน class, interface หรือ method ที่ **รับ type เป็น parameter** ทำให้โค้ดเดียวใช้กับหลาย type ได้ โดยยังมี **type safety** ตอน compile

คุณใช้ generics มาตลอดแล้วโดยไม่รู้ตัว — `List<String>`, `Map<String, Integer>`, `Optional<User>`, `Function<T, R>`

> 💡 **เทียบกับ JavaScript:** JS ไม่มี type จึงไม่ต้องมี generics แต่ถ้าเคยใช้ **TypeScript** แนวคิดและ syntax `<T>` แทบเหมือนกันทุกอย่าง

---

## 1. ทำไมต้องมี Generics?

### ❌ ก่อนมี Generics — ใช้ `Object` แล้วต้อง cast

```java
List list = new ArrayList();   // raw type
list.add("สมชาย");
list.add(123);                 // ใส่อะไรก็ได้ — compile ผ่าน (มีแค่ warning "unchecked")

String name = (String) list.get(0); // ต้อง cast เอง
String oops = (String) list.get(1); // 💥 ClassCastException ตอน runtime!
```

### ✅ ใช้ Generics — compiler ตรวจให้

```java
List<String> names = new ArrayList<>(); // <> = diamond operator (Java 7+) อนุมาน type ให้
names.add("สมชาย");
// names.add(123);                     // ❌ Compile error — จับได้ตั้งแต่ตอนเขียน
String name = names.get(0);             // ไม่ต้อง cast
```

### ข้อดี

- **Type safety** — จับ error ตอน compile แทน runtime
- **ไม่ต้อง cast**
- **Reusable** — เขียนครั้งเดียวใช้กับทุก type

---

## 2. Generic Class

```java
class Box<T> {           // T = Type parameter
    private T value;

    Box(T value) {
        this.value = value;
    }

    T get() {
        return value;
    }

    void set(T value) {
        this.value = value;
    }

    boolean isEmpty() {
        return value == null;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<String> nameBox = new Box<>("สมชาย");
        Box<Integer> ageBox = new Box<>(25);

        String name = nameBox.get(); // ไม่ต้อง cast
        int age = ageBox.get();      // auto-unboxing

        System.out.println(name + " อายุ " + age); // ผลลัพธ์: สมชาย อายุ 25
        // nameBox.set(100); // ❌ Compile error
    }
}
```

### หลาย Type Parameter

```java
class Pair<K, V> {
    private final K key;
    private final V value;

    Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    K getKey() { return key; }
    V getValue() { return value; }

    @Override
    public String toString() { return key + "=" + value; }
}

Pair<String, Integer> score = new Pair<>("สมหญิง", 95);
System.out.println(score); // ผลลัพธ์: สมหญิง=95

// หรือใช้ record (Java 16+) ซึ่ง generic ได้เหมือนกัน
record Tuple<A, B>(A first, B second) {}
Tuple<String, Double> gpa = new Tuple<>("สมศรี", 3.75);
System.out.println(gpa.first() + " " + gpa.second()); // ผลลัพธ์: สมศรี 3.75
```

### ธรรมเนียมการตั้งชื่อ Type Parameter

| ชื่อ  | ความหมาย             | ตัวอย่าง                   |
| ---- | ------------------- | ------------------------- |
| `T`  | Type                | `Box<T>`                  |
| `E`  | Element             | `List<E>`                 |
| `K`, `V` | Key, Value      | `Map<K, V>`               |
| `R`  | Return type         | `Function<T, R>`          |
| `ID` | Identifier          | `JpaRepository<T, ID>`    |
| `S`, `U` | Type ลำดับที่ 2, 3  | `BiFunction<T, U, R>`     |

---

## 3. Generic Method

ประกาศ type parameter **หน้า return type** ใช้ได้แม้ class ไม่ใช่ generic

```java
import java.util.*;

public class Main {
    // <T> ประกาศ type parameter ของ method นี้
    static <T> T firstOrDefault(List<T> list, T defaultValue) {
        return list.isEmpty() ? defaultValue : list.get(0);
    }

    static <T> void printAll(T[] items) {
        for (T item : items) {
            System.out.print(item + " ");
        }
        System.out.println();
    }

    static <K, V> Map<V, K> invert(Map<K, V> map) {
        Map<V, K> result = new HashMap<>();
        map.forEach((k, v) -> result.put(v, k));
        return result;
    }

    public static void main(String[] args) {
        System.out.println(firstOrDefault(List.of("ก", "ข"), "ว่าง")); // ผลลัพธ์: ก
        System.out.println(firstOrDefault(List.<Integer>of(), 0));    // ผลลัพธ์: 0

        printAll(new String[]{"สมชาย", "สมหญิง"}); // ผลลัพธ์: สมชาย สมหญิง
        printAll(new Integer[]{1, 2, 3});          // ผลลัพธ์: 1 2 3

        System.out.println(invert(Map.of("TH", "ไทย"))); // ผลลัพธ์: {ไทย=TH}
    }
}
```

> **หมายเหตุ:** ส่วนใหญ่ compiler **อนุมาน type ให้เอง** (type inference) ไม่ต้องเขียน `Main.<String>firstOrDefault(...)`

---

## 4. Bounded Type — จำกัด Type

### Upper Bound: `<T extends X>`

T ต้องเป็น X หรือ subclass ของ X (ใช้ `extends` ทั้งกับ class และ interface) — ทำให้เรียก method ของ X ได้

```java
// T ต้องเป็น Number (Integer, Double, Long ...)
static <T extends Number> double sum(List<T> numbers) {
    double total = 0;
    for (T n : numbers) {
        total += n.doubleValue(); // เรียกได้เพราะรู้ว่า T เป็น Number
    }
    return total;
}

System.out.println(sum(List.of(1, 2, 3)));       // ผลลัพธ์: 6.0
System.out.println(sum(List.of(1.5, 2.5)));      // ผลลัพธ์: 4.0
// sum(List.of("ก", "ข"));                       // ❌ Compile error — String ไม่ใช่ Number
```

### Multiple Bounds

```java
// T ต้องเป็น Comparable ของตัวเอง → เปรียบเทียบกันได้
static <T extends Comparable<T>> T max(List<T> list) {
    T best = list.get(0);
    for (T item : list) {
        if (item.compareTo(best) > 0) best = item;
    }
    return best;
}

System.out.println(max(List.of(3, 9, 4)));            // ผลลัพธ์: 9
System.out.println(max(List.of("กล้วย", "มะม่วง"))); // ผลลัพธ์: มะม่วง

// หลาย bound: class ก่อน แล้วตามด้วย interface คั่นด้วย &
// static <T extends Number & Comparable<T>> T clamp(T value, T min, T max) { ... }
```

---

## 5. Wildcards (`?`)

Wildcard ใช้ตอน **รับ** generic type เป็น parameter เมื่อไม่ต้องการผูกกับ type ใด type หนึ่ง

### ปัญหา: Generics ไม่ "สืบทอด" ตาม type ข้างใน

```java
List<Integer> ints = List.of(1, 2, 3);
// List<Number> nums = ints; // ❌ Compile error!
// แม้ Integer เป็น Number แต่ List<Integer> ไม่ใช่ List<Number> (invariant)
```

### 3 รูปแบบของ Wildcard

| รูปแบบ            | ความหมาย                         | อ่านได้ (get) | เขียนได้ (add) |
| ---------------- | -------------------------------- | ----------- | ------------ |
| `<?>`            | type อะไรก็ได้                     | เป็น `Object` | ไม่ได้ (ยกเว้น `null`) |
| `<? extends T>`  | T หรือ **ลูก** ของ T (upper bound) | เป็น `T`     | ไม่ได้          |
| `<? super T>`    | T หรือ **แม่** ของ T (lower bound) | เป็น `Object` | ใส่ `T` ได้     |

```java
import java.util.*;

public class Main {
    // ? — อ่านอย่างเดียว ไม่สนใจ type
    static void printSize(List<?> list) {
        System.out.println("ขนาด: " + list.size());
    }

    // ? extends Number — รับ List<Integer>, List<Double> ... อ่านเป็น Number ได้
    static double total(List<? extends Number> numbers) {
        double sum = 0;
        for (Number n : numbers) sum += n.doubleValue();
        // numbers.add(1); // ❌ ไม่รู้ว่าจริง ๆ เป็น List ของอะไร จึงห้ามใส่
        return sum;
    }

    // ? super Integer — รับ List<Integer>, List<Number>, List<Object> ... ใส่ Integer ได้
    static void fillNumbers(List<? super Integer> target) {
        for (int i = 1; i <= 3; i++) target.add(i);
    }

    public static void main(String[] args) {
        printSize(List.of("ก", "ข"));                  // ผลลัพธ์: ขนาด: 2
        System.out.println(total(List.of(1, 2.5, 3L))); // ผลลัพธ์: 6.5

        List<Number> numbers = new ArrayList<>();
        fillNumbers(numbers);
        System.out.println(numbers); // ผลลัพธ์: [1, 2, 3]

        List<Object> objects = new ArrayList<>();
        fillNumbers(objects);        // ใช้ได้เช่นกัน
    }
}
```

### PECS — Producer Extends, Consumer Super

กฎจำง่ายในการเลือก wildcard:
- ถ้า parameter **ผลิตค่าให้เราอ่าน** (Producer) → ใช้ `? extends T`
- ถ้า parameter **รับค่าที่เราใส่** (Consumer) → ใช้ `? super T`
- ถ้าทั้งอ่านและเขียน → ใช้ `T` ตรง ๆ

```java
// ตัวอย่างจาก JDK: Collections.copy
// public static <T> void copy(List<? super T> dest, List<? extends T> src)
//                                  ↑ consumer             ↑ producer

static <T> void copyAll(List<? extends T> src, List<? super T> dest) {
    for (T item : src) {
        dest.add(item);
    }
}

List<Integer> source = List.of(10, 20);
List<Number> destination = new ArrayList<>();
copyAll(source, destination);
System.out.println(destination); // ผลลัพธ์: [10, 20]
```

---

## 6. Type Erasure และข้อจำกัด

Generics ของ Java ทำงาน **เฉพาะตอน compile** — หลัง compile ข้อมูล type parameter จะถูก **ลบทิ้ง (erasure)** และแทนด้วย `Object` (หรือ bound) พร้อมใส่ cast ให้อัตโนมัติ

```java
List<String> a = new ArrayList<>();
List<Integer> b = new ArrayList<>();
System.out.println(a.getClass() == b.getClass()); // ผลลัพธ์: true — runtime เป็น ArrayList เหมือนกัน
```

### ข้อจำกัดที่ตามมา

| ทำไม่ได้                                  | เหตุผล / วิธีแก้                                          |
| --------------------------------------- | ------------------------------------------------------ |
| `new T()`                               | ไม่รู้ type ตอน runtime → รับ `Supplier<T>` หรือ `Class<T>` แทน |
| `new T[10]`                             | ใช้ `List<T>` แทน                                        |
| `obj instanceof List<String>`           | ใช้ `obj instanceof List<?>` ได้                           |
| `List<int>` (primitive)                 | ต้องใช้ wrapper `List<Integer>`                            |
| `static T field`                        | static เป็นของ class ไม่ผูกกับ T ของ instance ใด            |
| overload `m(List<String>)` กับ `m(List<Integer>)` | หลัง erasure ได้ signature เดียวกัน `m(List)`          |

```java
import java.util.function.Supplier;

// วิธีแก้ new T(): รับ factory เข้ามา
static <T> List<T> createMany(Supplier<T> factory, int count) {
    List<T> list = new ArrayList<>();
    for (int i = 0; i < count; i++) list.add(factory.get());
    return list;
}

List<StringBuilder> builders = createMany(StringBuilder::new, 3);
System.out.println(builders.size()); // ผลลัพธ์: 3
```

> 💡 **เทียบกับ TypeScript:** TypeScript ก็ลบ type ทิ้งตอน compile เป็น JS เหมือนกัน (type erasure) — แนวคิด "type มีแค่ตอน compile" จึงคุ้นเคยอยู่แล้ว

---

## 7. Generic Interface — `Repository<T, ID>`

ตัวอย่างนี้จำลองแนวคิดของ **Spring Data `JpaRepository<T, ID>`** — เขียน CRUD ครั้งเดียวใช้ได้กับทุก entity

```java
import java.util.*;

interface Identifiable<ID> {
    ID getId();
}

interface Repository<T extends Identifiable<ID>, ID> {
    T save(T entity);
    Optional<T> findById(ID id);
    List<T> findAll();
    void deleteById(ID id);
    default long count() { return findAll().size(); }
}

class InMemoryRepository<T extends Identifiable<ID>, ID> implements Repository<T, ID> {
    private final Map<ID, T> store = new LinkedHashMap<>();

    @Override
    public T save(T entity) {
        store.put(entity.getId(), entity);
        return entity;
    }

    @Override
    public Optional<T> findById(ID id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public List<T> findAll() {
        return new ArrayList<>(store.values());
    }

    @Override
    public void deleteById(ID id) {
        store.remove(id);
    }
}

record User(Long id, String name) implements Identifiable<Long> {
    @Override
    public Long getId() { return id; }
}

record Product(String sku, String title) implements Identifiable<String> {
    @Override
    public String getId() { return sku; }
}

public class Main {
    public static void main(String[] args) {
        Repository<User, Long> userRepo = new InMemoryRepository<>();
        userRepo.save(new User(1L, "สมชาย"));
        userRepo.save(new User(2L, "สมหญิง"));

        System.out.println(userRepo.findById(2L).map(User::name).orElse("ไม่พบ")); // ผลลัพธ์: สมหญิง
        System.out.println(userRepo.count()); // ผลลัพธ์: 2

        Repository<Product, String> productRepo = new InMemoryRepository<>();
        productRepo.save(new Product("TH-001", "ทุเรียนหมอนทอง"));
        System.out.println(productRepo.findById("TH-999").isPresent()); // ผลลัพธ์: false
    }
}
```

ใน Spring Data JPA เราแค่ประกาศ interface — Spring สร้าง implementation ให้ตอน runtime:

```java
// ตัวอย่างแนวคิด (ต้องมี Spring Data JPA)
public interface UserRepository extends JpaRepository<UserEntity, Long> {
    List<UserEntity> findByName(String name); // Spring สร้าง query ให้จากชื่อ method
}
```

---

## 8. เทียบกับ TypeScript Generics

```ts
// TypeScript
class Box<T> {
    constructor(private value: T) {}
    get(): T { return this.value; }
}

function first<T>(arr: T[]): T | undefined { return arr[0]; }

function sum<T extends number>(nums: T[]): number { /* ... */ }
```

| หัวข้อ                | Java                              | TypeScript                       |
| -------------------- | --------------------------------- | -------------------------------- |
| Generic class        | `class Box<T>`                    | `class Box<T>`                   |
| Generic method       | `static <T> T first(List<T> l)`   | `function first<T>(arr: T[]): T` |
| Upper bound          | `<T extends Number>`              | `<T extends number>`             |
| Lower bound          | `<? super T>`                     | ไม่มี                              |
| Wildcard             | `List<?>`                         | `unknown[]` / `any[]`            |
| Variance             | Invariant (ใช้ wildcard ช่วย)       | ส่วนใหญ่ covariant (structural)    |
| Default type         | ไม่มี                               | `<T = string>`                   |
| Primitive            | ไม่ได้ ต้องใช้ wrapper (`Integer`)    | ได้ (`number`)                    |
| Runtime type info    | ถูกลบ (erasure)                    | ถูกลบ (compile เป็น JS)            |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ห้ามใช้ raw type** (`List` เฉย ๆ) — ใช้ `List<String>` หรือ `List<?>` เสมอ
2. **ใช้ diamond operator `<>`** — `new ArrayList<>()` ให้ compiler อนุมานเอง
3. **ใช้ PECS** เมื่อออกแบบ API ที่รับ collection — ทำให้ method ยืดหยุ่นขึ้นมาก
4. **ใช้ bounded type** เมื่อต้องเรียก method ของ type นั้น (`<T extends Comparable<T>>`)
5. **ตั้งชื่อ type parameter ตามธรรมเนียม** (`T`, `E`, `K`, `V`, `R`)
6. **อย่าใช้ wildcard ใน return type** — ทำให้ผู้เรียกใช้งานยาก
7. **ใช้ generic method แทน `Object`** — ได้ type safety และไม่ต้อง cast

```java
// แนวทางที่แนะนำ
// รับ Collection ของ T หรือลูกของ T (producer → extends), T ต้องเปรียบเทียบได้
static <T extends Comparable<? super T>> List<T> topN(Collection<? extends T> items, int n) {
    List<T> copy = new ArrayList<>(items);   // ไม่แก้ collection ต้นฉบับ
    copy.sort(Comparator.reverseOrder());
    return List.copyOf(copy.subList(0, Math.min(n, copy.size())));
}

System.out.println(topN(List.of(5, 1, 9, 3), 2)); // ผลลัพธ์: [9, 5]
```

---
