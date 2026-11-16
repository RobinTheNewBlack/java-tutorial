# Advanced

# ส่วนที่ 5: Concurrency (การทำงานพร้อมกัน)

## Concurrency คืออะไร?

Concurrency คือการให้โปรแกรม **ทำหลายงานในช่วงเวลาเดียวกัน** เช่น เรียก API 3 ตัวพร้อมกัน, รับ HTTP request หลายพันตัว หรือแบ่งงานคำนวณไปใช้ CPU หลาย core — Java ใช้ **thread จริง** และใน Java 21 มี **Virtual Threads** ที่สร้างได้เป็นล้านตัว

> 💡 **เทียบกับ JavaScript:** JS รันโค้ดเราบน **thread เดียว** + **event loop** แล้วใช้ `Promise`/`async`/`await` รอ I/O ส่วน Java มี **หลาย thread รันโค้ดพร้อมกันจริง ๆ** — ใช้ CPU ได้ทุก core แต่ต้องระวัง **ข้อมูลที่แชร์กัน** ซึ่ง JS ไม่เคยต้องกังวล

---

## 1. Event Loop vs Multi-thread

```
JavaScript                               Java
───────────────────────                  ─────────────────────────────────
 ┌────────────┐                           Thread-1 ──[งาน A]──[งาน D]──▶
 │ Call Stack │ ← thread เดียว              Thread-2 ──[งาน B]──────────▶
 └─────┬──────┘                           Thread-3 ──[งาน C]──[งาน E]──▶
 ┌─────▼──────┐  I/O เสร็จ                  รันพร้อมกันจริงบนหลาย core
 │ Event Loop │◀── callback queue          ทุก thread แชร์ heap เดียวกัน
 └────────────┘                            → ต้องป้องกัน shared state
```

| หัวข้อ              | JavaScript                        | Java                                       |
| ------------------ | --------------------------------- | ------------------------------------------ |
| โมเดล               | Single thread + event loop        | Multi-thread (platform + virtual)          |
| งาน I/O             | non-blocking + Promise            | เขียน blocking ได้เลย (thread อื่นทำงานต่อ)     |
| งาน CPU หนัก         | บล็อกทั้งแอป (ต้องใช้ Worker)          | แบ่งไปหลาย thread ได้ตรง ๆ                   |
| Shared state       | ไม่มีปัญหา                           | **race condition** ถ้าไม่ป้องกัน                |
| เครื่องมือหลัก        | `Promise`, `async`/`await`        | `ExecutorService`, `CompletableFuture`, virtual threads |

---

## 2. Thread และ Runnable

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        // Runnable = functional interface "งานที่ไม่คืนค่า" → เขียนเป็น lambda ได้
        Runnable task = () -> System.out.println("รันบน " + Thread.currentThread().getName());

        Thread t1 = new Thread(task, "worker-1");
        Thread t2 = Thread.ofPlatform().name("worker-2").start(task); // Builder API (Java 21)

        t1.start(); // ✅ start() = สร้าง thread ใหม่แล้วเรียก run() บน thread นั้น
        // t1.run(); // ❌ แค่เรียก method ธรรมดาบน thread ปัจจุบัน — ไม่ได้ขนาน

        t1.join();  // รอให้ t1 จบ (คล้าย await)
        t2.join();
        System.out.println("main จบ");
    }
}
// ผลลัพธ์ (2 บรรทัดแรกอาจสลับลำดับ):
// รันบน worker-2
// รันบน worker-1
// main จบ
```

> **หมายเหตุ:** งานจริงแทบไม่ต้อง `new Thread(...)` เอง — ใช้ `ExecutorService` หรือ virtual threads

---

## 3. Thread Lifecycle

```
  ┌─────┐  start()  ┌──────────┐   run() จบ   ┌────────────┐
  │ NEW │ ────────▶ │ RUNNABLE │ ───────────▶ │ TERMINATED │
  └─────┘           └──────────┘              └────────────┘
                      │      ▲
     รอ lock / sleep / │      │ ได้ lock / หมดเวลา / ถูกปลุก
     join / wait       ▼      │
              BLOCKED / WAITING / TIMED_WAITING
