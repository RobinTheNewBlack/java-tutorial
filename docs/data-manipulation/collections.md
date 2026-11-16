# Collections

# ส่วนที่ 3: Collections Framework

## Collections Framework คืออะไร?

Collections Framework คือชุดของ **interface** และ **class** ใน `java.util` สำหรับเก็บและจัดการกลุ่มข้อมูล เช่น list, set, map, queue — เป็นเครื่องมือที่ใช้บ่อยที่สุดใน Java

> 💡 **เทียบกับ JavaScript:** JS มี `Array`, `Set`, `Map` และ `Object` ให้ใช้ ส่วน Java แยกเป็น **interface** (สัญญาว่าทำอะไรได้) กับ **implementation** (วิธีทำงานภายใน) — เช่น `List` คือ interface ส่วน `ArrayList` / `LinkedList` คือ implementation ที่ให้เลือกตามลักษณะงาน

---

## ภาพรวม (Diagram)

```
                    Iterable<E>
                        │
                  Collection<E>
          ┌─────────────┼──────────────────┐
          │             │                  │
       List<E>        Set<E>            Queue<E>
          │             │                  │
   ┌──────┴─────┐  ┌────┴─────────┐    Deque<E>
   │            │  │              │        │
ArrayList  LinkedList HashSet  SortedSet  ArrayDeque
                      │           │
               LinkedHashSet   TreeSet

   (SequencedCollection<E> — Java 21: อยู่ระหว่าง Collection กับ List / Deque / LinkedHashSet / SortedSet)


                      Map<K, V>          ← ไม่ได้สืบทอดจาก Collection
          ┌───────────┼─────────────┐
       HashMap    SortedMap      (SequencedMap — Java 21)
          │           │
   LinkedHashMap   TreeMap
```

### เลือกใช้อะไร?

| ต้องการ                                       | ใช้                    |
| --------------------------------------------- | ---------------------- |
| list ทั่วไป เข้าถึงด้วย index                  | `ArrayList`            |
| ไม่ให้มีค่าซ้ำ ไม่สนลำดับ                       | `HashSet`              |
| ไม่ให้มีค่าซ้ำ รักษาลำดับที่ใส่                 | `LinkedHashSet`        |
| ไม่ให้มีค่าซ้ำ เรียงลำดับอัตโนมัติ              | `TreeSet`              |
| key → value ไม่สนลำดับ                         | `HashMap`              |
| key → value รักษาลำดับที่ใส่                   | `LinkedHashMap`        |
| key → value เรียงตาม key                       | `TreeMap`              |
| stack / queue                                 | `ArrayDeque`           |
| queue ตามลำดับความสำคัญ                        | `PriorityQueue`        |

### ประกาศด้วย interface เสมอ

```java
import java.util.*;

List<String> names = new ArrayList<>();     // ✅ ประกาศด้วย interface
ArrayList<String> names2 = new ArrayList<>(); // ได้ แต่ผูกติดกับ implementation
```

> `<>` (diamond operator) ให้ compiler อนุมาน type จากฝั่งซ้าย และ collection เก็บ primitive ไม่ได้ — ต้องใช้ wrapper เช่น `List<Integer>` (ดู [Data Types](../fundamental/data-types.md))

---

## 1. List — เรียงตามลำดับ มีค่าซ้ำได้

### คุณสมบัติ

- เก็บตาม **ลำดับที่ใส่** เข้าถึงด้วย **index**
- **มีค่าซ้ำได้** และเก็บ `null` ได้ (ArrayList)

### `ArrayList` — ใช้บ่อยที่สุด

```java
import java.util.ArrayList;
import java.util.List;

List<String> fruits = new ArrayList<>();

fruits.add("มะม่วง");              // เพิ่มท้าย
fruits.add("ทุเรียน");
fruits.add(0, "มังคุด");           // แทรกที่ index 0
System.out.println(fruits);        // ผลลัพธ์: [มังคุด, มะม่วง, ทุเรียน]

fruits.get(1);                     // "มะม่วง"
fruits.set(1, "เงาะ");             // แทนที่ index 1
fruits.size();                     // 3
fruits.contains("ทุเรียน");        // true
fruits.indexOf("ทุเรียน");         // 2
fruits.isEmpty();                  // false

fruits.remove("เงาะ");             // ลบตามค่า
fruits.remove(0);                  // ลบตาม index
System.out.println(fruits);        // ผลลัพธ์: [ทุเรียน]

fruits.addAll(List.of("ลำไย", "ลิ้นจี่"));
fruits.removeIf(f -> f.startsWith("ล")); // ลบตามเงื่อนไข
fruits.clear();                    // ลบทั้งหมด
```

