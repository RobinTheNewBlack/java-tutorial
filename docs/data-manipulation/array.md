# Array

# ส่วนที่ 1: Array ใน Java

## Array คืออะไร?

Array คือโครงสร้างข้อมูลที่เก็บค่า **ชนิดเดียวกัน** หลายค่าเรียงต่อกัน เข้าถึงด้วย index (เริ่มที่ 0)

### คุณสมบัติ

- **ขนาดคงที่ (fixed size)** — กำหนดตอนสร้าง แล้วเปลี่ยนไม่ได้
- **ชนิดเดียวกันทั้งหมด** — `int[]` เก็บได้แค่ `int`
- เป็น **reference type** (object)
- เข้าถึง index เกินช่วง → `ArrayIndexOutOfBoundsException`

> 💡 **เทียบกับ JavaScript:** Array ของ JS คือ list ที่ยืดหดได้ เก็บได้ทุก type และมี method มากมาย (`push`, `map`, `filter`) แต่ array ของ Java **ขนาดคงที่และไม่มี method ในตัวเลย** — สิ่งที่เทียบเท่า Array ของ JS จริง ๆ คือ **`ArrayList`** (ดู [Collections](collections.md))

---

## 1. การประกาศและสร้าง Array

### ตัวอย่าง

```java
// แบบที่ 1: กำหนดค่าเริ่มต้นเลย (array initializer)
int[] scores = {90, 85, 70};
String[] names = {"สมชาย", "สมหญิง", "สมศรี"};

// แบบที่ 2: ระบุขนาด — สมาชิกได้ค่า default (0, false, null)
int[] empty = new int[5];          // [0, 0, 0, 0, 0]
String[] slots = new String[3];    // [null, null, null]
boolean[] flags = new boolean[2];  // [false, false]

// แบบที่ 3: new + initializer (ใช้เมื่อส่งเป็น argument ทันที)
printAll(new String[]{"ก", "ข", "ค"});

// syntax แบบ C (ได้ แต่ไม่นิยม)
int legacy[] = {1, 2, 3};
```

### เข้าถึงและแก้ไขสมาชิก

```java
String[] fruits = {"มะม่วง", "ทุเรียน", "มังคุด"};

System.out.println(fruits[0]);                 // ผลลัพธ์: มะม่วง
System.out.println(fruits[fruits.length - 1]); // ผลลัพธ์: มังคุด (ตัวสุดท้าย)
System.out.println(fruits.length);             // ผลลัพธ์: 3 (field ไม่ใช่ method — ไม่มีวงเล็บ)

fruits[1] = "เงาะ";                            // แก้ไขได้
// fruits[3] = "ลำไย";                          // ArrayIndexOutOfBoundsException
// fruits[-1];                                  // ArrayIndexOutOfBoundsException (ไม่มี index ติดลบ)
```

### พิมพ์ array

```java
import java.util.Arrays;

int[] nums = {3, 1, 2};
System.out.println(nums);                   // ผลลัพธ์: [I@1b6d3586 ⚠️ (พิมพ์ reference)
System.out.println(Arrays.toString(nums));  // ผลลัพธ์: [3, 1, 2] ✅
```

> 💡 **เทียบกับ JavaScript:** `arr[99]` ใน JS คืน `undefined` แต่ใน Java จะ **throw exception** และ `arr.at(-1)` ไม่มีใน array ของ Java

---

## 2. Multi-dimensional Array

Array ซ้อน array — ใช้แทนตาราง เช่น matrix, ตารางที่นั่ง

### ตัวอย่าง

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6}
};

System.out.println(matrix[1][2]);       // ผลลัพธ์: 6 (แถว 1, คอลัมน์ 2)
System.out.println(matrix.length);      // ผลลัพธ์: 2 (จำนวนแถว)
System.out.println(matrix[0].length);   // ผลลัพธ์: 3 (จำนวนคอลัมน์)

int[][] grid = new int[3][4];           // 3 แถว 4 คอลัมน์ ค่าเป็น 0 ทั้งหมด

System.out.println(java.util.Arrays.deepToString(matrix)); // ผลลัพธ์: [[1, 2, 3], [4, 5, 6]]
```

### วน 2 มิติ

```java
for (int[] row : matrix) {
    for (int value : row) {
        System.out.print(value + " ");
    }
    System.out.println();
}
// ผลลัพธ์:
// 1 2 3
// 4 5 6
```

### Jagged array — แต่ละแถวยาวไม่เท่ากัน

```java
int[][] triangle = new int[3][];
triangle[0] = new int[]{1};
triangle[1] = new int[]{1, 1};
triangle[2] = new int[]{1, 2, 1};
```

---

## 3. `Arrays` Utility Class

`java.util.Arrays` รวม static method สำหรับจัดการ array

### `toString` / `deepToString`

```java
import java.util.Arrays;

System.out.println(Arrays.toString(new int[]{1, 2, 3}));       // ผลลัพธ์: [1, 2, 3]
System.out.println(Arrays.deepToString(new int[][]{{1}, {2}})); // ผลลัพธ์: [[1], [2]]
```

### `sort` — เรียงลำดับ (แก้ไข array เดิม)

```java
int[] nums = {5, 2, 8, 1};
Arrays.sort(nums);
System.out.println(Arrays.toString(nums));   // ผลลัพธ์: [1, 2, 5, 8]

