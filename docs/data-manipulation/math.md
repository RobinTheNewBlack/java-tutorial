# Math

# ส่วนที่ 5: คณิตศาสตร์และตัวเลข

## ภาพรวม

Java มีเครื่องมือสำหรับงานตัวเลขหลายตัว

| เครื่องมือ                   | ใช้ทำอะไร                                         |
| ---------------------------- | ------------------------------------------------- |
| `Math`                       | ฟังก์ชันคณิตศาสตร์พื้นฐาน (static method ทั้งหมด) |
| `Random` / `ThreadLocalRandom` | สุ่มตัวเลข                                       |
| `BigDecimal`                 | ทศนิยมแม่นยำ — **เงิน**                           |
| `BigInteger`                 | จำนวนเต็มขนาดไม่จำกัด                              |
| `NumberFormat` / `String.format` | จัดรูปแบบการแสดงผลตัวเลข                      |

---

## 1. `Math` Class

### คุณสมบัติ

- อยู่ใน `java.lang` — **ไม่ต้อง import**
- ทุก method เป็น `static` — เรียกผ่าน `Math.xxx()` ได้เลย

### ค่าคงที่

```java
System.out.println(Math.PI);  // ผลลัพธ์: 3.141592653589793
System.out.println(Math.E);   // ผลลัพธ์: 2.718281828459045
```

### ค่าสัมบูรณ์ / ยกกำลัง / ราก

```java
Math.abs(-15);          // 15
Math.abs(-3.5);         // 3.5

Math.pow(2, 10);        // 1024.0 — คืน double เสมอ
Math.sqrt(144);         // 12.0
Math.cbrt(27);          // 3.0 — รากที่ 3
Math.hypot(3, 4);       // 5.0 — √(3² + 4²)

int squared = (int) Math.pow(5, 2); // 25 — ต้อง cast ถ้าต้องการ int
```

> 💡 **เทียบกับ JavaScript:** Java **ไม่มี operator `**`** — ใช้ `Math.pow(a, b)` ซึ่งคืน `double` เสมอ (ต้อง cast ถ้าต้องการ `int`)

### มากสุด / น้อยสุด

```java
Math.max(10, 20);       // 20
Math.min(3.5, 1.2);     // 1.2
Math.max(Math.max(4, 9), 7); // 9 — รับได้แค่ 2 ค่า

// หาค่ามากสุดจากหลายค่า
int[] nums = {4, 9, 7};
int max = java.util.Arrays.stream(nums).max().getAsInt(); // 9

Math.clamp(150, 0, 100); // 100 — บีบค่าให้อยู่ในช่วง (Java 21)
```

> 💡 **เทียบกับ JavaScript:** `Math.max(4, 9, 7)` ใน JS รับได้หลายค่า แต่ Java รับ **แค่ 2 ค่า** — ใช้ Stream หรือ `Collections.max(list)` แทน

### ปัดเศษ

| Method          | ความหมาย                    | `2.5`  | `-2.5` | `2.4`  | คืน type |
| --------------- | --------------------------- | ------ | ------ | ------ | -------- |
| `Math.round(x)` | ปัดครึ่งขึ้น (ไปทาง +∞)     | `3`    | `-2`   | `2`    | `long` (จาก double) / `int` (จาก float) |
| `Math.floor(x)` | ปัดลง                       | `2.0`  | `-3.0` | `2.0`  | `double` |
| `Math.ceil(x)`  | ปัดขึ้น                     | `3.0`  | `-2.0` | `3.0`  | `double` |
| `Math.rint(x)`  | ปัดไปหาเลขคู่เมื่ออยู่กึ่งกลาง | `2.0` | `-2.0` | `2.0`  | `double` |
| `(int) x`       | ตัดทศนิยม (ไปทาง 0)         | `2`    | `-2`   | `2`    | `int`    |

```java
long r = Math.round(4.6);          // 5 (long)
int r2 = (int) Math.round(4.6);    // 5 — cast เป็น int
double f = Math.floor(4.9);        // 4.0

// ปัดทศนิยม 2 ตำแหน่ง (สำหรับแสดงผล)
double price = 19.876;
double rounded = Math.round(price * 100) / 100.0;  // 19.88
```

### อื่น ๆ

```java
Math.signum(-5.0);      // -1.0 — เครื่องหมาย
Math.log(Math.E);       // 1.0 — ln
Math.log10(1000);       // 3.0
Math.sin(Math.PI / 2);  // 1.0
Math.toRadians(180);    // 3.141592653589793
Math.floorDiv(-7, 2);   // -4 — หารปัดลง (ต่างจาก -7 / 2 = -3)
Math.floorMod(-7, 2);   // 1  — mod ที่ได้ค่าบวก
```

