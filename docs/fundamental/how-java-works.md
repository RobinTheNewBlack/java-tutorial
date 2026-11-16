# How Java Works (เบื้องหลังการทำงานของ Java)

# ส่วนที่ 0.5: เจาะลึก Compile, Bytecode และ JVM

> อ่านต่อจาก [Setup & Hello World](setup-and-hello-world.md) — ไฟล์นี้ไม่จำเป็นต่อการเขียนโค้ดวันแรก แต่จะช่วยให้เข้าใจว่า **"ทำไม Java ถึงเป็นแบบนี้"** และอ่าน error แปลก ๆ ออก

## ภาพรวมทั้งหมด

```
                 ┌───────── Compile time ─────────┐   ┌──────────────────── Runtime (JVM) ─────────────────────┐
                 │                                │   │                                                       │
 Main.java ──► javac ──► Main.class (bytecode) ──────► Class Loader ──► Verifier ──► Execution Engine ──► CPU
 (source)        │       ไฟล์กลาง ไม่ผูกกับ OS     │   │                                 ├─ Interpreter        │
                 │                                │   │                                 └─ JIT Compiler       │
                 └────────────────────────────────┘   │        Memory: Heap / Stack / Metaspace + GC          │
                                                      └───────────────────────────────────────────────────────┘
```

Java มีการ "แปล" **สองรอบ**:

1. **`javac`** แปล source code → **bytecode** (ภาษาเครื่องของ "เครื่องสมมติ" ชื่อ JVM) — ทำครั้งเดียวตอน build
2. **JVM** แปล bytecode → **machine code** ของ CPU จริง (x86, ARM ...) — ทำตอนรัน

เพราะ bytecode ไม่ผูกกับ OS/CPU ไฟล์ `.class` ชุดเดียวจึงรันได้ทุกที่ที่มี JVM → **"Write once, run anywhere"**

> 💡 **เทียบกับ JavaScript:** V8 ก็ทำคล้ายกันมาก — parse JS → bytecode (Ignition) → JIT เป็น machine code (TurboFan) แต่ทั้งหมดเกิด **ตอนรัน** และเก็บใน memory เท่านั้น ส่วน Java แยกขั้นแรกออกมาทำล่วงหน้า และได้ไฟล์ `.class` ที่แจกจ่ายได้

---

## 1. ขั้นตอน Compile: `javac` ทำอะไรบ้าง?

| ขั้น | ชื่อ                    | สิ่งที่เกิดขึ้น                                                              |
| ---- | ----------------------- | ---------------------------------------------------------------------------- |
| 1    | Parse                   | อ่านโค้ด → แตกเป็น token → สร้าง **AST** (Abstract Syntax Tree)              |
| 2    | Enter / Resolve         | สร้างตารางสัญลักษณ์ (class, method, ตัวแปร) และหาว่าชื่อแต่ละชื่ออ้างถึงอะไร |
| 3    | **Type check (Attr)**   | ตรวจ type ทุกจุด — `int x = "abc";` ตายตรงนี้                                |
| 4    | Flow analysis           | ตรวจว่าตัวแปรถูกกำหนดค่าก่อนใช้, method ที่มี return type return ครบทุกทาง   |
| 5    | **Desugar (Lower)**     | แปลง syntax ระดับสูงให้เป็นแบบพื้นฐาน (ดูหัวข้อ 3)                           |
| 6    | Generate                | เขียน bytecode ลงไฟล์ `.class` — **1 class = 1 ไฟล์** (inner class ก็แยกไฟล์ เช่น `Outer$Inner.class`) |

```java
public class Main {
    static int check(int x) {
        if (x > 0) {
            return 1;
        }
        // ลืม return ตรงนี้
    }
}
```

```
Main.java:6: error: missing return statement
```

error นี้มาจากขั้น **Flow analysis** — โปรแกรม **ไม่ถูกรันเลยสักบรรทัด**

> 💡 **เกร็ด:** `javac` เขียนด้วยภาษา Java เอง (compiler ที่ compile ตัวเองได้เรียกว่า *self-hosting*) และ JDK มี API `javax.tools.ToolProvider.getSystemJavaCompiler()` ให้เรียก compiler จากในโค้ด Java ได้ — `java Main.java` (single-file launch) ก็ใช้วิธีนี้ compile ใน memory

---

## 2. Bytecode หน้าตาเป็นอย่างไร?

### ตัวอย่างโค้ด

```java
public class Add {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;
        int sum = a + b;
        System.out.println(sum);
    }
}
```

