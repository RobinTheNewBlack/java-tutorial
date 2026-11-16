# Advanced

# ส่วนที่ 3: วันที่และเวลา (Date & Time API)

## ภาพรวม

ตั้งแต่ **Java 8** มี package **`java.time`** ซึ่งเป็น API จัดการวันที่และเวลาที่ออกแบบมาใหม่ทั้งหมด

### คุณสมบัติ

- **Immutable** — ทุก method ที่ "แก้" ค่า จะคืน object ใหม่เสมอ (thread-safe)
- **แยก type ชัดเจน** — วันที่อย่างเดียว, เวลาอย่างเดียว, มี/ไม่มี timezone
- **เดือนเริ่มที่ 1** (มกราคม = 1) ไม่ใช่ 0
- รองรับ **ปฏิทินพุทธศักราช** (`ThaiBuddhistDate`) ในตัว

> 💡 **เทียบกับ JavaScript:** JS มีแค่ `Date` ตัวเดียว ซึ่ง **mutable**, เดือนเริ่มที่ 0 และผูกกับ timezone ของเครื่องเสมอ (ต้องพึ่ง `date-fns`, `dayjs` หรือรอ `Temporal` API) ส่วน `java.time` แก้ปัญหาเหล่านี้หมดแล้ว — จริง ๆ `Temporal` ของ JS ก็ได้แรงบันดาลใจมาจาก `java.time`

---

## 1. เลือกใช้ Type ไหนดี?

| Type             | เก็บอะไร                                 | ตัวอย่าง                                    | ใช้เมื่อ                                  |
| ---------------- | -------------------------------------- | ------------------------------------------ | --------------------------------------- |
| `LocalDate`      | วันที่ (ไม่มีเวลา, ไม่มี timezone)          | `2026-10-10`                               | วันเกิด, วันหยุด, วันครบกำหนด              |
| `LocalTime`      | เวลา (ไม่มีวันที่)                          | `14:30`                                    | เวลาเปิด-ปิดร้าน                         |
| `LocalDateTime`  | วันที่ + เวลา (ไม่มี timezone)              | `2026-10-10T09:00`                         | นัดหมายในประเทศเดียว                      |
| `ZonedDateTime`  | วันที่ + เวลา + timezone                  | `2026-10-10T09:00+07:00[Asia/Bangkok]`     | เวลาที่ต้องแปลงข้ามประเทศ                  |
| `OffsetDateTime` | วันที่ + เวลา + offset (`+07:00`)          | `2026-10-10T09:00+07:00`                   | เก็บลง DB / ส่งผ่าน API                   |
| `Instant`        | จุดเวลาบนเส้นเวลาสากล (UTC)              | `2026-10-10T02:00:00Z`                     | timestamp, log, `createdAt`              |
| `Duration`       | ระยะเวลาแบบ ชั่วโมง/นาที/วินาที              | `PT2H15M`                                  | เวลาที่ใช้ทำงาน, timeout                  |
| `Period`         | ระยะเวลาแบบ ปี/เดือน/วัน                   | `P31Y6M25D`                                | อายุ, ระยะสัญญา                          |

---

## 2. LocalDate — วันที่

```java
import java.time.*;
import java.time.temporal.*;

public class Main {
    public static void main(String[] args) {
        LocalDate today = LocalDate.now();                // วันนี้ (ตาม timezone ของเครื่อง)
        LocalDate date = LocalDate.of(2026, 10, 10);      // เดือน 10 = ตุลาคม (ไม่ต้อง -1)
        LocalDate parsed = LocalDate.parse("2026-10-10"); // จาก ISO string

        System.out.println(date.getYear());       // ผลลัพธ์: 2026
        System.out.println(date.getMonth());      // ผลลัพธ์: OCTOBER
        System.out.println(date.getMonthValue()); // ผลลัพธ์: 10
        System.out.println(date.getDayOfWeek());  // ผลลัพธ์: SATURDAY
        System.out.println(date.getDayOfYear());  // ผลลัพธ์: 283
        System.out.println(date.lengthOfMonth()); // ผลลัพธ์: 31
        System.out.println(date.isLeapYear());    // ผลลัพธ์: false

        // คำนวณ — คืน object ใหม่เสมอ
        System.out.println(date.plusDays(30));    // ผลลัพธ์: 2026-11-09
        System.out.println(date.minusMonths(1));  // ผลลัพธ์: 2026-09-10
        System.out.println(date.withDayOfMonth(1)); // ผลลัพธ์: 2026-10-01
        System.out.println(date.with(TemporalAdjusters.lastDayOfMonth())); // ผลลัพธ์: 2026-10-31

        // เปรียบเทียบ
        LocalDate newYear = LocalDate.of(2026, 12, 31);
        System.out.println(date.isBefore(newYear)); // ผลลัพธ์: true
        System.out.println(date.equals(parsed));    // ผลลัพธ์: true
    }
}
```

