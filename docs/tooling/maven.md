# Maven — Build Tool ของ Java

## Build tool ทำอะไร?

เมื่อโปรเจกต์มีหลายไฟล์และใช้ library ภายนอก การพิมพ์ `javac` / `java` เองจะไม่ไหว — **build tool** จะ **โหลด dependency**, **compile**, **รัน test** และ **package เป็น `.jar`** ให้ โดยทุกคนในทีมและ CI build ได้เหมือนกันจากไฟล์ config ไฟล์เดียว

```
pom.xml ──(mvn package)──►  โหลด dependency → compile → test → target/app.jar
```

> 💡 **เทียบกับ JavaScript:** Maven ≈ `npm` + `package.json` + `tsc` + test runner รวมเป็นเครื่องมือเดียว พร้อม "วิธีมาตรฐาน" ตั้งแต่ต้น

> 📖 ตัวอย่างจริงในเอกสารนี้ใช้ [`springboot-boilerplate/pom.xml`](../../../springboot-boilerplate/pom.xml) ส่วนคำสั่งของโปรเจกต์นั้นดูที่ [README → 📋 คำสั่งทั้งหมด](../../../springboot-boilerplate/README.md#-คำสั่งทั้งหมด)

---

## 1. Maven vs Gradle (สั้น ๆ)

| หัวข้อ      | **Maven**                         | **Gradle**                                   |
| ----------- | --------------------------------- | -------------------------------------------- |
| ไฟล์ config | `pom.xml` (XML, declarative)      | `build.gradle(.kts)` (script Groovy/Kotlin)  |
| แนวคิด      | Convention over configuration     | ยืดหยุ่น เขียน logic ได้                     |
| ความเร็ว    | ปานกลาง                           | เร็วกว่า (incremental, build cache)          |
| ใช้เยอะที่  | Spring Boot / องค์กร              | Android, โปรเจกต์ใหญ่                        |

ทั้งคู่ใช้ **Maven Central** และพิกัด dependency แบบเดียวกัน

---

## 2. `package.json` ↔ `pom.xml`

| `package.json`                  | `pom.xml`                                     |
| ------------------------------- | --------------------------------------------- |
| `"name"`                        | `<groupId>` + `<artifactId>`                  |
| `"version"`                     | `<version>`                                   |
| `"dependencies"`                | `<dependencies>` (scope `compile`/`runtime`)  |
| `"devDependencies"`             | `<dependency>` + `<scope>test</scope>`        |
| `"engines"`                     | `<java.version>` / `maven.compiler.release`   |
| `"scripts"`                     | lifecycle phases + plugins                    |
| `"overrides"`                   | `<dependencyManagement>`                      |
| `package-lock.json`             | ไม่มี — ใช้เวอร์ชันตายตัว + BOM               |
| `node_modules/`                 | `~/.m2/repository/` (cache กลางทั้งเครื่อง)   |
| `dist/`                         | `target/`                                     |

```xml
<groupId>com.example</groupId>                  <!-- เจ้าของ (โดเมนกลับด้าน) -->
<artifactId>springboot-boilerplate</artifactId> <!-- ชื่อโปรเจกต์ -->
<version>0.0.1-SNAPSHOT</version>               <!-- เวอร์ชันที่ยังพัฒนาอยู่ -->
```

---

## 3. GAV Coordinates

ทุก artifact ระบุด้วย **G**roupId : **A**rtifactId : **V**ersion

```
org.postgresql : postgresql : 42.7.7
```

> 💡 **เทียบกับ JavaScript:** `groupId` ≈ npm scope (`@nestjs`), `artifactId` ≈ ชื่อ package

| ประเภท       | ตัวอย่าง         | ความหมาย                                             |
| ------------ | ---------------- | ---------------------------------------------------- |
| **SNAPSHOT** | `0.0.1-SNAPSHOT` | กำลังพัฒนา เนื้อหาเปลี่ยนได้ Maven เช็กของใหม่เป็นระยะ |
| **Release**  | `3.5.6`          | ปล่อยแล้ว ห้ามเปลี่ยนเนื้อหา (immutable)              |

```xml
<version>2.8.13</version>       <!-- ✅ ระบุตายตัว -->
<version>^2.8.13</version>      <!-- ❌ ไม่มี ^ หรือ ~ -->
<version>[2.8,3.0)</version>    <!-- ⚠️ range มีได้ แต่ build ไม่ reproducible -->
```

- Maven **ไม่บังคับ semver** — เรียงเวอร์ชันตามกฎของตัวเอง (`1.0-alpha` < `1.0-RC1` < `1.0`)
- ทุกคนใช้เวอร์ชันตายตัว จึง **ไม่ต้องมี lock file**

> 💡 **เทียบกับ JavaScript:** `"^2.8.13"` ใน npm = อะไรก็ได้ที่ `<3.0.0` จึงต้องมี lock file ส่วน Maven `2.8.13` = `2.8.13` เท่านั้น เหมือน `npm i --save-exact` ตลอดเวลา

---

## 4. โครงสร้าง Directory มาตรฐาน

วางไฟล์ตามนี้แล้วไม่ต้องตั้งค่าอะไร (convention over configuration)

```
my-app/
├── pom.xml
├── mvnw, mvnw.cmd, .mvn/      # Maven Wrapper (ข้อ 10)
├── src/
│   ├── main/
│   │   ├── java/              # โค้ดหลัก → เข้า jar
│   │   └── resources/         # application.yml, SQL migration → เข้า jar (classpath)
│   └── test/
│       ├── java/              # โค้ด test → ไม่เข้า jar
│       └── resources/
└── target/                    # ผล build: classes/, surefire-reports/, *.jar (ใส่ .gitignore)
```

> 💡 **เทียบกับ JavaScript:** `src/main/java` ≈ `src/`, `src/test/java` ≈ `__tests__/`, `target/` ≈ `dist/` — แต่ Maven **บังคับ** ตำแหน่ง ไม่ต้องตั้ง `rootDir`/`outDir`

---

## 5. Dependencies และ Scope

| Scope                   | compile | test | อยู่ตอนรัน | ตัวอย่างใน boilerplate                 |
| ----------------------- | :-----: | :--: | :--------: | -------------------------------------- |
| `compile` (ค่าเริ่มต้น) | ✅      | ✅   | ✅         | `spring-boot-starter-web`              |
| `provided`              | ✅      | ✅   | ❌         | `servlet-api` เมื่อ deploy ลง Tomcat   |
| `runtime`               | ❌      | ✅   | ✅         | `postgresql` (JDBC driver)             |
| `test`                  | ❌      | ✅   | ❌         | `spring-boot-starter-test`             |

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>  <!-- โค้ดเราใช้ผ่าน JDBC interface ไม่ import driver ตรง ๆ -->
</dependency>
```

> 💡 **เทียบกับ JavaScript:** `test` ≈ `devDependencies`, `compile` ≈ `dependencies` แต่ Maven แยก "ใช้ตอน compile" กับ "ใช้ตอนรัน" ได้ละเอียดกว่า

### Lombok: `optional` + `annotationProcessorPaths`

Lombok เป็น **annotation processor** — generate getter/setter ตอน compile ไม่ต้องใช้ตอนรัน boilerplate จึงตั้งไว้ 3 จุด:

```xml
<!-- 1) import lombok.* ได้ แต่ไม่ส่งต่อให้โปรเจกต์ที่ใช้เรา -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- 2) บอก maven-compiler-plugin ให้รัน Lombok ตอน compile -->
<annotationProcessorPaths>
    <path>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version> <!-- property มาจาก parent -->
    </path>