### ดู bytecode ด้วย `javap -c`

```powershell
javac Add.java
javap -c Add
```

```
public static void main(java.lang.String[]);
  Code:
     0: bipush        10      // push ค่า 10 ลง operand stack
     2: istore_1              // pop ไปเก็บใน local variable ช่อง 1 (a)
     3: bipush        20
     5: istore_2              // b
     6: iload_1               // push a
     7: iload_2               // push b
     8: iadd                  // pop 2 ตัว บวกกัน push ผลลัพธ์
     9: istore_3              // sum
    10: getstatic     #2      // Field java/lang/System.out:Ljava/io/PrintStream;
    13: iload_3
    14: invokevirtual #3      // Method java/io/PrintStream.println:(I)V
    17: return
```

> หมายเลข `#2`, `#3` คือตำแหน่งใน **constant pool** ของไฟล์ อาจต่างกันไปตามเวอร์ชัน JDK

### สิ่งที่น่าสังเกต

- JVM เป็น **stack-based machine** — ไม่มี register ทุกคำสั่งทำงานผ่าน *operand stack* (`push` → คำนวณ → `pop`)
- ชื่อตัวแปร `a`, `b`, `sum` **หายไป** เหลือแค่หมายเลขช่อง (`_1`, `_2`, `_3`) — ช่อง 0 คือ `args`
- prefix บอก type: `i` = int, `l` = long, `d` = double, `a` = reference (object) → type ถูก "ฝัง" อยู่ในคำสั่งเลย
- แม้เราไม่ได้เขียน constructor, `javac` ก็สร้าง `public Add()` ให้อัตโนมัติ (default constructor)
- `(I)V` คือ **method descriptor** = รับ `int` คืน `void`

### `javap` ที่ใช้บ่อย

| คำสั่ง              | แสดงอะไร                                                     |
| ------------------- | ------------------------------------------------------------ |
| `javap Add`         | รายชื่อ method / field (public)                              |
| `javap -p Add`      | รวม private ด้วย                                             |
| `javap -c Add`      | bytecode ของแต่ละ method                                     |
| `javap -v Add`      | ทุกอย่าง: constant pool, version, stack size, line number   |
| `javap java.lang.String` | ดู method ของ class ใน JDK ได้ด้วย                     |

### เกร็ด: `CAFEBABE` และ class file version

ไฟล์ `.class` ทุกไฟล์ขึ้นต้นด้วย 4 byte เดียวกัน คือ **`CA FE BA BE`** (magic number ที่ James Gosling เลือกไว้เล่น ๆ ช่วงทีมชอบไปร้านกาแฟ) ตามด้วยเลข **major version**:

```
00000000: cafe babe 0000 0037 ...     ← 0x37 = 55 = Java 11
```

| Java  | major version |
| ----- | ------------- |
| 8     | 52            |
| 11    | 55            |
| 17    | 61            |
| 21    | 65            |

ถ้า compile ด้วย JDK 21 แล้วเอาไปรันบน JVM 11 จะเจอ error ที่ดูน่ากลัวนี้:

```
java.lang.UnsupportedClassVersionError: Main has been compiled by a more recent version
of the Java Runtime (class file version 65.0), this version of the Java Runtime only
recognizes class file versions up to 55.0
```

แปลว่า **"`javac` ใหม่กว่า `java`"** — แก้ได้โดยให้ `java` เป็นเวอร์ชันเดียวกัน หรือ compile ด้วย `javac --release 11 Main.java` เพื่อสร้าง bytecode ที่ JVM 11 อ่านออก

> 💡 **เทียบกับ JavaScript:** คล้ายเขียน syntax ใหม่แล้วไปรันบน Node รุ่นเก่า — ฝั่ง JS แก้ด้วย Babel/`tsconfig` `target` ฝั่ง Java ใช้ `--release`

---

## 3. Syntactic Sugar — สิ่งที่ `javac` แอบแปลงให้

ฟีเจอร์หลายอย่างใน Java **ไม่มีอยู่จริงใน JVM** — `javac` แปลงเป็นโค้ดพื้นฐานให้ก่อน