### ⚠️ กับดัก `remove` กับ `List<Integer>`

```java
List<Integer> nums = new ArrayList<>(List.of(10, 20, 30));
nums.remove(1);                    // ลบ index 1 → [10, 30]
nums.remove(Integer.valueOf(30));  // ลบค่า 30 → [10]
```

### เรียงลำดับ

```java
import java.util.Comparator;

List<String> names = new ArrayList<>(List.of("สมหญิง", "สมชาย", "อนันต์"));
names.sort(Comparator.naturalOrder());     // [สมชาย, สมหญิง, อนันต์]
names.sort(Comparator.reverseOrder());     // [อนันต์, สมหญิง, สมชาย]
names.sort(Comparator.comparing(String::length)); // เรียงตามความยาว
```

### `LinkedList`

```java
import java.util.LinkedList;

LinkedList<String> queue = new LinkedList<>();
queue.addFirst("A");
queue.addLast("B");
queue.removeFirst();               // "A"
```

| การทำงาน              | `ArrayList`     | `LinkedList`    |
| --------------------- | --------------- | --------------- |
| `get(i)`              | O(1) เร็ว       | O(n) ช้า        |
| เพิ่ม/ลบท้าย          | O(1)            | O(1)            |
| เพิ่ม/ลบตรงกลาง/หน้า  | O(n)            | O(1)*           |
| ใช้ memory            | น้อย            | มาก             |

\* เมื่ออยู่ที่ตำแหน่งนั้นแล้ว — ในทางปฏิบัติ **ใช้ `ArrayList` เกือบทุกกรณี** และใช้ `ArrayDeque` แทน `LinkedList` สำหรับ queue

### SequencedCollection (Java 21)

```java
List<String> list = new ArrayList<>(List.of("ก", "ข", "ค"));
list.getFirst();     // "ก"
list.getLast();      // "ค"
list.addFirst("0");  // [0, ก, ข, ค]
list.removeLast();   // [0, ก, ข]
list.reversed();     // [ข, ก, 0] — view กลับด้าน
```

---

## 2. Set — ไม่มีค่าซ้ำ

### คุณสมบัติ

- **ไม่เก็บค่าซ้ำ** — `add` ค่าที่มีอยู่แล้วจะคืน `false`
- **ไม่มี index** (`get(i)` ไม่มี)

### ตัวอย่าง

```java
import java.util.*;

Set<String> hash = new HashSet<>(List.of("ส้ม", "กล้วย", "แอปเปิ้ล", "ส้ม"));
Set<String> linked = new LinkedHashSet<>(List.of("ส้ม", "กล้วย", "แอปเปิ้ล", "ส้ม"));
Set<String> tree = new TreeSet<>(List.of("ส้ม", "กล้วย", "แอปเปิ้ล", "ส้ม"));

System.out.println(hash);   // ลำดับไม่แน่นอน
System.out.println(linked); // ผลลัพธ์: [ส้ม, กล้วย, แอปเปิ้ล] — ตามลำดับที่ใส่
System.out.println(tree);   // ผลลัพธ์: [กล้วย, ส้ม, แอปเปิ้ล] — เรียงตาม Unicode

boolean added = hash.add("ส้ม");
System.out.println(added);  // ผลลัพธ์: false — มีอยู่แล้ว
hash.contains("กล้วย");     // true — เร็วมาก O(1)
hash.remove("กล้วย");
```

### Set operations

```java
Set<Integer> a = new HashSet<>(Set.of(1, 2, 3, 4));
Set<Integer> b = Set.of(3, 4, 5);

Set<Integer> union = new HashSet<>(a);
union.addAll(b);            // [1, 2, 3, 4, 5] — ยูเนียน

Set<Integer> inter = new HashSet<>(a);
inter.retainAll(b);         // [3, 4] — อินเตอร์เซกชัน

Set<Integer> diff = new HashSet<>(a);
diff.removeAll(b);          // [1, 2] — ผลต่าง
```

### `TreeSet` — method สำหรับข้อมูลเรียงลำดับ

```java
TreeSet<Integer> scores = new TreeSet<>(List.of(55, 70, 85, 92));
scores.first();          // 55
scores.last();           // 92
scores.ceiling(80);      // 85 — ค่าที่ ≥ 80 ตัวแรก
scores.floor(80);        // 70 — ค่าที่ ≤ 80 ตัวสุดท้าย
scores.headSet(80);      // [55, 70] — น้อยกว่า 80
```

