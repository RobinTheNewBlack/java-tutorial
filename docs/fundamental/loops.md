# Java Fundamentals

# ส่วนที่ 5: การวนซ้ำ (Loops)

## Loop คืออะไร?

Loop ใช้ทำงานซ้ำ ๆ ตามเงื่อนไข Java มี `for`, enhanced `for` (for-each), `while`, `do-while` และยังวนผ่าน collection ด้วย `forEach` + lambda หรือ Stream ได้

> 💡 **เทียบกับ JavaScript:** syntax ของ `for`, `while`, `do-while` เหมือน JS เกือบทุกตัวอักษร ต่างกันที่ต้องระบุ type ของตัวแปร (`int i` แทน `let i`) และเงื่อนไขต้องเป็น `boolean`

---

## 1. `for` Loop

### คุณสมบัติ

- เหมาะเมื่อ **รู้จำนวนรอบ** หรือต้องการใช้ **index**
- โครงสร้าง: `for (เริ่มต้น; เงื่อนไข; อัปเดต)`
- ตัวแปรที่ประกาศใน `for` มี scope อยู่ใน loop เท่านั้น

### ตัวอย่าง

```java
for (int i = 1; i <= 5; i++) {
    System.out.print(i + " ");
}
// ผลลัพธ์: 1 2 3 4 5
```

### วนถอยหลัง / ข้ามทีละหลายค่า

```java
for (int i = 10; i > 0; i -= 3) {
    System.out.print(i + " ");
}
// ผลลัพธ์: 10 7 4 1
```

### วน array ด้วย index

```java
String[] names = {"สมชาย", "สมหญิง", "สมศรี"};

for (int i = 0; i < names.length; i++) {   // array ใช้ .length (ไม่มีวงเล็บ)
    System.out.println(i + ": " + names[i]);
}
// ผลลัพธ์:
// 0: สมชาย
// 1: สมหญิง
// 2: สมศรี
```

### หลายตัวแปรใน `for`

```java
for (int i = 0, j = 10; i < j; i++, j--) {
    System.out.print(i + "-" + j + " ");
}
// ผลลัพธ์: 0-10 1-9 2-8 3-7 4-6
```

---

## 2. Enhanced `for` (for-each)

วนผ่าน **ทุกสมาชิก** ของ array หรือ collection ที่เป็น `Iterable`

### คุณสมบัติ

- อ่านง่ายกว่า — ไม่ต้องจัดการ index
- **ไม่มี index** ให้ใช้
- **ห้ามเพิ่ม/ลบสมาชิก** ของ collection ระหว่างวน (จะเกิด `ConcurrentModificationException`)
- การกำหนดค่าให้ตัวแปร loop **ไม่ได้แก้ไขข้อมูลต้นทาง**

### ตัวอย่าง

```java
import java.util.List;

int[] scores = {80, 92, 75};
int total = 0;
for (int score : scores) {
    total += score;
}
System.out.println(total); // ผลลัพธ์: 247

List<String> fruits = List.of("มะม่วง", "ทุเรียน", "มังคุด");
for (String fruit : fruits) {
    System.out.println(fruit);
}

// ใช้ var ได้ (Java 10+)
for (var fruit : fruits) {
    System.out.println(fruit.length());
}
```

### กำหนดค่าให้ตัวแปร loop ไม่มีผลกับ array

```java
int[] nums = {1, 2, 3};
for (int n : nums) {
    n = n * 10;          // แก้แค่ตัวแปร n ภายใน loop
}
System.out.println(java.util.Arrays.toString(nums)); // ผลลัพธ์: [1, 2, 3]

for (int i = 0; i < nums.length; i++) {
    nums[i] = nums[i] * 10; // ✅ ต้องใช้ index ถ้าจะแก้ไข
}
System.out.println(java.util.Arrays.toString(nums)); // ผลลัพธ์: [10, 20, 30]
```

> 💡 **เทียบกับ JavaScript:** `for (String s : list)` ≈ `for (const s of list)` ของ JS — Java ใช้ `:` แทน `of` และ **ไม่มี `for...in`** สำหรับวน key ของ object (ใช้ `map.keySet()` / `map.entrySet()` แทน)

---

## 3. `while` Loop

ตรวจเงื่อนไข **ก่อน** ทำงาน — อาจไม่ทำงานเลยสักรอบ

