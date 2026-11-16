# Setup & Hello World

# ส่วนที่ 0: เริ่มต้นใช้งาน Java

## Java ทำงานอย่างไร?

Java เป็นภาษาแบบ **compiled + interpreted** — โค้ด `.java` จะถูก compile เป็น **bytecode** (`.class`) ก่อน แล้ว **JVM** จะนำ bytecode ไปรันบนเครื่องใดก็ได้ ("Write once, run anywhere")

```
Main.java  ──(javac)──►  Main.class (bytecode)  ──(java / JVM)──►  รันบน Windows / macOS / Linux
```

> 💡 **เทียบกับ JavaScript:** JS ถูกรันโดยตรงจาก source code ผ่าน engine (V8 ใน Node.js / Chrome) ไม่มีขั้นตอน compile แยก ส่วน Java ต้อง compile ก่อนเสมอ และ compiler จะ **ตรวจ type ทั้งหมดตั้งแต่ตอน compile** ทำให้ bug หลายอย่างถูกจับได้ก่อนรันจริง

> 📖 อยากรู้ลึกกว่านี้ (bytecode, class loading, JIT, memory, GC) อ่านต่อที่ [How Java Works](how-java-works.md)

---

## 1. JDK vs JRE vs JVM

| ตัวย่อ  | ชื่อเต็ม                   | คืออะไร                                                              |
| ------- | -------------------------- | -------------------------------------------------------------------- |
| **JVM** | Java Virtual Machine       | เครื่องเสมือนที่รัน bytecode, จัดการ memory (Garbage Collector)       |
| **JRE** | Java Runtime Environment   | JVM + standard library — พอสำหรับ "รัน" โปรแกรม                     |
| **JDK** | Java Development Kit       | JRE + เครื่องมือพัฒนา (`javac`, `jshell`, `jar`, `javadoc` ...)       |

```
┌─────────────────────── JDK ───────────────────────┐
│  javac, jshell, jar, javadoc, ...                 │
│  ┌──────────────────── JRE ────────────────────┐  │
│  │  Standard Library (java.lang, java.util ...)│  │
│  │  ┌──────────────── JVM ──────────────────┐  │  │
│  │  │  Class Loader, JIT Compiler, GC       │  │  │
│  │  └───────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────┘
```

> ตั้งแต่ Java 11 เป็นต้นมา ไม่มีการแจก JRE แยกอย่างเป็นทางการแล้ว — **ติดตั้ง JDK อย่างเดียว** ก็พอ

### เวอร์ชัน LTS (Long-Term Support)

| เวอร์ชัน | ปี   | ฟีเจอร์เด่น                                              |
| -------- | ---- | -------------------------------------------------------- |
| 8        | 2014 | Lambda, Stream API, `Optional`, `java.time`              |
| 11       | 2018 | `var` ใน lambda, `String` methods ใหม่, รันไฟล์เดียวได้  |
| 17       | 2021 | `record`, `sealed`, text blocks, switch expression       |
| **21**   | 2023 | Pattern matching for `switch`, Record patterns, Virtual Threads, `SequencedCollection` |

---

## 2. ติดตั้ง JDK 21 บน Windows

### ติดตั้งด้วย winget

```powershell
winget install EclipseAdoptium.Temurin.21.JDK
```

### ตั้งค่า `JAVA_HOME`

ตัวติดตั้ง Temurin มีตัวเลือกให้ตั้ง `JAVA_HOME` และ `PATH` ให้อัตโนมัติ ถ้าไม่ได้ตั้งไว้ให้ทำเอง:

```powershell
# ดู path ที่ติดตั้ง (ปกติอยู่ที่ C:\Program Files\Eclipse Adoptium\jdk-21.x.x-hotspot)
setx JAVA_HOME "C:\Program Files\Eclipse Adoptium\jdk-21.0.5.11-hotspot"
```

จากนั้นเพิ่ม `%JAVA_HOME%\bin` ไว้ **ด้านบนสุด** ของ `Path` ใน *System Properties → Environment Variables* แล้ว **เปิด terminal ใหม่**

### ตรวจสอบ

