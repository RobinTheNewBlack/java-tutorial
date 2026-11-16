# Advanced

# ส่วนที่ 1: การจัดการข้อผิดพลาด (Exception Handling)

## Exception คืออะไร?

Exception คือ **เหตุการณ์ผิดปกติ** ที่เกิดขึ้นระหว่างโปรแกรมทำงาน เช่น หารด้วยศูนย์, เปิดไฟล์ที่ไม่มีอยู่, เรียก method บน `null` — ถ้าไม่จัดการ โปรแกรมจะหยุดทำงานและแสดง stack trace

> 💡 **เทียบกับ JavaScript:** JS `throw` อะไรก็ได้ (string, number, object) และไม่มีการบังคับให้จัดการ error ส่วน Java `throw` ได้เฉพาะ object ที่เป็น `Throwable` และมี **checked exception** ที่ compiler **บังคับ** ให้จัดการ

---

## 1. ลำดับชั้นของ Exception (Exception Hierarchy)

```
                    Throwable
                   /         \
              Error           Exception
               |              /        \
     OutOfMemoryError   IOException    RuntimeException
     StackOverflowError SQLException     |
                        (checked)        ├── NullPointerException
                                         ├── IllegalArgumentException
                                         ├── IllegalStateException
                                         ├── ArithmeticException
                                         ├── IndexOutOfBoundsException
                                         └── ClassCastException
                                             (unchecked)
```

| กลุ่ม                 | ความหมาย                                          | ต้องจัดการ?          | ตัวอย่าง                                   |
| -------------------- | ------------------------------------------------- | ------------------ | ----------------------------------------- |
| `Error`              | ปัญหาร้ายแรงระดับ JVM — โปรแกรมกู้คืนไม่ได้              | ไม่ควร catch        | `OutOfMemoryError`, `StackOverflowError`  |
| Checked `Exception`  | ปัญหาจากภายนอกที่ "คาดได้" — compiler บังคับจัดการ      | **บังคับ**           | `IOException`, `SQLException`             |
| `RuntimeException`   | ส่วนใหญ่เป็น bug ของโปรแกรม (unchecked)               | ไม่บังคับ             | `NullPointerException`, `IllegalArgumentException` |

---

## 2. Checked vs Unchecked Exception

### Checked Exception

ต้อง **catch** หรือ **ประกาศ `throws`** ไม่งั้น compile ไม่ผ่าน

```java
import java.nio.file.*;
import java.io.IOException;

// ❌ Compile error: unreported exception IOException; must be caught or declared to be thrown
// static String read(Path p) { return Files.readString(p); }

// ✅ วิธีที่ 1: ประกาศ throws — ส่งต่อให้ผู้เรียกจัดการ
static String read(Path p) throws IOException {
    return Files.readString(p);
}

// ✅ วิธีที่ 2: catch เอง
static String readOrDefault(Path p) {
    try {
        return Files.readString(p);
    } catch (IOException e) {
        return "(อ่านไฟล์ไม่ได้)";
    }
}
```

### Unchecked Exception

ไม่บังคับจัดการ — มักบอกว่า **โค้ดมี bug** ควรแก้ที่ต้นเหตุมากกว่า catch

```java
String name = null;
// name.length(); // NullPointerException

int[] arr = {1, 2, 3};
// arr[5];        // ArrayIndexOutOfBoundsException

// Integer.parseInt("abc"); // NumberFormatException (extends IllegalArgumentException)
```

| หัวข้อ           | Checked                         | Unchecked                          |
| --------------- | ------------------------------- | ---------------------------------- |
| สืบทอดจาก       | `Exception` (ไม่ใช่ Runtime)     | `RuntimeException`                 |
| Compiler บังคับ  | ✅                              | ❌                                  |
| สาเหตุ           | ปัจจัยภายนอก (ไฟล์, network, DB)   | bug / ข้อมูลผิดรูปแบบ                  |
| ใช้ใน Spring     | น้อย                             | **นิยมมาก** (`DataAccessException` ฯลฯ) |

---

## 3. try / catch / finally