---

## 2. สุ่มตัวเลข

### `Math.random()`

คืน `double` ในช่วง **[0.0, 1.0)**

```java
double r = Math.random();                       // เช่น 0.7231...
int dice = (int) (Math.random() * 6) + 1;       // 1 ถึง 6
```

### `Random` class

```java
import java.util.Random;

Random random = new Random();
random.nextInt(10);          // 0 ถึง 9
random.nextInt(1, 7);        // 1 ถึง 6 (Java 17+)
random.nextDouble();         // 0.0 ถึง < 1.0
random.nextBoolean();        // true / false

Random seeded = new Random(42);   // ใส่ seed → ได้ลำดับเดิมทุกครั้ง (ใช้ในการทดสอบ)
```

### `ThreadLocalRandom` — แนะนำสำหรับงานทั่วไป

```java
import java.util.concurrent.ThreadLocalRandom;

int otp = ThreadLocalRandom.current().nextInt(100_000, 1_000_000); // เลข 6 หลัก
double temp = ThreadLocalRandom.current().nextDouble(25.0, 35.0);
```

### สุ่มจาก list

```java
import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

List<String> names = List.of("สมชาย", "สมหญิง", "สมศรี");
String winner = names.get(ThreadLocalRandom.current().nextInt(names.size()));

List<String> shuffled = new ArrayList<>(names);
Collections.shuffle(shuffled);
```

> สำหรับงานด้านความปลอดภัย (token, รหัสผ่าน) ใช้ `java.security.SecureRandom` แทน

---

## 3. `BigDecimal` — สำหรับเงิน

### ทำไมห้ามใช้ `double` กับเงิน?

`double` เก็บเลขฐาน 2 ทำให้ทศนิยมบางค่าเก็บได้ไม่ตรง

```java
System.out.println(0.1 + 0.2);          // ผลลัพธ์: 0.30000000000000004
System.out.println(1.10 - 1.00);        // ผลลัพธ์: 0.10000000000000009
System.out.println(0.1 + 0.2 == 0.3);   // ผลลัพธ์: false
```

> 💡 **เทียบกับ JavaScript:** ปัญหาเดียวกับ `0.1 + 0.2` ใน JS เป๊ะ ๆ (ทั้งคู่ใช้ IEEE 754) แต่ Java มี **`BigDecimal`** ในตัว ไม่ต้องลง library เพิ่มเหมือน `decimal.js`

### สร้าง `BigDecimal`

```java
import java.math.BigDecimal;

BigDecimal a = new BigDecimal("0.1");   // ✅ สร้างจาก String
BigDecimal b = BigDecimal.valueOf(0.2); // ✅ valueOf(double) ใช้ได้
BigDecimal bad = new BigDecimal(0.1);   // ❌ ได้ 0.1000000000000000055511151231257827...

System.out.println(a.add(b));           // ผลลัพธ์: 0.3
```

### การคำนวณ — ใช้ method ไม่ใช่ operator

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

BigDecimal price = new BigDecimal("199.50");
BigDecimal qty = BigDecimal.valueOf(3);
BigDecimal vatRate = new BigDecimal("0.07");

BigDecimal subtotal = price.multiply(qty);                        // 598.50
BigDecimal vat = subtotal.multiply(vatRate)
        .setScale(2, RoundingMode.HALF_UP);                       // 41.90 (41.895 → 41.90)
BigDecimal total = subtotal.add(vat);                             // 640.40

System.out.println("ยอดรวม: " + total);    // ผลลัพธ์: ยอดรวม: 640.40

BigDecimal discount = total.subtract(new BigDecimal("40.40"));    // 600.00
```

### การหาร — ต้องระบุ scale และ RoundingMode

```java
BigDecimal ten = new BigDecimal("10");
BigDecimal three = new BigDecimal("3");

