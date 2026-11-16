# Java Notes

รวมเนื้อหาและบันทึกการเรียน Java (เป้าหมาย **Java 21 LTS**) สำหรับคนที่เขียน JavaScript เป็นอยู่แล้ว

## เนื้อหา

| หมวด                        | หัวข้อ                                                          |
| :-------------------------- | :-------------------------------------------------------------- |
| 0. Getting Started          | ติดตั้ง JDK, Hello World, javac / java, jshell, เบื้องหลัง JVM   |
| 1. Fundamental              | ตัวแปร, ชนิดข้อมูล, Operators, Conditionals, Loops              |
| 2. Data Manipulation        | Array, String, Collections, Stream API, Math, Optional          |
| 3. Method                   | Method, Lambda & Functional Interface                           |
| 4. Object-Oriented Programming | Class, Inheritance, Abstract / Interface, Record / Enum      |
| 5. Advanced                 | Exception, Generics, Date & Time, File I/O, Concurrency         |
| 6. Tooling                  | Maven (build tool, dependency, lifecycle — เทียบ npm)            |

> 💡 ทุกไฟล์มีกล่อง **"💡 เทียบกับ JavaScript"** อธิบายจุดที่ Java ต่างจาก JS อย่างมีนัยสำคัญ

## 0.Getting Started

- [Setup & Hello World](docs/fundamental/setup-and-hello-world.md)
- [How Java Works (Compile, Bytecode, JVM)](docs/fundamental/how-java-works.md)

## 1.Fundamental

- [Variable](docs/fundamental/variable.md)
- [Data Types](docs/fundamental/data-types.md)
- [Operators](docs/fundamental/operators.md)
- [Conditional](docs/fundamental/conditional.md)
- [Loops](docs/fundamental/loops.md)

## 2.Data Manipulation

- [Array](docs/data-manipulation/array.md)
- [String Methods](docs/data-manipulation/string-methods.md)
- [Collections](docs/data-manipulation/collections.md)
- [Stream API](docs/data-manipulation/stream-api.md)
- [Math](docs/data-manipulation/math.md)
- [Optional](docs/data-manipulation/optional.md)

## 3.Method

- [Method](docs/method/method.md)
- [Lambda & Functional Interface](docs/method/lambda-functional-interface.md)

## 4.Object-Oriented Programming

- [Class & Object](docs/oop/class-object.md)
- [Inheritance & Polymorphism](docs/oop/inheritance-polymorphism.md)
- [Abstract Class & Interface](docs/oop/abstract-interface.md)
- [Record & Enum](docs/oop/record-enum.md)

## 5.Advanced

- [Exception](docs/advanced/exception.md)
- [Generics](docs/advanced/generics.md)
- [Date & Time](docs/advanced/date-time.md)
- [File I/O (java.nio.file)](docs/advanced/file-io.md)
- [Concurrency (Thread, ExecutorService, CompletableFuture, Virtual Threads)](docs/advanced/concurrency.md)

## 6.Tooling

- [Maven (เทียบ npm / package.json)](docs/tooling/maven.md)

---

## วิธีรันโค้ด

### 1. ติดตั้ง JDK 21 (Eclipse Temurin)

```powershell
winget install EclipseAdoptium.Temurin.21.JDK
```

หรือดาวน์โหลดจาก https://adoptium.net แล้วเลือก **Temurin 21 (LTS)**

### 2. ตรวจสอบเวอร์ชัน

```powershell
java -version
# openjdk version "21.0.x" ...
```

> ⚠️ **หมายเหตุ:** เครื่องนี้ตอนนี้มี **JDK 11** ติดตั้งอยู่ ซึ่งรันตัวอย่างพื้นฐานได้ แต่ฟีเจอร์ใหม่ ๆ เช่น `switch` expression (14), text blocks (15), `record` (16), `sealed` (17), pattern matching for `switch` (21) และ `SequencedCollection` (21) **ต้องใช้ JDK 17+ หรือ 21** — แนะนำให้ติดตั้ง JDK 21 แล้วตั้ง `JAVA_HOME` ให้ชี้ไปที่ JDK 21 (ดูรายละเอียดใน [Setup & Hello World](docs/fundamental/setup-and-hello-world.md))

### 3. รันไฟล์เดียวโดยไม่ต้อง compile (Java 11+)

สร้างไฟล์ `Main.java`:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("สวัสดี Java!");
    }
}
```

แล้วรัน:

```powershell
java Main.java
# ผลลัพธ์: สวัสดี Java!
```

### 4. ทดลองโค้ดสั้น ๆ ด้วย jshell

```powershell
jshell
jshell> 10 / 3
$1 ==> 3
```