| เขียนแบบนี้                          | `javac` แปลงเป็นประมาณนี้                                         |
| ------------------------------------ | ----------------------------------------------------------------- |
| `for (String s : list)`              | `Iterator<String> it = list.iterator(); while (it.hasNext()) ...` |
| `Integer x = 5;` (autoboxing)        | `Integer x = Integer.valueOf(5);`                                 |
| `int y = x;` (unboxing)              | `int y = x.intValue();` ← ถ้า `x` เป็น `null` ได้ `NullPointerException` |
| `List<String>` (generics)            | `List` เฉย ๆ + แทรก cast `(String)` ให้ตอนดึงค่า (**type erasure**) |
| `switch` บน `String`                 | `switch` บน `s.hashCode()` แล้วเช็ก `equals()` ซ้ำ                 |
| `enum Color { RED }`                 | `final class Color extends Enum<Color>` + field `static final RED` |
| `record Point(int x, int y)`         | `final class` + field `private final` + constructor + getter + `equals`/`hashCode`/`toString` |
| lambda `x -> x * 2`                  | private static method + คำสั่ง `invokedynamic` สร้าง object ตอนรัน |
| `"a" + b + "c"` (Java 9+)            | `invokedynamic` → `StringConcatFactory` (ก่อน Java 9 ใช้ `StringBuilder`) |

### Type Erasure — ทำไม generics มีข้อจำกัดแปลก ๆ

```java
List<String>  a = new ArrayList<>();
List<Integer> b = new ArrayList<>();
System.out.println(a.getClass() == b.getClass()); // ผลลัพธ์: true  (ตอนรันเป็น ArrayList เหมือนกัน)
```

เพราะ JVM ไม่รู้จัก `<String>` จึงทำสิ่งเหล่านี้ **ไม่ได้**:

```java
// new T()                        ✗ ไม่รู้ว่า T คืออะไรตอนรัน
// obj instanceof List<String>    ✗
// List<int>                      ✗ ต้องใช้ List<Integer> เพราะหลัง erasure กลายเป็น Object
```

ดูเพิ่มเติมใน [Generics](../advanced/generics.md)

> 💡 **เทียบกับ JavaScript:** เหมือน **TypeScript** เป๊ะ — type หายหมดหลัง compile เป็น JS เพียงแต่ Java ยังเก็บ type ของ class/primitive ไว้ใน bytecode ลบแค่ส่วน generics

---

## 4. Class Loading — JVM โหลดโค้ดเข้ามาอย่างไร?

JVM **ไม่ได้โหลดทุก class ตอนเริ่ม** แต่โหลดแบบ **lazy** — ใช้ class ไหนเมื่อไหร่ค่อยโหลดเมื่อนั้น

### 3 ขั้นตอน

```
Loading  ──►  Linking  ──────────────────────────────►  Initialization
(อ่าน .class)   ├─ Verify   ตรวจว่า bytecode ปลอดภัย          (รัน static { } และ
               ├─ Prepare  จองที่ static field (ค่า default)    กำหนดค่า static field)
               └─ Resolve  แปลงชื่อเป็นที่อยู่จริง
```

- **Verify** — JVM ไม่เชื่อไฟล์ `.class` ทันที (อาจถูกแก้มาหรือไม่ได้มาจาก `javac`) จะตรวจว่า stack ไม่ล้น, type ตรง, ไม่กระโดดไปกลางคำสั่ง ฯลฯ
- **Initialization** — เกิดตอนใช้ class ครั้งแรกจริง ๆ เท่านั้น

```java
class Config {
    static {
        System.out.println("โหลด Config แล้ว");
    }
    static String NAME = "app";
}

public class Main {
    public static void main(String[] args) {
        System.out.println("เริ่มโปรแกรม");
        System.out.println(Config.NAME);
    }
}
// ผลลัพธ์:
// เริ่มโปรแกรม
// โหลด Config แล้ว      ← เพิ่งโหลดตอนแตะ Config ครั้งแรก
// app
```

### Class Loader 3 ชั้น (Java 9+)

| Loader          | โหลดอะไร                                              |
| --------------- | ----------------------------------------------------- |
| **Bootstrap**   | class หลักของ JDK เช่น `java.lang.String`, `java.util.*` (เขียนด้วย C++) |
| **Platform**    | module อื่นของ JDK เช่น `java.sql`                    |
| **Application** | class ของเราจาก **classpath** (`-cp`)                 |

ใช้หลัก **parent delegation** — ถามแม่ก่อนเสมอ ทำให้เราสร้าง `java.lang.String` ปลอมมาแทนของจริงไม่ได้

```powershell
# ดูว่า class ไหนถูกโหลดบ้าง
java -Xlog:class+load Main.java
```