### Immutable — ต้องรับค่าที่คืนกลับมา

```java
LocalDate d = LocalDate.of(2026, 10, 10);
d.plusDays(1);              // ❌ ไม่มีผลอะไร — ค่าใหม่ถูกทิ้งไป
System.out.println(d);      // ผลลัพธ์: 2026-10-10

d = d.plusDays(1);          // ✅ ต้องเก็บค่าใหม่
System.out.println(d);      // ผลลัพธ์: 2026-10-11
```

### บวกเดือนข้ามเดือนที่วันไม่พอ

```java
LocalDate jan31 = LocalDate.of(2026, 1, 31);
System.out.println(jan31.plusMonths(1)); // ผลลัพธ์: 2026-02-28 (ปรับเป็นวันสุดท้ายของเดือนให้)
```

> 💡 **เทียบกับ JavaScript:** ใน JS `new Date(2026, 0, 31)` แล้ว `setMonth(1)` จะได้ **3 มีนาคม** (ล้นไปเดือนถัดไป) — `java.time` จัดการให้ถูกต้องกว่า

---

## 3. LocalTime และ LocalDateTime

```java
LocalTime open = LocalTime.of(14, 30);
System.out.println(open.plusHours(10));     // ผลลัพธ์: 00:30 (วนรอบ 24 ชม.)
System.out.println(open.minusMinutes(45));  // ผลลัพธ์: 13:45
System.out.println(open.isAfter(LocalTime.NOON)); // ผลลัพธ์: true

LocalDateTime meeting = LocalDateTime.of(2026, 10, 10, 9, 0);
System.out.println(meeting);                // ผลลัพธ์: 2026-10-10T09:00
System.out.println(meeting.plusMinutes(90)); // ผลลัพธ์: 2026-10-10T10:30

// รวม/แยก
LocalDateTime combined = LocalDate.of(2026, 10, 10).atTime(LocalTime.of(9, 0));
LocalDate onlyDate = meeting.toLocalDate();  // 2026-10-10
LocalTime onlyTime = meeting.toLocalTime();  // 09:00
```

---

## 4. ZonedDateTime และ Timezone `Asia/Bangkok`

`LocalDateTime` **ไม่รู้ว่าเป็นเวลาของที่ไหน** — ถ้าต้องสื่อสารข้ามประเทศ หรือแปลงเป็น timestamp ต้องผูกกับ `ZoneId`

```java
ZoneId bangkok = ZoneId.of("Asia/Bangkok"); // UTC+7 (ไทยไม่มี daylight saving)

ZonedDateTime bkk = LocalDateTime.of(2026, 10, 10, 9, 0).atZone(bangkok);
System.out.println(bkk);
// ผลลัพธ์: 2026-10-10T09:00+07:00[Asia/Bangkok]

// แปลงเป็นเวลาท้องถิ่นของประเทศอื่น (ช่วงเวลาเดียวกัน)
System.out.println(bkk.withZoneSameInstant(ZoneId.of("Asia/Tokyo")));
// ผลลัพธ์: 2026-10-10T11:00+09:00[Asia/Tokyo]

System.out.println(bkk.withZoneSameInstant(ZoneId.of("Europe/London")));
// ผลลัพธ์: 2026-10-10T03:00+01:00[Europe/London]  (ช่วงนั้นลอนดอนใช้ BST +01:00)

System.out.println(bkk.withZoneSameInstant(ZoneOffset.UTC));
// ผลลัพธ์: 2026-10-10T02:00Z

// เวลาปัจจุบันในไทย ไม่ว่าเซิร์ฟเวอร์จะอยู่ที่ไหน
ZonedDateTime nowInThailand = ZonedDateTime.now(bangkok);
```