```

| สถานะ            | ความหมาย                                         |
| --------------- | ----------------------------------------------- |
| `NEW`           | สร้างแล้วแต่ยังไม่ `start()`                         |
| `RUNNABLE`      | กำลังรัน หรือพร้อมรัน (รอ CPU)                       |
| `BLOCKED`       | รอเข้า `synchronized` ที่ thread อื่นถือ lock อยู่     |
| `WAITING`       | รอไม่มีกำหนด เช่น `join()`                          |
| `TIMED_WAITING` | รอมีกำหนด เช่น `sleep(500)`                         |
| `TERMINATED`    | จบแล้ว — `start()` ซ้ำไม่ได้                         |

ตรวจสถานะได้ด้วย `t.getState()` เช่น ก่อน `start()` ได้ `NEW`, ระหว่าง `sleep(500)` ได้ `TIMED_WAITING`, หลัง `join()` ได้ `TERMINATED`

---

## 4. ExecutorService และ Thread Pool

Platform thread แต่ละตัวมีต้นทุนสูง (เป็น thread ของ OS, stack ~1MB) จึงควร **สร้างไว้ชุดหนึ่งแล้วใช้ซ้ำ** = **thread pool**

```
 submit(task) ──▶ [ Queue: T4 T5 T6 ... ] ──▶ Thread-1 [T1]
                                          ──▶ Thread-2 [T2]   (pool ขนาด 3)
                                          ──▶ Thread-3 [T3]
```

ตั้งแต่ **Java 19** `ExecutorService` เป็น `AutoCloseable` — ใช้ try-with-resources ได้ โดย `close()` จะ **รองานทั้งหมดเสร็จ** แล้วปิด pool ให้

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        try (ExecutorService executor = Executors.newFixedThreadPool(3)) {
            for (int i = 1; i <= 5; i++) {
                int orderId = i; // lambda ใช้ได้เฉพาะตัวแปร effectively final
                executor.submit(() -> System.out.println(
                        "order #" + orderId + " บน " + Thread.currentThread().getName()));
            }
        } // ← close(): รองานครบแล้ว shutdown
        System.out.println("เสร็จทุก order");
    }
}
// ผลลัพธ์ (ลำดับไม่แน่นอน):
// order #1 บน pool-1-thread-1
// order #3 บน pool-1-thread-3
// order #2 บน pool-1-thread-2
// order #4 บน pool-1-thread-1   ← thread ถูกใช้ซ้ำ
// order #5 บน pool-1-thread-2
// เสร็จทุก order
```

> **ก่อน Java 19** ต้องเรียก `executor.shutdown()` + `awaitTermination(...)` เองใน `finally` — ❌ ถ้าลืม thread ใน pool ยังมีชีวิตและโปรแกรมจะไม่จบ

| Factory                                       | ลักษณะ                         | เหมาะกับ                           |
| --------------------------------------------- | ----------------------------- | --------------------------------- |
| `newFixedThreadPool(n)`                       | thread คงที่ n ตัว               | CPU-bound (n ≈ จำนวน core)         |
| `newCachedThreadPool()`                       | สร้างเพิ่มตามต้องการ             | งานสั้น ๆ จำนวนไม่แน่นอน              |
| `newSingleThreadExecutor()`                   | thread เดียว ตามลำดับ            | งานที่ต้องเรียงลำดับ                   |
| `newScheduledThreadPool(n)`                   | ตั้งเวลา/รันซ้ำ                   | คล้าย `setTimeout`/`setInterval`   |
| `newVirtualThreadPerTaskExecutor()`           | **Java 21** virtual thread ต่องาน | I/O-bound จำนวนมาก                 |

---

## 5. Callable และ Future

