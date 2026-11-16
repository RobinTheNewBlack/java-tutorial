# Advanced

# ส่วนที่ 4: การอ่านและเขียนไฟล์ (File I/O ด้วย `java.nio.file`)

## ภาพรวม

Java มี API จัดการไฟล์ 2 รุ่น: **`java.io.File`** (รุ่นเก่า) และ **`java.nio.file`** (NIO.2, Java 7+) ที่ควรใช้ในโค้ดใหม่ หัวใจคือ **`Path`** (ตำแหน่งไฟล์) และ **`Files`** (static method ที่ทำทุกอย่างกับไฟล์)

> 💡 **เทียบกับ JavaScript:** Node ส่ง **string** เป็น path ให้ `fs` ตรง ๆ ส่วน Java แยก "ตำแหน่ง" (`Path`) ออกจาก "การกระทำ" (`Files`) และ method ของ `Files` เป็น **blocking** ทั้งหมด คล้าย `fs.readFileSync` ไม่ใช่ `fs/promises` — ใน Java 21 ถ้าต้องการ concurrency ให้รันบน **virtual threads** แทนการเขียน async

---

## 1. Path — ตำแหน่งของไฟล์

```java
import java.nio.file.*;

Path a = Path.of("data", "products.csv");     // ✅ Java 11+ — ต่อชิ้นส่วนให้ตาม OS
Path b = Paths.get("data/products.csv");      // แบบเก่า (Java 7) ผลเหมือนกัน, ใช้ / ได้แม้บน Windows
Path cwd = Path.of("").toAbsolutePath();      // working directory ปัจจุบัน

System.out.println(a);                 // Windows: data\products.csv   Unix: data/products.csv
System.out.println(a.getFileName());   // ผลลัพธ์: products.csv
System.out.println(a.getParent());     // ผลลัพธ์: data
```

> **หมายเหตุ:** สร้าง `Path` **ไม่ได้แตะดิสก์** — ไฟล์ไม่จำเป็นต้องมีอยู่จริง

### resolve / relativize / normalize

```java
Path base = Path.of("/home/somchai/project");

System.out.println(base.resolve("src/Main.java"));  // ผลลัพธ์: /home/somchai/project/src/Main.java
System.out.println(base.resolve("/etc/hosts"));     // ผลลัพธ์: /etc/hosts  ← ตัวที่ต่อเป็น absolute ได้ตัวนั้นเลย
System.out.println(base.resolveSibling("other"));   // ผลลัพธ์: /home/somchai/other

Path target = Path.of("/home/somchai/docs/report.txt");
System.out.println(base.relativize(target));        // ผลลัพธ์: ../docs/report.txt

System.out.println(Path.of("a/./b/../c").normalize()); // ผลลัพธ์: a/c
// (บน Windows ตัวคั่นที่พิมพ์ออกมาจะเป็น \ เช่น ..\docs\report.txt)
```

> **ข้อควรระวัง:** `relativize` ต้องใช้ path ชนิดเดียวกัน (absolute ทั้งคู่ หรือ relative ทั้งคู่) ไม่งั้นได้ `IllegalArgumentException`

### Absolute vs Relative และ Windows vs Unix

```java
Path rel = Path.of("data/products.csv");
System.out.println(rel.isAbsolute());       // ผลลัพธ์: false
System.out.println(rel.toAbsolutePath());   // ต่อกับ working directory เช่น D:\learn\java\data\products.csv

System.out.println(FileSystems.getDefault().getSeparator()); // Windows: \   Unix: /

String bad = "data" + "\\" + "products.csv"; // ❌ ต่อ string เอง — พังบน Linux
Path good = Path.of("data", "products.csv");  // ✅
```

> **Tip:** relative path คำนวณจาก **working directory ตอนรัน** (ไม่ใช่ตำแหน่งไฟล์ `.java`) — รันจาก IDE กับ terminal อาจได้ผลต่างกัน และ Windows ไม่สนตัวพิมพ์เล็ก/ใหญ่ (`A.txt` = `a.txt`) แต่ Linux สนใจ