> **ข้อควรระวัง:** ใช้ชื่อ region (`"Asia/Bangkok"`) แทน offset ตายตัว (`"+07:00"`) เพราะ region รู้กฎ daylight saving และการเปลี่ยนแปลงในอดีต — สำคัญมากกับประเทศที่มี DST

> **Tip:** เซิร์ฟเวอร์บน cloud มักตั้ง timezone เป็น UTC — ถ้าใช้ `LocalDateTime.now()` จะได้เวลา UTC ไม่ใช่เวลาไทย ให้ระบุ `ZoneId` เสมอ

---

## 5. Instant — Timestamp

`Instant` คือจุดเวลาบนเส้นเวลาสากล (UTC) ไม่สนใจปฏิทินหรือ timezone เหมาะสำหรับเก็บ "เวลาที่เหตุการณ์เกิดขึ้น"

```java
Instant now = Instant.now();
System.out.println(now); // เช่น 2026-10-10T02:00:00.123456Z

Instant fromBkk = bkk.toInstant();
System.out.println(fromBkk);                // ผลลัพธ์: 2026-10-10T02:00:00Z
System.out.println(fromBkk.toEpochMilli()); // ผลลัพธ์: 1791597600000

// แปลงกลับเป็นเวลาไทยเพื่อแสดงผล
Instant fromApi = Instant.parse("2026-10-10T02:00:00Z");
System.out.println(fromApi.atZone(ZoneId.of("Asia/Bangkok")));
// ผลลัพธ์: 2026-10-10T09:00+07:00[Asia/Bangkok]

System.out.println(Instant.ofEpochMilli(0)); // ผลลัพธ์: 1970-01-01T00:00:00Z
```

> 💡 **เทียบกับ JavaScript:** `Instant.now().toEpochMilli()` เทียบเท่า `Date.now()` และ `Instant.parse(...)` คล้าย `new Date("...Z")`

### แนวทางการเก็บเวลา

| สถานการณ์                       | Type ที่แนะนำ                     |
| ------------------------------ | -------------------------------- |
| `createdAt` / `updatedAt` ใน DB | `Instant` หรือ `OffsetDateTime` |
| ส่งผ่าน JSON API                 | ISO-8601 string (`2026-10-10T02:00:00Z`) |
| แสดงผลให้ผู้ใช้                    | แปลงเป็น `ZonedDateTime` ตาม timezone ผู้ใช้ |
| วันเกิด, วันหยุดราชการ             | `LocalDate`                      |

---

## 6. Duration และ Period

| หัวข้อ     | `Duration`                       | `Period`                       |
| --------- | -------------------------------- | ------------------------------ |
| หน่วย      | วินาที/นาที/ชั่วโมง (เวลาจริง)         | ปี/เดือน/วัน (ตามปฏิทิน)          |
| ใช้กับ     | `LocalTime`, `LocalDateTime`, `Instant` | `LocalDate`              |
| รูปแบบ     | `PT2H15M`                        | `P1Y2M3D`                      |

```java
import java.time.*;
import java.time.temporal.ChronoUnit;

// Duration — ชั่วโมงทำงาน
Duration work = Duration.between(LocalTime.of(8, 30), LocalTime.of(17, 45));
System.out.println(work);                 // ผลลัพธ์: PT9H15M
System.out.println(work.toMinutes());     // ผลลัพธ์: 555

Duration d = Duration.ofMinutes(135);
System.out.println(d.toHours() + " ชม. " + d.toMinutesPart() + " นาที"); // ผลลัพธ์: 2 ชม. 15 นาที (toMinutesPart ต้อง Java 9+)

// Period — คำนวณอายุ
LocalDate birthday = LocalDate.of(1995, 3, 15);
Period age = Period.between(birthday, LocalDate.of(2026, 10, 10));
System.out.println(age); // ผลลัพธ์: P31Y6M25D
System.out.println("สมชายอายุ " + age.getYears() + " ปี " + age.getMonths() + " เดือน " + age.getDays() + " วัน");
// ผลลัพธ์: สมชายอายุ 31 ปี 6 เดือน 25 วัน

// ChronoUnit — นับจำนวนหน่วยทั้งหมดระหว่างสองวัน
long daysLeft = ChronoUnit.DAYS.between(LocalDate.of(2026, 10, 10), LocalDate.of(2026, 12, 31));
System.out.println("อีก " + daysLeft + " วันจะสิ้นปี"); // ผลลัพธ์: อีก 82 วันจะสิ้นปี
```