`Runnable` คืนค่าไม่ได้ — ถ้าต้องการผลลัพธ์ใช้ **`Callable<V>`** (`V call() throws Exception`) แล้ว `submit()` จะคืน **`Future<V>`** = "ตั๋วรับผลในอนาคต"

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws Exception {
        try (ExecutorService executor = Executors.newFixedThreadPool(2)) {
            Callable<Integer> slowTask = () -> {
                Thread.sleep(500); // Callable โยน checked exception ได้
                return 42;
            };
            Future<Integer> future = executor.submit(slowTask); // คืนทันที ไม่รอ
            System.out.println("ทำอย่างอื่นไปก่อน... isDone=" + future.isDone()); // false
            System.out.println("ผลลัพธ์: " + future.get());      // ⏳ block จนได้ผล → 42

            Future<Integer> bad = executor.submit(() -> Integer.parseInt("abc"));
            try {
                bad.get(1, TimeUnit.SECONDS); // มี timeout ได้ (TimeoutException)
            } catch (ExecutionException e) {  // exception จาก task ถูกห่อไว้
                System.out.println("ผิดพลาด: " + e.getCause().getClass().getSimpleName());
            }
        }
    }
}
// ผลลัพธ์:
// ทำอย่างอื่นไปก่อน... isDone=false
// ผลลัพธ์: 42
// ผิดพลาด: NumberFormatException
```

> 💡 **เทียบกับ JavaScript:** `Future` คล้าย `Promise` แต่ **ต่อ chain ไม่ได้** — มีแค่ `get()` ที่ block รอ ไม่มี `.then()` ถ้าอยากได้แบบ Promise จริง ๆ ให้ใช้ `CompletableFuture`

---

## 6. CompletableFuture — "Promise ของ Java"

```java
import java.util.*;
import java.util.concurrent.*;

public class Main {
    record User(long id, String name) {}

    static void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }

    // จำลองการเรียก API (เหมือน async function ใน JS)
    static CompletableFuture<User> fetchUser(long id) {
        return CompletableFuture.supplyAsync(() -> { sleep(300); return new User(id, "สมชาย"); });
    }

    static CompletableFuture<Double> fetchTotal(User user) {
        return CompletableFuture.supplyAsync(() -> { sleep(300); return 470.0; });
    }

    static CompletableFuture<Integer> fetchPoints(long userId) {
        return CompletableFuture.supplyAsync(() -> { sleep(200); return 1500; });
    }

    public static void main(String[] args) {
        // thenApply = .then(x => ค่าใหม่)
        String greet = fetchUser(1).thenApply(u -> "คุณ" + u.name()).join(); // join() ≈ await
        System.out.println(greet); // ผลลัพธ์: คุณสมชาย

        // thenCompose = .then(x => promiseใหม่) — ไม่ซ้อนเป็น CF<CF<T>>
        double total = fetchUser(1).thenCompose(Main::fetchTotal).join();
        System.out.println(total); // ผลลัพธ์: 470.0

        // thenCombine = รวม 2 งานที่ไม่ขึ้นต่อกัน (รันขนาน)
        String profile = fetchUser(1)
                .thenCombine(fetchPoints(1), (u, p) -> u.name() + " มี " + p + " แต้ม")
                .join();
        System.out.println(profile); // ผลลัพธ์: สมชาย มี 1500 แต้ม

        // allOf = Promise.all — คืน CF<Void> จึงต้อง join แต่ละตัวเอง
        List<CompletableFuture<User>> futures = List.of(fetchUser(1), fetchUser(2), fetchUser(3));
        List<Long> ids = CompletableFuture.allOf(futures.toArray(CompletableFuture[]::new))
                .thenApply(v -> futures.stream().map(CompletableFuture::join).map(User::id).toList())
                .join();
        System.out.println(ids); // ผลลัพธ์: [1, 2, 3]  (~300ms ไม่ใช่ 900ms)

        // exceptionally = .catch()
        String safe = CompletableFuture.<String>supplyAsync(() -> {
                    throw new IllegalStateException("API ล่ม");
                })
                .exceptionally(ex -> "ค่าสำรอง (" + ex.getCause().getMessage() + ")")
                .join();
        System.out.println(safe); // ผลลัพธ์: ค่าสำรอง (API ล่ม)
    }
}
```

> **หมายเหตุ:** ใน `exceptionally` ตัว `ex` มักเป็น `CompletionException` ที่ห่อ exception จริงไว้ จึงใช้ `ex.getCause()` และ `join()` จะโยน `CompletionException` (unchecked) ส่วน `get()` โยน `ExecutionException` (checked)

| JavaScript                       | Java `CompletableFuture`                      |
| -------------------------------- | --------------------------------------------- |
| `async () => value`              | `supplyAsync(() -> value)`                    |
| `Promise.resolve(v)`             | `completedFuture(v)`                          |
| `.then(x => y)`                  | `.thenApply(x -> y)`                          |
| `.then(x => fetchMore(x))`       | `.thenCompose(x -> fetchMore(x))`             |
| `.then(x => console.log(x))`     | `.thenAccept(x -> System.out.println(x))`     |
| `Promise.all([a, b])` (2 ตัว)     | `a.thenCombine(b, (x, y) -> ...)`             |
| `Promise.all([...])`             | `allOf(...)` + `join()` แต่ละตัว                 |
| `Promise.race([...])`            | `anyOf(...)`                                  |
| `.catch(err => fallback)`        | `.exceptionally(ex -> fallback)` / `.handle(...)` |
| `.finally(() => ...)`            | `.whenComplete((v, ex) -> ...)`               |
| `await p`                        | `p.join()` / `p.get()` (block thread)          |

> 💡 **เทียบกับ JavaScript:** `supplyAsync()` ที่ไม่ระบุ executor จะรันบน **`ForkJoinPool.commonPool()`** (thread ≈ จำนวน core − 1) ถ้างานเป็น **blocking I/O** ควรส่ง executor ของตัวเอง เช่น `supplyAsync(task, virtualExecutor)`

---

## 7. Race Condition และ Thread Safety

```java
public class Main {
    static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> { for (int i = 0; i < 100_000; i++) count++; }; // ❌ ไม่ atomic
        Thread t1 = new Thread(task), t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join();  t2.join();
        System.out.println(count); // คาดว่า 200000 แต่ได้ เช่น 137482 (เปลี่ยนทุกครั้ง)
    }
}
```

`count++` คือ 3 ขั้น: **อ่าน → บวก → เขียนกลับ** ถ้า 2 thread สลับกันกลางทาง การบวกจะหาย

```
 count = 5
 Thread-1: อ่าน 5 ──▶ 5+1=6 ──────────▶ เขียน 6
 Thread-2:      อ่าน 5 ──▶ 5+1=6 ──────────▶ เขียน 6   ← ควรเป็น 7!
