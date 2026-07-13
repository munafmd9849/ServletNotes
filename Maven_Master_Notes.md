# Maven Master Notes (Interview + Developer Edition)

> Based on faculty notes and expanded with industry practices.

## SECTION 1 — BUILD TOOLS FUNDAMENTALS

### What is a Build Process?
- Compile source code
- Run tests
- Package artifacts (JAR/WAR)
- Manage dependencies
- Generate reports
- Deploy applications

### Evolution
| Tool | Description |
|--------|--------|
| Batch | Manual command automation |
| ANT | XML-based build automation |
| Maven | Build + Dependency + Project Management |
| Gradle | Flexible DSL-based build system |

### Why Maven Won
- Convention over Configuration
- Dependency management
- Standard structure
- Reproducible builds
- CI/CD friendly

---

## SECTION 2 — MAVEN INTRODUCTION

### Definition
Maven is a Build Automation and Dependency Management Tool.

### Maven Workflow

```text
Developer
    |
    v
 pom.xml
    |
    v
  Maven
    |
    +----> Local Repository (.m2)
    |
    +----> Maven Central / Nexus
    |
    v
 Build Artifact (jar/war)
```

Key identity:
```xml
<groupId>com.company</groupId>
<artifactId>order-service</artifactId>
<version>1.0.0</version>
```

---

## SECTION 3 — INSTALLATION

### Windows
```bash
JAVA_HOME=C:\Java\jdk21
MAVEN_HOME=C:\apache-maven
PATH=%JAVA_HOME%\bin;%MAVEN_HOME%\bin
```

### Ubuntu
```bash
sudo apt update
sudo apt install maven
mvn -version
```

Common issues:
- JAVA_HOME missing
- PATH misconfigured
- Multiple JDK versions

---

## SECTION 4 — PROJECT STRUCTURE

```text
project
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       ├── java
│       └── resources
└── target
```

Convention over Configuration:
- Standard structure
- Less XML
- Faster onboarding

---

## SECTION 5 — REPOSITORIES

### Local Repository
```text
Windows:
C:\Users\<user>\.m2

Linux:
~/.m2
```

### Central Repository
https://repo.maven.apache.org/maven2

### Remote Repository
- Nexus
- Artifactory (JFrog)

Dependency Resolution:

```text
Project
  |
Local Repo?
  |
 yes -> use
 no
  |
Remote Repo?
  |
 yes -> download
 no
  |
 Build Failure
```

---

## SECTION 6 — POM.XML GUIDE

```xml
<project>
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.company</groupId>
  <artifactId>order-service</artifactId>
  <version>1.0.0</version>

  <packaging>jar</packaging>

  <dependencies>
  </dependencies>

  <build>
  </build>
</project>
```

Important tags:
- groupId
- artifactId
- version
- packaging
- dependencies
- dependencyManagement
- build
- plugins
- parent
- modules
- profiles

---

## SECTION 7 — DEPENDENCIES

### Dependency
External library required by project.

### Transitive Dependency
A -> B -> C

Project automatically gets C.

Commands:

```bash
mvn dependency:tree
mvn dependency:list
mvn dependency:analyze
```

### Nearest Definition Rule
Closest dependency version wins.

---

## SECTION 8 — DEPENDENCY SCOPES

| Scope | Compile | Runtime | Test |
|---------|---------|---------|------|
| compile | Yes | Yes | Yes |
| provided | Yes | No | Yes |
| runtime | No | Yes | Yes |
| test | No | No | Yes |
| system | Manual | Manual | Manual |
| import | BOM | BOM | BOM |

Examples:
- Servlet API -> provided
- MySQL -> runtime
- JUnit -> test

---

## SECTION 9 — MAVEN LIFECYCLES

### Clean
```text
pre-clean
clean
post-clean
```

### Default

Important phases:

```text
validate
compile
test
package
verify
install
deploy
```

Faculty note mapping:

```text
validate
compile -> compiler plugin
test -> surefire plugin
package -> jar/war plugin
install -> local repo
deploy -> remote repo
```

Commands:

```bash
mvn clean package
mvn clean install
mvn deploy
```

---

## SECTION 10 — COMMANDS

```bash
mvn compile
mvn test
mvn package
mvn verify
mvn install
mvn deploy
mvn clean
mvn dependency:tree
mvn help:effective-pom
mvn help:effective-settings
```

---

## SECTION 11 — PLUGINS

### Plugin Concepts

Plugin -> Goal -> Execution

Mojo = Maven Plain Old Java Object

Important Plugins:

- maven-compiler-plugin
- maven-surefire-plugin
- maven-failsafe-plugin
- maven-jar-plugin
- maven-war-plugin
- maven-shade-plugin
- maven-assembly-plugin
- exec-maven-plugin
- spring-boot-maven-plugin

### Exec Plugin

Faculty Notes:

```xml
<plugin>
 <groupId>org.codehaus.mojo</groupId>
 <artifactId>exec-maven-plugin</artifactId>
</plugin>
```

Run:

```bash
mvn exec:java
```

---

## SECTION 12 — TESTING

### Surefire
Runs unit tests.

```bash
mvn test
```