| Implementation  | ลำดับ                   | ความเร็ว (add/contains) |
| --------------- | ----------------------- | ----------------------- |
| `HashSet`       | ไม่แน่นอน               | O(1)                    |
| `LinkedHashSet` | ตามลำดับที่ใส่          | O(1)                    |
| `TreeSet`       | เรียงลำดับ              | O(log n)                |

---

## 3. Map — คู่ key → value

### คุณสมบัติ

- key **ไม่ซ้ำ** — `put` key เดิมจะ **ทับค่าเก่า**
- `get` key ที่ไม่มีจะคืน **`null`**

### ตัวอย่าง

```java
import java.util.*;

Map<String, Integer> stock = new HashMap<>();

stock.put("กาแฟ", 20);
stock.put("ชาเขียว", 15);
stock.put("กาแฟ", 25);             // ทับค่าเก่า

stock.get("กาแฟ");                 // 25
stock.get("โกโก้");                // null
stock.containsKey("ชาเขียว");      // true
stock.containsValue(15);           // true
stock.size();                      // 2
stock.remove("ชาเขียว");

stock.keySet();                    // Set ของ key
stock.values();                    // Collection ของ value
stock.entrySet();                  // Set ของ Map.Entry<K, V>
```

### Method ที่มีประโยชน์มาก

```java
Map<String, Integer> stock = new HashMap<>(Map.of("กาแฟ", 20));

// getOrDefault — คืนค่า default ถ้าไม่มี key
stock.getOrDefault("โกโก้", 0);          // 0

// putIfAbsent — ใส่เฉพาะเมื่อยังไม่มี key
stock.putIfAbsent("กาแฟ", 99);            // ไม่เปลี่ยน (ยังเป็น 20)
stock.putIfAbsent("โกโก้", 10);           // เพิ่ม โกโก้=10

// merge — รวมค่าเดิมกับค่าใหม่
stock.merge("กาแฟ", 5, Integer::sum);     // 20 + 5 = 25
stock.merge("ชาไทย", 5, Integer::sum);    // ไม่มี key → ใส่ 5

// compute / computeIfPresent
stock.computeIfPresent("กาแฟ", (k, v) -> v - 1); // 24
```

### นับจำนวนคำ — ใช้ `merge`

```java
String[] words = {"แมว", "หมา", "แมว", "นก", "แมว"};
Map<String, Integer> count = new TreeMap<>();
for (String w : words) {
    count.merge(w, 1, Integer::sum);
}
System.out.println(count); // ผลลัพธ์: {นก=1, หมา=1, แมว=3} — TreeMap เรียง key ตาม Unicode
```

### จัดกลุ่ม — ใช้ `computeIfAbsent`

```java
Map<String, List<String>> byDept = new HashMap<>();

byDept.computeIfAbsent("IT", k -> new ArrayList<>()).add("สมชาย");
byDept.computeIfAbsent("IT", k -> new ArrayList<>()).add("สมหญิง");
byDept.computeIfAbsent("HR", k -> new ArrayList<>()).add("สมศรี");

System.out.println(byDept.get("IT")); // ผลลัพธ์: [สมชาย, สมหญิง]
```

### วน Map

```java
Map<String, Integer> prices = new LinkedHashMap<>();
prices.put("กาแฟ", 50);
prices.put("ชา", 40);

for (Map.Entry<String, Integer> e : prices.entrySet()) {
    System.out.println(e.getKey() + " = " + e.getValue());
}
prices.forEach((name, price) -> System.out.println(name + " = " + price));
```

| Implementation  | ลำดับ key               | ความเร็ว     | `null` key |
| --------------- | ----------------------- | ------------ | ---------- |
| `HashMap`       | ไม่แน่นอน               | O(1)         | ได้ 1 ตัว  |
| `LinkedHashMap` | ตามลำดับที่ใส่          | O(1)         | ได้ 1 ตัว  |
| `TreeMap`       | เรียงตาม key            | O(log n)     | ไม่ได้     |

> 💡 **เทียบกับ JavaScript:** `Map` ของ Java ≈ `Map` ของ JS (ไม่ใช่ plain object) — ใช้ `get`/`put` แทน `obj[key]` และ `HashMap` **ไม่รักษาลำดับ** (ต่างจาก JS Map ที่รักษาลำดับเสมอ) ถ้าต้องการลำดับให้ใช้ `LinkedHashMap`

---