```

> 💡 **เทียบกับ JavaScript:** ใน JS โค้ดนี้ไม่มีทางพัง เพราะไม่มี 2 บรรทัดรันพร้อมกัน (ยกเว้น `SharedArrayBuffer` + Worker ซึ่งต้องใช้ `Atomics`) — ใน Java นี่คือ bug ที่เจอบ่อยที่สุด

### `synchronized` — เข้าได้ทีละ thread

```java
class Counter {
    private int count = 0;
    public synchronized void increment() { count++; } // lock ด้วย this
    public synchronized int get() { return count; }

    public void reset() {
        synchronized (this) { count = 0; } // หรือ lock เฉพาะบาง block
    }
}
```

### `AtomicInteger` — ง่ายและเร็วสำหรับตัวนับ

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Main {
    static final AtomicInteger count = new AtomicInteger();

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> { for (int i = 0; i < 100_000; i++) count.incrementAndGet(); }; // ✅
        Thread t1 = new Thread(task), t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join();  t2.join();
        System.out.println(count.get()); // ผลลัพธ์: 200000 เสมอ
    }
}
```

ตัวอื่น: `AtomicLong`, `AtomicBoolean`, `AtomicReference<T>`, `LongAdder` (เร็วกว่าเมื่อแย่งกันหนัก)

### `volatile` — แก้เรื่อง "มองไม่เห็นค่า"

แต่ละ thread อาจ cache ค่าตัวแปรไว้ จน **ไม่เห็นค่าที่ thread อื่นเปลี่ยน** — `volatile` บังคับให้อ่าน/เขียนจาก main memory

```java
public class Main {
    static volatile boolean running = true; // ❌ ถ้าไม่มี volatile worker อาจวนไม่รู้จบ

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            while (running) { /* ทำงาน */ }
            System.out.println("worker หยุดแล้ว");
        });
        worker.start();
        Thread.sleep(100);
        running = false;
        worker.join(); // ผลลัพธ์: worker หยุดแล้ว
    }
}
```