</annotationProcessorPaths>

<!-- 3) spring-boot-maven-plugin: <excludes> lombok ออกจาก jar สุดท้าย -->
```

> ⚠️ ตั้งแต่ JDK 23 `javac` ไม่รัน processor ที่เจอใน classpath ให้อัตโนมัติแล้ว การระบุ `annotationProcessorPaths` จึงเป็นวิธีที่ถูกต้อง

> 💡 **เทียบกับ JavaScript:** ≈ Babel/SWC plugin ที่แปลงโค้ดตอน build

---

## 6. Transitive Dependencies

เพิ่ม `spring-boot-starter-web` ตัวเดียว ได้ Spring MVC, Jackson, Tomcat ฯลฯ มาด้วย เพราะ dependency มี dependency ของมันเอง

```powershell
.\mvnw.cmd dependency:tree
# [INFO] com.example:springboot-boilerplate:jar:0.0.1-SNAPSHOT
# [INFO] +- org.springframework.boot:spring-boot-starter-web:jar:3.5.6:compile
# [INFO] |  +- org.springframework.boot:spring-boot-starter-json:jar:3.5.6:compile
# [INFO] |  |  +- com.fasterxml.jackson.core:jackson-databind:jar:...:compile
# [INFO] |  +- org.springframework.boot:spring-boot-starter-tomcat:jar:3.5.6:compile
# [INFO] +- org.postgresql:postgresql:jar:...:runtime