> **ข้อควรระวัง:** `Period.getDays()` คือ "ส่วนที่เป็นวัน" (25) **ไม่ใช่จำนวนวันทั้งหมด** — ถ้าต้องการจำนวนวันทั้งหมดใช้ `ChronoUnit.DAYS.between(...)`

---

## 7. DateTimeFormatter — Format และ Parse

### Pattern ที่ใช้บ่อย

| Pattern | ความหมาย                  | ตัวอย่าง            |
| ------- | ------------------------ | ----------------- |
| `yyyy`  | ปี (ของ era)              | `2026`            |
| `uuuu`  | ปี (proleptic — ใช้กับ STRICT) | `2026`       |
| `MM`    | เดือนเป็นตัวเลข             | `10`              |
| `MMM` / `MMMM` | ชื่อเดือนย่อ / เต็ม    | `ต.ค.` / `ตุลาคม` |
| `dd`    | วันที่                      | `10`              |
| `EEEE`  | ชื่อวัน                     | `วันเสาร์`          |
| `HH`    | ชั่วโมง 0–23               | `09`              |
| `hh` + `a` | ชั่วโมง 1–12 + AM/PM     | `09 AM`           |
| `mm`    | นาที                      | `00`              |
| `ss`    | วินาที                     | `00`              |
| `G`     | ศักราช                    | `ค.ศ.` / `พ.ศ.`    |

> **ข้อควรระวัง:** `MM` = เดือน, `mm` = นาที, `HH` = 24 ชม., `hh` = 12 ชม. — สลับกันบ่อยมาก!

### Format (วันที่ → String)

```java
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.util.Locale;

LocalDateTime dt = LocalDateTime.of(2026, 10, 10, 9, 0);
Locale thai = Locale.forLanguageTag("th-TH");

System.out.println(dt.format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm")));
// ผลลัพธ์: 10/10/2026 09:00

System.out.println(dt.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
// ผลลัพธ์: 2026-10-10T09:00:00

System.out.println(dt.format(DateTimeFormatter.ofPattern("EEEE d MMMM yyyy", thai)));
// ผลลัพธ์: วันเสาร์ 10 ตุลาคม 2026

System.out.println(dt.format(DateTimeFormatter.ofPattern("d MMM yyyy HH:mm", Locale.ENGLISH)));
// ผลลัพธ์: 10 Oct 2026 09:00
```

### Parse (String → วันที่)

```java
import java.time.format.*;

DateTimeFormatter thaiInput = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate d = LocalDate.parse("10/10/2026", thaiInput);
System.out.println(d); // ผลลัพธ์: 2026-10-10

// รูปแบบไม่ตรง → DateTimeParseException
try {
    LocalDate.parse("2026/10/10"); // ค่า default คาดหวัง ISO (yyyy-MM-dd)
} catch (DateTimeParseException e) {
    System.out.println(e.getMessage()); // ผลลัพธ์: Text '2026/10/10' could not be parsed at index 4
}

// ค่า default (SMART) จะปรับวันที่ไม่มีจริงให้เอง
System.out.println(LocalDate.parse("31/02/2026", thaiInput)); // ผลลัพธ์: 2026-02-28

// STRICT — ปฏิเสธวันที่ไม่มีจริง (ต้องใช้ uuuu แทน yyyy)
DateTimeFormatter strict = DateTimeFormatter.ofPattern("dd/MM/uuuu")
        .withResolverStyle(ResolverStyle.STRICT);
try {
    LocalDate.parse("31/02/2026", strict);
} catch (DateTimeParseException e) {
    System.out.println(e.getMessage());
    // ผลลัพธ์: Text '31/02/2026' could not be parsed: Invalid date 'FEBRUARY 31'
}
```

> **Tip:** `DateTimeFormatter` เป็น immutable และ thread-safe — ประกาศเป็น `static final` แล้วใช้ซ้ำได้ (ต่างจาก `SimpleDateFormat` แบบเก่าที่ **ไม่ thread-safe**)

---

## 8. ปฏิทินพุทธศักราช — `ThaiBuddhistDate`