```powershell
java -version
# openjdk version "21.0.5" 2024-10-15 LTS

javac -version
# javac 21.0.5
```

> ⚠️ ถ้ามีหลายเวอร์ชัน (เช่น JDK 11 อยู่แล้ว) ตัวที่อยู่บนสุดของ `Path` จะถูกใช้ — ถ้า `java -version` ยังขึ้น 11 ให้ตรวจลำดับใน `Path`

---

## 3. Hello World

### สร้างไฟล์ `Main.java`

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("สวัสดี Java!");
    }
}
```

### คุณสมบัติ

- ชื่อไฟล์ **ต้องตรงกับชื่อ `public class`** (`Main` → `Main.java`)
- ทุกโค้ดต้องอยู่ใน **class** (ไม่มีฟังก์ชันลอย ๆ ระดับไฟล์)
- ทุก statement ต้องจบด้วย **`;`** (ไม่มี automatic semicolon insertion แบบ JS)

### วิธีที่ 1: compile แล้วรัน (แบบดั้งเดิม)

```powershell
javac Main.java     # ได้ไฟล์ Main.class
java Main           # รันโดยระบุชื่อ class (ไม่ต้องใส่ .class)
# ผลลัพธ์: สวัสดี Java!
```

### วิธีที่ 2: Single-File Source Launch (Java 11+)

```powershell
java Main.java
# ผลลัพธ์: สวัสดี Java!
```

compile ใน memory แล้วรันเลย ไม่สร้างไฟล์ `.class` — เหมาะกับการทดลองและการเรียน

> 💡 **เทียบกับ JavaScript:** คล้าย `node main.js` แต่เบื้องหลังยังมีการ compile + ตรวจ type อยู่ ถ้า type ผิด โปรแกรมจะ **ไม่รันเลยสักบรรทัด**

---

## 4. กายวิภาคของ `main` method

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("จำนวน arguments: " + args.length);
    }
}
```

| ส่วน            | ความหมาย                                                               |
| --------------- | ---------------------------------------------------------------------- |
| `public`        | JVM เรียกใช้ได้จากภายนอก                                                |
| `static`        | เรียกได้โดยไม่ต้องสร้าง object ของ `Main` ก่อน                          |
| `void`          | ไม่คืนค่า (ใช้ `System.exit(code)` ถ้าต้องการส่ง exit code)            |
| `main`          | ชื่อที่ JVM มองหาเป็นจุดเริ่มต้นโปรแกรม                                  |
| `String[] args` | argument จาก command line                                              |

### รับ argument จาก command line

```java
public class Main {
    public static void main(String[] args) {
        for (String arg : args) {
            System.out.println("arg = " + arg);
        }
    }
}
```

```powershell
java Main.java สมชาย 25
# ผลลัพธ์:
# arg = สมชาย
# arg = 25
```

### การแสดงผล

```java
System.out.println("ขึ้นบรรทัดใหม่");      // พิมพ์แล้วขึ้นบรรทัดใหม่
System.out.print("ไม่ขึ้นบรรทัดใหม่");     // พิมพ์ต่อกัน
System.out.printf("ชื่อ %s อายุ %d%n", "สมชาย", 25); // ผลลัพธ์: ชื่อ สมชาย อายุ 25
System.err.println("ข้อความ error");       // พิมพ์ไปที่ stderr
```

> 💡 **เทียบกับ JavaScript:** `System.out.println(x)` ≈ `console.log(x)` แต่รับได้ทีละ 1 argument — ต้องต่อ string เองด้วย `+` หรือใช้ `printf` / `String.format`

---

## 5. Package และ Naming Conventions

### Package

Package คือ namespace สำหรับจัดกลุ่ม class (คล้าย folder) และต้องตรงกับโครงสร้าง directory

```
src/
└── com/
    └── example/
        └── shop/
            ├── Main.java
            └── Product.java
```

```java
package com.example.shop;   // บรรทัดแรกของไฟล์

import java.util.List;      // import class จาก package อื่น
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        List<String> items = new ArrayList<>();
        items.add("กาแฟ");
        System.out.println(items); // ผลลัพธ์: [กาแฟ]
    }
}
```