.\mvnw.cmd dependency:tree "-Dincludes=com.fasterxml.jackson.core"   # ใครดึง jackson มา?
```

> 💡 **เทียบกับ JavaScript:** `dependency:tree` ≈ `npm ls --all`, `-Dincludes` ≈ `npm why`

**ต่างจาก npm:** `node_modules` ซ้อนได้หลายเวอร์ชัน แต่ **classpath มี class ชื่อเดียวกันได้เวอร์ชันเดียว** Maven เลือกด้วยกฎ:

1. `<dependencyManagement>` ชนะเสมอ
2. ตัวที่ **อยู่ใกล้ root ของ tree** ที่สุดชนะ (nearest wins)
3. ลึกเท่ากัน → ตัวที่ **ประกาศก่อน** ชนะ

### Exclusions

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion> <!-- ตัด Tomcat ออก (เช่นจะใช้ Jetty แทน) -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

---

## 7. Parent POM และ BOM — ทำไมไม่ต้องใส่ version

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.5.6</version>
    <relativePath/> <!-- โหลดจาก repository ไม่หาในโฟลเดอร์แม่ -->
</parent>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- ✅ ไม่มี version — parent กำหนดให้ -->
</dependency>
```

`pom.xml` ของเรา **สืบทอด** จาก parent:

- **`<dependencyManagement>`** — ตารางเวอร์ชันที่ทดสอบแล้วว่าเข้ากันได้ (Spring, Jackson, Hibernate, PostgreSQL, Flyway, Lombok, JUnit ...)
- **`<pluginManagement>`** — เวอร์ชันและ config ของ plugin
- **properties** เช่น `${lombok.version}`, encoding UTF-8, แปลง `<java.version>` เป็น compiler release

| Tag                      | ผล                                                     |
| ------------------------ | ------------------------------------------------------ |
| `<dependencies>`         | **ใช้จริง** — ใส่ลง classpath                         |
| `<dependencyManagement>` | **กำหนดเวอร์ชันเฉย ๆ** — มีผลเมื่อประกาศใน `<dependencies>` |

### BOM (Bill of Materials)

Maven มี parent ได้ตัวเดียว ถ้ามี parent อื่นอยู่แล้วให้ import ตารางเวอร์ชันเป็น BOM:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.5.6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

> 💡 **เทียบกับ JavaScript:** เหมือนทีม framework เขียน `overrides` ชุดที่เข้ากันได้มาให้ — อัปเกรด Spring Boot แค่แก้ `3.5.6` จุดเดียว คล้าย `ng update` ที่อัปทั้งชุด

---

## 8. `<properties>`

```xml
<properties>
    <java.version>21</java.version>               <!-- property ของ Spring Boot parent -->
    <springdoc.version>2.8.13</springdoc.version> <!-- library ที่ BOM ไม่ได้คุม -->
    <bucket4j.version>8.14.0</bucket4j.version>
</properties>

<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>${springdoc.version}</version>
</dependency>
```

- library ที่ไม่อยู่ใน BOM (springdoc, bucket4j) ต้องใส่ version เอง → เก็บเป็น property
- **override เวอร์ชันของ parent** ได้ด้วย property ชื่อเดียวกัน เช่น `<lombok.version>...</lombok.version>`
- ส่งจาก command line ได้ด้วย `-Dชื่อ=ค่า`

---

## 9. Lifecycle — แทน npm scripts

Maven ไม่มี `"scripts"` อิสระ แต่มี **lifecycle** ลำดับตายตัว — สั่ง phase ไหน = รันทุก phase ก่อนหน้าด้วย