```java
public class Main {
    public static void main(String[] args) {
        try {
            int result = 10 / 0;
            System.out.println("ไม่ถึงบรรทัดนี้");
        } catch (ArithmeticException e) {
            System.out.println("เกิดข้อผิดพลาด: " + e.getMessage());
        } finally {
            System.out.println("finally ทำงานเสมอ");
        }
        // ผลลัพธ์:
        // เกิดข้อผิดพลาด: / by zero
        // finally ทำงานเสมอ
    }
}
```

### หลาย catch — เรียงจาก "เฉพาะเจาะจง" ไป "กว้าง"

```java
try {
    String input = "abc";
    int n = Integer.parseInt(input);
} catch (NumberFormatException e) {          // เฉพาะเจาะจงกว่า → ต้องอยู่ก่อน
    System.out.println("รูปแบบตัวเลขไม่ถูกต้อง");
} catch (IllegalArgumentException e) {       // กว้างกว่า
    System.out.println("argument ไม่ถูกต้อง");
}
// ผลลัพธ์: รูปแบบตัวเลขไม่ถูกต้อง
```

> **หมายเหตุ:** ถ้าเอา catch ที่กว้างกว่าไว้ก่อน จะ **compile error** เพราะ catch ตัวหลังไม่มีทางทำงาน

### Multi-catch (Java 7+)

จัดการหลาย exception ด้วยโค้ดเดียวกันด้วย `|`

```java
try {
    Object o = "สมชาย";
    Integer n = (Integer) o;
} catch (ClassCastException | NullPointerException e) {
    System.out.println("ข้อมูลผิดประเภท: " + e.getClass().getSimpleName());
}
// ผลลัพธ์: ข้อมูลผิดประเภท: ClassCastException
```

### Method ที่ใช้บ่อยของ Exception

| Method              | คืนค่า                                    |
| ------------------- | ---------------------------------------- |
| `getMessage()`      | ข้อความ error                             |
| `getCause()`        | exception ต้นเหตุ (ถ้ามี)                    |
| `printStackTrace()` | พิมพ์ stack trace (ใช้ตอน debug — production ให้ใช้ logger) |
| `getStackTrace()`   | array ของ stack frame                     |

---

## 4. try-with-resources และ AutoCloseable (Java 7+)

Resource ที่ต้องปิด (ไฟล์, DB connection, stream) ควรประกาศใน `try (...)` — Java จะเรียก `close()` ให้อัตโนมัติ **แม้เกิด exception**

```java
import java.io.*;

// ❌ แบบเก่า — ยาวและลืมปิดง่าย
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("data.txt"));
    System.out.println(reader.readLine());
} catch (IOException e) {
    System.out.println("อ่านไม่ได้");
} finally {
    if (reader != null) {
        try { reader.close(); } catch (IOException ignored) { }
    }
}

// ✅ try-with-resources
try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    System.out.println(br.readLine());
} catch (IOException e) {
    System.out.println("อ่านไม่ได้: " + e.getMessage());
}
```

### สร้าง Resource เอง ด้วย `AutoCloseable`

```java
class DatabaseConnection implements AutoCloseable {
    private final String name;

    DatabaseConnection(String name) {
        this.name = name;
        System.out.println("เปิด " + name);
    }

    void query(String sql) {
        System.out.println("รัน: " + sql);
    }

    @Override
    public void close() {
        System.out.println("ปิด " + name);
    }
}

public class Main {
    public static void main(String[] args) {
        try (DatabaseConnection db1 = new DatabaseConnection("DB-1");
             DatabaseConnection db2 = new DatabaseConnection("DB-2")) {
            db1.query("SELECT * FROM users");
            throw new IllegalStateException("เกิดปัญหา!");
        } catch (IllegalStateException e) {
            System.out.println("จับได้: " + e.getMessage());
        }
    }
}
// ผลลัพธ์:
// เปิด DB-1
// เปิด DB-2
// รัน: SELECT * FROM users
// ปิด DB-2          ← ปิดย้อนลำดับการเปิด และปิดก่อนเข้า catch
// ปิด DB-1
// จับได้: เกิดปัญหา!
```