> ⚠️ `volatile` **ไม่ได้ทำให้ `count++` ปลอดภัย** — แก้แค่ visibility ไม่ใช่ atomicity

### `ReentrantLock` (ย่อ)

```java
import java.util.concurrent.locks.ReentrantLock;

class BankAccount {
    private final ReentrantLock lock = new ReentrantLock();
    private double balance = 1000;

    void withdraw(double amount) {
        lock.lock();
        try {                     // ✅ unlock ใน finally เสมอ
            if (balance >= amount) balance -= amount;
        } finally {
            lock.unlock();
        }
    }
}
```

ข้อดีเหนือ `synchronized`: มี `tryLock(timeout)`, fairness และใน Java 21 ไม่ทำให้ virtual thread "pin" (ดูข้อ 9)

| เครื่องมือ         | แก้ปัญหา                | ใช้เมื่อ                                  |
| ---------------- | ---------------------- | --------------------------------------- |
| `synchronized`   | atomicity + visibility | ป้องกันหลายบรรทัดพร้อมกัน แบบง่าย             |
| `AtomicXxx`      | atomicity + visibility | ตัวนับ / ค่าเดียวที่ update บ่อย               |
| `volatile`       | visibility เท่านั้น       | flag ที่เขียนจากที่เดียว                       |
| `ReentrantLock`  | atomicity + visibility | ต้องการ `tryLock`/timeout หรือใช้กับ virtual thread |
| Immutable object | ไม่มี state ให้แย่ง       | **ดีที่สุด** — `record`, `List.of(...)`     |

---

## 8. Concurrent Collections

`HashMap`/`ArrayList` **ไม่ thread-safe** — ใช้ collection จาก `java.util.concurrent` แทน

| แทนที่              | ใช้                       | ลักษณะ                                     |
| ------------------ | ------------------------ | ----------------------------------------- |
| `HashMap`          | `ConcurrentHashMap`      | lock ละเอียดระดับ bucket, อ่านไม่ต้อง lock    |
| `ArrayList`        | `CopyOnWriteArrayList`   | เขียนแล้ว copy ใหม่ — อ่านเร็ว เขียนช้า         |
| `Queue`            | `LinkedBlockingQueue`    | `put()`/`take()` สำหรับ producer/consumer |

```java
import java.util.*;
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        Map<String, Integer> wordCount = new ConcurrentHashMap<>();
        List<String> words = List.of("ไทย", "java", "ไทย", "spring", "java", "ไทย");

        try (ExecutorService executor = Executors.newFixedThreadPool(4)) {
            for (String w : words) {
                executor.submit(() -> wordCount.merge(w, 1, Integer::sum)); // ✅ atomic
            }
        }
        System.out.println(new TreeMap<>(wordCount)); // ผลลัพธ์: {java=2, spring=1, ไทย=3}

        // ❌ check-then-act แม้บน ConcurrentHashMap ก็ยัง race ได้
        // if (!wordCount.containsKey("go")) wordCount.put("go", 0);
        wordCount.putIfAbsent("go", 0); // ✅ atomic

        // CopyOnWriteArrayList — วน loop บน snapshot จึงแก้ระหว่างวนได้
        List<String> listeners = new CopyOnWriteArrayList<>(List.of("email", "sms"));
        for (String l : listeners) listeners.add("push"); // ArrayList จะโยน ConcurrentModificationException
        System.out.println(listeners); // ผลลัพธ์: [email, sms, push, push]
    }
}
```

> **หมายเหตุ:** `Collections.synchronizedMap(...)` lock ทั้ง map ทุกครั้ง — ช้ากว่า `ConcurrentHashMap` ไม่แนะนำในโค้ดใหม่

---

## 9. Virtual Threads (Java 21) ⭐

**Virtual thread** (JEP 444) คือ thread น้ำหนักเบาที่ **JVM จัดการเอง** — ใช้ memory ไม่กี่ KB ต่อตัว สร้างได้เป็นล้าน