> 💡 **เทียบกับ JavaScript:**
>
> | Node `path`           | Java `Path`                      |
> | --------------------- | -------------------------------- |
> | `path.join(a, b)`     | `Path.of(a, b)` / `a.resolve(b)` |
> | `path.resolve(p)`     | `Path.of(p).toAbsolutePath()`    |
> | `path.relative(a, b)` | `a.relativize(b)`                |
> | `path.normalize(p)`   | `p.normalize()`                  |
> | `path.basename(p)` / `path.dirname(p)` | `p.getFileName()` / `p.getParent()` |
> | `path.sep` / `process.cwd()` | `FileSystems.getDefault().getSeparator()` / `Path.of("").toAbsolutePath()` |

---

## 2. อ่าน/เขียนไฟล์ขนาดเล็ก

สำหรับไฟล์ที่ใส่ memory ได้ (config, JSON, ข้อความไม่กี่ MB) — บรรทัดเดียวจบ

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.List;

public class Main {
    public static void main(String[] args) throws IOException {
        Path file = Path.of("hello.txt");

        Files.writeString(file, "สวัสดี Java\n");        // สร้างใหม่ หรือเขียนทับ
        System.out.print(Files.readString(file));       // ผลลัพธ์: สวัสดี Java

        Files.write(file, List.of("บรรทัด 1", "บรรทัด 2")); // เขียนเป็นรายบรรทัด
        List<String> lines = Files.readAllLines(file);
        System.out.println(lines);                      // ผลลัพธ์: [บรรทัด 1, บรรทัด 2]

        byte[] bytes = Files.readAllBytes(file);        // ไฟล์ binary (รูป, PDF)
    }
}
```

### StandardOpenOption

| Option              | ความหมาย                                                | Node flag |
| ------------------- | ------------------------------------------------------ | --------- |
| `CREATE`            | สร้างไฟล์ถ้ายังไม่มี                                         | —         |
| `CREATE_NEW`        | สร้างใหม่เท่านั้น — มีอยู่แล้วโยน `FileAlreadyExistsException` | `'wx'`    |
| `TRUNCATE_EXISTING` | ล้างเนื้อหาเดิมก่อนเขียน                                      | `'w'`     |
| `APPEND`            | เขียนต่อท้าย                                              | `'a'`     |

ถ้าไม่ระบุ option ค่า default ของ `writeString` / `write` คือ `CREATE, TRUNCATE_EXISTING, WRITE` (เขียนทับ)

```java
Path log = Path.of("app.log");

// ✅ append ต้องมี CREATE ด้วย
Files.writeString(log, "ผู้ใช้ สมชาย login\n", StandardOpenOption.CREATE, StandardOpenOption.APPEND);

// ❌ APPEND อย่างเดียว ถ้าไฟล์ยังไม่มี → NoSuchFileException
// Files.writeString(Path.of("new.log"), "x", StandardOpenOption.APPEND);
```

> 💡 **เทียบกับ JavaScript:** `Files.readString(p)` ≈ `fs.readFileSync(p, 'utf8')`, `Files.writeString(p, s)` ≈ `fs.writeFileSync(p, s)`, `writeString(p, s, CREATE, APPEND)` ≈ `fs.appendFileSync(p, s)`

---

## 3. Encoding: UTF-8 และ Charset (สำคัญกับภาษาไทย)

ไฟล์คือ **byte** — การแปลง byte ↔ ตัวอักษรต้องรู้ **charset** ถ้าผิด ภาษาไทยจะกลายเป็น `à¸ªà¸§...` หรือ `???`

```java
import java.nio.charset.*;