- package ใช้ชื่อโดเมนกลับด้าน เช่น `com.example.shop` (ตัวพิมพ์เล็กทั้งหมด)
- `java.lang.*` (เช่น `String`, `Math`, `System`) ถูก import อัตโนมัติ

> 💡 **เทียบกับ JavaScript:** `import java.util.List;` ≈ `import { List } from "java.util"` แต่ Java import เป็น **ชื่อ class** ไม่ใช่ไฟล์ และไม่มี default export

### Naming Conventions

| สิ่งที่ตั้งชื่อ    | รูปแบบ                 | ตัวอย่าง                       |
| ----------------- | ---------------------- | ------------------------------ |
| Class / Interface | `PascalCase`           | `OrderService`, `Comparable`   |
| Method / ตัวแปร   | `camelCase`            | `calculateTotal`, `userName`   |
| ค่าคงที่          | `UPPER_SNAKE_CASE`     | `MAX_SIZE`, `DEFAULT_TIMEOUT`  |
| Package           | ตัวพิมพ์เล็ก คั่นด้วย `.` | `com.example.shop`          |

---

## 6. IDE และเครื่องมือ

| เครื่องมือ                           | จุดเด่น                                                                    |
| ------------------------------------ | -------------------------------------------------------------------------- |
| **IntelliJ IDEA** (Community ฟรี)    | IDE ยอดนิยมที่สุดสำหรับ Java, refactor / autocomplete ดีมาก               |
| **VS Code + Extension Pack for Java** | เบา คุ้นเคยสำหรับคนเขียน JS — ติดตั้ง extension `vscjava.vscode-java-pack` |
| **Eclipse**                          | IDE รุ่นเก่า ยังใช้ในองค์กร                                                |

ใน VS Code เมื่อติดตั้ง Extension Pack for Java แล้ว จะมีปุ่ม **Run | Debug** เหนือ `main` method ให้กดรันได้ทันที

---

## 7. jshell — REPL ของ Java (Java 9+)

`jshell` ใช้ทดลองโค้ดสั้น ๆ โดยไม่ต้องสร้าง class หรือ `main`

```
> jshell
|  Welcome to JShell -- Version 21.0.5

jshell> int x = 10
x ==> 10

jshell> x / 3
$2 ==> 3

jshell> "สมชาย".length()
$3 ==> 5

jshell> /exit
```

| คำสั่ง      | ความหมาย                     |
| ----------- | ---------------------------- |
| `/vars`     | แสดงตัวแปรทั้งหมด              |
| `/methods`  | แสดง method ที่ประกาศไว้       |
| `/imports`  | แสดง import ปัจจุบัน           |
| `/exit`     | ออกจาก jshell                |

> 💡 **เทียบกับ JavaScript:** `jshell` ≈ การพิมพ์ `node` เข้า REPL หรือใช้ DevTools Console ในเบราว์เซอร์ (ใน jshell ไม่ต้องใส่ `;` ท้ายบรรทัด)

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ JDK เวอร์ชัน LTS** (17 หรือ 21) — ได้รับ security update ยาวนาน
2. **ตั้ง `JAVA_HOME` ให้ถูกต้อง** — build tool อย่าง Maven / Gradle อ่านค่านี้
3. **ใช้ `java Main.java`** สำหรับทดลองโค้ด และใช้ build tool (Maven/Gradle) เมื่อโปรเจกต์มีหลายไฟล์หรือมี dependency
4. **ตั้งชื่อตาม convention** — `PascalCase` สำหรับ class, `camelCase` สำหรับ method/ตัวแปร
5. **ใช้ `jshell`** เมื่ออยากเช็กพฤติกรรมของ method หรือ expression สั้น ๆ

```java
// โครงสร้างพื้นฐานที่ใช้ในทุกตัวอย่างของเอกสารนี้
public class Main {
    public static void main(String[] args) {
        String name = "สมชาย";
        System.out.println("สวัสดี " + name); // ผลลัพธ์: สวัสดี สมชาย
    }
}
```

---
