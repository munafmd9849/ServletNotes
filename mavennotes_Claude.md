---
title: "The Maven Master Notes"
subtitle: "The Only Maven Reference a Java Backend / Spring Boot Developer Needs — Interview + Practical Guide"
author: "Prepared for Munaf"
---

# Introduction

These notes convert classroom-style Maven notes into a complete, interview-ready and production-ready Maven handbook. Every topic is explained from two angles — **Interview Perspective** (what an interviewer wants to hear) and **Practical Development Perspective** (what you actually do at your keyboard, in Eclipse, IntelliJ, Ubuntu, or Windows). Wherever the original faculty notes are stated informally, they are preserved, corrected, and expanded.

---

# SECTION 1 — Build Tools Fundamentals

## 1.1 What is a Build Process?

A **build process** converts human-written source code into a runnable artifact. For Java this typically means:

```
.java  --(compile)-->  .class  --(package)-->  .jar / .war  --(deploy)-->  running application
```

A "build" is not just compilation — it is the *entire pipeline*: fetching dependencies, compiling, running tests, packaging, and (optionally) installing/deploying the artifact.

## 1.2 Manual Build Process (What developers did before build tools)

Before Maven/Ant, a Java developer would manually:

1. Run `javac` on every source file, in the correct dependency order.
2. Manually download `.jar` files from vendor websites and place them on the classpath.
3. Manually create the `WEB-INF/lib` folder structure and copy jars into it for a web app.
4. Manually zip class files and resources into a `.jar` or `.war` using the `jar` command.
5. Manually copy the artifact to the server and restart it.

### Problems with Manual Build