String word = "สวัสดี";
System.out.println(word.length());                                    // ผลลัพธ์: 6 (จำนวน char)
System.out.println(word.getBytes(StandardCharsets.UTF_8).length);     // ผลลัพธ์: 18 (ไทย 1 ตัว = 3 byte)
System.out.println(word.getBytes(Charset.forName("TIS-620")).length); // ผลลัพธ์: 6
System.out.println(Charset.defaultCharset());                         // ผลลัพธ์: UTF-8 (Java 18+)
```

```java
// Files.* ใช้ UTF-8 เป็น default อยู่แล้ว แต่ระบุเองชัดเจนกว่า
Files.writeString(Path.of("thai.txt"), "ภาษาไทย", StandardCharsets.UTF_8);

// แปลงไฟล์ภาษาไทยแบบเก่า (TIS-620 / windows-874 จากโปรแกรมบัญชี, Excel รุ่นเก่า) เป็น UTF-8
Path legacy = Path.of("old-report.txt");
String text = Files.readString(legacy, Charset.forName("TIS-620"));   // ✅ อ่านด้วย charset ที่ถูก
Files.writeString(Path.of("new-report.txt"), text, StandardCharsets.UTF_8);

// ❌ อ่านไฟล์ TIS-620 ด้วย UTF-8 → MalformedInputException: Input length = 1
// Files.readString(legacy);
```

> **ข้อควรระวัง:** ก่อน Java 18 default charset ขึ้นกับ OS (Windows ไทยมักเป็น `windows-874`) ทำให้ `FileReader`, `FileWriter`, `getBytes()`, `new String(bytes)` ที่ไม่ระบุ charset ให้ผลต่างจาก Linux — Java 18+ (JEP 400) เป็น UTF-8 ทุก OS แล้ว แต่ **ระบุ charset เองไว้เสมอ**

> 💡 **เทียบกับ JavaScript:** Node รับ encoding เป็น string `'utf8'` และ **ไม่รองรับ TIS-620 ในตัว** (ต้องใช้ `iconv-lite`) ส่วน Java มีมาให้

---

## 4. ไฟล์ขนาดใหญ่ — Files.lines และ Buffered Reader/Writer

`readString` / `readAllLines` โหลด **ทั้งไฟล์** เข้า memory — ไฟล์ log หลาย GB จะ `OutOfMemoryError` ต้องอ่านทีละบรรทัด

```java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.stream.Stream;

Path log = Path.of("access.log");

// ✅ Files.lines — Stream แบบ lazy แต่ถือ file handle ไว้ ต้องปิดด้วย try-with-resources
try (Stream<String> lines = Files.lines(log, StandardCharsets.UTF_8)) {
    long errors = lines.filter(line -> line.contains("ERROR")).count();
    System.out.println("พบ ERROR " + errors + " บรรทัด");
}
// ❌ long n = Files.lines(log).count();  — handle รั่ว (Windows จะลบ/ย้ายไฟล์ไม่ได้)

// BufferedReader — คุม loop เอง (break, เก็บ state)
try (BufferedReader reader = Files.newBufferedReader(log, StandardCharsets.UTF_8)) {
    String line;
    while ((line = reader.readLine()) != null) {   // null = จบไฟล์
        if (line.startsWith("#")) continue;
        System.out.println(line);
    }
}