## 4. Queue / Deque

### `ArrayDeque` — ใช้ได้ทั้ง Stack และ Queue

```java
import java.util.ArrayDeque;
import java.util.Deque;

// ใช้เป็น Queue (FIFO)
Deque<String> queue = new ArrayDeque<>();
queue.offer("ลูกค้า 1");     // เพิ่มท้าย
queue.offer("ลูกค้า 2");
queue.peek();                // "ลูกค้า 1" — ดูหัวแถว (ไม่ลบ)
queue.poll();                // "ลูกค้า 1" — เอาออกจากหัวแถว
queue.poll();                // "ลูกค้า 2"
queue.poll();                // null — คิวว่าง

// ใช้เป็น Stack (LIFO)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);
stack.pop();                 // 3
stack.peek();                // 2
```

### `PriorityQueue` — เอาค่าน้อยสุดออกก่อน

```java
import java.util.Comparator;
import java.util.List;
import java.util.PriorityQueue;

PriorityQueue<Integer> pq = new PriorityQueue<>(List.of(5, 1, 8, 3));
pq.poll();   // 1
pq.poll();   // 3

PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.addAll(List.of(5, 1, 8));
maxHeap.poll(); // 8
```

| การทำงาน       | throw exception ถ้าล้มเหลว | คืนค่าพิเศษ (`null`/`false`) |
| -------------- | -------------------------- | ---------------------------- |
| เพิ่ม          | `add(e)`                   | `offer(e)`                   |
| เอาออก         | `remove()`                 | `poll()`                     |
| ดูหัว          | `element()`                | `peek()`                     |

---

## 5. Immutable Collections — `List.of`, `Set.of`, `Map.of` (Java 9+)

```java
List<String> days = List.of("จันทร์", "อังคาร", "พุธ");
Set<Integer> primes = Set.of(2, 3, 5, 7);
Map<String, Integer> ages = Map.of("สมชาย", 25, "สมหญิง", 30);
Map<String, Integer> many = Map.ofEntries(
        Map.entry("ก", 1),
        Map.entry("ข", 2)
);

// days.add("พฤหัส");        // UnsupportedOperationException
// List.of("a", null);       // NullPointerException — ไม่รับ null
// Set.of(1, 1);             // IllegalArgumentException — ค่าซ้ำ

List<String> copy = List.copyOf(new ArrayList<>(days)); // สำเนาแบบ immutable
List<String> mutable = new ArrayList<>(days);           // ✅ ต้องการแก้ไข → ห่อด้วย ArrayList
```

> `Map.of` รับได้สูงสุด 10 คู่ — มากกว่านั้นใช้ `Map.ofEntries`

---

## 6. `Collections` Utility Class

```java
import java.util.*;

List<Integer> nums = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9));

Collections.sort(nums);                    // [1, 1, 3, 4, 5, 9]
Collections.reverse(nums);                 // [9, 5, 4, 3, 1, 1]
Collections.shuffle(nums);                 // สลับแบบสุ่ม
Collections.max(nums);                     // 9
Collections.min(nums);                     // 1
Collections.frequency(nums, 1);            // 2
Collections.swap(nums, 0, 1);              // สลับตำแหน่ง 0 กับ 1

List<String> readOnly = Collections.unmodifiableList(new ArrayList<>(List.of("ก")));
List<Integer> zeros = Collections.nCopies(3, 0);    // [0, 0, 0]
List<String> none = Collections.emptyList();         // list ว่าง (immutable)
```

---

## 7. Iterator และ `ConcurrentModificationException`

### ปัญหา: ลบสมาชิกระหว่าง for-each

```java
List<String> names = new ArrayList<>(List.of("สมชาย", "สมหญิง", "อนันต์"));

for (String name : names) {
    if (name.startsWith("สม")) {
        // names.remove(name);  // ConcurrentModificationException!
    }
}
```

### วิธีแก้ 1: `Iterator.remove()`

```java
import java.util.Iterator;

Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    if (name.startsWith("สม")) {
        it.remove();         // ✅ ลบผ่าน iterator
    }
}
System.out.println(names);   // ผลลัพธ์: [อนันต์]
```

### วิธีแก้ 2: `removeIf` (แนะนำ)

```java
names.removeIf(name -> name.startsWith("สม"));
```

---

## 8. `equals` และ `hashCode` — สำคัญเมื่อใช้ object ของเราเอง