### Failsafe
Runs integration tests.

```bash
mvn verify
```

### JaCoCo
Coverage reports.

---

## SECTION 13 — WEB APPLICATIONS

WAR Packaging:

```xml
<packaging>war</packaging>
```

Servlet dependency:

```xml
<scope>provided</scope>
```

Tomcat:

```bash
mvn tomcat7:run
```

---

## SECTION 14 — SPRING BOOT

Parent:

```xml
<parent>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-parent</artifactId>
</parent>
```

Starter example:

```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Build:

```bash
mvn spring-boot:run
mvn clean package
```

Fat Jar:
Contains application + dependencies.

Faculty shade-plugin example is one way to create executable jars.

---

## SECTION 15 — INHERITANCE

Parent POM:

```xml
<parent>
 ...
</parent>
```

Benefits:
- Common versions
- Shared plugins
- Shared repositories

---

## SECTION 16 — MULTI MODULE PROJECTS

Faculty Concept:

```text
Parent
 |
 +-- Service Module
 |
 +-- Web Module
```

Modern Microservice Example:

```text
parent
├── common
├── user-service
├── order-service
├── payment-service
```

---

## SECTION 17 — PROFILES

```xml
<profiles>
</profiles>
```

Examples:
- dev
- test
- prod

Run:

```bash
mvn clean package -Pprod
```

---

## SECTION 18 — PROPERTIES

Faculty Example:

```xml
<properties>
 <spring.version>5.3.26</spring.version>
</properties>
```

Usage:

```xml
<version>${spring.version}</version>
```

---

## SECTION 19 — EXCLUSIONS

Faculty Example:

```xml
<exclusions>
 <exclusion>
 </exclusion>
</exclusions>
```

Use case:
Remove conflicting transitive dependency.

---

## SECTION 20 — TOP INTERVIEW QUESTIONS (Sample)

1. What is Maven?
2. Difference between ANT and Maven?
3. What is POM?
4. What is dependency scope?
5. Difference between install and deploy?
6. What is transitive dependency?
7. What is dependency mediation?
8. What is dependencyManagement?
9. What is a parent POM?
10. What is a BOM?

Senior Questions:
- Explain version conflict resolution.
- Explain multi-module build order.
- Explain reproducible builds.

---

## SECTION 21 — DEBUGGING GUIDE

### Dependency Not Found

```bash
mvn -U clean install
```

Check:
- groupId
- artifactId
- version
- repository

### Corrupted Local Repository

```bash
rm -rf ~/.m2/repository
```

### Dependency Tree

```bash
mvn dependency:tree
```

### Effective POM

```bash
mvn help:effective-pom
```

---

## SECTION 22 — MAVEN FOR SPRING BOOT

Daily Commands:

```bash
mvn spring-boot:run
mvn clean package
mvn dependency:tree
mvn test
```

Skip tests:

```bash
mvn clean package -DskipTests
```

---

## SECTION 23 — BEST PRACTICES

- Use dependencyManagement
- Lock versions
- Avoid SNAPSHOT in production
- Keep builds reproducible
- Use Nexus/Artifactory
- Scan dependencies for CVEs
- Separate dev/test/prod profiles

---

## SECTION 24 — QUICK CHEATSHEET

### Lifecycle

```text
validate
compile
test
package
verify
install
deploy
```

### Scopes

```text
compile
provided
runtime
test
system
import
```

### Repositories

```text
Local -> .m2
Central -> Maven Central
Remote -> Nexus/Artifactory
```

---

## SECTION 25 — FACULTY NOTES RECONCILIATION

| Faculty Topic | Covered |
|--------------|----------|
| Build Process | Section 1 |
| Batch Files | Section 1 |
| ANT | Section 1 |
| Maven Features | Section 2 |
| Archetypes | Section 2 |
| Repositories | Section 5 |
| Local Repository .m2 | Section 5 |
| settings.xml | Section 5 |
| Lifecycle | Section 9 |
| Plugins | Section 11 |
| Exec Plugin | Section 11 |
| Surefire | Section 12 |
| WAR Deployment | Section 13 |
| Tomcat Plugin | Section 13 |
| Spring Properties | Section 18 |
| Exclusions | Section 19 |
| Inheritance | Section 15 |
| Multi Module | Section 16 |
| Install Local Jar | Sections 5,7 |
| Shade Plugin / Fat Jar | Section 14 |
| Dependency Scopes | Section 8 |

---

# 30 Minute Revision Plan

- 5 min: Repositories
- 5 min: POM tags
- 5 min: Scopes
- 5 min: Lifecycles
- 5 min: Plugins
- 5 min: Interview Questions

# 1 Day Maven Mastery Plan

Hour 1: Build fundamentals
Hour 2: POM
Hour 3: Dependencies
Hour 4: Scopes
Hour 5: Lifecycles
Hour 6: Plugins
Hour 7: Spring Boot Maven
Hour 8: Debugging

# 7 Day Deep Dive

Day1 Foundations
Day2 Repositories + POM
Day3 Dependencies
Day4 Lifecycles + Plugins
Day5 Testing + WAR
Day6 Spring Boot + Multi Module
Day7 Interview Revision + Debugging