// BufferedWriter — เขียนไฟล์ใหญ่ทีละส่วน
try (BufferedWriter writer = Files.newBufferedWriter(Path.of("numbers.txt"), StandardCharsets.UTF_8)) {
    for (int i = 1; i <= 100_000; i++) {
        writer.write("รายการที่ " + i);
        writer.newLine();                          // ขึ้นบรรทัดตาม OS (Windows = \r\n)
    }
} // close() จะ flush buffer ลงดิสก์ให้
```

> 💡 **เทียบกับ JavaScript:** `Files.lines` ≈ `readline.createInterface({ input: fs.createReadStream(p) })` + `for await`, `newBufferedWriter` ≈ `fs.createWriteStream(p)` — แต่ Java เป็น blocking ธรรมดา ไม่ต้องรอ event `'line'` / `'finish'`

---

## 5. IOException และ try-with-resources

แทบทุก method ใน `Files` โยน **`IOException`** ซึ่งเป็น **checked exception** — ต้อง catch หรือประกาศ `throws` และ resource ทุกตัวต้องปิดด้วย try-with-resources (ดู [exception.md](./exception.md) หัวข้อ 2 และ 4)

Subclass ที่เจอบ่อย: `NoSuchFileException` (Node `ENOENT`), `FileAlreadyExistsException` (`EEXIST`), `DirectoryNotEmptyException` (`ENOTEMPTY`), `AccessDeniedException` (`EACCES`)

```java
static String loadConfig(Path path) {
    try {
        return Files.readString(path);
    } catch (NoSuchFileException e) {        // เฉพาะเจาะจงก่อน
        return "";                           // ไม่มี config → ใช้ค่าว่าง
    } catch (IOException e) {                // กว้างกว่าไว้ทีหลัง
        throw new UncheckedIOException("อ่าน config ไม่สำเร็จ: " + path, e);
    }
}
```

> **หมายเหตุ:** lambda ใน Stream (`map`, `forEach`) **โยน checked exception ไม่ได้** — `files.stream().map(Files::readString)` จึง compile ไม่ผ่าน ต้องห่อด้วย `try/catch` แล้วโยน `UncheckedIOException` (ดูตัวอย่างลบโฟลเดอร์ในหัวข้อ 6) และ `Files.lines` ก็โยน `UncheckedIOException` ออกมาเช่นกันเมื่ออ่านพลาดกลางทาง

---

## 6. จัดการโฟลเดอร์ (Directory)

```java
Files.createDirectories(Path.of("output", "reports", "2026")); // ✅ สร้างทุกชั้น (mkdir -p) มีอยู่แล้วไม่ error

// list — ชั้นเดียว (ต้องปิด, ลำดับไม่รับประกัน)
try (Stream<Path> entries = Files.list(Path.of("output"))) {
    entries.sorted().forEach(p -> System.out.println(p.getFileName()));
}

// walk — ทุกชั้นแบบ recursive (กำหนดความลึกได้)
try (Stream<Path> all = Files.walk(Path.of("src"), 5)) {
    all.filter(Files::isRegularFile).forEach(System.out::println);
}

// find — walk + เงื่อนไขที่เข้าถึง attribute ได้ทันที
try (Stream<Path> javaFiles = Files.find(Path.of("src"), 10,
        (path, attrs) -> attrs.isRegularFile() && path.toString().endsWith(".java"))) {
    System.out.println("ไฟล์ .java: " + javaFiles.count());
}

Files.delete(Path.of("old.txt"));               // ไม่มี → NoSuchFileException, โฟลเดอร์ไม่ว่าง → DirectoryNotEmptyException
boolean deleted = Files.deleteIfExists(Path.of("old.txt")); // ไม่มีก็ไม่ error
```

### ลบทั้งโฟลเดอร์แบบ recursive

Java **ไม่มี** `rm -rf` ในตัว — ต้องไล่ลบจาก "ลูก" ไป "พ่อ"

```java
import java.io.*;
import java.nio.file.*;
import java.util.Comparator;
import java.util.stream.Stream;

static void deleteRecursively(Path root) throws IOException {
    if (Files.notExists(root)) return;
    try (Stream<Path> paths = Files.walk(root)) {
        paths.sorted(Comparator.reverseOrder())   // path ลึกกว่ามาก่อน → ลบไฟล์ก่อนโฟลเดอร์
             .forEach(p -> {
                 try {
                     Files.delete(p);
                 } catch (IOException e) {
                     throw new UncheckedIOException(e); // lambda โยน checked ไม่ได้
                 }
             });
    } catch (UncheckedIOException e) {
        throw e.getCause();                       // แกะกลับเป็น IOException
    }
}
```

> 💡 **เทียบกับ JavaScript:** `createDirectories` ≈ `fs.mkdirSync(p, { recursive: true })`, `Files.list` ≈ `fs.readdirSync` (แต่ได้ `Path` เต็ม ไม่ใช่แค่ชื่อ), `Files.walk` ≈ `fs.readdirSync(p, { recursive: true })`, `deleteRecursively` ≈ `fs.rmSync(p, { recursive: true, force: true })`

---

## 7. Copy และ Move

```java
import static java.nio.file.StandardCopyOption.*;