// ten.divide(three);  // ArithmeticException: Non-terminating decimal expansion
BigDecimal result = ten.divide(three, 2, RoundingMode.HALF_UP);
System.out.println(result); // ผลลัพธ์: 3.33
```

### RoundingMode ที่ใช้บ่อย

| RoundingMode   | ความหมาย                                   | `2.345 → 2 ตำแหน่ง` | `2.355` |
| -------------- | ------------------------------------------ | ------------------- | ------- |
| `HALF_UP`      | ปัดครึ่งขึ้น (แบบที่เรียนในโรงเรียน)       | `2.35`              | `2.36`  |
| `HALF_EVEN`    | ปัดครึ่งไปเลขคู่ (Banker's rounding)       | `2.34`              | `2.36`  |
| `HALF_DOWN`    | ปัดครึ่งลง                                 | `2.34`              | `2.35`  |
| `UP`           | ปัดออกจาก 0 เสมอ                           | `2.35`              | `2.36`  |
| `DOWN`         | ตัดทิ้ง (เข้าหา 0)                         | `2.34`              | `2.35`  |
| `CEILING`      | ปัดไปทาง +∞                                | `2.35`              | `2.36`  |
| `FLOOR`        | ปัดไปทาง -∞                                | `2.34`              | `2.35`  |

### ⚠️ เปรียบเทียบด้วย `compareTo` ไม่ใช่ `equals`

```java
BigDecimal x = new BigDecimal("2.0");
BigDecimal y = new BigDecimal("2.00");

System.out.println(x.equals(y));          // ผลลัพธ์: false ⚠️ (scale ต่างกัน)
System.out.println(x.compareTo(y) == 0);  // ผลลัพธ์: true ✅

BigDecimal balance = new BigDecimal("500");
if (balance.compareTo(new BigDecimal("1000")) < 0) {
    System.out.println("ยอดเงินไม่พอ");
}
```

| ต้องการ   | เขียน                      |
| --------- | -------------------------- |
| `a == b`  | `a.compareTo(b) == 0`      |
| `a > b`   | `a.compareTo(b) > 0`       |
| `a < b`   | `a.compareTo(b) < 0`       |
| `a >= b`  | `a.compareTo(b) >= 0`      |

---

## 4. `BigInteger` — จำนวนเต็มไม่จำกัดขนาด

```java
import java.math.BigInteger;

BigInteger big = new BigInteger("123456789012345678901234567890");
BigInteger two = BigInteger.TWO;            // Java 9+

BigInteger sum = big.add(BigInteger.ONE);
BigInteger power = two.pow(100);
System.out.println(power); // ผลลัพธ์: 1267650600228229401496703205376

// แฟกทอเรียล 30!
BigInteger fact = BigInteger.ONE;
for (int i = 2; i <= 30; i++) {
    fact = fact.multiply(BigInteger.valueOf(i));
}
System.out.println(fact); // ผลลัพธ์: 265252859812191058636308480000000
```

> 💡 **เทียบกับ JavaScript:** `BigInteger` ≈ `BigInt` ของ JS (`123n`) แต่ใช้ method (`add`, `multiply`) แทน operator `+`, `*`

---

## 5. Overflow และ `Math.*Exact`

`int` และ `long` ล้นแบบ **เงียบ ๆ** (วนกลับเป็นค่าติดลบ) ไม่มี error

```java
int max = Integer.MAX_VALUE;           // 2147483647
System.out.println(max + 1);           // ผลลัพธ์: -2147483648 ⚠️

int a = 50_000;
int b = 50_000;
System.out.println(a * b);             // ผลลัพธ์: -1794967296 ⚠️ (ควรเป็น 2,500,000,000)
System.out.println((long) a * b);      // ผลลัพธ์: 2500000000 ✅ cast ก่อนคูณ
```

### ใช้ `Math.addExact` / `multiplyExact` ให้ throw เมื่อล้น

```java
try {
    int result = Math.addExact(Integer.MAX_VALUE, 1);
} catch (ArithmeticException e) {
    System.out.println("ล้น: " + e.getMessage()); // ผลลัพธ์: ล้น: integer overflow
}

Math.multiplyExact(50_000L, 50_000L);   // 2500000000 (long ไม่ล้น)
Math.subtractExact(10, 3);              // 7
Math.toIntExact(3_000_000_000L);        // ArithmeticException — แปลง long → int อย่างปลอดภัย
```

> 💡 **เทียบกับ JavaScript:** JS ไม่มี integer overflow (ตัวเลขใหญ่จะเสียความแม่นยำหลัง `Number.MAX_SAFE_INTEGER` แทน) แต่ Java `int` ล้นที่ ~2.1 พันล้าน — ระวังเวลาคูณตัวเลขใหญ่

---

## 6. จัดรูปแบบตัวเลข

### `String.format` / `printf`

```java
double amount = 1234567.891;