### ตัวอย่าง

```java
int balance = 1000;
int month = 0;

while (balance > 0) {
    balance -= 300;
    month++;
}
System.out.println("เงินหมดในเดือนที่ " + month); // ผลลัพธ์: เงินหมดในเดือนที่ 4
```

### Infinite loop + `break`

```java
import java.util.Scanner;

Scanner scanner = new Scanner(System.in);
while (true) {
    System.out.print("พิมพ์คำสั่ง (exit เพื่อออก): ");
    String input = scanner.nextLine();
    if ("exit".equals(input)) {
        break;
    }
    System.out.println("คุณพิมพ์: " + input);
}
```

---

## 4. `do-while` Loop

ทำงาน **ก่อน 1 รอบ** แล้วค่อยตรวจเงื่อนไข

```java
int attempt = 0;
do {
    attempt++;
    System.out.println("พยายามครั้งที่ " + attempt);
} while (attempt < 3);
// ผลลัพธ์:
// พยายามครั้งที่ 1
// พยายามครั้งที่ 2
// พยายามครั้งที่ 3

int x = 100;
do {
    System.out.println("ทำงาน 1 รอบแม้เงื่อนไขเป็น false");
} while (x < 10);
```

---

## 5. `break` และ `continue`

| คำสั่ง     | ความหมาย                                  |
| ---------- | ----------------------------------------- |
| `break`    | ออกจาก loop ทันที                         |
| `continue` | ข้ามรอบปัจจุบัน ไปเริ่มรอบถัดไป           |

```java
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
        continue;       // ข้ามเลขคู่
    }
    if (i > 7) {
        break;          // หยุดเมื่อเกิน 7
    }
    System.out.print(i + " ");
}
// ผลลัพธ์: 1 3 5 7
```

---

## 6. Labeled `break` / `continue`

ใช้กับ **loop ซ้อน** เพื่อ break/continue loop ด้านนอกโดยตรง

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};
int target = 5;

outer:
for (int row = 0; row < matrix.length; row++) {
    for (int col = 0; col < matrix[row].length; col++) {
        if (matrix[row][col] == target) {
            System.out.println("เจอที่ [" + row + "][" + col + "]");
            break outer;    // ออกจากทั้ง 2 loop
        }
    }
}
// ผลลัพธ์: เจอที่ [1][1]
```

```java
outer:
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {
        if (j == 2) {
            continue outer; // ข้ามไปรอบถัดไปของ loop นอก
        }
        System.out.print(i + "" + j + " ");
    }
}
// ผลลัพธ์: 11 21 31
```

> 💡 **เทียบกับ JavaScript:** JS ก็มี labeled statement เหมือนกัน (`outer: for (...) { break outer; }`) syntax เหมือนกันทุกอย่าง

---

## 7. วนผ่าน Collections

### List

```java
import java.util.List;

List<String> names = List.of("สมชาย", "สมหญิง", "สมศรี");

for (String name : names) {
    System.out.println(name);
}

for (int i = 0; i < names.size(); i++) {      // List ใช้ .size()
    System.out.println(i + ": " + names.get(i)); // และ .get(i)
}
```

### Map

```java
import java.util.LinkedHashMap;
import java.util.Map;

Map<String, Integer> stock = new LinkedHashMap<>();
stock.put("กาแฟ", 20);
stock.put("ชาเขียว", 15);

for (Map.Entry<String, Integer> entry : stock.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

for (var entry : stock.entrySet()) {          // ใช้ var ให้สั้นลง
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

for (String key : stock.keySet()) { }          // วนเฉพาะ key
for (int qty : stock.values()) { }             // วนเฉพาะ value
```

### ลบสมาชิกระหว่างวนอย่างปลอดภัย

```java
import java.util.ArrayList;
import java.util.List;

List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5));

// for (Integer n : nums) { if (n % 2 == 0) nums.remove(n); } // ConcurrentModificationException

nums.removeIf(n -> n % 2 == 0);   // ✅
System.out.println(nums);         // ผลลัพธ์: [1, 3, 5]
```

---

## 8. `forEach` กับ Lambda (Java 8+)

```java
import java.util.List;
import java.util.Map;

List<String> fruits = List.of("มะม่วง", "ทุเรียน");
fruits.forEach(fruit -> System.out.println(fruit));
fruits.forEach(System.out::println);           // method reference — สั้นกว่า