Path src = Path.of("data", "products.csv");
Path backup = Path.of("backup", "products.csv");
Files.createDirectories(backup.getParent());

Files.copy(src, backup, REPLACE_EXISTING);                  // ไม่ใส่ option + ปลายทางมีอยู่ → FileAlreadyExistsException
Files.copy(src, backup, REPLACE_EXISTING, COPY_ATTRIBUTES); // คัดลอกวันที่แก้ไขด้วย
Files.move(backup, backup.resolveSibling("products-old.csv"), REPLACE_EXISTING); // ย้าย/เปลี่ยนชื่อ

// เขียนลง .tmp แล้ว ATOMIC_MOVE — ไม่มีใครอ่านเจอไฟล์ที่เขียนไม่เสร็จ
Path tmp = Path.of("summary.csv.tmp");
Files.writeString(tmp, "ข้อมูลใหม่");
Files.move(tmp, Path.of("summary.csv"), REPLACE_EXISTING, ATOMIC_MOVE);
```

> **ข้อควรระวัง:** `Files.copy` กับโฟลเดอร์จะ **สร้างแค่โฟลเดอร์เปล่า** ไม่คัดลอกของข้างใน — ต้องไล่ `Files.walk` แล้ว copy ทีละไฟล์ และ `ATOMIC_MOVE` ใช้ได้เฉพาะภายใน filesystem/drive เดียวกัน

---

## 8. ตรวจสอบไฟล์และ Attribute

```java
import java.nio.file.attribute.*;
import java.time.ZoneId;

Path p = Path.of("data", "products.csv");

Files.exists(p);          // มีอยู่ไหม
Files.isDirectory(p);     // false
Files.isRegularFile(p);   // true
Files.isReadable(p);      // อ่านได้ไหม

long bytes = Files.size(p);                        // ขนาดเป็น byte (ไม่ใช่จำนวนตัวอักษร!)
FileTime modified = Files.getLastModifiedTime(p);
System.out.println(modified.toInstant().atZone(ZoneId.of("Asia/Bangkok"))); // แปลงเป็น java.time

// อ่านหลาย attribute ในครั้งเดียว
BasicFileAttributes attrs = Files.readAttributes(p, BasicFileAttributes.class);
System.out.println(attrs.size() + " bytes, สร้างเมื่อ " + attrs.creationTime());
```

> **ข้อควรระวัง:** `if (Files.exists(p)) Files.delete(p);` มี race condition — ไฟล์อาจหายไประหว่างสองบรรทัด ใช้ `Files.deleteIfExists(p)` หรือ `CREATE_NEW` + catch exception แทน

> 💡 **เทียบกับ JavaScript:** `Files.exists` ≈ `fs.existsSync`, `Files.readAttributes` ≈ `fs.statSync` (`attrs.size()` ≈ `stats.size`, `attrs.lastModifiedTime()` ≈ `stats.mtime`)

---

## 9. อ่านไฟล์จาก Classpath (Resources)

ไฟล์ใน `src/main/resources` ถูก **รวมเข้าไปใน `.jar`** ตอน build — ไม่ใช่ไฟล์บนดิสก์อีกต่อไป ต้องอ่านเป็น `InputStream` ผ่าน classpath

```java
import java.io.*;
import java.nio.charset.StandardCharsets;