String.format("%.2f", amount);       // "1234567.89"
String.format("%,.2f", amount);      // "1,234,567.89"
String.format("%,d", 1_000_000);     // "1,000,000"
String.format("%08.2f", 3.14159);    // "00003.14"
String.format("%.1f%%", 45.678);     // "45.7%" — %% คือเครื่องหมาย %
System.out.printf("ราคา %,.2f บาท%n", amount); // ผลลัพธ์: ราคา 1,234,567.89 บาท
```

### `NumberFormat` — จัดรูปแบบตาม locale

```java
import java.text.NumberFormat;
import java.util.Locale;

Locale th = Locale.of("th", "TH");    // Java 19+ (เวอร์ชันเก่าใช้ new Locale("th", "TH"))

NumberFormat currency = NumberFormat.getCurrencyInstance(th);
System.out.println(currency.format(1234.5));   // ผลลัพธ์: ฿1,234.50

NumberFormat percent = NumberFormat.getPercentInstance();
System.out.println(percent.format(0.256));     // ผลลัพธ์: 26%

NumberFormat number = NumberFormat.getNumberInstance(Locale.US);
number.setMaximumFractionDigits(2);
System.out.println(number.format(9876.54321)); // ผลลัพธ์: 9,876.54

NumberFormat compact = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
System.out.println(compact.format(1_500_000)); // ผลลัพธ์: 2M (Java 12+)
```

> 💡 **เทียบกับ JavaScript:** `NumberFormat` ≈ `Intl.NumberFormat` และ `"%.2f".formatted(x)` ≈ `x.toFixed(2)`

---

## ตารางเปรียบเทียบกับ JavaScript

| JavaScript                       | Java                                            |
| -------------------------------- | ----------------------------------------------- |
| `Math.abs(x)`                    | `Math.abs(x)`                                   |
| `x ** y` / `Math.pow(x, y)`      | `Math.pow(x, y)` (คืน `double`)                 |
| `Math.max(a, b, c)`              | `Math.max(a, Math.max(b, c))` / Stream          |
| `Math.round(2.5)` → `3`          | `Math.round(2.5)` → `3` (คืน `long`)            |
| `Math.trunc(x)`                  | `(int) x` / `(long) x`                          |
| `Math.random()`                  | `Math.random()` / `ThreadLocalRandom`           |
| `x.toFixed(2)`                   | `String.format("%.2f", x)`                      |
| `parseInt("42")`                 | `Integer.parseInt("42")`                        |
| `Number.MAX_SAFE_INTEGER`        | `Integer.MAX_VALUE` / `Long.MAX_VALUE`          |
| `123n` (BigInt)                  | `BigInteger`                                    |
| ไม่มี (ใช้ library)               | `BigDecimal`                                    |
| `Intl.NumberFormat`              | `NumberFormat`                                  |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ `BigDecimal` กับเงินเสมอ** และสร้างจาก `String` หรือ `BigDecimal.valueOf()` — ไม่ใช่ `new BigDecimal(double)`
2. **ระบุ `RoundingMode`** ทุกครั้งที่ `divide` หรือ `setScale`
3. **เทียบ `BigDecimal` ด้วย `compareTo`** ไม่ใช่ `equals`
4. **ระวัง overflow** — cast เป็น `long` ก่อนคูณ หรือใช้ `Math.multiplyExact`
5. **ใช้ `ThreadLocalRandom`** สำหรับสุ่มทั่วไป และ `SecureRandom` สำหรับงานความปลอดภัย
6. **แยกการคำนวณกับการแสดงผล** — คำนวณด้วยค่าเต็ม แล้วค่อยปัด/จัดรูปแบบตอนแสดงผล

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<BigDecimal> prices = List.of(
                new BigDecimal("59.90"),
                new BigDecimal("120.00"),
                new BigDecimal("35.50")
        );

        BigDecimal subtotal = prices.stream().reduce(BigDecimal.ZERO, BigDecimal::add);
        BigDecimal vat = subtotal.multiply(new BigDecimal("0.07")).setScale(2, RoundingMode.HALF_UP);
        BigDecimal total = subtotal.add(vat);

        System.out.printf("ยอดก่อน VAT: %,.2f%n", subtotal); // ผลลัพธ์: ยอดก่อน VAT: 215.40
        System.out.printf("VAT 7%%: %,.2f%n", vat);           // ผลลัพธ์: VAT 7%: 15.08
        System.out.printf("ยอดสุทธิ: %,.2f%n", total);        // ผลลัพธ์: ยอดสุทธิ: 230.48
    }
}
```

---