String[] names = {"สมหญิง", "สมชาย", "อนันต์"};
Arrays.sort(names);
System.out.println(Arrays.toString(names));  // ผลลัพธ์: [สมชาย, สมหญิง, อนันต์]

// เรียงมากไปน้อย — ต้องเป็น object array (Integer[]) ไม่ใช่ int[]
Integer[] boxed = {5, 2, 8, 1};
Arrays.sort(boxed, java.util.Collections.reverseOrder());
System.out.println(Arrays.toString(boxed));  // ผลลัพธ์: [8, 5, 2, 1]

// เรียงตามความยาว
String[] words = {"ช้าง", "มด", "กระต่าย"};
Arrays.sort(words, java.util.Comparator.comparingInt(String::length));
System.out.println(Arrays.toString(words));  // ผลลัพธ์: [มด, ช้าง, กระต่าย]
```

> 💡 **เทียบกับ JavaScript:** `[10, 9, 1].sort()` ใน JS เรียงแบบ string ได้ `[1, 10, 9]` แต่ `Arrays.sort` ของ Java เรียงตัวเลขตามค่าจริงเลย ไม่ต้องส่ง comparator

### `binarySearch` — ค้นหาใน array ที่ **เรียงแล้ว**

```java
int[] sorted = {1, 3, 5, 7, 9};
System.out.println(Arrays.binarySearch(sorted, 7));  // ผลลัพธ์: 3 (index)
System.out.println(Arrays.binarySearch(sorted, 4));  // ผลลัพธ์: -3 (ไม่เจอ → ค่าติดลบ)
```

### `fill` — เติมค่าเดียวกันทั้งหมด

```java
int[] seats = new int[5];
Arrays.fill(seats, -1);
System.out.println(Arrays.toString(seats)); // ผลลัพธ์: [-1, -1, -1, -1, -1]
```

### `copyOf` / `copyOfRange` — คัดลอก (และขยายขนาด)

```java
int[] original = {1, 2, 3};

int[] bigger = Arrays.copyOf(original, 5);          // [1, 2, 3, 0, 0]
int[] smaller = Arrays.copyOf(original, 2);         // [1, 2]
int[] middle = Arrays.copyOfRange(original, 1, 3);  // [2, 3] (end ไม่รวม)
int[] clone = original.clone();                     // คัดลอกทั้งหมด
```

### `equals` — เทียบเนื้อหา

```java
int[] a = {1, 2, 3};
int[] b = {1, 2, 3};

System.out.println(a == b);              // ผลลัพธ์: false (คนละ object)
System.out.println(a.equals(b));         // ผลลัพธ์: false ⚠️ array ไม่ override equals
System.out.println(Arrays.equals(a, b)); // ผลลัพธ์: true ✅
```

### `asList` — แปลงเป็น `List` (ขนาดคงที่)

```java
import java.util.List;

String[] arr = {"ก", "ข", "ค"};
List<String> list = Arrays.asList(arr);

list.set(0, "ง");          // ได้ — และ arr[0] เปลี่ยนตามด้วย (ใช้ข้อมูลร่วมกัน)
System.out.println(arr[0]); // ผลลัพธ์: ง
// list.add("จ");           // UnsupportedOperationException — ขนาดคงที่

List<String> mutable = new java.util.ArrayList<>(Arrays.asList(arr)); // ✅ ถ้าต้องการเพิ่ม/ลบ
```

### `stream` — ใช้ Stream API กับ array

```java
int[] scores = {80, 92, 75, 60};

int sum = Arrays.stream(scores).sum();                       // 307
double avg = Arrays.stream(scores).average().orElse(0);      // 76.75
int max = Arrays.stream(scores).max().getAsInt();            // 92
int[] passed = Arrays.stream(scores).filter(s -> s >= 75).toArray(); // [80, 92, 75]
```

---

## 4. Varargs — รับ argument จำนวนไม่จำกัด

`Type... name` ทำให้ method รับ argument กี่ตัวก็ได้ ภายใน method จะได้เป็น **array**

```java
public class Main {
    static int sum(int... numbers) {          // numbers เป็น int[]
        int total = 0;
        for (int n : numbers) {
            total += n;
        }
        return total;
    }

    public static void main(String[] args) {
        System.out.println(sum());             // ผลลัพธ์: 0
        System.out.println(sum(1, 2, 3));      // ผลลัพธ์: 6
        System.out.println(sum(new int[]{4, 5})); // ผลลัพธ์: 9 — ส่ง array ได้ด้วย
    }
}
```

- varargs ต้องเป็น **parameter ตัวสุดท้าย** และมีได้ตัวเดียว
- ตัวอย่างใน JDK: `String.format(String, Object...)`, `List.of(E...)`

> 💡 **เทียบกับ JavaScript:** `int... numbers` ≈ rest parameter `...numbers` ของ JS แต่ Java ไม่มี spread operator ตอนเรียก (`sum(...arr)`) — ส่ง array ตรง ๆ แทน

---

## 5. Array vs ArrayList

ในงานจริงส่วนใหญ่ใช้ `ArrayList` มากกว่า array เพราะยืดหดได้

```java
import java.util.ArrayList;
import java.util.List;