public class Main {
    public static void main(String[] args) throws IOException {
        // "/" นำหน้า = เริ่มจาก root ของ classpath (src/main/resources/data/provinces.txt)
        try (InputStream in = Main.class.getResourceAsStream("/data/provinces.txt")) {
            if (in == null) {   // ไม่พบ → คืน null ไม่ใช่ exception!
                throw new FileNotFoundException("ไม่พบ resource data/provinces.txt");
            }
            String text = new String(in.readAllBytes(), StandardCharsets.UTF_8);
            System.out.println(text.lines().count() + " จังหวัด");
        }
    }
}
```

> **ข้อควรระวัง:** ❌ `Path.of(Main.class.getResource("/data/provinces.txt").toURI())` ใช้ได้ตอนรันใน IDE แต่ **พังเมื่อรันจาก `.jar`** (`FileSystemNotFoundException`)

ใน **Spring Boot** ใช้ `Resource` — ทำงานได้ทั้งใน IDE และใน jar:

```java
@Value("classpath:data/provinces.txt")
private Resource provinces;   // org.springframework.core.io.Resource

String load() throws IOException {
    try (InputStream in = provinces.getInputStream()) {          // ✅
        return new String(in.readAllBytes(), StandardCharsets.UTF_8);
    }
    // ❌ provinces.getFile() — พังใน jar เหมือนกัน
}
```

> 💡 **เทียบกับ JavaScript:** Node ใช้ `path.join(__dirname, 'data/provinces.txt')` ส่วน Java ไม่มี `__dirname` (ไฟล์ `.class` อาจอยู่ใน jar) จึงใช้ classpath resource แทน

---

## 10. `java.io.File` — API รุ่นเก่า

```java
import java.io.File;

File f = new File("data/products.csv");
boolean ok = f.delete();                        // ❌ ล้มเหลวคืน false — ไม่บอกเหตุผล
File[] children = new File("data").listFiles(); // ❌ ไม่ใช่โฟลเดอร์ → null

java.nio.file.Path path = f.toPath();           // ✅ แปลงเป็น Path ทันที
File back = path.toFile();                      // ส่งให้ API เก่าที่ต้องการ File
```

**ยังเจอได้ที่:** library เก่า, `MultipartFile.transferTo(File)` ใน Spring (มี overload รับ `Path` แล้ว), Swing `JFileChooser`, ตัวอย่างโค้ดรุ่นเก่าบนเว็บ

---

## 11. ตัวอย่างจริง: CSV สินค้า → กรองด้วย Stream → เขียนไฟล์สรุป

```java
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.*;
import java.util.stream.*;

public class ProductReport {

    record Product(String name, String category, double price, int stock) {
        static Product fromCsv(String line) {
            String[] c = line.split(",");
            return new Product(c[1].strip(), c[2].strip(),
                    Double.parseDouble(c[3].strip()), Integer.parseInt(c[4].strip()));
        }
        double stockValue() { return price * stock; }
    }

    record CategorySummary(String category, long count, double totalValue) {}

    static List<Product> readProducts(Path csv) throws IOException {
        try (Stream<String> lines = Files.lines(csv, StandardCharsets.UTF_8)) {
            return lines.skip(1)                         // ข้าม header
                        .filter(line -> !line.isBlank()) // ข้ามบรรทัดว่าง
                        .map(Product::fromCsv)
                        .toList();
        }
    }

    static List<CategorySummary> summarize(List<Product> products) {
        return products.stream()
                .filter(p -> p.stock() > 0)              // เฉพาะสินค้าที่มีของ
                .collect(Collectors.groupingBy(Product::category))
                .entrySet().stream()
                .map(e -> new CategorySummary(e.getKey(), e.getValue().size(),
                        e.getValue().stream().mapToDouble(Product::stockValue).sum()))
                .sorted(Comparator.comparingDouble(CategorySummary::totalValue).reversed())
                .toList();
    }