```
validate → compile → test → package → verify → install → deploy
```

| Phase      | ทำอะไร                                         | เทียบ npm           |
| ---------- | ---------------------------------------------- | ------------------- |
| `validate` | ตรวจ `pom.xml`                                 | —                   |
| `compile`  | `src/main/java` → `target/classes`             | `tsc`               |
| `test`     | รัน test ใน `src/test/java`                    | `npm test`          |
| `package`  | สร้าง `target/*.jar`                           | `npm run build`     |
| `verify`   | integration test / ตรวจเพิ่ม — นิยมใช้ใน CI   | build + test        |
| `install`  | copy jar ลง `~/.m2` ให้โปรเจกต์อื่นในเครื่องใช้ | `npm link`          |
| `deploy`   | upload ไป remote repo (Nexus/Artifactory)      | `npm publish`       |

`clean` เป็น lifecycle แยก (ลบ `target/`) จึงมักสั่งคู่กัน: `mvn clean package`

> ⚠️ `mvn install` **ไม่ใช่** `npm install` — Maven โหลด dependency ให้เองทุกครั้งที่ build ส่วน `install` คือติดตั้ง "โปรเจกต์เรา" ลง local repo

---

## 10. Plugins และ Goals

งานจริงทำโดย **plugin** แต่ละตัวมี **goal** ที่ผูกกับ phase

| Plugin                     | ผูกกับ phase | หน้าที่                                                  |
| -------------------------- | ------------ | -------------------------------------------------------- |
| `maven-compiler-plugin`    | `compile`    | เรียก `javac`, ตั้ง Java release, annotation processor   |
| `maven-surefire-plugin`    | `test`       | รัน JUnit → `target/surefire-reports`                   |
| `spring-boot-maven-plugin` | `package`    | repackage เป็น **fat jar** + goal `spring-boot:run`      |

เรียก goal ตรง ๆ ด้วย `prefix:goal` เช่น `spring-boot:run`, `dependency:tree`

> 💡 **fat jar** = jar ที่รวม dependency ทั้งหมดไว้ รันด้วย `java -jar app.jar` ไฟล์เดียว ≈ bundle ด้วย esbuild จนไม่ต้องมี `node_modules` บน server

---

## 11. Maven Wrapper (`mvnw`)

```
springboot-boilerplate/
├── mvnw                                  # Git Bash / macOS / Linux
├── mvnw.cmd                              # Windows
└── .mvn/wrapper/maven-wrapper.properties
```

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.16/apache-maven-3.9.16-bin.zip
```

รันครั้งแรก wrapper จะโหลด Maven 3.9.16 ไปไว้ใน `~/.m2/wrapper/` แล้วใช้ตัวนั้นเสมอ — **ไม่ต้องติดตั้ง Maven** (แต่ต้องมี JDK) ✅ commit ทั้ง 3 อย่างเข้า git เพิ่ม wrapper ให้โปรเจกต์ใหม่ด้วย `mvn wrapper:wrapper`

> 💡 **เทียบกับ JavaScript:** ≈ pin package manager ด้วย `"packageManager"` + Corepack หรือ `.nvmrc` — ทุกคนได้ Maven เวอร์ชันเดียวกัน

---

## 12. `~/.m2` vs `node_modules`

```
C:\Users\<you>\.m2\
├── repository\org\postgresql\postgresql\42.7.7\postgresql-42.7.7.jar
├── wrapper\          # Maven ที่ mvnw โหลดมา
└── settings.xml      # (ถ้ามี) mirror / proxy / credential ของ repo บริษัท
```

| หัวข้อ         | `node_modules`          | `~/.m2/repository`                   |
| -------------- | ----------------------- | ------------------------------------ |
| ที่อยู่        | ในแต่ละโปรเจกต์         | **ที่เดียวทั้งเครื่อง** แชร์ทุกโปรเจกต์ |
| ซ้ำซ้อน        | มาก                     | ไม่ซ้ำ (คล้าย global store ของ pnpm) |
| ลบแล้ว         | ต้อง `npm install` ใหม่ | Maven โหลดใหม่เองตอน build ถัดไป     |

---

## 13. คำสั่งและ Flag ที่ใช้บ่อย

> PowerShell: ใช้ `.\mvnw.cmd` และครอบ `-D...` ด้วย `"..."` (รายการเต็มของ boilerplate อยู่ใน [README](../../../springboot-boilerplate/README.md#-คำสั่งทั้งหมด))

| Flag                    | ความหมาย                                                          |
| ----------------------- | ----------------------------------------------------------------- |
| `-DskipTests`           | compile test แต่ไม่รัน                                            |
| `-Dtest=CustomerServiceTest` | รันเฉพาะ test class (หรือ `Class#method`)                    |
| `-pl api -am`           | (multi-module) build เฉพาะโมดูล `api` + โมดูลที่มันพึ่ง           |
| `-U`                    | บังคับเช็ก SNAPSHOT / ผลโหลดที่เคยล้มเหลวใหม่                    |
| `-q`                    | quiet — แสดงเฉพาะ error                                           |
| `-o`                    | offline — ใช้แค่ของใน `~/.m2`                                     |
| `-B`                    | batch mode สำหรับ CI                                              |
| `-X`                    | debug log ละเอียด                                                 |
| `spring-boot:run`       | รันแอป Spring Boot จากโค้ดเลย (`-Dspring-boot.run.profiles=dev`) |