> 💡 **เกร็ด:** error `ClassNotFoundException` / `NoClassDefFoundError` คือหา `.class` ใน classpath ไม่เจอ — มักเกิดจากลืมใส่ `.jar` ใน `-cp` หรือรันผิด directory (ต้องรันจาก root ของ package ไม่ใช่จากใน folder `com/example`)

> 💡 **เทียบกับ JavaScript:** classpath ≈ `node_modules` + การ resolve `require()` — แต่ Java หาจาก **ชื่อเต็มของ class** (`com.example.shop.Main` → `com/example/shop/Main.class`)

---

## 5. Execution Engine — Interpreter + JIT

### ทำไมต้องมีทั้งสองแบบ?

| วิธี             | เริ่มเร็ว? | รันเร็ว? | หลักการ                                       |
| ---------------- | ---------- | -------- | --------------------------------------------- |
| **Interpreter**  | ✅ ทันที   | ❌ ช้า   | อ่าน bytecode ทีละคำสั่งแล้วทำตาม              |
| **JIT Compiler** | ❌ ต้องรอ compile | ✅ เร็วมาก | แปลง bytecode เป็น machine code ทั้ง method |

JVM (ชื่อ **HotSpot**) จึงใช้ทั้งคู่: เริ่มด้วย interpreter แล้ว **นับว่า method ไหนถูกเรียกบ่อย ("hot spot")** แล้วค่อยส่งไป JIT compile

### Tiered Compilation

```
Interpreter ──(เรียกบ่อย)──► C1 compiler ──(เรียกบ่อยมาก)──► C2 compiler
  ช้า               compile เร็ว, optimize น้อย          compile ช้า, optimize สุด
```

C2 ใช้ข้อมูลจริงที่เก็บระหว่างรัน (profiling) มา optimize เช่น:

- **Inlining** — เอาโค้ดของ method เล็ก ๆ มาแปะแทนการเรียก (getter/setter แทบไม่มีต้นทุน)
- **Escape analysis** — ถ้า object ไม่หลุดออกนอก method อาจไม่สร้างบน heap เลย
- **Loop unrolling**, ตัด null check / bounds check ที่พิสูจน์ได้ว่าไม่จำเป็น
- **Deoptimization** — ถ้าสมมติฐานผิด (เช่น โหลด subclass ใหม่เข้ามา) JVM ย้อนกลับไป interpreter ได้

```powershell
# ดูว่า method ไหนถูก JIT compile
java -XX:+PrintCompilation Main.java
```

### ผลที่ตามมา: JVM ต้อง "warm up"

โปรแกรม Java มักช้าช่วงแรก แล้วเร็วขึ้นเรื่อย ๆ — เพราะฉะนั้น **อย่าวัด performance ด้วย `System.currentTimeMillis()` รอบเดียว** ให้ใช้เครื่องมืออย่าง **JMH** ที่ warm up ให้ก่อน

> 💡 **เทียบกับ JavaScript:** V8 ทำแบบเดียวกัน (Ignition interpreter → Sparkplug/Maglev → TurboFan) และมี deoptimization เหมือนกัน — ต่างกันที่ JIT ของ Java ได้ type ที่แน่นอนจาก bytecode ทำให้ optimize ได้มั่นใจกว่า ไม่ต้องเดาว่าตัวแปรเป็น number หรือ string

### ทางเลือก: AOT (Ahead-of-Time)

**GraalVM Native Image** compile Java เป็นไฟล์ `.exe` ล่วงหน้าเลย → เปิดเร็วระดับ millisecond ใช้ RAM น้อย เหมาะกับ serverless / CLI แต่แลกกับ build ช้าและข้อจำกัดเรื่อง reflection

---

## 6. Memory ใน JVM

```
┌──────────────────────────── JVM Process ────────────────────────────┐
│                                                                     │
│  แชร์ทุก thread                       แยกต่อ thread                  │
│  ┌──────────────────────┐            ┌──────────────┐               │
│  │ Heap                 │            │ Stack (T1)   │  ┌──────────┐ │
│  │  object ทุกตัว, array  │◄───────────┤  frame: main │  │Stack (T2)│ │
│  │  (GC ดูแล)           │  reference │  frame: add  │  │   ...    │ │
│  └──────────────────────┘            └──────────────┘  └──────────┘ │
│  ┌──────────────────────┐            PC Register, Native Stack      │
│  │ Metaspace            │                                           │
│  │  ข้อมูล class, method │                                           │
│  └──────────────────────┘                                           │
└─────────────────────────────────────────────────────────────────────┘
```