    static void writeSummary(Path out, List<CategorySummary> summaries) throws IOException {
        Files.createDirectories(out.getParent());
        try (BufferedWriter w = Files.newBufferedWriter(out, StandardCharsets.UTF_8)) {
            w.write("หมวดหมู่,จำนวนสินค้า,มูลค่าสต็อก");
            w.newLine();
            for (CategorySummary s : summaries) {
                w.write("%s,%d,%.2f".formatted(s.category(), s.count(), s.totalValue()));
                w.newLine();
            }
        }
    }

    public static void main(String[] args) throws IOException {
        Path input = Path.of("data", "products.csv");
        Path output = Path.of("output", "summary.csv");

        Files.createDirectories(input.getParent());     // เตรียมไฟล์ตัวอย่าง
        Files.writeString(input, """
                id,name,category,price,stock
                1,กาแฟเย็น,เครื่องดื่ม,55.00,120
                2,ชาเขียว,เครื่องดื่ม,45.00,0
                3,ข้าวผัดกะเพรา,อาหาร,60.00,35
                4,ผัดไทย,อาหาร,70.00,20
                5,ขนมครก,ของหวาน,40.00,15
                6,บัวลอย,ของหวาน,35.00,0
                """, StandardCharsets.UTF_8);

        List<Product> products = readProducts(input);
        List<CategorySummary> summaries = summarize(products);
        writeSummary(output, summaries);

        System.out.println("อ่าน " + products.size() + " รายการ → สรุป " + summaries.size() + " หมวด");
        System.out.print(Files.readString(output, StandardCharsets.UTF_8));
    }
}
// ผลลัพธ์:
// อ่าน 6 รายการ → สรุป 3 หมวด
// หมวดหมู่,จำนวนสินค้า,มูลค่าสต็อก
// เครื่องดื่ม,1,6600.00
// อาหาร,2,3500.00
// ของหวาน,1,600.00
```

> **Tip:** `split(",")` ใช้ได้กับ CSV ง่าย ๆ เท่านั้น — ถ้าในช่องมี comma หรือ `"` (เช่น `"ข้าวมันไก่, ไก่ทอด"`) ให้ใช้ **Apache Commons CSV**, **OpenCSV** หรือ **Jackson CSV**

---

## 12. ข้อผิดพลาดที่พบบ่อย (Pitfalls)

| ❌ ผิดพลาด                                          | ✅ แก้ไข                                               |
| -------------------------------------------------- | ----------------------------------------------------- |
| ไม่ปิด `Files.lines` / `list` / `walk` / `find`       | ใส่ใน `try (Stream<...> s = ...)` เสมอ                  |
| `readAllLines` กับไฟล์หลาย GB                         | `Files.lines` หรือ `BufferedReader`                   |
| `APPEND` โดยไม่มี `CREATE`                             | `writeString(p, s, CREATE, APPEND)`                   |
| ต่อ path ด้วย `"dir" + "\\" + "file"`                 | `Path.of("dir", "file")`                              |
| ไม่ระบุ charset กับ `getBytes()`, `new String(bytes)` | ระบุ `StandardCharsets.UTF_8`                         |
| อ่าน resource ด้วย `Path`/`File`                      | `getResourceAsStream` / Spring `Resource`             |
| คิดว่า `Files.size()` คือจำนวนตัวอักษร                    | เป็น byte — ไทย 1 ตัว = 3 byte ใน UTF-8                 |
| ลบไฟล์บน Windows ไม่ได้ (`AccessDeniedException`)     | มักเพราะ stream/reader ยังไม่ปิด                          |

**BOM จาก Excel:** ไฟล์ "CSV UTF-8" ของ Excel มี BOM (`\uFEFF`) นำหน้า และ Java **ไม่ตัดให้** — column แรกจะกลายเป็น `"\uFEFFid"` ให้ตัดเอง:

```java
String header = Files.readAllLines(Path.of("from-excel.csv")).get(0);
if (header.startsWith("\uFEFF")) header = header.substring(1); // ✅ ตัด BOM
```