`HashSet` / `HashMap` ใช้ `hashCode()` หาตำแหน่ง แล้วใช้ `equals()` เทียบ ถ้า class ของเราไม่ override ทั้งสองตัว object ที่ข้อมูลเหมือนกันจะถูกมองว่า **คนละตัว**

```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    static class PersonClass {               // ไม่ override equals/hashCode
        String name;
        PersonClass(String name) { this.name = name; }
    }

    record Person(String name) {}            // record สร้าง equals/hashCode ให้อัตโนมัติ (Java 16+)

    public static void main(String[] args) {
        Set<PersonClass> set1 = new HashSet<>();
        set1.add(new PersonClass("สมชาย"));
        set1.add(new PersonClass("สมชาย"));
        System.out.println(set1.size());     // ผลลัพธ์: 2 ⚠️

        Set<Person> set2 = new HashSet<>();
        set2.add(new Person("สมชาย"));
        set2.add(new Person("สมชาย"));
        System.out.println(set2.size());     // ผลลัพธ์: 1 ✅
    }
}
```

> กฎ: ถ้า override `equals` **ต้อง override `hashCode` ด้วยเสมอ** — ดูเพิ่มใน [Record & Enum](../oop/record-enum.md)

> 💡 **เทียบกับ JavaScript:** `new Set([{a: 1}, {a: 1}]).size` ใน JS ได้ `2` เพราะเทียบ reference เสมอ แต่ Java ให้เรากำหนดเองได้ว่า "เท่ากัน" หมายถึงอะไรผ่าน `equals`/`hashCode`

---

## ตารางเปรียบเทียบกับ JavaScript

| JavaScript                           | Java                                            |
| ------------------------------------ | ----------------------------------------------- |
| `Array` (`[]`)                       | `List` → `ArrayList`                            |
| `arr.push(x)` / `arr.pop()`          | `list.add(x)` / `list.removeLast()` (21)        |
| `arr.length`                         | `list.size()`                                   |
| `arr[i]`                             | `list.get(i)`                                   |
| `Set`                                | `Set` → `HashSet` / `LinkedHashSet`             |
| `set.has(x)`                         | `set.contains(x)`                               |
| `set.size`                           | `set.size()`                                    |
| `Map`                                | `Map` → `LinkedHashMap` (รักษาลำดับเหมือน JS)   |
| `map.set(k, v)` / `map.get(k)`       | `map.put(k, v)` / `map.get(k)`                  |
| `map.has(k)`                         | `map.containsKey(k)`                            |
| `Object` `{ key: value }`            | `Map<String, V>` หรือสร้าง class / `record`     |
| `obj[key] ?? 0`                      | `map.getOrDefault(key, 0)`                      |
| `Object.keys(obj)`                   | `map.keySet()`                                  |
| `Object.entries(obj)`                | `map.entrySet()`                                |
| `Object.freeze([...])`               | `List.of(...)`                                  |
| `arr.shift()` / queue                | `ArrayDeque.poll()`                             |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ประกาศด้วย interface** — `List<String> x = new ArrayList<>();`
2. **ใช้ `ArrayList`, `HashMap`, `HashSet` เป็นค่าเริ่มต้น** — เปลี่ยนเมื่อต้องการลำดับ (`Linked...`) หรือเรียงลำดับ (`Tree...`)
3. **ใช้ `List.of` / `Map.of`** สำหรับข้อมูลที่ไม่เปลี่ยน
4. **ใช้ `getOrDefault`, `merge`, `computeIfAbsent`** แทนการเช็ก `containsKey` แล้ว `put`
5. **อย่าแก้ไข collection ระหว่าง for-each** — ใช้ `removeIf` หรือ `Iterator`
6. **override `equals` + `hashCode` คู่กัน** (หรือใช้ `record`) เมื่อใช้ object เป็น key/สมาชิกของ Set
7. **ใช้ `ArrayDeque`** แทน `Stack` และ `LinkedList` (class รุ่นเก่า)
8. **คืน collection ว่าง แทน `null`** — `List.of()` / `Collections.emptyList()`

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<String> orders = List.of("กาแฟ", "ชา", "กาแฟ", "โกโก้", "กาแฟ");

        Map<String, Integer> summary = new LinkedHashMap<>();
        for (String item : orders) {
            summary.merge(item, 1, Integer::sum);
        }

        Set<String> uniqueItems = new TreeSet<>(orders);

        System.out.println(summary);     // ผลลัพธ์: {กาแฟ=3, ชา=1, โกโก้=1}
        System.out.println(uniqueItems); // ผลลัพธ์: [กาแฟ, ชา, โกโก้]
    }
}
```

---