| พื้นที่         | เก็บอะไร                                                             | error เมื่อเต็ม                              |
| --------------- | -------------------------------------------------------------------- | -------------------------------------------- |
| **Stack**       | 1 frame ต่อการเรียก method: local variable (primitive + **reference**) | `StackOverflowError` (เช่น recursion ไม่มีจุดจบ) |
| **Heap**        | **object ทุกตัว** (`new ...`), array, String                          | `OutOfMemoryError: Java heap space`          |
| **Metaspace**   | โครงสร้าง class, bytecode ของ method (แทน PermGen ตั้งแต่ Java 8)     | `OutOfMemoryError: Metaspace`                |

### Primitive vs Reference ในหน่วยความจำ

```java
public static void main(String[] args) {
    int age = 25;                       // ค่า 25 อยู่บน stack ตรง ๆ
    int[] scores = {90, 80};            // ตัวแปร scores (บน stack) เก็บ "ที่อยู่" → array อยู่บน heap
    int[] copy = scores;                // copy ที่อยู่ ไม่ได้ copy array
    copy[0] = 0;
    System.out.println(scores[0]);      // ผลลัพธ์: 0
}
```

```
Stack (main)              Heap
┌────────────────┐        ┌──────────────┐
│ age    = 25    │        │ int[] {0, 80}│
│ scores = ──────┼───────►│              │
│ copy   = ──────┼───────►│              │
└────────────────┘        └──────────────┘
```

> 💡 **เกร็ด: Java เป็น pass-by-value เสมอ** — เวลาส่ง object เข้า method สิ่งที่ถูก copy คือ "ค่าของ reference" ดังนั้นแก้ข้างใน object ได้ แต่สั่ง `param = new X()` จะไม่กระทบตัวแปรต้นทาง

> 💡 **เทียบกับ JavaScript:** เหมือนกันทุกประการ — primitive (`number`, `string`, `boolean`) copy ค่า, object/array copy reference

---

## 7. Garbage Collector (GC)

Java **ไม่มี `free()` / `delete`** — GC จะหา object ที่ **ไม่มีใครอ้างถึงแล้ว** (unreachable จาก stack/static) แล้วคืน memory ให้อัตโนมัติ

### Generational Hypothesis

> "object ส่วนใหญ่ตายตั้งแต่ยังเด็ก" — เช่น object ชั่วคราวใน loop

```
Heap
┌───────────── Young Generation ─────────────┬──── Old Generation ────┐
│  Eden (เกิดใหม่)  │ Survivor 0 │ Survivor 1 │  object อายุยืน          │
└───────────────────────────────────────────┴────────────────────────┘
   GC บ่อย เร็ว (Minor GC)                     GC นาน ๆ ครั้ง (Major GC)
```

object รอดจาก GC หลายรอบ → ถูกย้ายไป Old Gen

### GC ที่ควรรู้จัก

| GC              | จุดเด่น                                                      |
| --------------- | ------------------------------------------------------------ |
| **G1**          | **ค่าเริ่มต้นตั้งแต่ Java 9** สมดุลระหว่าง throughput กับเวลาหยุด |
| **ZGC**         | หยุดโปรแกรมต่ำกว่า 1 ms แม้ heap หลายร้อย GB (Generational ZGC ใน Java 21) |
| **Parallel**    | throughput สูงสุด เหมาะกับงาน batch                          |
| **Serial**      | thread เดียว เหมาะกับเครื่องเล็ก/container                    |

```powershell
java -Xms256m -Xmx1g -XX:+UseZGC -Xlog:gc Main.java
#    ^heap เริ่มต้น  ^heap สูงสุด  ^เลือก GC   ^พิมพ์ log การ GC
```

> ⚠️ **GC ไม่ได้กัน memory leak ทุกแบบ** — ถ้ายังมี reference ค้างอยู่ (เช่น ใส่ไว้ใน `static List` แล้วไม่เคยลบ) GC ก็เก็บไม่ได้

> 💡 **เทียบกับ JavaScript:** V8 ก็เป็น generational GC (Scavenger สำหรับ young, Mark-Compact สำหรับ old) แนวคิดเดียวกัน ต่างกันที่ Java ให้เลือกและปรับ GC ได้ละเอียดกว่ามาก

---

## 8. เกร็ดความรู้ที่มักสะดุด

### 8.1 `==` กับ Integer Cache

```java
Integer a = 127, b = 127;
System.out.println(a == b);       // ผลลัพธ์: true

Integer c = 128, d = 128;
System.out.println(c == d);       // ผลลัพธ์: false  😱
System.out.println(c.equals(d));  // ผลลัพธ์: true   ✅
```