> 💡 **เทียบกับ JavaScript:** คล้าย `using` declaration (Explicit Resource Management) ที่เพิ่งเข้ามาใน JS/TypeScript 5.2 — ก่อนหน้านั้น JS ต้องใช้ `try/finally` ปิดเอง

---

## 5. `throw` และ `throws`

| Keyword  | ใช้ที่                 | ความหมาย                                   |
| -------- | -------------------- | ----------------------------------------- |
| `throw`  | ในตัว method          | **โยน** exception ออกไปตอนนี้                |
| `throws` | ใน signature ของ method | **ประกาศ** ว่า method นี้อาจโยน exception ชนิดนี้ |

```java
static void withdraw(double balance, double amount) {
    if (amount <= 0) {
        throw new IllegalArgumentException("จำนวนเงินต้องมากกว่า 0");
    }
    if (amount > balance) {
        throw new IllegalStateException("ยอดเงินไม่พอ: มี " + balance + " ต้องการ " + amount);
    }
    System.out.println("ถอนสำเร็จ");
}

static void loadConfig(String path) throws java.io.IOException { // checked → ต้องประกาศ
    throw new java.io.FileNotFoundException("ไม่พบไฟล์ " + path);
}

try {
    withdraw(500, 1000);
} catch (IllegalStateException e) {
    System.out.println(e.getMessage()); // ผลลัพธ์: ยอดเงินไม่พอ: มี 500.0 ต้องการ 1000.0
}
```

---

## 6. Custom Exception

สร้าง exception ของ domain เอง ทำให้โค้ดสื่อความหมายและจัดการแยกประเภทได้

```java
// Unchecked (นิยมใน Spring Boot)
class ResourceNotFoundException extends RuntimeException {
    private final String resource;
    private final Object id;

    public ResourceNotFoundException(String resource, Object id) {
        super(resource + " id=" + id + " ไม่พบในระบบ");
        this.resource = resource;
        this.id = id;
    }

    public String getResource() { return resource; }
    public Object getId() { return id; }
}

// Checked (ใช้เมื่อผู้เรียก "ควร" จัดการจริง ๆ)
class InsufficientBalanceException extends Exception {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}
```

```java
import java.util.*;

public class Main {
    record User(long id, String name) {}

    static final Map<Long, User> USERS = Map.of(1L, new User(1, "สมชาย"));

    static User findUser(long id) {
        User user = USERS.get(id);
        if (user == null) {
            throw new ResourceNotFoundException("User", id);
        }
        return user;
    }

    public static void main(String[] args) {
        System.out.println(findUser(1).name()); // ผลลัพธ์: สมชาย

        try {
            findUser(99);
        } catch (ResourceNotFoundException e) {
            System.out.println(e.getMessage()); // ผลลัพธ์: User id=99 ไม่พบในระบบ
        }
    }
}
```

> **Tip:** ใน Spring Boot มักใช้ `@RestControllerAdvice` + `@ExceptionHandler(ResourceNotFoundException.class)` แปลง exception เป็น HTTP 404 ที่จุดเดียว ไม่ต้อง try/catch ทุก controller

---

## 7. Exception Chaining (Cause)

เมื่อ catch exception ระดับล่างแล้วโยน exception ระดับสูงกว่า **ต้องแนบ exception เดิมเป็น cause** เพื่อไม่ให้ข้อมูลต้นเหตุหาย

```java
class DataLoadException extends RuntimeException {
    public DataLoadException(String message, Throwable cause) {
        super(message, cause); // ส่ง cause ต่อไป
    }
}

static int loadAge(String raw) {
    try {
        return Integer.parseInt(raw);
    } catch (NumberFormatException e) {
        throw new DataLoadException("โหลดข้อมูลอายุไม่สำเร็จ", e); // ✅ แนบ e
        // throw new DataLoadException("โหลดข้อมูลอายุไม่สำเร็จ", null); // ❌ ข้อมูลต้นเหตุหาย
    }
}

try {
    loadAge("ยี่สิบ");
} catch (DataLoadException e) {
    System.out.println(e.getMessage());            // ผลลัพธ์: โหลดข้อมูลอายุไม่สำเร็จ
    System.out.println(e.getCause().getMessage()); // ผลลัพธ์: For input string: "ยี่สิบ"
}
```