```
 Virtual:   V1  V2  V3  V4  V5  V6 ... (เป็นล้านตัว)
              \  |  /      \  |  /
 Carrier:      [ P1 ]       [ P2 ]  ... (platform thread ≈ จำนวน core)
 เมื่อ V ใด block รอ I/O → JVM ถอดออกจาก carrier แล้วเอา V อื่นขึ้นมารันแทน
```

| หัวข้อ          | Platform Thread       | Virtual Thread             |
| -------------- | --------------------- | -------------------------- |
| จัดการโดย        | OS                    | JVM                        |
| ต้นทุน           | ~1MB, สร้างช้า          | ไม่กี่ KB, สร้างเร็วมาก        |
| จำนวน           | หลักพัน                 | หลักล้าน                     |
| ควร pool ไหม    | ✅                    | ❌ **ห้าม** — สร้างใหม่ทุกงาน   |
| เหมาะกับ        | CPU-bound             | I/O-bound (HTTP, DB, file) |

```java
import java.time.Duration;
import java.util.concurrent.*;
import java.util.stream.IntStream;

public class Main {
    public static void main(String[] args) throws Exception {
        Thread vt = Thread.ofVirtual().name("vt-1").start(() ->
                System.out.println("virtual? " + Thread.currentThread().isVirtual()));
        vt.join(); // ผลลัพธ์: virtual? true

        Thread.startVirtualThread(() -> System.out.println("สวัสดี")).join(); // shortcut

        // ✅ แบบที่ใช้บ่อยที่สุด
        long start = System.currentTimeMillis();
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            IntStream.range(0, 10_000).forEach(i -> executor.submit(() -> {
                Thread.sleep(Duration.ofSeconds(1)); // จำลองรอ I/O — เขียนแบบ blocking ได้เลย
                return i;
            }));
        }
        System.out.println("ใช้เวลา ~" + (System.currentTimeMillis() - start) + " ms");
        // ผลลัพธ์: ใช้เวลา ~1000 ms — ถ้าใช้ newFixedThreadPool(100) จะใช้ ~100 วินาที!
    }
}
```

ด้วย virtual thread หลายครั้ง **ไม่ต้องใช้ `CompletableFuture`** — `submit()` แล้ว `get()` แบบ blocking ตรง ๆ อ่านง่ายกว่า เพราะการ block แทบไม่มีต้นทุน

### ควรใช้เมื่อไหร่ และข้อควรระวัง

- ✅ Web server / งานที่รอ DB, HTTP, file เป็นจำนวนมาก
- ❌ งานคำนวณหนัก (CPU-bound) — ไม่เร็วขึ้น ใช้ `newFixedThreadPool(cores)`
- ❌ **อย่า pool** virtual thread — ถ้าต้องจำกัดจำนวนพร้อมกัน (เช่น DB มี 10 connection) ใช้ `Semaphore`
- ⚠️ **Pinning (Java 21):** ถ้า block I/O ขณะอยู่ใน `synchronized` virtual thread จะติดกับ carrier — ใช้ `ReentrantLock` แทน (แก้แล้วใน Java 24, JEP 491)
- ⚠️ ระวัง `ThreadLocal` ขนาดใหญ่ — thread เป็นล้าน × ThreadLocal = memory บาน

### Virtual Threads กับ Spring Boot

ตั้งแต่ **Spring Boot 3.2** (บน Java 21+) เปิดได้ด้วยบรรทัดเดียว:

```properties
# application.properties
spring.threads.virtual.enabled=true
```

Tomcat/Jetty จะรับแต่ละ request บน virtual thread รวมถึง `@Async` และ `@Scheduled` — controller ที่เขียนแบบ blocking (JPA, `RestClient`) รับโหลดพร้อมกันได้มากขึ้นมาก **โดยไม่ต้องเปลี่ยนไปเขียน reactive (WebFlux)**

> 💡 **เทียบกับ JavaScript:** Virtual thread ให้ข้อดีแบบ Node.js (รอ I/O เป็นหมื่นงานด้วย thread น้อย ๆ) แต่ **ไม่ต้องเขียน `async`/`await`** และไม่มีปัญหา "function coloring" (async ลามไปทั้ง call chain) — โค้ดดูเป็น synchronous ทุกบรรทัด