```powershell
.\mvnw.cmd -q clean package -DskipTests
.\mvnw.cmd spring-boot:run "-Dspring-boot.run.profiles=dev"
```

> ⚠️ Maven profile (`-P prod`) กับ Spring profile **คนละเรื่องกัน** — อันแรกเปลี่ยนวิธี build อันหลังเปลี่ยน config ตอนรัน

---

## 14. Step-by-step: โปรเจกต์ Maven เปล่า (ไม่ใช้ Spring)

เป้าหมาย: Java 21 + library Gson แปลง object เป็น JSON

### ขั้นที่ 1: สร้างโฟลเดอร์

```powershell
New-Item -ItemType Directory -Force hello-maven\src\main\java\com\example\hello
cd hello-maven
```

### ขั้นที่ 2: `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hello-maven</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <properties>
        <!-- ไม่มี parent จึงต้องตั้งเอง -->
        <maven.compiler.release>21</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- ไม่มี BOM → ใส่ version เอง -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.11.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin> <!-- pin ให้ใหม่พอรองรับ release 21 -->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
            </plugin>
            <plugin> <!-- ใช้รัน main ด้วย mvn exec:java -->
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.5.0</version>
                <configuration>
                    <mainClass>com.example.hello.Main</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### ขั้นที่ 3: `src/main/java/com/example/hello/Main.java`

```java
package com.example.hello;

import com.google.gson.Gson;

public class Main {
    record Product(String name, int price) {} // data class แบบย่อ

    public static void main(String[] args) {
        String json = new Gson().toJson(new Product("กาแฟ", 55));
        System.out.println(json); // ผลลัพธ์: {"name":"กาแฟ","price":55}
    }
}
```

### ขั้นที่ 4: รัน

```powershell
mvn wrapper:wrapper            # ครั้งเดียว (หรือ copy mvnw + .mvn จาก boilerplate)
.\mvnw.cmd compile exec:java   # ผลลัพธ์: {"name":"กาแฟ","price":55}
.\mvnw.cmd package             # ได้ target\hello-maven-1.0.0-SNAPSHOT.jar
```

> ⚠️ jar นี้ **ไม่ใช่ fat jar** (ไม่มี Gson ข้างใน) `java -jar` จะไม่รัน — ต้องเพิ่ม `maven-shade-plugin` หรือใช้ `spring-boot-maven-plugin` แบบ boilerplate

> 💡 **เทียบกับ JavaScript:** ≈ `npm init -y` → `npm i gson` → `node index.js` แต่ Maven ไม่มีคำสั่ง add — แก้ `pom.xml` เอง (IDE ช่วยได้) ส่วนโปรเจกต์ Spring ให้เริ่มจาก Spring Initializr ตาม [README ของ boilerplate](../../../springboot-boilerplate/README.md)

---

## 15. Troubleshooting

### ❌ `release version 21 not supported`

Maven ถูกรันด้วย JDK ที่เก่ากว่า 21 — Maven ใช้ JDK ตาม **`JAVA_HOME`** ไม่ใช่ `java` ใน `Path` เสมอไป