- **No dependency management** — you had to hunt for the correct jar version yourself, and transitive dependencies (a jar's own dependencies) were invisible.
- **Not reproducible** — a build that works on your machine may fail on a teammate's machine because of a different jar version or missing jar ("works on my machine" syndrome).
- **No standard project structure** — every developer organized folders differently, making onboarding and tooling (IDEs, CI) painful.
- **Error-prone and slow** — manual compilation order, manual classpath strings, manual packaging.
- **No lifecycle** — there was no single command that would clean, compile, test, package, and install in one repeatable sequence.
- **No collaboration story** — no shared/central place to publish a library your team produced for other teams to consume.

## 1.3 Evolution of Build Tools

### Batch Files / Shell Scripts
The most primitive automation — a `.bat` or `.sh` file that strings together `javac` and `jar` commands.
- **Pros:** Full control, no learning curve.
- **Cons:** Not portable (Windows vs Linux scripts differ), no dependency management, no standard structure, brittle, hard to maintain as the project grows.

### ANT (Apache Ant, 2000)
Ant introduced **XML-based build scripts** (`build.xml`) with explicit **targets** (like `compile`, `jar`, `clean`) that you define yourself.
- **Pros:** Portable XML syntax, flexible, can do literally anything you script.
- **Cons:** No standard project structure (you must tell Ant every path), no dependency management out of the box (Ivy was added later to bolt this on), every project's `build.xml` looks different — nothing is "convention," everything is "configuration."

### Maven (2004)
Maven flipped the model: instead of *telling* the tool what to do step by step, you *declare* what your project is (its coordinates and dependencies) and Maven's predefined **lifecycle** does the rest.
- **Convention over configuration**: standard folder layout (`src/main/java`, etc.) is assumed.
- Built-in **dependency management** with **transitive resolution** from remote repositories.
- A predictable, standardized **lifecycle** (`validate → compile → test → package → verify → install → deploy`) shared by every Maven project on the planet.

### Gradle (2007/2012)
Gradle is the "next generation" build tool — it uses a **Groovy or Kotlin DSL** (`build.gradle` / `build.gradle.kts`) instead of XML, and it is optimized for **build performance** (incremental builds, build caching, parallel execution).
- Same core ideas as Maven (dependency management, lifecycle/tasks) but scriptable and faster for very large builds (this is why Android and many large monorepos prefer it).

### Comparison Table

| Feature | Batch/Shell | ANT | Maven | Gradle |
|---|---|---|---|---|
| Configuration style | Imperative script | Imperative XML (`build.xml`) | Declarative XML (`pom.xml`) | Declarative + programmable DSL (Groovy/Kotlin) |
| Project structure | None enforced | None enforced | **Standard, convention-based** | Standard, but configurable |
| Dependency management | Manual | Manual (Ivy plugin optional) | **Built-in, transitive** | Built-in, transitive |
| Lifecycle | None — you write every step | You define every target yourself | **Predefined standard lifecycle** | Task graph, customizable, incremental |
| Build speed | Fast (trivial builds only) | Moderate | Moderate | **Fast** (incremental + cache + parallel) |
| Learning curve | Low | Moderate | Moderate | Higher (DSL, task graph) |
| IDE / ecosystem support | Minimal | Legacy, shrinking | **Excellent** (Eclipse, IntelliJ, Spring) | Excellent, default for Android |
| Multi-module support | Manual | Manual (`ant` sub-calls) | **First-class** (`<modules>`) | First-class, more flexible |
| Industry usage today | Rare (small scripts only) | Legacy/maintenance projects | **Dominant in enterprise Java/Spring** | Dominant in Android, growing in backend |

## 1.4 Why Maven Became the Industry Standard

- **Convention over configuration** dramatically reduces the decisions a new developer or a new project must make — every Maven project "looks the same."
- **Central Repository** (Maven Central) gave the Java ecosystem one trusted place to publish and consume libraries — this network effect is why almost every Java library ships a Maven-compatible artifact.
- **Standardized lifecycle** means CI/CD tools, IDEs, and static analysis tools can integrate with *any* Maven project the same way (`mvn clean install` works everywhere).
- **Plugin ecosystem** covers virtually every need: compiling, testing, packaging, code coverage, static analysis, Docker image building, deployment.
- Mature, stable, extremely well documented, huge StackOverflow/community knowledge base — critical for enterprise risk-aversion.

## 1.5 Why Spring Boot Prefers Maven/Gradle

- Spring Boot ships a **Spring Boot Parent POM** / **BOM (Bill of Materials)** that pins compatible versions of hundreds of libraries — this only works cleanly with a tool that has first-class **dependency management** (Maven's `<dependencyManagement>` and `<parent>` inheritance, or Gradle's platform/BOM support).
- The **Spring Boot Maven Plugin** (and its Gradle equivalent) knows how to repackage your app into an **executable "fat jar"** with an embedded Tomcat/Jetty/Undertow server — this is central to Spring Boot's "just run the jar" philosophy.
- Spring Initializr (`start.spring.io`) generates Maven or Gradle projects by default because both give reproducible, IDE-agnostic builds that work identically on a laptop and inside a CI/CD pipeline or Docker build.

---

# SECTION 2 — Maven Introduction

## 2.1 What is Maven?

> **Maven is a build automation and project/dependency management tool for Java projects**, based on the concept of a **Project Object Model (POM)** declared in `pom.xml`.

It automates:
1. Compiling source code
2. Running tests
3. Packaging code into distributable formats (`jar`, `war`, `ear`)
4. Managing dependencies (including transitive ones) from local/remote repositories
5. Installing/deploying artifacts for reuse by other projects/teams

## 2.2 Why Maven is Called a "Project Management Tool" (not just a build tool)

Maven manages far more than compilation:
- **Project metadata** — name, version, description, organization, licenses, developers, SCM (source control) info.
- **Dependency graph** — including transitive dependencies and conflict resolution.
- **Build lifecycle & reporting** — site generation, test reports, code coverage.
- **Multi-module orchestration** — an entire suite of related modules (e.g., a microservices monorepo) can be built, versioned, and released together.
- **Plugin execution framework** — anything from database migrations to Docker builds can be plugged into the same lifecycle.

This is why "Project Management Tool" is the historically correct term Apache uses, even though in casual conversation people just say "build tool."

## 2.3 Brief History of Maven

- Created inside the **Apache Jakarta Turbine** project (early 2000s) because every sub-project had a slightly different Ant `build.xml` and there was no consistent way to build them.
- Maven 1 (2004) introduced POM + lifecycle but had many pain points.
- **Maven 2 (2005)** is the version that established the current `pom.xml` structure, the standard directory layout, and the dependency model we still use today.
- **Maven 3 (2010–present)** improved performance, parallel builds, and better plugin/POM validation. Maven 3.9.x/4.x are current as of today.

## 2.4 Maven Architecture

Maven's architecture has 3 major participants:

1. **Maven Core** — the engine that reads `pom.xml`, resolves the effective POM (after merging parent/profiles), and executes the lifecycle.
2. **Maven Plugins** — reusable units of build logic (compiler plugin, surefire plugin, jar plugin, shade plugin, etc.). Maven itself does very little "real work" — almost everything is delegated to a plugin goal bound to a lifecycle phase.
3. **Repositories** — Local (`~/.m2/repository`), Remote/Central, and private/enterprise repositories (Nexus, Artifactory, JFrog) that store and serve dependency artifacts and plugins.

## 2.5 Maven Workflow (End-to-End)

```
 Developer writes code
        │
        ▼
   pom.xml (declares GAV coordinates + dependencies + plugins)
        │
        ▼
   Maven reads pom.xml → resolves effective POM
        │
        ▼
 ┌───────────────────────────────────────────┐
 │  Dependency Resolution:                   │
 │  1) Check Local Repository (~/.m2)        │
 │  2) If missing → check Remote/Central     │
 │  3) Download → cache into Local Repo      │
 └───────────────────────────────────────────┘
        │
        ▼
   Maven executes Lifecycle Phases
   validate → compile → test → package → verify → install → deploy
        │
        ▼
   Build Artifact (.jar / .war) produced in target/
        │
        ▼
 (install) copied to Local Repository for reuse by other local projects
        │
        ▼
 (deploy) pushed to Remote/Enterprise Repository (Nexus/Artifactory) for the team
        │
        ▼
   CI/CD (Jenkins/GitHub Actions) picks it up → Testing → QA → Production
```

### Step-by-step explanation

1. **Developer → pom.xml**: You declare *what* your project is (coordinates: `groupId:artifactId:version`), *what* it needs (`<dependencies>`), and *how* it should be packaged (`<packaging>jar|war</packaging>`).
2. **pom.xml → Maven**: Maven reads this file (and merges in any `<parent>` POM and active `<profiles>`) to compute the **Effective POM** — the fully resolved configuration Maven will actually use.
3. **Maven → Local Repository**: For every declared dependency, Maven first checks `~/.m2/repository`. This is Maven's on-disk cache, keyed by GAV coordinates.
4. **Local Repository → Central/Remote Repository**: If a dependency (or its transitive dependencies) isn't cached locally, Maven downloads it from Maven Central (`https://repo.maven.apache.org`) or any configured remote/enterprise repository (Nexus/Artifactory/JFrog), then caches it locally for next time.
5. **Maven → Build Artifact**: Once everything needed is present, Maven runs the lifecycle phases in order, ultimately producing a `.jar`/`.war` in the `target/` folder.
6. This artifact can then be **installed** (copied to your local repo, for other local projects to consume) or **deployed** (pushed to a shared remote repository so other developers/teams/CI systems can consume it).

**Interview one-liner:** *"Maven resolves the dependency graph declared in pom.xml, downloads missing artifacts from local → remote repositories, executes a standardized lifecycle of phases bound to plugin goals, and produces a build artifact that can be installed locally or deployed to a shared repository."*

---

# SECTION 3 — Maven Installation

## 3.1 Windows Installation

1. Download the Maven binary zip from `https://maven.apache.org/download.cgi`.
2. Extract it to a folder, e.g. `C:\Program Files\Apache\maven`.
3. Set environment variables (System Properties → Environment Variables):
   - `JAVA_HOME` → path to your JDK, e.g. `C:\Program Files\Java\jdk-21`
   - `MAVEN_HOME` (or `M2_HOME`) → `C:\Program Files\Apache\maven`
   - Add `%MAVEN_HOME%\bin` to the system `PATH`
4. Open a new terminal and verify:
```
mvn -version
```

## 3.2 Ubuntu/Linux Installation

**Option A — via apt (simplest, may not be the newest version):**
```bash
sudo apt update
sudo apt install maven -y
mvn -version
```

**Option B — manual install (get the exact latest version):**
```bash
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
sudo tar -xzf apache-maven-3.9.9-bin.tar.gz -C /opt
echo 'export MAVEN_HOME=/opt/apache-maven-3.9.9' >> ~/.bashrc
echo 'export PATH=$MAVEN_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
mvn -version
```

## 3.3 Common Installation Issues & Fixes

| Issue | Cause | Fix |
|---|---|---|
| `'mvn' is not recognized` | `PATH` doesn't include Maven's `bin` folder | Re-check environment variable, open a **new** terminal |
| `JAVA_HOME is not defined correctly` | `JAVA_HOME` missing or pointing to a JRE, not a JDK | Point `JAVA_HOME` to a full JDK install |
| `mvn -version` shows wrong Java version | Multiple JDKs installed, `JAVA_HOME` points to the wrong one | Update `JAVA_HOME`; on Ubuntu use `update-alternatives --config java` |
| IDE uses a different Maven than terminal | Eclipse/IntelliJ ships an "embedded" Maven | In IntelliJ: Settings → Build Tools → Maven → set "Maven home path" explicitly; in Eclipse: Preferences → Maven → Installations |
| Corporate proxy blocks downloads | No proxy configured in `settings.xml` | Add a `<proxies>` block in `~/.m2/settings.xml` |

---

# SECTION 4 — Maven Project Structure (Convention Over Configuration)

```
my-app/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/                → application source code (.java)
│   │   ├── resources/           → non-code files bundled into the artifact
│   │   │                          (application.properties, logback.xml, static templates)
│   │   └── webapp/              → (only for war packaging)
│   │       └── WEB-INF/
│   │           └── web.xml
│   └── test/
│       ├── java/                → test source code (JUnit/Mockito classes)
│       └── resources/           → test-only resources (application-test.properties)
└── target/                      → generated output (compiled classes, jar/war, reports)
    ├── classes/
    ├── test-classes/
    └── my-app-1.0.jar
```

- **`src/main/java`** — production source code, becomes `target/classes/*.class` after `compile`.
- **`src/main/resources`** — copied verbatim onto the classpath (`target/classes`) during `process-resources`. This is where Spring Boot's `application.yml`/`application.properties` live.
- **`src/test/java`** — JUnit/TestNG/Mockito test classes, compiled during `test-compile`, run during `test`.
- **`src/test/resources`** — test-only configuration (e.g., an in-memory H2 datasource config).
- **`target/`** — fully disposable; `mvn clean` deletes it. Never commit this folder to Git.
- **`pom.xml`** — the single source of truth for the project's identity, dependencies, plugins, and build configuration.

## Why Maven Enforces Convention Over Configuration

- Any developer who knows Maven can open **any** Maven project and immediately know where the source, tests, resources, and output live — **zero onboarding cost**.
- IDEs (Eclipse, IntelliJ) auto-detect this structure and configure the classpath/build path automatically — no manual `.classpath` wrangling.
- Tools (Sonar, JaCoCo, Surefire, CI systems) can assume this layout, so plugins "just work" without per-project path configuration.
- It eliminates an entire category of bugs and arguments ("where should this file go?") by making the answer non-negotiable.

*(This can still be overridden via `<build><sourceDirectory>` etc. in the pom, but doing so is rare and discouraged.)*

---

# SECTION 5 — Maven Repositories

## 5.1 Types of Repositories

| Repository | Location | Purpose |
|---|---|---|
| **Local Repository** | `~/.m2/repository` (Linux/Mac) or `C:\Users\<user>\.m2\repository` (Windows) | Per-machine cache of every dependency/plugin ever downloaded; also where `mvn install` places your own build's jar |
| **Central Repository** | Maven Central, browsable via `https://mvnrepository.com` / `https://repo.maven.apache.org` | The global public repository maintained by Apache/Sonatype where almost every open-source Java library is published |
| **Remote/Enterprise Repository** | Company-hosted (Nexus / Artifactory / JFrog) | Proxies Central (so the company has one controlled egress point) **and** hosts the company's own private/internal artifacts |

## 5.2 `settings.xml`

Located at `~/.m2/settings.xml` (user-level) or `$MAVEN_HOME/conf/settings.xml` (global). Used to configure:
- Custom **mirrors** (e.g., route all requests through your company's Nexus instead of Central directly)
- **Server credentials** (username/password or token for a private repository, referenced by `<id>` from `pom.xml`'s `<distributionManagement>`)
- **Proxy** settings for corporate networks
- Active **profiles**

```xml
<settings>
  <mirrors>
    <mirror>
      <id>company-nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>https://nexus.company.com/repository/maven-public/</url>
    </mirror>
  </mirrors>
  <servers>
    <server>
      <id>company-releases</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
```

## 5.3 Enterprise Repository Managers

- **Nexus Repository (Sonatype)** — most common in enterprises; hosts release/snapshot repos, proxies Central, supports Docker/npm/PyPI repos too.
- **JFrog Artifactory** — similar role, popular in larger enterprises, strong support for many artifact types and fine-grained access control.
- These exist so that: (1) internal/private artifacts never need to be published to the public internet, (2) the company has a single controlled point of egress/audit for all third-party downloads, (3) builds are faster (repo manager caches everything close to the CI servers).

## 5.4 Dependency Resolution Process (Step by Step)

```
mvn compile is run
   │
   ▼
Maven reads <dependencies> in effective POM
   │
   ▼
For each dependency (groupId:artifactId:version):
   │
   ├─► Is it already in Local Repository (~/.m2)?
   │        │
   │        YES → use cached jar, done
   │        │
   │        NO
   │        ▼
   ├─► Check configured Remote Repositories (in order: mirrors → repositories in pom → Central)
   │        │
   │        FOUND → download the jar (+ its own pom.xml to discover ITS dependencies)
   │        │        cache it into Local Repository
   │        │
   │        NOT FOUND (in ANY repo)
   │        ▼
   └─► BUILD FAILURE: "Could not resolve dependency"
```

This is a **recursive** process: every dependency's own `pom.xml` is inspected for *its* dependencies (transitive dependencies), and the whole graph is resolved and flattened before compilation begins.

---

# SECTION 6 — `pom.xml` Complete Guide

## 6.1 Minimal Skeleton

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                              https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>My Application</name>
    <description>A sample Spring Boot backend service</description>
</project>
```

## 6.2 Tag-by-Tag Explanation

| Tag | Meaning |
|---|---|
| `<project>` | Root element; declares the POM XML namespace/schema |
| `<modelVersion>` | Always `4.0.0` for modern Maven — declares which POM schema version is used |
| `<groupId>` | Organization/company/package namespace, e.g. `com.example` (reverse-domain convention) |
| `<artifactId>` | The project's unique name within the group, becomes the base jar/war filename |
| `<version>` | The project's version, e.g. `1.0.0` or `1.0.0-SNAPSHOT` (SNAPSHOT = in-development, mutable) |
| `<packaging>` | Output type: `jar` (default), `war`, `pom` (parent/aggregator), `ear` |
| `<name>` / `<description>` | Human-readable metadata, shown in generated docs/IDE |
| `<properties>` | Key-value placeholders reusable via `${property.name}` throughout the POM |
| `<dependencies>` | Direct list of libraries this project needs |
| `<dependencyManagement>` | Declares versions/scopes **without** adding the dependency — children/this-module can then declare the dependency **without repeating the version** |
| `<build>` | Build customization: `<plugins>`, `<finalName>`, `<sourceDirectory>`, `<resources>` |
| `<plugins>` | Plugin executions bound to lifecycle phases |
| `<profiles>` | Conditional configuration blocks activated by environment/flag/OS |
| `<repositories>` | Additional remote repositories (beyond Central) to search for dependencies |
| `<parent>` | Points to a parent POM to inherit configuration/dependency versions from |
| `<modules>` | Lists child modules in a multi-module/aggregator project |
| `<distributionManagement>` | Declares *where* `mvn deploy` should publish this artifact (which remote repo) |

## 6.3 Fully Worked Example (Spring Boot Style)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.4</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>order-service</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
        <mysql.version>8.3.0</mysql.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>${mysql.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

Notice how, because of the `<parent>`, the versions of `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, and `spring-boot-starter-test` are **not specified** — the parent POM's `<dependencyManagement>` already pins them to versions known to work together.

---

# SECTION 7 — Dependencies

## 7.1 Core Concepts

- **Dependency** — a library your project directly needs, declared in `<dependencies>`.
- **Transitive Dependency** — a dependency *of your dependency*, pulled in automatically. Example: adding `spring-boot-starter-web` transitively brings in Jackson, Tomcat embedded, spring-core, etc., without you declaring them.
- **Dependency Tree** — the full graph of direct + transitive dependencies.

## 7.2 Version Conflicts & Resolution Rules

When two different paths in the dependency graph pull in **different versions of the same artifact**, Maven must pick one:

- **Nearest Definition Rule ("nearest wins")** — Maven picks the version declared **closest to your project** in the dependency tree (fewest hops). If module A depends directly on `libX:2.0`, and also depends on B which depends on `libX:1.0`, Maven picks `2.0` because it's "nearer" (depth 1 vs depth 2).
- **First Declaration Wins (tie-break)** — if two dependencies are at the *same* depth, the one declared **first** in the POM wins.
- **Dependency Mediation** — the general term for this whole "which version wins" algorithm.
- **Dependency Convergence** — the desired end-state where every module in a multi-module build agrees on the same version of a shared library (checked via the `maven-enforcer-plugin`'s `dependencyConvergence` rule).

## 7.3 Useful Commands

```bash
mvn dependency:tree        # print the full resolved dependency tree with versions
mvn dependency:analyze     # find unused declared dependencies AND used-but-undeclared ones
mvn dependency:list        # flat list of all resolved dependencies
mvn dependency:tree -Dincludes=com.fasterxml.jackson.core   # filter tree to one artifact
```

### Real-world example
```bash
$ mvn dependency:tree
[INFO] com.example:order-service:jar:1.0.0
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:3.3.4:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:3.3.4:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-json:jar:3.3.4:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.17.2:compile
[INFO] |  \- org.apache.tomcat.embed:tomcat-embed-core:jar:10.1.28:compile
[INFO] \- com.mysql:mysql-connector-j:jar:8.3.0:runtime
```
This instantly tells you *why* a jar ended up on your classpath — critical when debugging `NoSuchMethodError` caused by two different versions of the same library colliding.

---

# SECTION 8 — Dependency Scopes

| Scope | Available at Compile | Available at Runtime | Available in Tests | Packaged in Final Artifact | Typical Use Case |
|---|---|---|---|---|---|
| `compile` (default) | ✅ | ✅ | ✅ | ✅ | Regular libraries, e.g. `spring-boot-starter-web` |
| `provided` | ✅ | ❌ (expected to be supplied by the runtime container) | ✅ | ❌ | Servlet API when deploying a WAR to Tomcat, which already provides it |
| `runtime` | ❌ | ✅ | ✅ | ✅ | JDBC drivers like MySQL connector — code only references the interface (`java.sql.Driver`), never the class directly |
| `test` | ❌ | ❌ | ✅ | ❌ | JUnit, Mockito |
| `system` | ✅ (path given explicitly via `<systemPath>`) | ✅ | ✅ | ❌ | Legacy: a local jar not available in any repository (**discouraged today**) |
| `import` | N/A (only valid inside `<dependencyManagement>`, `packaging=pom`) | N/A | N/A | N/A | Importing a BOM, e.g. `spring-boot-dependencies`, to inherit its managed versions |

### Concrete Examples

```xml
<!-- Servlet API: container (Tomcat) already provides it -->
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>

<!-- MySQL: only needed at runtime, code talks to java.sql interfaces -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.3.0</version>
    <scope>runtime</scope>
</dependency>

<!-- JUnit: only needed to compile/run tests -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.2</version>
    <scope>test</scope>
</dependency>

<!-- Spring Boot: plain compile scope (default) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### Interview Questions — Scopes
- **Q: Why is the Servlet API marked `provided` instead of `compile`?**
  A: Because the servlet container (Tomcat/Jetty) already has the Servlet API jar on its own classpath at runtime. If you packaged it into your WAR too, you'd get duplicate/conflicting classes (`ClassCastException`, `LinkageError`) since two classloaders would load the same class.
- **Q: Why is a JDBC driver `runtime` scope, not `compile`?**
  A: Your code never imports `com.mysql.cj.jdbc.Driver` directly — it only uses `java.sql.Connection`/`DriverManager`, which are part of the JDK. The driver is loaded reflectively at runtime, so it isn't needed to *compile* your code, only to *run* it.
- **Q: What happens if a `test`-scoped dependency is imported in `src/main/java`?**
  A: Compilation of `src/main/java` will fail — test-scope dependencies are not on the main compile classpath, only the test compile/runtime classpath.

---

# SECTION 9 — Maven Lifecycles

Maven has **3 independent lifecycles**, each made of an ordered sequence of **phases**. Running a phase always runs every phase **before** it in the same lifecycle too.

## 9.1 Clean Lifecycle (3 phases)

| Phase | Purpose |
|---|---|
| `pre-clean` | Executes processes needed prior to the actual project cleaning |
| `clean` | Deletes all files generated by the previous build (i.e., deletes `target/`) |
| `post-clean` | Executes processes needed to finalize project cleaning |

## 9.2 Default Lifecycle (the main one — 23 phases, most important listed)

| # | Phase | What happens internally |
|---|---|---|
| 1 | `validate` | Verifies the project is correct and all necessary information is available (valid `pom.xml`) |
| 2 | `initialize` | Sets up build state, e.g. properties or creates directories |
| 3 | `generate-sources` | Generates any source code for inclusion in compilation (e.g., code-gen from an OpenAPI spec) |
| 4 | `process-sources` | Processes source code, e.g. filtering placeholder values |
| 5 | `generate-resources` | Generates resources for inclusion in the package |
| 6 | `process-resources` | Copies/filters `src/main/resources` into `target/classes` |
| 7 | **`compile`** | `maven-compiler-plugin` invokes `javac` → produces `target/classes/*.class` |
| 8 | `process-classes` | Post-processes compiled classes, e.g. bytecode enhancement |
| 9-14 | `generate-test-sources` … `process-test-classes` | Same idea as steps 3–8, but for the `src/test` tree |
| 15 | **`test`** | `maven-surefire-plugin` runs unit tests. These tests must not require the code to be packaged or deployed |
| 16 | `prepare-package` | Any operations needed before actual packaging (e.g., unpacking dependencies for a fat jar) |
| 17 | **`package`** | `maven-jar-plugin` / `maven-war-plugin` takes compiled code and packages it as a distributable `.jar`/`.war` |
| 18-20 | `pre-integration-test` … `post-integration-test` | Sets up environment, runs integration tests (via `maven-failsafe-plugin`), tears down environment |
| 21 | `verify` | Runs checks to validate the package is valid and meets quality criteria |
| 22 | **`install`** | `maven-install-plugin` copies the artifact into the **Local Repository** (`~/.m2`) so other local projects can use it as a dependency |
| 23 | **`deploy`** | `maven-deploy-plugin` copies the final package to a **remote/enterprise repository** for sharing with other developers/projects/CI |

## 9.3 Site Lifecycle (4 phases)

| Phase | Purpose |
|---|---|
| `pre-site` | Executes processes needed prior to actual project site generation |
| `site` | Generates the project's documentation site (`target/site/index.html`) |
| `post-site` | Executes processes needed to finalize site generation, prepares for deployment |
| `site-deploy` | Deploys the generated site documentation to a specified web server |

## 9.4 Lifecycle Flow Diagram

```
CLEAN LIFECYCLE          DEFAULT LIFECYCLE (main)                       SITE LIFECYCLE
pre-clean                validate → initialize → generate-sources →     pre-site
   │                      process-sources → generate-resources →           │
   ▼                      process-resources → compile → process-classes    ▼
 clean                    → generate-test-sources → process-test-sources  site
   │                      → generate-test-resources → process-test-       │
   ▼                      resources → test-compile → process-test-classes ▼
post-clean                → test → prepare-package → package →         post-site
                          pre-integration-test → integration-test →        │
                          post-integration-test → verify → install →       ▼
                          deploy                                    site-deploy
```

## 9.5 `mvn clean package` vs `mvn clean install` vs `mvn deploy`

| Command | What it does |
|---|---|
| `mvn clean package` | Deletes `target/`, then runs the **default** lifecycle up to `package` — produces the jar/war **inside `target/`** only. Nothing is shared with other projects. |
| `mvn clean install` | Everything `package` does, **plus** copies the artifact into your **local repository (`~/.m2`)** so other projects on the same machine can declare it as a dependency. |
| `mvn deploy` | Everything `install` does, **plus** uploads the artifact to a **remote/enterprise repository** (Nexus/Artifactory) so other developers, other machines, and CI/CD pipelines can consume it. Requires `<distributionManagement>` + credentials in `settings.xml`. |

**Interview one-liner:** *"package builds a local artifact, install shares it with other projects on my machine via the local repo, deploy shares it with the whole team/organization via a remote repo."*

---

# SECTION 10 — Maven Commands Cheat-Sheet

| Command | Purpose | Typical Output | When to Use | Interview Relevance |
|---|---|---|---|---|
| `mvn validate` | Checks project is correct/complete | Pass/fail, no artifact | Rarely run alone | Shows you know the full lifecycle |
| `mvn compile` | Compiles main source | `target/classes/*.class` | Quick compile-check during development | Basic |
| `mvn test` | Runs unit tests | Surefire reports in `target/surefire-reports` | Before every commit | Very common |
| `mvn package` | Builds jar/war | `target/*.jar` or `*.war` | Producing a deployable artifact locally | Very common |
| `mvn verify` | Runs integration tests + quality checks | Failsafe reports | CI pipelines, before merging | Common in senior interviews |
| `mvn install` | package + copy to local repo | Artifact appears in `~/.m2` | Sharing a library with sibling local projects | Very common |
| `mvn deploy` | install + upload to remote repo | Artifact appears in Nexus/Artifactory | Publishing a release/snapshot | Common in senior/DevOps interviews |
| `mvn clean` | Deletes `target/` | Empty target dir | Before a fresh build, when stale classes suspected | Basic |
| `mvn dependency:tree` | Prints dependency graph | Tree listing with scopes | Debugging version conflicts | Very common |
| `mvn dependency:analyze` | Flags unused/undeclared deps | Warnings list | Cleaning up bloated poms | Senior-level |
| `mvn help:effective-pom` | Prints the fully resolved POM (after parent/profile merge) | Full XML | Debugging "why is this version being used" | Senior-level |
| `mvn help:effective-settings` | Prints fully resolved `settings.xml` | Full XML | Debugging repo/mirror/proxy issues | DevOps-relevant |
| `mvn versions:display-dependency-updates` | Shows newer available versions of your dependencies | List of outdated deps | Dependency hygiene / security patching | Senior-level, security-conscious teams |
| `mvn -o ...` | Offline mode — never hits network | Uses local repo cache only | No internet / airplane / restricted CI runner | Practical |
| `mvn -DskipTests package` | Skips running tests (still compiles them) | Faster package | Quick local iteration (not for CI) | Very common, watch the "don't do this in CI" follow-up |
| `mvn -Dmaven.test.skip=true package` | Skips **compiling and running** tests entirely | Fastest | Emergency builds | Interviewers ask the difference vs `-DskipTests` |

---

# SECTION 11 — Maven Plugins

## 11.1 What is a Plugin?

A **plugin** is a collection of **goals** (a goal = one unit of work, technically implemented as a **Mojo** = "Maven plain Old Java Object"). Maven itself is almost an empty shell — literally every real action (compiling, testing, packaging) is delegated to a plugin goal.

## 11.2 Goals, Mojo, Lifecycle Binding

- A **goal** is `pluginPrefix:goalName`, e.g. `compiler:compile`, `surefire:test`, `jar:jar`.
- Goals are **bound** to lifecycle phases by default (this is why typing `mvn compile` silently triggers `compiler:compile`), but you can also invoke a goal **directly** without going through the lifecycle: `mvn dependency:tree`, `mvn exec:java`.
- **Mojo** = the actual Java class implementing a goal's logic.

## 11.3 Default Plugin Bindings (jar packaging)

| Phase | Plugin:Goal |
|---|---|
| compile | `maven-compiler-plugin:compile` |
| test | `maven-surefire-plugin:test` |
| package | `maven-jar-plugin:jar` |
| install | `maven-install-plugin:install` |
| deploy | `maven-deploy-plugin:deploy` |

## 11.4 Important Plugins Reference

| Plugin | Goals | Purpose |
|---|---|---|
| `maven-compiler-plugin` | `compile`, `testCompile` | Compiles main and test `.java` files |
| `maven-surefire-plugin` | `test` | Runs unit tests during the `test` phase |
| `maven-failsafe-plugin` | `integration-test`, `verify` | Runs integration tests (naming convention `*IT.java`), separately from unit tests |
| `maven-jar-plugin` | `jar` | Packages compiled classes into a `.jar` |
| `maven-war-plugin` | `war` | Packages a web app into a `.war` |
| `maven-shade-plugin` | `shade` | Builds an **uber/fat jar** — bundles all dependency classes into one jar, with relocation/merging support |
| `maven-assembly-plugin` | `single` | Similar to shade, more general-purpose (can also build zips/tarballs of a distribution) |
| `exec-maven-plugin` | `exec`, `java` | Runs a Java class or external process directly through Maven (`mvn exec:java`) |
| `spring-boot-maven-plugin` | `repackage`, `run` | Repackages the jar into a Spring Boot **executable fat jar** with an embedded server, and can run the app directly (`mvn spring-boot:run`) |
| `maven-antrun-plugin` | `run` | Lets you embed raw Ant tasks inside a Maven build (legacy interop) |
| `tomcat-maven-plugin` | `start`, `stop`, `deploy`, `redeploy`, `undeploy` | Deploys the project directly into an embedded/external Tomcat |
| `jboss-maven-plugin` | (same idea) | Deploys into JBoss/WildFly |

### Example — `exec-maven-plugin`
```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.5.0</version>
    <configuration>
        <mainClass>in.ioi.pw.TestApp</mainClass>
    </configuration>
</plugin>
```
Run with: `mvn compile exec:java` — this is exactly the pattern from the faculty notes (`CalculatorApp` → `CalculatorService`, consumed by `OrderCartService`/`TestApp`).

### Example — `maven-shade-plugin` (Fat Jar)
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals><goal>shade</goal></goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.abc.edtech.MainApp</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```
Run with `mvn clean package`, then `java -jar OrderService-2.0.jar` — no external classpath needed, because `DBUtil.jar`, `mysql-connector-j`, and `hibernate-core` classes are all merged **inside** `OrderService-2.0.jar`, and the `Manifest.MF`'s `Main-Class` entry (written by the `ManifestResourceTransformer`) tells the JVM which class contains `main()`. This is exactly the "Fat Jar" flow from the diagrammed notes: `DBUtil` (with `mvn install`) → consumed by `OrderService` → shaded into one runnable jar.

---

# SECTION 12 — Testing in Maven

## 12.1 Unit Tests vs Integration Tests

| | Unit Test | Integration Test |
|---|---|---|
| Scope | Single class/method, isolated (mocks for collaborators) | Multiple components together (DB, web layer, message broker) |
| Speed | Fast (milliseconds) | Slower (seconds+) |
| Naming convention | `*Test.java` | `*IT.java` (Integration Test) |
| Maven plugin | `maven-surefire-plugin` | `maven-failsafe-plugin` |
| Bound to phase | `test` | `integration-test` / `verify` |
| Should require packaging? | No | Often yes (may run against the packaged artifact) |

## 12.2 JUnit + Mockito + Surefire/Failsafe

- **JUnit** (4 or 5/Jupiter) — the testing framework/annotations (`@Test`, `@BeforeEach`).
- **Mockito** — mocking framework for isolating a unit from its dependencies.
- **Surefire** — the Maven plugin that *discovers and executes* `*Test.java` classes and produces `target/surefire-reports/*.txt` / `.xml`.
- **Failsafe** — same idea, but for `*IT.java`, and — critically — it does **not** fail the build immediately on a test failure; it waits until `verify` so that `post-integration-test` cleanup (e.g., stopping a test DB container) still runs.

## 12.3 Code Coverage — JaCoCo

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>verify</phase>
            <goals><goal>report</goal></goals>
        </execution>
    </executions>
</plugin>
```
Produces an HTML coverage report at `target/site/jacoco/index.html`. Widely used with a minimum coverage threshold enforced via the `check` goal in CI pipelines.

---

# SECTION 13 — Web Applications with Maven

## 13.1 WAR Packaging & Structure

```xml
<packaging>war</packaging>
```

```
src/main/webapp/
└── WEB-INF/
    ├── web.xml
    └── lib/          ← Maven auto-copies your compile/runtime-scoped jars here during packaging
```

## 13.2 `provided` Scope for Servlet API

Because Tomcat itself supplies `jakarta.servlet-api` on its own classpath, your project declares it `provided` so it's available to compile against, but **not bundled** into `WEB-INF/lib` (avoiding classloader conflicts).

## 13.3 Tomcat Maven Plugin

```xml
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <path>/myapp</path>
        <port>8081</port>
    </configuration>
</plugin>
```
```bash
mvn tomcat7:run        # runs an embedded Tomcat directly from Maven — fast dev loop
mvn tomcat7:deploy      # deploys the WAR to a remote/managed Tomcat instance
```

## 13.4 Real-World Deployment Flow

```
mvn clean package  → target/EmployeeCrudApp-3.0.war
        │
        ▼
Copy .war into <TOMCAT_HOME>/webapps/
        │
        ▼
Tomcat auto-explodes the WAR → deploys the web app
        │
        ▼
Access via http://localhost:8080/EmployeeCrudApp-3.0/
```
This matches the faculty note's `EmployeeCrudApp` example: servlet + JSP (or Thymeleaf) + MySQL + Hibernate, packaged as a `.war`, dropped into Tomcat's `webapps` folder.

---

# SECTION 14 — Spring Boot + Maven

## 14.1 Why Spring Boot Uses Maven

Spring Boot leans entirely on Maven's inheritance + BOM model to give you **curated, tested dependency versions** without you ever specifying a version number yourself, plus a plugin that turns your app into a **self-contained runnable jar**.

## 14.2 Spring Boot Parent POM vs BOM Import

**Option A — inherit from `spring-boot-starter-parent` (simplest, most common):**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.4</version>
</parent>
```

**Option B — import the BOM only (when you already have your own corporate parent POM):**
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.3.4</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
This is exactly where scope `import` is used — it pulls in the **managed dependency versions** from `spring-boot-dependencies` without inheriting the parent's plugin/build configuration too.

## 14.3 Starter Dependencies

"Starters" are curated dependency bundles: `spring-boot-starter-web` = Spring MVC + embedded Tomcat + Jackson; `spring-boot-starter-data-jpa` = Spring Data JPA + Hibernate + transaction management. They exist to eliminate manual assembly of "which 6 jars do I need for a REST API."

## 14.4 Fat Jar / Executable Jar (`spring-boot-maven-plugin`)

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
```bash
mvn clean package
java -jar target/order-service-1.0.0.jar
```
Internally, `spring-boot-maven-plugin` **repackages** the plain jar (produced by `maven-jar-plugin`) into a **fat/executable jar** with a special nested-jar layout (`BOOT-INF/classes`, `BOOT-INF/lib`) and a custom `Main-Class` (`JarLauncher`) that knows how to load classes from those nested jars — this is different from the shade plugin's flat-merge approach, but achieves the same practical outcome: **one jar, `java -jar` it, done.**

## 14.5 Complete Spring Boot `pom.xml` Example

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.4</version>
        <relativePath/>
    </parent>

    <groupId>com.abc.edtech</groupId>
    <artifactId>order-service</artifactId>
    <version>2.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

# SECTION 15 — Maven Inheritance (Parent/Child POM)

A **Parent POM** centralizes shared configuration (dependency versions, plugin versions, Java version, common properties) so **Child POMs** don't repeat it.

```xml
<!-- parent/pom.xml -->
<packaging>pom</packaging>
<properties>
    <java.version>21</java.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
```xml
<!-- child/pom.xml -->
<parent>
    <groupId>com.company</groupId>
    <artifactId>company-parent</artifactId>
    <version>1.0.0</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <!-- no version needed — inherited from parent's dependencyManagement -->
    </dependency>
</dependencies>
```

**Benefits:** one place to bump a shared library's version across dozens of microservices; enforces consistent Java version, encoding, and plugin versions org-wide; reduces copy-paste drift.

**Real-world enterprise pattern:** a company maintains its own `company-parent-pom` (itself inheriting from `spring-boot-starter-parent`), pinning internal libraries, shared checkstyle/PMD rules, and internal Nexus `<distributionManagement>` — every microservice repo's POM just does `<parent>company-parent-pom</parent>` and inherits all of it.

---

# SECTION 16 — Multi-Module Projects

```
ecommerce-platform/                (packaging = pom, the "Aggregator")
├── pom.xml                        (<modules>order-service, inventory-service, common-lib</modules>)
├── common-lib/
│   └── pom.xml
├── order-service/
│   └── pom.xml   (depends on common-lib)
└── inventory-service/
    └── pom.xml   (depends on common-lib)
```

```xml
<!-- ecommerce-platform/pom.xml -->
<packaging>pom</packaging>
<modules>
    <module>common-lib</module>
    <module>order-service</module>
    <module>inventory-service</module>
</modules>
```

- **Aggregator POM** — the top-level POM whose only job is to list `<modules>` so `mvn install` from the root builds everything in the correct dependency order.
- **Build order** is computed automatically by Maven's **Reactor** based on inter-module dependencies (it topologically sorts the modules — `common-lib` builds before `order-service` because `order-service` depends on it).
- **Dependency sharing** — `common-lib` is `mvn install`-ed into the local repo first (by the reactor), then immediately available to sibling modules in the same reactor build.

**Real-world microservices example:** a monorepo with `common-lib` (shared DTOs/exceptions), `order-service`, `inventory-service`, `payment-service` — each is its own deployable Spring Boot jar, but they all share validation logic and DTOs through `common-lib`, versioned and released together via the aggregator POM.

---

# SECTION 17 — Maven Profiles

Profiles let the **same POM** behave differently depending on environment/flag.

```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation><activeByDefault>true</activeByDefault></activation>
        <properties>
            <db.url>jdbc:mysql://localhost:3306/dev_db</db.url>
        </properties>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <db.url>jdbc:mysql://test-server:3306/test_db</db.url>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <db.url>${env.PROD_DB_URL}</db.url>
        </properties>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <skipTests>true</skipTests>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```
```bash
mvn clean package -Pprod          # activates the 'prod' profile explicitly
```
Common real-world use: different database URLs per environment, skipping slow integration tests locally but not in CI, enabling a code-coverage gate only in the `ci` profile.

---

# SECTION 18 — Maven Properties

```xml
<properties>
    <java.version>21</java.version>
    <spring-boot.version>3.3.4</spring-boot.version>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```
- Reused anywhere via `${property.name}` — dependency versions, plugin configuration, even inside `src/main/resources` files if `resource filtering` is enabled.
- **Best practice:** define every shared version **once** as a property, reference it everywhere, so upgrading a library is a one-line change.

---

# SECTION 19 — Exclusions

When a **transitive dependency** conflicts with a version you need (or pulls in something insecure/unwanted), exclude it:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```
**Real-world example:** swapping the embedded server from Tomcat to Undertow/Jetty in a Spring Boot app, or excluding an old `commons-logging` transitively pulled in by a legacy library so it doesn't clash with your SLF4J/Logback setup.

---

# SECTION 20 — Maven Interview Questions

### Fundamentals
1. **What is Maven?** A build automation and project management tool that uses a declarative `pom.xml` and a standardized lifecycle to compile, test, package, and distribute Java projects.
2. **Why Maven over Ant?** Maven gives you convention-over-configuration, built-in transitive dependency management, and a standard lifecycle; Ant requires you to script every step and manage dependencies manually.
3. **What is the difference between Maven and Gradle?** Maven is declarative XML with a fixed lifecycle; Gradle is a Groovy/Kotlin DSL with a flexible, incremental task graph, generally faster on very large builds.
4. **What is a POM?** Project Object Model — the XML file (`pom.xml`) describing a project's identity, dependencies, plugins, and build configuration.
5. **What is GAV?** groupId + artifactId + version — the unique coordinate identifying any Maven artifact.
   - *Follow-up:* What happens if two dependencies share GAV but different scope? → Scope conflicts are resolved separately from version conflicts; the "widest" applicable scope generally wins, though this is a lesser-known edge case worth verifying with `dependency:tree`.

### Repositories & Dependencies
6. **Where is the local repository located?** `~/.m2/repository` by default (configurable in `settings.xml`).
7. **What's the difference between local, central, and remote repositories?** Local is your machine's cache; Central is Apache's public global repository; remote/enterprise (Nexus/Artifactory) is a company-hosted repository that proxies Central and hosts private artifacts.
8. **What is a transitive dependency?** A dependency of your dependency, pulled in automatically without you declaring it directly.
9. **How does Maven resolve version conflicts?** "Nearest wins" (nearest definition rule) — the version declared at the shallowest depth in the dependency tree wins; ties go to the first declaration.
   - *Senior follow-up:* How would you enforce dependency convergence across a multi-module build? → `maven-enforcer-plugin` with the `dependencyConvergence` rule, or explicitly pin versions via `<dependencyManagement>`.
10. **How do you view the dependency tree?** `mvn dependency:tree`.
11. **How do you exclude a transitive dependency?** `<exclusions><exclusion>` inside the `<dependency>` block.
12. **Difference between `dependencies` and `dependencyManagement`?** `dependencyManagement` only *declares* versions/scopes for children/BOM consumers to reference — it does **not** add the dependency to the classpath by itself; `dependencies` actually adds it.

### Scopes
13. **Explain all Maven dependency scopes.** (see Section 8 table)
14. **Why is Servlet API `provided`?** Because the servlet container already supplies it at runtime; bundling it too would cause classloader conflicts.
15. **Why is a JDBC driver `runtime` scope?** Code only references JDK interfaces (`java.sql.*`); the driver class is loaded reflectively at runtime only.
16. **When would you use `system` scope, and why is it discouraged?** Only for a jar not available in any repository, referenced via an absolute local path; discouraged because it breaks reproducibility across machines/CI.
17. **What is `import` scope used for?** Only inside `<dependencyManagement>` to import a BOM's managed versions (e.g., `spring-boot-dependencies`).

### Lifecycle
18. **Name Maven's three lifecycles.** clean, default, site.
19. **What phases does the default lifecycle include (name the key ones)?** validate, compile, test, package, verify, install, deploy (plus several intermediate phases).
20. **Difference between `package`, `install`, and `deploy`?** package → artifact in `target/`; install → also copied to local repo; deploy → also uploaded to remote/enterprise repo.
21. **If I run `mvn install`, does it run `test` first?** Yes — running any phase runs every earlier phase in that lifecycle, so `test` (and `compile`, `validate`, etc.) all run before `install`.
22. **What is the difference between Surefire and Failsafe?** Surefire runs unit tests during `test` and fails the build immediately; Failsafe runs integration tests (`*IT.java`) bound to `integration-test`/`verify`, deferring failure reporting to `verify` so cleanup still happens.

### Plugins
23. **What is a Maven plugin?** A reusable unit of build logic exposing one or more "goals," bound to lifecycle phases.
24. **What is a Mojo?** The Java class implementing a plugin goal ("Maven plain Old Java Object").
25. **How do you run a goal without invoking the full lifecycle?** Call it directly, e.g. `mvn dependency:tree`, `mvn exec:java`.
26. **Difference between shade plugin and Spring Boot Maven plugin for building a runnable jar?** Shade flat-merges all dependency classes into one jar (uber jar); Spring Boot's plugin nests dependencies under `BOOT-INF/lib` and uses a custom `JarLauncher`, avoiding class-merge conflicts and preserving each jar's own manifest.

### Spring Boot Specific
27. **Why doesn't Spring Boot require you to specify dependency versions?** Because of the parent POM/BOM's `<dependencyManagement>`, which pins tested-compatible versions.
28. **What does `spring-boot-starter-parent` give you besides dependency versions?** Default plugin configuration, UTF-8 encoding, Java version defaults, resource filtering defaults.
29. **How do you create an executable jar for a Spring Boot app?** Include `spring-boot-maven-plugin` in `<build><plugins>`, then `mvn clean package`, then `java -jar target/app.jar`.
30. **How would you use a corporate parent POM AND Spring Boot's version management together?** Import `spring-boot-dependencies` as a BOM via `<scope>import</scope>` inside your own corporate parent's `<dependencyManagement>`.

### Multi-Module / Enterprise
31. **What is an aggregator/parent POM with `packaging=pom`?** A POM whose only purpose is to list child `<modules>` so a single `mvn install` builds them all in dependency order via Maven's Reactor.
32. **How does Maven decide build order in a multi-module project?** Topological sort of inter-module dependencies performed by the Reactor.
33. **What is the benefit of Maven inheritance in a microservices org?** Centralizes shared dependency/plugin versions and build standards, so every service inherits consistency and upgrades happen in one place.

### Debugging / Senior-Level
34. **A dependency shows in `dependency:tree` at an unexpected version — how do you fix it?** Explicitly declare the desired version directly, or in `<dependencyManagement>`, so it "wins" regardless of transitive depth; alternatively use `<exclusions>` on the transitive path pulling in the wrong version.
35. **`mvn clean install` works locally but fails in CI — what do you check first?** Whether the same Maven/JDK version is used, whether `settings.xml`/repository mirrors differ, and whether any `SNAPSHOT` dependency changed between the two runs (SNAPSHOTs are mutable and not reproducible by design).
36. **What causes `NoClassDefFoundError` vs `ClassNotFoundException` in a Maven-built app?** `ClassNotFoundException` = the class truly isn't on the classpath at all (missing dependency/wrong scope); `NoClassDefFoundError` = the class *was* available at compile time but is missing/mismatched at runtime (commonly a `provided`/`test`-scope dependency needed at runtime, or a version conflict where an incompatible class was loaded instead).
37. **How do you check what will actually be built after all inheritance/profiles are merged?** `mvn help:effective-pom`.

---

# SECTION 21 — Maven Debugging Guide

## 21.1 Troubleshooting Flowchart (general)

```
Build fails
   │
   ▼
Read the FIRST "ERROR" line (Maven often prints many; the root cause is usually the first)
   │
   ├─ "Could not resolve dependency" / "Could not find artifact" → §21.2
   ├─ "Plugin execution failed" → §21.3
   ├─ ClassNotFoundException / NoClassDefFoundError at runtime → §21.4
   ├─ Test failures → §21.5
   └─ Network / proxy / SSL errors → §21.6
```

## 21.2 Dependency Not Found / Could Not Resolve Artifact

- Verify the exact `groupId:artifactId:version` on `mvnrepository.com` — typos are the #1 cause.
- Run `mvn -U clean install` to force Maven to re-check remote repositories (ignores cached "not found" markers, especially important for `SNAPSHOT` versions).
- Delete the specific corrupted folder from `~/.m2/repository/<group>/<artifact>/<version>/` and rebuild — Maven sometimes caches a partially-downloaded/corrupted jar.
- Check `settings.xml` mirrors — if a corporate Nexus mirror is misconfigured or down, Maven may never reach Central at all.

## 21.3 Plugin Execution Failed

- Read the full stack trace with `mvn -e` (errors) and `mvn -X` (full debug) — the actual cause is usually a few lines into the Mojo's own exception, not the generic "Plugin execution failed" summary line.
- Check the plugin's configuration in your POM against its documentation — a common cause is a missing `<mainClass>` for `exec-maven-plugin`/shade's `ManifestResourceTransformer`.
- Check plugin **version compatibility** with your Maven/JDK version — very old or bleeding-edge plugin versions are common culprits after a JDK upgrade.

## 21.4 ClassNotFoundException / NoClassDefFoundError

| Symptom | Likely Cause | Fix |
|---|---|---|
| `ClassNotFoundException` at runtime, but compiles fine | Dependency scoped `provided`/`test`, so it's absent at runtime; or it's simply missing from the fat/executable jar | Fix scope, or verify shade/spring-boot-maven-plugin actually ran during `package` |
| `NoClassDefFoundError` | A class *was* on the compile classpath but a conflicting/older version is loaded at runtime | Run `dependency:tree`, find the colliding version, pin the correct one via `dependencyManagement` or `exclusions` |
| Works in IDE, fails via `java -jar` | IDE uses its own classpath resolution; the fat jar may not have been rebuilt, or the shade/Boot plugin isn't configured to run on `package` | Rebuild with `mvn clean package`, verify `<phase>package</phase>` binding on the plugin execution |

## 21.5 Test Failures Breaking the Build

- Read `target/surefire-reports/*.txt` for the actual assertion failure/stack trace, not just the console summary.
- If tests fail only in CI (e.g., DB/network dependent) — this usually indicates a unit test that should have been an **integration test** (`*IT.java` via Failsafe) with proper environment setup/teardown.
- `-DskipTests` still **compiles** tests (catches compile errors) but doesn't run them; `-Dmaven.test.skip=true` skips compiling tests entirely — use the former for a "just skip the slow tests" scenario, never disable tests permanently in CI.

## 21.6 Corrupted Local Repository / Network / Proxy / SSL / Firewall

| Issue | Fix |
|---|---|
| Corrupted local repo (partial download, `.lastUpdated` files) | Delete the offending artifact folder under `~/.m2/repository`, rerun `mvn -U clean install` |
| Corporate proxy blocking direct internet | Add `<proxies>` in `settings.xml`, or point `<mirrors>` at the company Nexus so Maven never talks to the raw internet |
| SSL handshake errors to Central/Nexus | Usually a missing corporate root CA in the JDK's truststore — import the cert via `keytool -importcert`, or ensure `JAVA_HOME` points at a JDK with the corporate CA already installed |
| Port already in use (e.g., embedded Tomcat/Boot dev server) | `lsof -i:8080` (Linux) / `netstat -ano | findstr 8080` (Windows) to find and kill the conflicting process, or change `server.port` |
| Build failure with no clear reason | Run `mvn -X` for full debug trace; check `mvn -v` matches expected Java/Maven version; try `mvn help:effective-pom` to rule out unexpected inherited configuration |

---

# SECTION 22 — Maven for Spring Boot Developers: Daily Tasks

| Task | Command / Action |
|---|---|
| Add a dependency | Add `<dependency>` block; if version is unmanaged, check `mvnrepository.com` for latest stable |
| Remove a dependency | Delete the `<dependency>` block, then run `mvn dependency:analyze` to confirm nothing still needs it |
| Check dependency tree | `mvn dependency:tree` |
| Create an executable jar | Ensure `spring-boot-maven-plugin` is in `<build><plugins>`, run `mvn clean package` |
| Run the application via Maven | `mvn spring-boot:run` |
| Skip tests for a quick local build | `mvn clean package -DskipTests` |
| Build a Docker image | Use `spring-boot:build-image` (Cloud Native Buildpacks, no Dockerfile needed) or a multi-stage `Dockerfile` that runs `mvn clean package` then copies the jar |
| CI/CD integration — GitHub Actions | `actions/setup-java` + cache `~/.m2` + `mvn -B clean verify` |
| CI/CD integration — Jenkins | `withMaven` pipeline step, or a `sh 'mvn clean verify'` stage, with Nexus credentials injected for `deploy` |

### Minimal GitHub Actions snippet
```yaml
- uses: actions/setup-java@v4
  with:
    distribution: 'temurin'
    java-version: '21'
    cache: 'maven'
- run: mvn -B clean verify
```

---

# SECTION 23 — Real-World Maven Best Practices

- **Version locking:** never leave a dependency version to "whatever transitively resolves" for anything security-sensitive — pin explicit versions via `<dependencyManagement>`.
- **Dependency management centralization:** use a corporate parent POM or BOM so every microservice shares consistent versions.
- **Repository strategy:** always route through a company Nexus/Artifactory mirror, both for security auditing and to avoid Central outages blocking your CI.
- **Build reproducibility:** avoid `SNAPSHOT` dependencies in anything that ships to production — pin exact release versions. Use `mvn versions:display-dependency-updates` periodically to review upgrades deliberately, not accidentally.
- **Security:** run `mvn dependency:tree`/`org.owasp:dependency-check-maven` in CI to catch known-vulnerable transitive dependencies (e.g., a stale Log4j version).
- **Performance:** enable the local build cache (`-o` when offline is possible), use `mvn -T 1C` for parallel multi-module builds, keep the Reactor's module graph as flat/decoupled as practical.
- **CI/CD:** always run `mvn clean verify` (not just `package`) in CI so integration tests and quality gates actually execute; separate a fast "PR check" pipeline (unit tests only) from a slower "merge to main" pipeline (full verify + deploy).
- **Release management:** use the `maven-release-plugin` or a simple version-bump + `mvn deploy` convention for cutting releases; tag the exact commit in Git that produced a released artifact.

---

# SECTION 24 — Maven Quick Revision Sheet (1 Page)

**Most-used commands**
```
mvn clean            # delete target/
mvn compile           # compile src/main
mvn test              # run unit tests (Surefire)
mvn package            # build jar/war in target/
mvn verify             # run integration tests (Failsafe) + quality checks
mvn install            # package + copy to ~/.m2 (local repo)
mvn deploy             # install + upload to remote/enterprise repo
mvn clean install      # most common single command for "give me a fresh, shareable build"
mvn dependency:tree    # see resolved dependency graph
mvn dependency:analyze # find unused/undeclared dependencies
mvn help:effective-pom # see the fully merged/resolved POM
mvn spring-boot:run    # run a Spring Boot app directly
mvn -DskipTests package    # build fast, skip running tests
```

**Lifecycle (default):** validate → compile → test → package → verify → install → deploy
**Lifecycle (clean):** pre-clean → clean → post-clean
**Lifecycle (site):** pre-site → site → post-site → site-deploy

**Scopes:** compile (default, everywhere) · provided (compile+test only, container supplies at runtime) · runtime (not needed to compile) · test (tests only) · system (local path, legacy) · import (BOM only)

**Repositories:** Local (`~/.m2/repository`) → Remote/Enterprise (Nexus/Artifactory) → Central (Maven Central)

**Key plugins:** compiler · surefire · failsafe · jar · war · shade · assembly · exec · spring-boot-maven-plugin · tomcat

**Troubleshooting reflexes:** `mvn -U` (force refresh) · `mvn -X` (full debug trace) · `mvn -e` (error trace) · delete the bad folder under `~/.m2/repository` · `mvn help:effective-pom` (see what's really being used)

---

# SECTION 25 — Concepts from Faculty Notes: Reconciliation Map

This maps every concept present in the original faculty notes/diagrams to where it is fully covered here — confirming nothing was lost.

| Faculty Note Concept | Covered In |
|---|---|
| "Maven is a build automation and dependency management tool" | Section 2.1 |
| Without Maven: `.java → .class → .jar/.war → run/deploy` manual flow | Section 1.2 |
| Life Cycle terminology (phase → plugin → goal) | Sections 9, 11 |
| `validate, compile, test, package, install, deploy` phases | Section 9.2 |
| `maven-compiler-plugin` → `javac` → `target/classes` | Section 9.2, 11.4 |
| `maven-surefire-plugin` for tests | Section 9.2, 12.2 |
| `maven-war-plugin` / `maven-jar-plugin` → package | Section 9.2, 11.4, 13 |
| `maven-install-plugin` / `maven-deploy-plugin` | Section 9.2, 9.5 |
| Running a Java program via Maven — `exec:java` (Mojo plugin) | Section 11.4 |
| `EmployeeCrudApp` WAR / Tomcat / `WEB-INF/lib` / servlet + JSP/Thymeleaf + MySQL + Hibernate | Section 13 |
| Dependency scopes: `compile`, `runtime`, `test`, `provided` | Section 8 |
| `clean` lifecycle — deletes `target/` | Section 9.1 |
| `site` lifecycle — documentation, `target/site/index.html` | Section 9.3 |
| Creating a jar and using it as a dependency in another project (`CalculatorApp` → `CalculatorService` consumed by `OrderCartService`/`TestApp`) | Section 7, 11.4 |
| `mvn install` copying jar to local repository | Section 9.2, 9.5 |
| `exec-maven-plugin` configuration with `<mainClass>` | Section 11.4 |
| `DBUtil` (Hibernate + MySQL + Lombok) consumed by `OrderService` | Section 6.3, 15 |
| `maven-shade-plugin` with `ManifestResourceTransformer` for building a runnable Fat Jar | Section 11.4, 14.4 |
| `java -jar OrderService-2.0.jar` running a shaded fat jar | Section 11.4 |
| Diagram: Developer → Project → Maven[Dependency, .jar] → Testing Team → GIT/Jenkins/`mvn deploy` → Nexus Repository → QA → Production | Section 2.5, 5.3, 22 |
| Diagram: Maven[pom.xml, archetype] → Local Repository (`.m2`) → Remote/Central Repository, packaging `jar`(standalone)/`war`(webapplication), `src/main/java|resources|webapp/WEB-INF/web.xml` | Sections 2.5, 4, 5, 6 |
| `settings.xml`, `mvnrepository.com`, Nexus, artifact terminology | Section 5.2, 5.3 |
| Maven Life Cycle table — 3 lifecycles (clean/default/site), 23 default phases, phase descriptions | Section 9 (full table reproduced/expanded) |
| Plugin table: `maven-compiler-plugin`, `exec-maven-plugin`, `maven-antrun-plugin`, `tomcat-maven-plugin`, `jboss-maven-plugin` and their goals/purpose | Section 11.4 |
| GAV (`groupId`, `artifactId`, `version`) + `<dependencies>`/`<repository>` structure in `pom.xml`, flow to Local/Remote/Central Repository | Sections 6, 5.4 |
| Double-posting / GET-POST / 302 redirect diagram (Post-Redirect-Get pattern) | *Out of scope for Maven — this is a web/servlet request-handling pattern, not a Maven concept; noted here for completeness but not expanded in this Maven-focused document.* |

---

# Revision Plans

## 30-Minute Maven Revision Plan (day-of-interview, quick refresh)
1. **(5 min)** Re-read Section 24 (1-Page Cheat Sheet) top to bottom.
2. **(5 min)** Say out loud, from memory: the 7 key default-lifecycle phases and what plugin backs each.
3. **(5 min)** Re-read the Scopes table (Section 8) — be able to justify `provided` (Servlet API) and `runtime` (JDBC driver) without looking.
4. **(5 min)** Re-read `package` vs `install` vs `deploy` (Section 9.5) and rehearse the one-liner.
5. **(5 min)** Skim Section 20's Q1–Q22 — these are the highest-frequency questions.
6. **(5 min)** Mentally rehearse the Fat Jar flow (shade plugin vs Spring Boot plugin) — Section 11.4/14.4 — a very common "explain what happens when you run `java -jar`" question.

## 1-Day Maven Mastery Plan
- **Morning (2 hrs):** Sections 1–6 (fundamentals, architecture, installation, project structure, repositories, pom.xml). Actually create a tiny Maven project by hand (`mvn archetype:generate`) and inspect its structure.
- **Midday (2 hrs):** Sections 7–11 (dependencies, scopes, lifecycles, commands, plugins). Run `mvn dependency:tree` on a real Spring Boot project you have; intentionally break a scope (change a runtime dep to `provided`) and observe the runtime failure.
- **Afternoon (2 hrs):** Sections 12–16 (testing, web apps, Spring Boot, inheritance, multi-module). Build a 2-module toy project (a `common-lib` + a consumer) exactly like the `DBUtil`/`OrderService` example, `mvn install` the first, then build the fat jar for the second.
- **Evening (2 hrs):** Sections 17–23 (profiles, properties, exclusions, interview Q&A, debugging guide, best practices). Do a full read-through of Section 20 and 21, out loud, as if answering an interviewer.

## 7-Day Maven Deep Dive Roadmap
- **Day 1:** Build tools history + Maven architecture/workflow (Sections 1–2). Deliverable: be able to draw the full Developer→pom→Maven→Local/Central Repo→Artifact diagram from memory.
- **Day 2:** Installation + project structure + repositories (Sections 3–5). Deliverable: install Maven fresh on a VM/WSL, inspect `~/.m2`, explain every folder.
- **Day 3:** `pom.xml` deep dive + dependencies + scopes (Sections 6–8). Deliverable: write a pom from scratch (no copy-paste) covering all six scopes correctly.
- **Day 4:** Lifecycles + commands + plugins (Sections 9–11). Deliverable: explain, phase by phase, what happens when you type `mvn clean install` on a WAR project.
- **Day 5:** Testing + Web apps + Spring Boot integration (Sections 12–14). Deliverable: build and run a Spring Boot fat jar end-to-end, then break and fix a `NoClassDefFoundError` on purpose.
- **Day 6:** Inheritance + multi-module + profiles + properties + exclusions (Sections 15–19). Deliverable: build a 3-module aggregator project (`common-lib`, `service-a`, `service-b`) with a shared parent POM and a `dev`/`prod` profile pair.
- **Day 7:** Full interview Q&A pass + debugging guide + best practices + mock interview (Sections 20–23). Deliverable: answer every question in Section 20 out loud without notes, then do a 30-minute mock interview focused only on Maven.

---

*End of Maven Master Notes.*