> **Structured Concurrency:** Java 21 มี `StructuredTaskScope` (JEP 453) สำหรับจัดกลุ่ม subtask ให้จบ/ยกเลิกพร้อมกัน แต่ยังเป็น **preview feature** (ต้อง `--enable-preview`, API อาจเปลี่ยน) — ยังไม่แนะนำใน production

---

## 10. ข้อผิดพลาดที่พบบ่อย (Common Pitfalls)

| ❌ ผิดพลาด                                  | ✅ วิธีแก้                                                   |
| ------------------------------------------ | --------------------------------------------------------- |
| เรียก `run()` แทน `start()`                  | ใช้ `start()` เสมอ                                          |
| ลืมปิด `ExecutorService` → โปรแกรมไม่จบ          | try-with-resources (Java 19+) หรือ `shutdown()` ใน `finally` |
| `submit()` แล้วไม่เคย `get()` → exception หายเงียบ | `get()` ผลลัพธ์ หรือ try/catch + log ภายใน task              |
| แชร์ `HashMap`/`ArrayList` ระหว่าง thread      | `ConcurrentHashMap`, `CopyOnWriteArrayList`, immutable     |
| `containsKey` แล้ว `put` (check-then-act)    | `putIfAbsent`, `computeIfAbsent`, `merge`                 |
| Blocking I/O บน `commonPool` ของ CF          | ส่ง executor ของตัวเองให้ `supplyAsync`                        |
| ใช้ virtual thread กับงาน CPU หรือเอาไป pool     | fixed pool สำหรับ CPU, `Semaphore` สำหรับจำกัดจำนวน          |
| `submit(...).get()` ในลูป → ทำทีละงาน            | submit ให้หมดก่อน แล้วค่อยวน `get()` (เหมือน `Promise.all` แทน `await` ในลูป) |

### กลืน `InterruptedException`

```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // ❌ ปล่อยว่าง — สัญญาณขอให้หยุดหายไป
}

try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt(); // ✅ คืนสถานะ interrupt ให้ชั้นบนรู้
}
```

### Deadlock

```
 Thread-1: lock(A) ──▶ รอ lock(B) ┐
                                  ├── รอกันตลอดกาล 💀
 Thread-2: lock(B) ──▶ รอ lock(A) ┘
```

ป้องกัน: **lock ตามลำดับเดียวกันเสมอ**, ถือ lock ให้สั้นที่สุด หรือใช้ `tryLock(timeout)`

---

## สรุป (Cheat Sheet)

| ต้องการ                         | ใช้                                                  | เทียบ JS                 |
| ------------------------------ | ---------------------------------------------------- | ----------------------- |
| รันงานบน thread ใหม่              | `Thread.ofVirtual().start(r)` / `new Thread(r).start()` | —                    |
| รอ thread จบ                    | `thread.join()`                                      | `await`                 |
| Thread pool                    | `try (var ex = Executors.newFixedThreadPool(n)) {}`  | Worker pool             |
| งาน I/O จำนวนมาก (Java 21)       | `Executors.newVirtualThreadPerTaskExecutor()`        | event loop              |
| งานที่คืนค่า                       | `Callable<V>` + `Future<V>.get()`                    | Promise (ไม่มี `.then`)  |
| Async chain                    | `supplyAsync().thenApply()` / `.thenCompose()`       | `.then()`               |
| รวม / รอหลายงาน                  | `thenCombine` / `allOf` / `anyOf`                    | `Promise.all` / `race`  |
| จัดการ error                     | `.exceptionally()` / `.handle()`                     | `.catch()`              |
| Critical section               | `synchronized` / `ReentrantLock`                     | ไม่ต้อง                   |
| ตัวนับข้าม thread                 | `AtomicInteger` / `LongAdder`                        | ไม่ต้อง                   |
| Flag ข้าม thread                 | `volatile boolean`                                   | ไม่ต้อง                   |
| Map / List ข้าม thread           | `ConcurrentHashMap` / `CopyOnWriteArrayList`         | `Map` / `Array`         |
| จำกัดงานพร้อมกัน                   | `Semaphore`                                          | p-limit                 |
| Spring Boot + virtual threads  | `spring.threads.virtual.enabled=true` (Boot 3.2+)    | —                       |

---