Java รองรับปฏิทินไทย (พ.ศ. = ค.ศ. + 543) ในตัวผ่าน `java.time.chrono`

```java
import java.time.*;
import java.time.chrono.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoField;
import java.util.Locale;

public class Main {
    public static void main(String[] args) {
        LocalDate date = LocalDate.of(2026, 10, 10);
        Locale thai = Locale.forLanguageTag("th-TH");

        // แปลง ค.ศ. → พ.ศ.
        ThaiBuddhistDate thaiDate = ThaiBuddhistDate.from(date);
        System.out.println(thaiDate);                       // ผลลัพธ์: ThaiBuddhist BE 2569-10-10
        System.out.println(thaiDate.get(ChronoField.YEAR)); // ผลลัพธ์: 2569

        // แปลง พ.ศ. → ค.ศ.
        LocalDate fromThai = LocalDate.from(ThaiBuddhistDate.of(2569, 1, 1));
        System.out.println(fromThai);                       // ผลลัพธ์: 2026-01-01

        // Format เป็นภาษาไทยแบบ พ.ศ. — ใช้ withChronology
        DateTimeFormatter thaiFull = DateTimeFormatter
                .ofPattern("EEEE d MMMM G yyyy", thai)
                .withChronology(ThaiBuddhistChronology.INSTANCE);
        System.out.println(date.format(thaiFull)); // ผลลัพธ์: วันเสาร์ 10 ตุลาคม พ.ศ. 2569

        DateTimeFormatter thaiShort = DateTimeFormatter
                .ofPattern("d MMM yy", thai)
                .withChronology(ThaiBuddhistChronology.INSTANCE);
        System.out.println(date.format(thaiShort)); // ผลลัพธ์: 10 ต.ค. 69
    }
}
```

> **ข้อควรระวัง:** **เก็บข้อมูลเป็น ค.ศ. (`LocalDate`) เสมอ** แล้วแปลงเป็น พ.ศ. เฉพาะตอนแสดงผลหรือรับ input — การเก็บปี พ.ศ. ลง database มักทำให้เกิดบั๊ก "ปี 2569 + 543" ซ้ำซ้อน

> **หมายเหตุ:** ระวัง default locale ของเครื่อง/เซิร์ฟเวอร์ที่ตั้งเป็น `th-TH` — บาง API แบบเก่า (`Calendar.getInstance()`) จะได้ปฏิทินพุทธศักราชโดยไม่ตั้งใจ ควรระบุ `Locale` ให้ชัดเจนเสมอ

---

## 9. Legacy API — `Date` และ `Calendar` (ทำไมควรเลี่ยง)

ก่อน Java 8 ใช้ `java.util.Date`, `java.util.Calendar` และ `java.text.SimpleDateFormat` ซึ่งมีปัญหามาก

```java
import java.util.*;

Date date = new Date(126, 9, 10);  // ⚠️ deprecated — ปีเริ่มนับจาก 1900, เดือนเริ่มที่ 0
System.out.println(date.getYear()); // ผลลัพธ์: 126 (ไม่ใช่ 2026!)

Calendar cal = Calendar.getInstance();
cal.set(2026, 9, 10);              // เดือน 9 = ตุลาคม 😵
System.out.println(cal.get(Calendar.MONTH)); // ผลลัพธ์: 9
```

| ปัญหาของ API เก่า                   | `java.time` แก้อย่างไร                  |
| ---------------------------------- | -------------------------------------- |
| Mutable — ส่งต่อแล้วถูกแก้ค่าได้          | Immutable ทั้งหมด                       |
| เดือนเริ่มที่ 0, ปีนับจาก 1900          | เดือนเริ่มที่ 1, ปีตามจริง                   |
| `SimpleDateFormat` ไม่ thread-safe  | `DateTimeFormatter` thread-safe        |
| `Date` ชื่อว่า date แต่เก็บเวลาด้วย      | แยก `LocalDate`, `LocalTime`, `Instant` |
| จัดการ timezone สับสน               | `ZoneId`, `ZonedDateTime` ชัดเจน       |

### แปลงระหว่าง Legacy ↔ `java.time` (เมื่อต้องใช้กับ library เก่า)

```java
Date legacy = new Date();
Instant instant = legacy.toInstant();                     // Date → Instant
Date back = Date.from(instant);                           // Instant → Date

LocalDateTime ldt = LocalDateTime.ofInstant(instant, ZoneId.of("Asia/Bangkok"));
```