Stack trace จะแสดงทั้งสองชั้น:

```
DataLoadException: โหลดข้อมูลอายุไม่สำเร็จ
    at Main.loadAge(Main.java:12)
    ...
Caused by: java.lang.NumberFormatException: For input string: "ยี่สิบ"
    ...
```

> 💡 **เทียบกับ JavaScript:** ES2022 เพิ่ม `new Error("msg", { cause: err })` ซึ่งเป็นแนวคิดเดียวกัน

---

## 8. เทียบกับ JavaScript: try/catch และ Promise Rejection

```js
// JavaScript
try {
    JSON.parse("{bad json");
} catch (err) {               // catch ได้ทุกอย่าง แยกประเภทเองด้วย instanceof
    if (err instanceof SyntaxError) console.log("JSON ผิด");
}

// Async — error มาในรูป Promise rejection
fetch("/api/users/99")
    .then(res => res.json())
    .catch(err => console.log("ผิดพลาด", err));

// หรือ async/await
try {
    await fetchUser(99);
} catch (err) { /* ... */ }
```

```java
// Java — แยก catch ตาม type ได้ตรง ๆ
try {
    Integer.parseInt("abc");
} catch (NumberFormatException e) {
    System.out.println("ตัวเลขผิด");
}

// Async — CompletableFuture มี exceptionally() คล้าย .catch()
java.util.concurrent.CompletableFuture
        .supplyAsync(() -> { throw new IllegalStateException("โหลดไม่ได้"); })
        .exceptionally(ex -> { System.out.println("ผิดพลาด: " + ex.getCause().getMessage()); return null; })
        .join();
// ผลลัพธ์: ผิดพลาด: โหลดไม่ได้
```

| หัวข้อ                    | JavaScript                        | Java                                 |
| ------------------------ | --------------------------------- | ------------------------------------ |
| โยนอะไรได้                 | ทุกอย่าง (`throw "oops"`)          | เฉพาะ `Throwable`                     |
| catch หลายแบบ             | catch เดียว + `instanceof`         | หลาย `catch` แยกตาม type / multi-catch |
| บังคับจัดการ                | ไม่มี                               | checked exception                    |
| ปิด resource อัตโนมัติ        | `using` (ใหม่) / `finally`         | try-with-resources                   |
| Error chaining           | `{ cause }` (ES2022)              | `super(message, cause)`              |
| Async error              | Promise rejection / `await` + try | `CompletableFuture.exceptionally()` หรือ try/catch ใน thread นั้น |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **อย่ากลืน exception (swallow)** — catch แล้วไม่ทำอะไรเลยทำให้ bug หายเงียบ อย่างน้อยต้อง log หรือโยนต่อ
2. **อย่า catch `Exception` หรือ `Throwable` แบบกว้าง ๆ** — catch เฉพาะที่จัดการได้จริง (ยกเว้นที่จุดบนสุด เช่น global handler)
3. **ใช้ try-with-resources** กับทุกอย่างที่เป็น `AutoCloseable`
4. **แนบ cause เสมอ** เมื่อห่อ exception ใหม่
5. **โยนเร็ว จับช้า (Throw early, catch late)** — ตรวจ input ตั้งแต่ต้น method แต่จัดการ exception ในชั้นที่รู้ว่าควรทำอะไร
6. **ใช้ exception มาตรฐานก่อน** (`IllegalArgumentException`, `IllegalStateException`) สร้าง custom เมื่อมีความหมายเชิง domain
7. **อย่าใช้ exception ควบคุม flow ปกติ** — exception มีต้นทุนสูง (สร้าง stack trace)
8. **ข้อความ error ต้องมีบริบท** — บอกค่าที่ผิดด้วย เช่น `"User id=99 ไม่พบในระบบ"` ไม่ใช่แค่ `"error"`

```java
// ❌ แย่
try {
    process();
} catch (Exception e) {
    // เงียบ...
}

// ✅ ดี
try {
    process();
} catch (IOException e) {
    throw new DataLoadException("ประมวลผลไฟล์ไม่สำเร็จ", e);
}
```

---