// Array — ขนาดคงที่
String[] arr = new String[2];
arr[0] = "สมชาย";
arr[1] = "สมหญิง";
// ต้องการเพิ่มคนที่ 3 → ต้องสร้าง array ใหม่ที่ใหญ่กว่าแล้ว copy

// ArrayList — ยืดหดได้
List<String> list = new ArrayList<>();
list.add("สมชาย");
list.add("สมหญิง");
list.add("สมศรี");          // เพิ่มได้เรื่อย ๆ
list.remove("สมหญิง");
System.out.println(list);   // ผลลัพธ์: [สมชาย, สมศรี]
```

### แปลงไปมา

```java
String[] arr = {"ก", "ข"};
List<String> list = new ArrayList<>(List.of(arr));   // array → ArrayList
String[] back = list.toArray(new String[0]);         // List → array
String[] back2 = list.toArray(String[]::new);        // Java 11+

int[] primitives = {1, 2, 3};
List<Integer> boxed = Arrays.stream(primitives).boxed().toList(); // int[] → List<Integer> (Java 16+)
```

### ตารางเปรียบเทียบ

| คุณสมบัติ          | Array (`int[]`)          | `ArrayList<Integer>`         |
| ------------------ | ------------------------ | ---------------------------- |
| ขนาด               | คงที่                    | ยืดหดได้                     |
| เก็บ primitive     | ได้ (`int[]`)            | ไม่ได้ (ต้องใช้ `Integer`)   |
| ความยาว            | `arr.length`             | `list.size()`                |
| เข้าถึง            | `arr[i]`                 | `list.get(i)`                |
| แก้ไข              | `arr[i] = x`             | `list.set(i, x)`             |
| เพิ่ม / ลบ         | ไม่ได้                   | `add()`, `remove()`          |
| พิมพ์              | `Arrays.toString(arr)`   | `System.out.println(list)`   |
| ประสิทธิภาพ        | เร็วกว่าเล็กน้อย          | สะดวกกว่า                    |
| Generic            | ไม่เหมาะ                 | รองรับเต็มที่                |

---

## ตารางเปรียบเทียบกับ JavaScript

| JavaScript                    | Java array                          | Java `ArrayList`                 |
| ----------------------------- | ----------------------------------- | -------------------------------- |
| `const a = [1, 2, 3]`         | `int[] a = {1, 2, 3};`              | `new ArrayList<>(List.of(1,2,3))` |
| `a.length`                    | `a.length`                          | `list.size()`                    |
| `a.push(x)`                   | ไม่ได้                              | `list.add(x)`                    |
| `a.sort((x, y) => x - y)`     | `Arrays.sort(a)`                    | `list.sort(null)` / `Collections.sort(list)` |
| `a.includes(x)`               | `Arrays.stream(a).anyMatch(v -> v == x)` | `list.contains(x)`          |
| `a.indexOf(x)`                | วน loop / `binarySearch` (ถ้าเรียงแล้ว) | `list.indexOf(x)`             |
| `[...a]` / `a.slice()`        | `a.clone()` / `Arrays.copyOf`       | `new ArrayList<>(list)`          |
| `new Array(5).fill(0)`        | `new int[5]` (เป็น 0 อยู่แล้ว)       | `new ArrayList<>(Collections.nCopies(5, 0))` |
| `console.log(a)`              | `Arrays.toString(a)`                | `System.out.println(list)`       |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ `ArrayList` เป็นหลัก** — ใช้ array เมื่อขนาดคงที่แน่นอน, ต้องการประสิทธิภาพสูง หรือทำงานกับ primitive จำนวนมาก
2. **พิมพ์ array ด้วย `Arrays.toString`** ไม่ใช่ `println(arr)` ตรง ๆ
3. **เทียบ array ด้วย `Arrays.equals`** ไม่ใช่ `==` หรือ `.equals()`
4. **ระวัง `Arrays.asList`** — ขนาดคงที่และแชร์ข้อมูลกับ array เดิม
5. **ใช้ for-each** เมื่อไม่ต้องการ index เพื่อเลี่ยง `ArrayIndexOutOfBoundsException`
6. **คืน array ว่าง (`new int[0]`) แทน `null`** เมื่อไม่มีข้อมูล

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] scores = {72, 95, 88, 60, 79};

        int[] sorted = Arrays.copyOf(scores, scores.length); // ไม่แก้ array เดิม
        Arrays.sort(sorted);

        System.out.println("เดิม: " + Arrays.toString(scores));  // ผลลัพธ์: เดิม: [72, 95, 88, 60, 79]
        System.out.println("เรียง: " + Arrays.toString(sorted)); // ผลลัพธ์: เรียง: [60, 72, 79, 88, 95]
        System.out.println("สูงสุด: " + sorted[sorted.length - 1]); // ผลลัพธ์: สูงสุด: 95
    }
}
```

---