```powershell
.\mvnw.cmd -v           # ดูบรรทัด "Java version: ..."
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-21.0.5.11-hotspot"  # แก้ชั่วคราว
```

### ❌ `JAVA_HOME environment variable is not defined correctly`

ไม่ได้ตั้ง หรือชี้ไปที่ `...\bin` — ต้องชี้ **โฟลเดอร์รากของ JDK** แล้วเปิด terminal ใหม่ (ดู [Setup & Hello World](../fundamental/setup-and-hello-world.md))

### ❌ `Could not resolve dependencies` / `Could not find artifact`

| สาเหตุ                                       | วิธีแก้                                              |
| -------------------------------------------- | ---------------------------------------------------- |
| พิมพ์ GAV ผิด                                | ค้นที่ [central.sonatype.com](https://central.sonatype.com) |
| ลืม version ของ library ที่ไม่อยู่ใน BOM     | เพิ่ม `<version>`                                    |
| อยู่หลัง proxy บริษัท                        | ตั้ง proxy / mirror ใน `~/.m2/settings.xml`          |
| เคยโหลดล้มเหลว (ไฟล์ `*.lastUpdated`)        | รันด้วย `-U` หรือลบโฟลเดอร์นั้นใน `~/.m2/repository` |

### ❌ Lombok ไม่ทำงาน / `cannot find symbol getXxx()`

ตรวจ `annotationProcessorPaths` (ข้อ 5) และเปิด annotation processing ใน IDE

### ❌ `NoSuchMethodError` / `ClassNotFoundException` ตอนรัน

มักเป็นเวอร์ชัน transitive ชนกัน → `dependency:tree -Dincludes=...` แล้วแก้ด้วย `<dependencyManagement>` หรือ `<exclusions>`

### ❌ PowerShell: `Unknown lifecycle phase ".run.profiles=dev"`

ครอบด้วย quote: `"-Dspring-boot.run.profiles=dev"`

---

## 16. Cheat Sheet: npm → Maven

| npm                             | Maven (`./mvnw` / `.\mvnw.cmd`)                    |
| ------------------------------- | -------------------------------------------------- |
| `npm init`                      | Spring Initializr / เขียน `pom.xml` เอง            |
| `npm install`                   | ไม่ต้อง — โหลดอัตโนมัติตอน build                   |
| `npm i gson` / `npm i -D jest`  | เพิ่ม `<dependency>` (+ `<scope>test</scope>`)     |
| `npm ls --all` / `npm why x`    | `mvn dependency:tree` (`-Dincludes=g:a`)           |
| `npm outdated`                  | `mvn versions:display-dependency-updates`          |
| `npm run build`                 | `mvn package`                                      |
| `npm test`                      | `mvn test`                                         |
| `npx jest file`                 | `mvn test -Dtest=ClassName`                        |
| `npm start`                     | `mvn spring-boot:run` / `mvn exec:java`            |
| `node dist/main.js`             | `java -jar target/app.jar`                         |
| `rm -rf dist`                   | `mvn clean`                                        |
| `npm ci` (CI)                   | `mvn -B clean verify`                              |
| `npm link` / `npm publish`      | `mvn install` / `mvn deploy`                       |
| `.nvmrc` / `"packageManager"`   | `.mvn/wrapper/maven-wrapper.properties`            |

---

## แนวทางปฏิบัติที่ดี (Best Practices)

1. **ใช้ `./mvnw` เสมอ** และ commit `mvnw`, `mvnw.cmd`, `.mvn/`
2. **ระบุเวอร์ชันตายตัว** — ไม่ใช้ range
3. **ให้ parent / BOM คุมเวอร์ชัน** — ใส่ `<version>` เฉพาะ library นอก BOM โดยเก็บเป็น `<properties>`
4. **เลือก scope ให้ถูก** — test library เป็น `test`, JDBC driver เป็น `runtime`
5. **CI ใช้ `mvn -B verify`** และอย่าติดนิสัย `-DskipTests`
6. **build พังให้เช็ก `mvn -v` ก่อน** — ปัญหาส่วนใหญ่มาจาก `JAVA_HOME` ผิด JDK

---