**ภาษาไทยเป็น `???` ใน console ของ Windows:** ไฟล์อาจถูกต้องแล้ว แต่ console แสดงไม่ได้ — รัน `chcp 65001` หรือ `java -Dstdout.encoding=UTF-8 Main` (Java 19+) แล้วตรวจเนื้อหาจริงด้วย editor

---

## 13. Cheat Sheet: Node `fs` → Java `Files`

| Node.js                                          | Java 21                                              |
| ------------------------------------------------ | ---------------------------------------------------- |
| `fs.readFileSync(p, 'utf8')`                     | `Files.readString(p)`                                |
| `fs.readFileSync(p)` (Buffer)                    | `Files.readAllBytes(p)`                              |
| `fs.writeFileSync(p, s)`                         | `Files.writeString(p, s)`                            |
| `fs.writeFileSync(p, s, { flag: 'wx' })`         | `Files.writeString(p, s, CREATE_NEW)`                |
| `fs.appendFileSync(p, s)`                        | `Files.writeString(p, s, CREATE, APPEND)`            |
| `readline` + `fs.createReadStream(p)`            | `Files.lines(p)` / `Files.newBufferedReader(p)`      |
| `fs.createWriteStream(p)`                        | `Files.newBufferedWriter(p)`                         |
| `fs.existsSync(p)`                               | `Files.exists(p)`                                    |
| `fs.statSync(p)`                                 | `Files.readAttributes(p, BasicFileAttributes.class)` |
| `fs.statSync(p).size` / `.mtime`                 | `Files.size(p)` / `Files.getLastModifiedTime(p)`     |
| `fs.mkdirSync(p, { recursive: true })`           | `Files.createDirectories(p)`                         |
| `fs.readdirSync(p)`                              | `Files.list(p)` (ต้องปิด)                              |
| `fs.readdirSync(p, { recursive: true })`         | `Files.walk(p)` (ต้องปิด)                              |
| `fs.unlinkSync(p)` / `fs.rmdirSync(p)`           | `Files.delete(p)`                                    |
| `fs.rmSync(p, { recursive: true, force: true })` | `Files.walk` + `reverseOrder()` + `delete` (หัวข้อ 6) |
| `fs.copyFileSync(a, b)`                          | `Files.copy(a, b, REPLACE_EXISTING)`                 |
| `fs.renameSync(a, b)`                            | `Files.move(a, b, REPLACE_EXISTING)`                 |
| `await fs.promises.readFile(p)`                  | `Files.readString(p)` บน virtual thread               |
| `err.code === 'ENOENT'` / `'EEXIST'`             | `catch (NoSuchFileException e)` / `FileAlreadyExistsException` |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ `Path` + `Files` เสมอ** — เจอ `java.io.File` ให้ `toPath()` ทันที
2. **ระบุ `StandardCharsets.UTF_8` ชัดเจน** — โดยเฉพาะเมื่อมีภาษาไทย
3. **ปิดทุก Stream/Reader/Writer ด้วย try-with-resources** — รวมถึง `Files.lines`, `list`, `walk`, `find`
4. **เลือกวิธีอ่านตามขนาดไฟล์** — เล็กใช้ `readString`, ใหญ่ใช้ `Files.lines` / `BufferedReader`
5. **สร้าง path ด้วย `Path.of(...)` / `resolve`** — ห้ามต่อ string ด้วย `/` หรือ `\` เอง
6. **อ่านไฟล์ใน jar ผ่าน classpath** (`getResourceAsStream`, Spring `Resource`) ไม่ใช่ `Path`
7. **จับ exception เฉพาะเจาะจง** และแนบ cause เมื่อห่อเป็น `UncheckedIOException`
8. **เขียนไฟล์สำคัญแบบ atomic** — เขียนลง `.tmp` แล้ว `Files.move(..., ATOMIC_MOVE)`

---