---

## 10. เปรียบเทียบกับ JavaScript `Date`

```js
// JavaScript
const d = new Date(2026, 9, 10);          // เดือน 9 = ตุลาคม (เริ่มที่ 0)
d.setDate(d.getDate() + 30);               // แก้ object เดิม (mutable)
d.toLocaleDateString("th-TH");             // "9/11/2569" — th-TH ใช้ พ.ศ. อัตโนมัติ
d.toISOString();                           // แปลงเป็น UTC เสมอ
Date.now();                                // epoch milliseconds
```

```java
// Java
LocalDate d = LocalDate.of(2026, 10, 10);  // เดือน 10 = ตุลาคม
d = d.plusDays(30);                        // คืน object ใหม่ (immutable)
d.format(DateTimeFormatter.ofPattern("d/M/yyyy")
        .withChronology(ThaiBuddhistChronology.INSTANCE)); // "9/11/2569"
Instant.now().toString();                  // ISO-8601 UTC
Instant.now().toEpochMilli();              // epoch milliseconds
```

| หัวข้อ               | JavaScript `Date`           | Java `java.time`                         |
| ------------------- | --------------------------- | ---------------------------------------- |
| จำนวน type           | 1 (`Date`)                   | หลาย type ตามการใช้งาน                     |
| Mutable             | ✅ (แก้ค่าเดิม)                 | ❌ (immutable)                            |
| เดือนแรก              | 0                           | 1                                        |
| Timezone            | ใช้ของเครื่องเสมอ หรือ UTC        | ระบุ `ZoneId` ได้ทุกที่                       |
| วันที่อย่างเดียว         | ไม่มี (มีเวลาติดมาเสมอ)           | `LocalDate`                              |
| ระยะเวลา             | ลบ milliseconds เอง          | `Duration`, `Period`, `ChronoUnit`       |
| Format              | `toLocaleString`, `Intl`    | `DateTimeFormatter`                      |
| ปฏิทินไทย             | `th-TH` locale              | `ThaiBuddhistChronology`                 |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ `java.time` เสมอ** — หลีกเลี่ยง `Date`, `Calendar`, `SimpleDateFormat` ในโค้ดใหม่
2. **เลือก type ให้ตรงความหมาย** — วันเกิดใช้ `LocalDate`, timestamp ใช้ `Instant`
3. **ระบุ `ZoneId` ชัดเจน** — `ZonedDateTime.now(ZoneId.of("Asia/Bangkok"))` แทน `LocalDateTime.now()` บนเซิร์ฟเวอร์
4. **เก็บเป็น UTC / ค.ศ. แปลงตอนแสดงผล** — ทั้ง timezone และปี พ.ศ.
5. **ใช้ ISO-8601 ใน API** — Spring Boot (Jackson) แปลง `java.time` ↔ ISO string ให้อัตโนมัติ
6. **ประกาศ `DateTimeFormatter` เป็น `static final`** และใช้ `ResolverStyle.STRICT` เมื่อตรวจสอบ input ผู้ใช้
7. **ใช้ `Clock` เพื่อให้ทดสอบได้** — inject `Clock` แทนการเรียก `now()` ตรง ๆ

```java
// แนวทางที่แนะนำ
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.util.Locale;

class OrderService {
    private static final ZoneId BANGKOK = ZoneId.of("Asia/Bangkok");
    private static final DateTimeFormatter DISPLAY =
            DateTimeFormatter.ofPattern("d MMM yyyy HH:mm", Locale.forLanguageTag("th-TH"));

    private final Clock clock; // inject เข้ามา → test ใช้ Clock.fixed(...) ได้

    OrderService(Clock clock) { this.clock = clock; }

    Instant createdAt() {
        return Instant.now(clock);                   // เก็บเป็น UTC
    }

    String displayTime(Instant instant) {
        return instant.atZone(BANGKOK).format(DISPLAY); // แสดงเป็นเวลาไทย
    }
}

OrderService service = new OrderService(
        Clock.fixed(Instant.parse("2026-10-10T02:00:00Z"), ZoneOffset.UTC));
System.out.println(service.displayTime(service.createdAt())); // ผลลัพธ์: 10 ต.ค. 2026 09:00
```

---