**ทำไม?** autoboxing เรียก `Integer.valueOf()` ซึ่ง **cache object ไว้เฉพาะ -128 ถึง 127** — นอกช่วงนี้ได้ object ใหม่ทุกครั้ง และ `==` บน object เปรียบเทียบ **ที่อยู่** ไม่ใช่ค่า

### 8.2 String Pool

```java
String s1 = "java";
String s2 = "java";
String s3 = new String("java");

System.out.println(s1 == s2);       // ผลลัพธ์: true   (literal ใช้ object เดียวกันจาก pool)
System.out.println(s1 == s3);       // ผลลัพธ์: false  (new สร้าง object ใหม่บน heap)
System.out.println(s1.equals(s3));  // ผลลัพธ์: true
```

String literal ถูกเก็บใน **String Pool** (อยู่ใน heap) และใช้ซ้ำ — ทำได้เพราะ `String` เป็น **immutable**

> **กฎ:** เปรียบเทียบ object (รวมถึง `String`, `Integer`) ด้วย **`.equals()`** เสมอ ใช้ `==` กับ primitive เท่านั้น

> 💡 **เทียบกับ JavaScript:** ใน JS `"java" === "java"` เป็น `true` เสมอเพราะ string เป็น primitive — ใน Java `String` เป็น object จึงต้องระวัง

### 8.3 JVM ไม่ได้มีแค่ Java

ภาษาอะไรก็ได้ที่ compile เป็น bytecode ได้ก็รันบน JVM — **Kotlin** (ภาษาหลักของ Android), **Scala**, **Groovy** (ใช้ใน Gradle), **Clojure** และเรียกใช้ library Java ร่วมกันได้

### 8.4 JVM มีหลายเจ้า

`java` ที่เราติดตั้งเป็นแค่ **implementation หนึ่ง** ตาม spec — OpenJDK HotSpot (Temurin, Oracle, Amazon Corretto, Microsoft Build of OpenJDK ใช้ตัวนี้), **Eclipse OpenJ9** (เน้นใช้ RAM น้อย), **GraalVM** (JIT เขียนด้วย Java + native image)

---

## 9. เครื่องมือส่องดู JVM (มากับ JDK)

| เครื่องมือ        | ใช้ทำอะไร                                                          |
| ----------------- | ------------------------------------------------------------------ |
| `jps`             | แสดง process Java ที่รันอยู่ (≈ `ps` เฉพาะ Java)                   |
| `jcmd <pid> help` | สั่งงาน JVM ที่รันอยู่: ดู heap, thread, flag, สั่ง GC              |
| `jstack <pid>`    | ดู stack ของทุก thread — ใช้หา deadlock / โปรแกรมค้าง               |
| `jconsole`        | GUI ดู memory, thread, CPU แบบ real-time                            |
| **JFR** (Flight Recorder) | บันทึก profile ระดับ production ต้นทุนต่ำ: `java -XX:StartFlightRecording ...` |
| VisualVM / JDK Mission Control | GUI วิเคราะห์ heap dump และไฟล์ JFR (ดาวน์โหลดแยก)    |

> 💡 **เทียบกับ JavaScript:** คล้าย `node --inspect` + Chrome DevTools (Memory / Performance tab)

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ให้ `java` และ `javac` เป็นเวอร์ชันเดียวกัน** — หรือใช้ `javac --release N` เมื่อต้องรองรับ JVM เก่า
2. **อย่ากลัว method เล็ก ๆ** — JIT inline ให้อยู่แล้ว เขียนให้อ่านง่ายก่อน optimize ทีหลัง
3. **ใช้ `.equals()` เปรียบเทียบ object** — `==` ใช้กับ primitive เท่านั้น
4. **วัด performance ด้วย JMH** ไม่ใช่จับเวลารอบเดียว เพราะ JVM ต้อง warm up
5. **ปล่อยค่า GC เป็นค่าเริ่มต้น (G1)** จนกว่าจะมีปัญหาจริงและวัดผลได้ — ส่วนใหญ่ปรับแค่ `-Xmx` ก็พอ
6. **เมื่อสงสัยว่า `javac` แปลงโค้ดเป็นอะไร ให้ลอง `javap -c`** — เป็นวิธีเรียนรู้ภาษาที่ดีมาก

```powershell
# ลองเองทั้งหมดในคำสั่งเดียว
javac Add.java ; javap -c Add ; java -XX:+PrintCompilation Add
```

---