Map<String, Integer> prices = Map.of("กาแฟ", 50);
prices.forEach((name, price) -> System.out.println(name + ": " + price));
// ผลลัพธ์: กาแฟ: 50
```

### วนตามช่วงตัวเลขด้วย `IntStream`

```java
import java.util.stream.IntStream;

IntStream.range(0, 5).forEach(i -> System.out.print(i + " "));       // ผลลัพธ์: 0 1 2 3 4
IntStream.rangeClosed(1, 5).forEach(i -> System.out.print(i + " ")); // ผลลัพธ์: 1 2 3 4 5
```

### ข้อจำกัดของ lambda ใน `forEach`

```java
int total = 0;
List<Integer> nums = List.of(1, 2, 3);
// nums.forEach(n -> total += n); // Error: local variables referenced from a lambda must be final or effectively final

int sum = nums.stream().mapToInt(Integer::intValue).sum(); // ✅ ใช้ stream แทน
for (int n : nums) total += n;                             // ✅ หรือ for-each ธรรมดา
```

- ใน `forEach` ใช้ `break` / `continue` **ไม่ได้** (ใช้ `return` เพื่อข้ามรอบปัจจุบันเท่านั้น)

> 💡 **เทียบกับ JavaScript:** `list.forEach(x -> ...)` ≈ `arr.forEach(x => ...)` — ใช้ `->` แทน `=>` แต่ lambda ใน Java **แก้ไขตัวแปร local ด้านนอกไม่ได้** ต่างจาก closure ของ JS

ดูเพิ่มเติมใน [Lambda & Functional Interface](../method/lambda-functional-interface.md) และ [Stream API](../data-manipulation/stream-api.md)

---

## ตารางเปรียบเทียบกับ JavaScript

| JavaScript                          | Java                                        |
| ----------------------------------- | ------------------------------------------- |
| `for (let i = 0; i < n; i++)`       | `for (int i = 0; i < n; i++)`               |
| `for (const x of arr)`              | `for (Type x : arr)` / `for (var x : arr)`  |
| `for (const key in obj)`            | `for (String key : map.keySet())`           |
| `for (const [k, v] of Object.entries(obj))` | `for (var e : map.entrySet())`      |
| `arr.forEach(x => ...)`             | `list.forEach(x -> ...)`                    |
| `arr.length`                        | `arr.length` (array) / `list.size()`        |
| `while`, `do...while`               | เหมือนกัน                                   |
| labeled `break` / `continue`        | เหมือนกัน                                   |

---

## ตารางเปรียบเทียบ Loop แต่ละแบบ

| Loop         | เหมาะกับ                                  | มี index | แก้ไขระหว่างวน   |
| ------------ | ----------------------------------------- | -------- | ---------------- |
| `for`        | รู้จำนวนรอบ, ต้องการ index                 | มี       | ได้ (ระวัง)      |
| for-each     | อ่านทุกสมาชิก                              | ไม่มี    | ไม่ได้           |
| `while`      | ไม่รู้จำนวนรอบ                             | -        | -                |
| `do-while`   | ต้องทำอย่างน้อย 1 รอบ (เช่น เมนู, retry)   | -        | -                |
| `forEach`    | ทำงานสั้น ๆ กับแต่ละสมาชิก                 | ไม่มี    | ไม่ได้           |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ for-each เป็นหลัก** เมื่อไม่ต้องการ index — อ่านง่ายและไม่มี off-by-one error
2. **ใช้ `removeIf`** หรือ `Iterator.remove()` เมื่อต้องลบสมาชิกระหว่างวน
3. **ระวัง infinite loop** — ตรวจให้แน่ใจว่าเงื่อนไขจะเป็น `false` ได้
4. **ใช้ labeled break** เมื่อต้องออกจาก loop ซ้อน แทนการใช้ flag
5. **ใช้ Stream API** เมื่อเป็นการแปลง/กรอง/สรุปข้อมูล แทนการเขียน loop สะสมค่าเอง

```java
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Integer> prices = List.of(120, 80, 250, 40);
        int total = 0;

        for (int price : prices) {
            if (price < 50) {
                continue; // ข้ามสินค้าราคาต่ำกว่า 50
            }
            total += price;
        }

        System.out.println("ยอดรวม: " + total); // ผลลัพธ์: ยอดรวม: 450
    }
}
```

---
