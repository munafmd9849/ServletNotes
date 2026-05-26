# Backend Engineering Handbook
### Java · Servlet · Tomcat · Spring MVC · JDBC · JVM · Threading

> Built from classroom notes (27 Apr – 25 May 2026) + expanded production-grade content.  
> Covers: Beginner → Senior Engineer level.

---

## TABLE OF CONTENTS

1. [Web Applications & HTTP Fundamentals](#1-web-applications--http-fundamentals)
2. [Servlet Architecture — The Complete Picture](#2-servlet-architecture--the-complete-picture)
3. [Servlet Interface — The Contract](#3-servlet-interface--the-contract)
4. [GenericServlet — Adapter Pattern](#4-genericservlet--adapter-pattern)
5. [HttpServlet — Template Method Pattern](#5-httpservlet--template-method-pattern)
6. [Servlet Lifecycle — Deep Dive](#6-servlet-lifecycle--deep-dive)
7. [Tomcat Internals](#7-tomcat-internals)
8. [Threading — Process vs Thread vs JVM](#8-threading--process-vs-thread-vs-jvm)
9. [Single Instance Multi-Threaded Model](#9-single-instance-multi-threaded-model)
10. [HttpServletRequest — The Request Object](#10-httpservletrequest--the-request-object)
11. [Request Parameters vs Attributes](#11-request-parameters-vs-attributes)
12. [HttpServletResponse — The Response Object](#12-httpservletresponse--the-response-object)
13. [ServletConfig vs ServletContext](#13-servletconfig-vs-servletcontext)
14. [Listeners](#14-listeners)
15. [JDBC & Database Connectivity](#15-jdbc--database-connectivity)
16. [Connection Pooling · DataSource · JNDI](#16-connection-pooling--datasource--jndi)
17. [RequestDispatcher — Forward & Include](#17-requestdispatcher--forward--include)
18. [Forward vs Redirect — Master Comparison](#18-forward-vs-redirect--master-comparison)
19. [Resource Streaming](#19-resource-streaming)
20. [Filters & FilterChain](#20-filters--filterchain)
21. [Dispatcher Types](#21-dispatcher-types)
22. [Spring MVC Flow](#22-spring-mvc-flow)
23. [HTTP Status Codes](#23-http-status-codes)
24. [Interview Master Reference](#24-interview-master-reference)

---

## 1. Web Applications & HTTP Fundamentals

### Level 1 — Beginner

**What is a Web Application?**

A web application is software that runs on a server and delivers content to users through a browser over a network. Unlike a desktop app installed on your machine, a web app lives on a remote computer (server) and you access it via a URL.

There are two types of content a web application serves:

| Type | What it is | Examples |
|------|-----------|---------|
| **Static** | Files that never change per-request | HTML, CSS, JS, images |
| **Dynamic** | Generated fresh per-request using server-side logic | User dashboards, order history, search results |

**What is HTTP?**

HTTP (HyperText Transfer Protocol) is the language browsers and servers use to talk to each other. Every time you click a link or submit a form, your browser sends an HTTP **request**. The server reads it, does work, and sends back an HTTP **response**.

```
Browser                              Server
  |                                    |
  |-------- HTTP Request ------------->|
  |   GET /products HTTP/1.1           |
  |   Host: amazon.com                 |
  |   Accept: text/html                |
  |                                    |
  |<------- HTTP Response -------------|
  |   HTTP/1.1 200 OK                  |
  |   Content-Type: text/html          |
  |                                    |
  |   <html>...product list...</html>  |
```

**Raw HTTP Request Structure:**

```
REQUEST LINE:   GET /login?user=sachin HTTP/1.1
                ^^^  ^^^^^^^^^^^^^^^^^  ^^^^^^^
                |    |                  |
                |    Resource path      HTTP version
                HTTP Method (GET/POST/PUT/DELETE...)

REQUEST HEADERS (key:value pairs sent by browser):
  Host: localhost:9999
  User-Agent: Mozilla/5.0 (Windows NT 10.0...)
  Accept: text/html,application/xhtml+xml
  Accept-Language: en-US,en;q=0.9
  Connection: keep-alive
  Cookie: sessionId=abc123

REQUEST BODY (only for POST/PUT):
  username=sachin&password=secret
```

**Raw HTTP Response Structure:**

```
STATUS LINE:    HTTP/1.1 200 OK
                         ^^^  ^^
                         |    |
                         |    Reason phrase
                         Status code

RESPONSE HEADERS:
  Content-Type: text/html; charset=UTF-8
  Content-Length: 1234
  Set-Cookie: JSESSIONID=xyz789
  Cache-Control: no-cache

RESPONSE BODY:
  <html><body><h1>Welcome Sachin!</h1></body></html>
```

### Level 2 — Internal Working

**How a browser builds an HTTP request:**

1. DNS lookup: `amazon.com` → `54.239.28.85` (IP address)
2. TCP handshake: Browser opens a socket to `IP:443`
3. TLS handshake (for HTTPS): Certificates exchanged, encrypted tunnel established
4. HTTP request bytes sent over the socket
5. Server reads bytes, parses them into a request object
6. Server writes response bytes back through the same socket
7. Browser receives bytes, parses HTML, renders the page

**HTTP/1.1 vs HTTP/2 vs HTTP/3:**

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Multiplexing | No (one request/connection) | Yes (multiple streams) | Yes (QUIC protocol) |
| Header compression | No | Yes (HPACK) | Yes (QPACK) |
| Transport | TCP | TCP | UDP (QUIC) |
| Used by | Legacy apps | Most modern apps | Google, Cloudflare |

**HTTP Methods and Their Semantics:**

| Method | Safe? | Idempotent? | Has Body? | Use |
|--------|-------|-------------|-----------|-----|
| GET | Yes | Yes | No | Read data |
| POST | No | No | Yes | Create resource |
| PUT | No | Yes | Yes | Replace resource |
| PATCH | No | No | Yes | Partial update |
| DELETE | No | Yes | No | Delete resource |
| HEAD | Yes | Yes | No | Get headers only |
| OPTIONS | Yes | Yes | No | CORS preflight |

> **Safe** = server state unchanged. **Idempotent** = calling N times = calling once.

### Level 3 — Production Perspective

- **Amazon**: Every product page is a dynamic response. When you load `amazon.com/dp/B08N5LNQCX`, Amazon's servers read your session cookie, load personalized recommendations, check inventory in real time, and assemble a response — all within 200ms. They use HTTP/2 with server push to pre-send CSS/JS before the browser even asks.

- **Zomato**: When you search "pizza near me", a GET request goes out with your coordinates as query parameters. The server returns JSON (not HTML) which the React frontend renders. REST API over HTTP/2.

- **Paytm**: Payment flows use POST requests with HTTPS. The request body contains transaction details. TLS ensures the data is encrypted in transit. Response includes a transaction ID and status.

### Analogies

1. **Airport**: HTTP is like the flight booking system. You (browser) send a request form (HTTP request). The airline system (server) reads it, checks availability, and sends back a boarding pass (HTTP response).
2. **Restaurant**: HTTP is like placing an order. You tell the waiter (browser) what you want. The waiter takes the order slip (HTTP request) to the kitchen (server). The kitchen prepares and sends back the dish (HTTP response).
3. **Bank**: Going to a bank teller — you fill a form (request), the teller processes it (server), and hands you cash or a receipt (response).

### Revision — HTTP

- **One-line**: HTTP is a stateless request-response protocol used by browsers and servers to exchange data.
- **Key facts**: Stateless (no memory between requests), text-based, port 80 (HTTP) / 443 (HTTPS).
- **Common mistake**: Confusing URL (full address) with URI (path portion). `http://localhost:9999/app/login` is the URL; `/app/login` is the URI.

---

## 2. Servlet Architecture — The Complete Picture

### Level 1 — Beginner

A **Servlet** is a Java program that runs inside a web server (like Tomcat) to generate dynamic responses. Think of it as the "brain" behind a web page — it receives your request, does business logic, and writes back HTML or JSON or a file.

**Why was it needed?**

Before Servlets (pre-1997), web servers could only serve static files. CGI (Common Gateway Interface) scripts existed but spawned a new OS process per request — extremely expensive at scale. Sun Microsystems designed the Servlet specification: a Java class loaded **once** and reused for thousands of requests.

```
Before Servlet (CGI):
  1000 users → 1000 OS processes → System crashes

With Servlet:
  1000 users → 1 Servlet object + 1000 threads → Efficient
```

### Level 2 — Full Architecture Walkthrough

```
┌─────────────────────────────────────────────────────────────────┐
│  USER'S BROWSER                                                  │
│  Chrome/Firefox/Safari                                           │
│  Builds HTTP Request bytes                                       │
└───────────────────────────────┬─────────────────────────────────┘
                                │  TCP/IP Socket (port 9999)
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  TOMCAT SERVER                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  COYOTE  (HTTP Connector)                               │    │
│  │  • Listens on port 9999                                 │    │
│  │  • Accepts TCP connections                              │    │
│  │  • Parses raw bytes → HttpRequest object                │    │
│  │  • Thread Pool: picks an idle thread for each request   │    │
│  └───────────────────────────┬─────────────────────────────┘    │
│                              │                                   │
│  ┌───────────────────────────▼─────────────────────────────┐    │
│  │  CATALINA  (Servlet Container / Engine)                 │    │
│  │  • Routes request to correct Web Application            │    │
│  │  • Manages Servlet lifecycle                            │    │
│  │  • Creates HttpServletRequest wrapper                   │    │
│  │  • Creates HttpServletResponse wrapper                  │    │
│  │                                                         │    │
│  │  ┌───────────────────────────────────────────────────┐  │    │
│  │  │  ENGINE (Catalina)                                │  │    │
│  │  │  └── HOST (localhost)                             │  │    │
│  │  │       └── CONTEXT (/FirstServletApp)              │  │    │
│  │  │            └── WRAPPER (MyServlet → /test)        │  │    │
│  │  └───────────────────────────────────────────────────┘  │    │
│  └───────────────────────────┬─────────────────────────────┘    │
│                              │                                   │
│  ┌───────────────────────────▼─────────────────────────────┐    │
│  │  YOUR SERVLET (MyServlet.class)                         │    │
│  │  • doGet() / doPost() executes                          │    │
│  │  • Business logic runs                                  │    │
│  │  • Reads from DB (JDBC)                                 │    │
│  │  • Writes response HTML/JSON                            │    │
│  └───────────────────────────┬─────────────────────────────┘    │
│                              │                                   │
│  ┌───────────────────────────▼─────────────────────────────┐    │
│  │  DATABASE (MySQL / Oracle / PostgreSQL)                 │    │
│  │  • SQL queries execute                                  │    │
│  │  • ResultSet returned                                   │    │
│  └─────────────────────────────────────────────────────────┘    │
└───────────────────────────────┬─────────────────────────────────┘
                                │  HTTP Response bytes
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  USER'S BROWSER — renders the response                          │
└─────────────────────────────────────────────────────────────────┘
```

**WAR File Structure (what you deploy into Tomcat):**

```
MyApp.war  (ZIP file)
  │
  ├── index.html          ← public static resources
  ├── css/
  │    └── style.css
  ├── js/
  │    └── app.js
  ├── images/
  │
  └── WEB-INF/            ← PRIVATE — browser CANNOT access this directly
       ├── web.xml         ← deployment descriptor (legacy config)
       ├── classes/
       │    └── in/pw/ioi/controller/MyServlet.class
       └── lib/
            └── mysql-connector.jar
```

> **Security rule**: Anything inside `WEB-INF/` is protected. The browser can never directly request `/WEB-INF/web.xml`. Only the server can read it. This is enforced by Catalina.

**URL Pattern Matching (how Tomcat routes requests):**

Tomcat matches URL patterns in priority order:

1. **Exact match**: `/login` → `LoginServlet`
2. **Directory match**: `/user/*` → `UserServlet` (handles `/user/profile`, `/user/settings`)
3. **Extension match**: `*.do` → `ActionServlet` (rarely used today)

```
Request: /FirstServletApp/user/login/*

Catalina breaks it down:
  Context Path  = /FirstServletApp
  Servlet Path  = /user/login
  Path Info     = /*
  Query String  = null
```

### Level 3 — Production Perspective

- **Netflix**: Each microservice is a web application. The `streaming-service` war contains servlets/controllers that handle `/play`, `/stop`, `/seek`. Requests come from Netflix clients (apps) via HTTP/2.
- **Uber**: The `driver-location-service` processes thousands of GPS coordinate updates per second. Each update is a POST request hitting a servlet/controller that validates, stores, and publishes the location.
- **Swiggy**: The `order-service` war handles `/place-order`, `/track-order` endpoints. The Servlet/Spring MVC layer receives the HTTP, maps to business logic, and returns JSON to the React Native mobile app.

### Analogies

1. **Airport security gate**: Tomcat is the entire terminal. Catalina is the gate agent who checks your boarding pass (URL) and routes you to the correct gate (Servlet). The Servlet is the aircraft itself where your actual "journey" (business logic) happens.
2. **Hospital**: Tomcat is the hospital. Catalina is the reception that reads your appointment slip (URL) and sends you to the right department (Servlet — Cardiology/Orthopedics). The doctor (business logic) examines you and gives a prescription (response).
3. **Shopping Mall**: Tomcat is the mall. Catalina is the directory/map. Each shop (Servlet) serves specific customers routed to it.

---

## 3. Servlet Interface — The Contract

### Level 1 — Beginner

`Servlet` is a Java **interface** defined by the Jakarta EE (formerly Java EE) specification. It's the fundamental contract that every servlet must implement.

```java
// javax.servlet.Servlet (the full interface)
public interface Servlet {

    // Called ONCE when servlet is first initialized
    void init(ServletConfig config) throws ServletException;

    // Called for EVERY request — the main worker method
    void service(ServletRequest req, ServletResponse res)
        throws ServletException, IOException;

    // Called ONCE when servlet is being destroyed
    void destroy();

    // Returns the config object given during init
    ServletConfig getServletConfig();

    // Returns human-readable info about the servlet
    String getServletInfo();
}
```

**Where does this JAR come from?**

```
Tomcat installation folder:
  tomcat/
    lib/
      servlet-api.jar   ← This JAR contains the Servlet interface

You need it on the classpath to COMPILE your servlet:
  javac -cp "C:\Tomcat 9.0\lib\servlet-api.jar" MyServlet.java

At RUNTIME, Tomcat provides it — don't package it in your WAR.
```

**Modern Jakarta EE note**: In Tomcat 10+, the package changed from `javax.servlet.*` to `jakarta.servlet.*`. Spring Boot 3.x uses `jakarta.*`. Your notes use `javax.*` which is Tomcat 9 / Spring Boot 2.x.

### Level 2 — Internal Working

**How does the Container know which class to load?**

When you annotate with `@WebServlet(urlPatterns="/test")`, Tomcat's deployment scanner reads this annotation at startup (or at first request) and builds an internal mapping:

```
Internal Map (simplified):
  "/test"    → "in.pw.ioi.controller.MyServlet"
  "/login"   → "in.pw.ioi.controller.LoginServlet"
  "/cart"    → "in.pw.ioi.controller.CartServlet"
```

When request `/test` arrives, Catalina does:

```java
// Catalina's simplified internal logic
String urlPattern = extractPattern(request); // "/test"
String className  = urlPatternMap.get(urlPattern); // "in.pw.ioi.controller.MyServlet"

// Reflective instantiation (zero-param constructor required!)
Class<?> clazz   = Class.forName(className);
Servlet servlet  = (Servlet) clazz.newInstance();

// Initialize
servlet.init(servletConfig);

// Serve
servlet.service(request, response);
```

This is why your Servlet class **must have a public no-arg constructor**. If you define a constructor with parameters and no no-arg constructor, `clazz.newInstance()` throws `InstantiationException`.

**Memory impact of the Servlet object:**

The Servlet object lives in the **JVM Heap** inside Tomcat's JVM process. It is loaded by a `WebAppClassLoader` (a child classloader per web application). This isolation means two web apps can have different versions of the same class without conflict.

```
JVM Heap
  └── Tomcat ClassLoader hierarchy
       ├── Bootstrap ClassLoader (JDK classes)
       ├── System ClassLoader (Tomcat core)
       ├── Common ClassLoader (shared jars in tomcat/lib)
       └── WebApp ClassLoader (per-app — WEB-INF/classes, WEB-INF/lib)
            └── MyServlet object (singleton in heap)
```

### Revision — Servlet Interface

- **One-line**: The `Servlet` interface is the specification contract defining the 5 methods every servlet must implement.
- **Key interview point**: Direct `Servlet` implementation is impractical (5 methods). Use `GenericServlet` or `HttpServlet`.
- **WODA**: Write Once, Deploy Anywhere — because the Servlet spec is vendor-neutral. Works on Tomcat, JBoss, GlassFish, WebLogic.

---

## 4. GenericServlet — Adapter Pattern

### Level 1 — Beginner

**Problem with implementing `Servlet` directly:**

Every class that implements `Servlet` must provide bodies for all 5 methods — even if you only need `service()`. This creates unnecessary boilerplate:

```java
// Painful: implementing Servlet directly
public class MyServlet implements Servlet {
    public void init(ServletConfig c) throws ServletException { /* nothing */ }
    public void destroy() { /* nothing */ }
    public ServletConfig getServletConfig() { return null; /* bug! */ }
    public String getServletInfo() { return ""; }

    public void service(ServletRequest req, ServletResponse res)
        throws ServletException, IOException {
        // actual logic here
    }
}
```

**Solution — GenericServlet (Adapter Pattern):**

`GenericServlet` is an **abstract class** that implements the `Servlet` interface and provides default (empty) implementations for all methods except `service()`. You only override what you need.

```
Servlet (I)           ← 5 abstract methods
    |
GenericServlet (AC)   ← implements Servlet, provides defaults
    |                   Leaves service() abstract
    |
YourServlet           ← only override service() (mandatory)
                        override init() / destroy() if needed
```

### Level 2 — Adapter Pattern Explained

The **Adapter Design Pattern** says: _"Provide a default implementation of an interface so subclasses don't have to implement everything."_

**GenericServlet internal source (simplified):**

```java
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {

    private transient ServletConfig config;

    // ─── LIFECYCLE METHODS ───────────────────────────────────────────

    @Override
    public void init(ServletConfig config) throws ServletException {
        this.config = config;   // STORES the config reference
        this.init();            // Calls the no-arg init() for subclasses
    }

    // Subclasses override THIS (not the one above) to avoid losing config
    public void init() throws ServletException {
        // default: do nothing
    }

    @Override
    public abstract void service(ServletRequest req, ServletResponse res)
        throws ServletException, IOException;
    // ↑ Still abstract — subclass MUST provide this

    @Override
    public void destroy() {
        // default: do nothing
    }

    // ─── CONFIG DELEGATION ───────────────────────────────────────────

    @Override
    public ServletConfig getServletConfig() {
        return config;  // Returns the stored config
    }

    public String getInitParameter(String name) {
        return config.getInitParameter(name);  // Delegates to config
    }

    public ServletContext getServletContext() {
        return config.getServletContext();     // Delegates to config
    }

    @Override
    public String getServletInfo() {
        return "";
    }
}
```

**The three cases your notes describe:**

```java
// CASE 1: Override init(ServletConfig config) — DANGEROUS
// --------------------------------------------------------
// init(SC) runs, but init() does NOT run.
// this.config is NOT set (you bypassed GenericServlet's init(SC))
// getServletConfig() returns null → NullPointerException later!

@Override
public void init(ServletConfig config) {
    System.out.println("init with config");
    // If you don't call super.init(config), this.config stays null
}

// CASE 2: Override init() — CORRECT way ✓
// ----------------------------------------
// GenericServlet.init(SC) runs first → sets this.config
// then calls your init()
// getServletConfig() returns config correctly

@Override
public void init() throws ServletException {
    System.out.println("My startup logic here");
    // getServletConfig() works fine here
}

// CASE 3: Override nothing — FINE
// --------------------------------
// GenericServlet.init(SC) runs → sets config → calls empty init()
// Everything works as expected
```

**Why this matters in an interview:**

> Q: "If I override `init(ServletConfig config)` in my servlet without calling `super.init(config)`, what happens?"
> A: `getServletConfig()` will return `null` because `GenericServlet.init(SC)` is never called. Any call to `getServletContext()` or `getInitParameter()` will throw a `NullPointerException`.

**GenericServlet Limitations:**

`GenericServlet` is protocol-agnostic. It doesn't know about HTTP methods (GET, POST, PUT). If you need to handle GET differently from POST, you'd have to manually check `((HttpServletRequest)req).getMethod()` — messy. Solution: `HttpServlet`.

---

## 5. HttpServlet — Template Method Pattern

### Level 1 — Beginner

`HttpServlet` is an abstract class that extends `GenericServlet` and adds HTTP-specific support. It knows about GET, POST, PUT, DELETE, etc. You override `doGet()`, `doPost()` instead of `service()`.

```
Servlet (I)
    |
GenericServlet (AC)  ← Adapter Pattern
    |
HttpServlet (AC)     ← Template Method Pattern
    |
YourServlet          ← override doGet(), doPost(), etc.
```

### Level 2 — Template Method Pattern & Internal Dispatch

The **Template Method Pattern** says: _"Define the skeleton of an algorithm in a base class, but let subclasses fill in specific steps."_

HttpServlet's `service()` IS the template. It decides WHICH `doXXX()` to call based on the HTTP method. You fill in the `doXXX()` steps.

**HttpServlet internal source (simplified):**

```java
public abstract class HttpServlet extends GenericServlet {

    // STEP 1: GenericServlet calls this
    @Override
    public void service(ServletRequest req, ServletResponse res)
        throws ServletException, IOException {

        // Type-cast to HTTP-specific objects
        HttpServletRequest  hReq  = (HttpServletRequest)  req;
        HttpServletResponse hResp = (HttpServletResponse) res;

        // Delegate to HTTP-aware service
        service(hReq, hResp);
    }

    // STEP 2: Dispatches based on HTTP method
    protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {

        String method = req.getMethod();

        if ("GET".equals(method))         doGet(req, resp);
        else if ("POST".equals(method))   doPost(req, resp);
        else if ("PUT".equals(method))    doPut(req, resp);
        else if ("DELETE".equals(method)) doDelete(req, resp);
        else if ("HEAD".equals(method))   doHead(req, resp);
        else if ("OPTIONS".equals(method))doOptions(req, resp);
        else if ("TRACE".equals(method))  doTrace(req, resp);
        else {
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED,
                "Method " + method + " not implemented");
        }
    }

    // STEP 3: Default implementations return 405/400
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        String protocol = req.getProtocol();
        if (protocol.endsWith("1.1")) {
            resp.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED);  // 405
        } else {
            resp.sendError(HttpServletResponse.SC_BAD_REQUEST);         // 400
        }
    }

    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        // same default: return error
    }
    // ... doPut, doDelete, etc. all default to error
}
```

**Your servlet only overrides what it needs:**

```java
@WebServlet(urlPatterns="/test", loadOnStartup=5)
public class MyServlet extends HttpServlet {

    static {
        System.out.println("Loading...");      // class loading
    }

    public MyServlet() {
        System.out.println("Instantiation");   // zero-param constructor
    }

    @Override
    public void init() throws ServletException {
        System.out.println("Initialization");  // setup logic
    }

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        useMe(req, resp);
    }

    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        useMe(req, resp);
    }

    // Common logic shared by GET and POST
    private void useMe(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter out = resp.getWriter();
        out.println("<html><body><h1>Hello!</h1></body></html>");
        out.close();
    }

    @Override
    public void destroy() {
        System.out.println("Destroy");
    }
}
```

**The 6 cases your notes describe (service method override behavior):**

| Case | What your Servlet has | GET request result | POST request result |
|------|-----------------------|-------------------|---------------------|
| 1 | `public service(SR,SR)` | public service() runs | public service() runs |
| 2 | Both `public service(SR,SR)` AND `protected service(HSR,HSResp)` | public service() wins for both | public service() wins for both |
| 3 | `protected service(HSR,HSResp)` + `doGet()` | protected service → doGet | protected service → doPost (default: error) |
| 4 | Only `doPost()` | HttpServlet.doGet → 405 error | doPost() runs |
| 5 | Only `doGet()` | doGet() runs | HttpServlet.doPost → 405 error |
| 6 | Both `doGet()` and `doPost()` | doGet() runs | doPost() runs |

> **Production best practice**: Always override `doGet()` and `doPost()` separately (Case 6). Call a common private method if logic is shared. This makes it explicit which HTTP methods are supported.

**Real concrete classes at runtime (visible from your notes' code):**

```java
System.out.println("Request class:  " + request.getClass().getName());
System.out.println("Response class: " + response.getClass().getName());

// Output:
// Request class:  org.apache.catalina.connector.RequestFacade
// Response class: org.apache.catalina.connector.ResponseFacade
```

Tomcat wraps its internal `Request` and `Response` objects in **Facade** objects before passing to your servlet. This is the **Facade Design Pattern** — it exposes only the public API methods and hides Tomcat's internal implementation details, preventing servlets from down-casting to access internals.

### Level 3 — Production Perspective

- **Spring Boot**: `DispatcherServlet` extends `HttpServlet`. Every Spring MVC application is ultimately backed by one HttpServlet. Spring adds its own routing layer on top.
- **REST APIs**: In modern microservices, `doGet()` maps to `@GetMapping`, `doPost()` to `@PostMapping`, etc. The mapping logic is the same Template Method pattern, just expressed through annotations.

### Revision — HttpServlet

- **One-line**: `HttpServlet` uses the Template Method pattern to dispatch HTTP requests to `doGet()`, `doPost()`, etc.
- **Common mistake**: Overriding `public service(SR, SR)` accidentally — this bypasses all HTTP method dispatching.
- **Spring Boot equivalent**: `@GetMapping`, `@PostMapping` on `@Controller` methods.

---

## 6. Servlet Lifecycle — Deep Dive

### Level 1 — Beginner

A Servlet has 5 lifecycle stages. Think of it like an employee's career:

1. **Loading** — Company decides to hire (classloader loads the `.class` file)
2. **Instantiation** — Employee joins (object created with `new`)
3. **Initialization** — Orientation/training (`init()` called once)
4. **Request Processing** — Daily work (`service()` called per request)
5. **Destroy** — Employee retires (`destroy()` called once)

### Level 2 — Who Calls What, When, and Why

```
SERVER STARTUP (if loadOnStartup >= 0)
or
FIRST REQUEST (if loadOnStartup < 0, default)
        │
        ▼
┌───────────────────────────────────────────────────────┐
│  STAGE 1: LOADING                                     │
│  • Who: JVM ClassLoader (WebAppClassLoader)           │
│  • What: Reads MyServlet.class bytes from             │
│          WEB-INF/classes/                             │
│  • When: Once per deployment                          │
│  • Memory: Class object created in Method Area (JVM)  │
│  • Your code: static {} block runs here               │
│                                                       │
│  static {                                             │
│      System.out.println("Loading...");                │
│  }                                                    │
└───────────────────────────┬───────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────┐
│  STAGE 2: INSTANTIATION                               │
│  • Who: Catalina container                            │
│  • What: Class.forName("MyServlet").newInstance()     │
│  • When: Once (creates the singleton servlet object)  │
│  • Memory: Object created in JVM Heap                 │
│  • Your code: zero-param constructor runs here        │
│                                                       │
│  public MyServlet() {                                 │
│      System.out.println("Instantiation");             │
│  }                                                    │
└───────────────────────────┬───────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────┐
│  STAGE 3: INITIALIZATION                              │
│  • Who: Catalina container                            │
│  • What: servlet.init(servletConfig) called           │
│  • When: Once after instantiation                     │
│  • Thread: Container's deployment thread (NOT         │
│    a request thread)                                  │
│  • Your code: init() or init(SC) — setup DB pools,    │
│    load config, create expensive objects              │
│                                                       │
│  public void init() {                                 │
│      // Ideal place to get DataSource from JNDI       │
│      ds = (DataSource) ctx.lookup("java:/comp/env/"); │
│  }                                                    │
└───────────────────────────┬───────────────────────────┘
                            │ (for every request)
                            ▼
┌───────────────────────────────────────────────────────┐
│  STAGE 4: REQUEST PROCESSING                          │
│  • Who: Thread from Tomcat's thread pool              │
│  • What: service() → doGet()/doPost()                 │
│  • When: For EVERY incoming HTTP request              │
│  • Thread: A different thread per request             │
│  • CRITICAL: The servlet OBJECT is shared across      │
│    all threads. Instance variables = RACE CONDITION   │
│                                                       │
│  public void doGet(HSR req, HSResp resp) {            │
│      // req and resp are thread-local (new per req)   │
│      // this (servlet object) is shared — be careful! │
│  }                                                    │
└───────────────────────────┬───────────────────────────┘
                            │ (on undeploy/server stop)
                            ▼
┌───────────────────────────────────────────────────────┐
│  STAGE 5: DESTROY                                     │
│  • Who: Catalina container                            │
│  • What: servlet.destroy() called                     │
│  • When: On undeploy or server shutdown               │
│  • Your code: close DB connections, flush buffers,    │
│    release resources                                  │
│                                                       │
│  public void destroy() {                              │
│      if (dataSource != null) dataSource.close();      │
│  }                                                    │
└───────────────────────────────────────────────────────┘
```

**loadOnStartup explained:**

```
@WebServlet(urlPatterns="/test", loadOnStartup=5)
                                 ^^^^^^^^^^^^^^
                                 Controls when Stages 1-3 happen

loadOnStartup < 0 (default: -1):
  → Lazy loading: Stages 1-3 happen on FIRST REQUEST
  → Faster server startup, slower first request

loadOnStartup >= 0:
  → Eager loading: Stages 1-3 happen at SERVER STARTUP
  → Slower server startup, faster first request
  → RECOMMENDED for servlets that acquire DB connections

loadOnStartup ordering:
  Servlet A: loadOnStartup=2  → initializes BEFORE Servlet B
  Servlet B: loadOnStartup=3  → initializes AFTER Servlet A
  Servlet C: loadOnStartup=2  → order with A is UNDEFINED (container decides)
```

**Full lifecycle execution log (from your notes):**

```
Server starts:
  ***ThirdServletLoading...****       ← static block
  ***ThirdServlet Instantiation...*** ← constructor
  ***ThirdServlet Initialziation...** ← init()

Request arrives:
  ***Request Processing****           ← doGet()/doPost()

Request arrives again (same object!):
  ***Request Processing****           ← same object, new thread

Server stops/undeploy:
  <<<DEINSTANTIATION>>>               ← destroy()
```

### Level 3 — Production Perspective

- **Zomato**: The `OrderServlet` has `loadOnStartup=1`. During `init()`, it acquires 20 database connections from HikariCP pool and caches configuration like tax rates. First request is fast.
- **Paytm**: The `PaymentGatewayServlet` in `init()` loads RSA private keys for signing transactions. Keys are expensive to load (disk I/O + parsing). Doing it once in `init()` instead of per-request saves 50ms per payment.
- **Netflix**: Content catalog servlet pre-warms a Redis connection in `init()`. When `destroy()` is called (during a rolling deploy), it waits for in-flight requests to complete before releasing the Redis connection.

### Interview Questions — Lifecycle

**Beginner:**
1. Q: What are the 5 lifecycle methods of a Servlet?  
   A: Loading (static block) → Instantiation (constructor) → Initialization (init) → Request Processing (service/doXXX) → Destroy (destroy)

2. Q: Is a Servlet instantiated once or per request?  
   A: Once. Only one object is created. All requests share it.

3. Q: What is `loadOnStartup`?  
   A: Annotation attribute that makes the container initialize the servlet at server startup rather than on first request.

**Intermediate:**
4. Q: What happens if you throw an exception in `init()`?  
   A: `UnavailableException` is thrown. The container may mark the servlet as unavailable and return 503. The servlet is NOT put into service.

5. Q: Can `destroy()` be called while `service()` is still running in another thread?  
   A: Yes. The container should wait for active requests to finish before calling `destroy()`, but this is container-dependent. Production code should count active requests and wait.

**Advanced:**
6. Q: Why must the Servlet constructor be public with no parameters?  
   A: Catalina uses `Class.forName().newInstance()` (reflection) which requires a public no-arg constructor. Adding a parameterized-only constructor causes `InstantiationException`.

7. Q: What is `UnavailableException` and when is it used?  
   A: Thrown from `init()` or `service()` to signal the servlet is unavailable. If thrown with `isPermanent()=true`, the container removes the servlet permanently.

---

## 7. Tomcat Internals

### Level 1 — Beginner

Tomcat is an **open-source application server** (technically a "servlet container") developed by the Apache Software Foundation. It implements the Servlet and JSP specifications. When you run a Java web app, Tomcat is the program that:
- Listens for HTTP requests on a port
- Routes them to the right Servlet
- Returns the response

### Level 2 — Tomcat Component Architecture

```
TOMCAT (Server)
│
└── SERVICE (Catalina)
     │
     ├── CONNECTOR(s)
     │    ├── Coyote HTTP/1.1 Connector  (port 8080/9999)
     │    ├── Coyote HTTPS Connector     (port 8443)
     │    └── AJP Connector              (port 8009) ← for Apache httpd
     │
     └── ENGINE (Catalina)
          │
          └── HOST (localhost)   ← virtual host
               │
               ├── CONTEXT (/FirstServletApp)  ← one WAR = one Context
               │    ├── Servlet: MyServlet    → /test
               │    ├── Servlet: LoginServlet → /login
               │    └── Filter: LoggingFilter → /*
               │
               └── CONTEXT (/SecondApp)
                    └── Servlet: ...
```

**Key Components Explained:**

**Coyote** — The HTTP Connector
- Listens on a TCP port (default 8080)
- Accepts incoming socket connections
- Reads raw bytes and parses them into an HTTP request structure
- Manages the thread pool — assigns one thread per connection (HTTP/1.1) or multiplexes (HTTP/2)
- Hands the request to Catalina

**Catalina** — The Servlet Container
- The "brain" of Tomcat
- Implements the Servlet specification
- Manages the lifecycle of all servlets across all deployed apps
- Routes requests to the correct Context → Wrapper → Servlet
- Creates `HttpServletRequest` and `HttpServletResponse` wrapper objects
- `RequestFacade` wraps the internal `Request` for safe exposure to servlets

**Jasper** — The JSP Engine
- Compiles `.jsp` files into Servlet source code (`.java`)
- Compiles that source to `.class` files
- On first request to a JSP, Jasper generates `MyPage_jsp.java` and compiles it
- Subsequent requests reuse the compiled class (unless the JSP changes)

**Thread Pool Configuration (server.xml):**

```xml
<Connector port="8080"
           maxThreads="200"         ← max concurrent requests
           minSpareThreads="10"     ← threads kept alive when idle
           acceptCount="100"        ← queue size when all threads busy
           connectionTimeout="20000"
           protocol="HTTP/1.1"/>
```

- `maxThreads=200` means at most 200 concurrent requests can be processed
- Request 201 goes into the accept queue (size `acceptCount=100`)
- If the queue is full, new connections are refused (connection reset)

**Hot vs Cold Deployment:**

| | Cold Deployment | Hot Deployment |
|--|----------------|----------------|
| Server state | Stopped | Running |
| Method | Copy files to webapps/, start server | Deploy WAR through Manager App |
| URL | `http://localhost:9999/` | `http://localhost:9999/manager` |
| Config | `tomcat/conf/tomcat-users.xml` — add manager role | Same |
| Use case | Initial setup, dev environment | Production rolling update |

```xml
<!-- tomcat-users.xml -->
<user username="root" password="root123" roles="manager-gui,admin-gui"/>
```

Creating a WAR file:
```bash
# From inside FirstServletApp directory:
jar -cvf MyApp.war WEB-INF index.html css js

# c = create, v = verbose, f = filename
```

**Classloader Architecture (critical for interviews):**

```
Bootstrap ClassLoader
  └── loads: rt.jar (String, Object, etc.)

Extension ClassLoader
  └── loads: jre/lib/ext/*.jar

Application/System ClassLoader
  └── loads: Tomcat's own classes

Common ClassLoader
  └── loads: tomcat/lib/*.jar (shared between all web apps)
        └── includes: servlet-api.jar, jsp-api.jar

WebApp ClassLoader (one per deployed app)
  └── loads: WEB-INF/classes/ + WEB-INF/lib/*.jar
  └── ISOLATED: App1 and App2 cannot see each other's classes
```

> This isolation is why you can deploy two apps with different versions of the same library without conflict.

---

## 8. Threading — Process vs Thread vs JVM

### Level 1 — Beginner

**Process:**
- A running instance of a program
- Has its own memory space (RAM)
- Managed by the Operating System
- Expensive to create

**Thread:**
- A lightweight unit of execution within a process
- Shares memory with other threads in the same process
- Managed by the JVM (via the OS thread scheduler)
- Cheap to create relative to a process

```
Opening Notepad = 1 process (its own memory)
Opening 5 Notepads = 5 processes (5 separate memories)

One JVM process with 5 threads = 5 units of work sharing ONE memory
```

### Level 2 — JVM Thread Architecture

```
OS
 └── JVM Process  (one Tomcat = one JVM process)
      │
      ├── Heap Memory  (SHARED by all threads)
      │    ├── Servlet objects (singletons)
      │    ├── DataSource pool objects
      │    └── String pool, class metadata
      │
      ├── Method Area  (SHARED — class definitions, static vars)
      │
      └── Per-Thread:
           ├── Thread Stack (private to each thread)
           │    ├── Frame for doGet()
           │    │    ├── local var: req (reference)
           │    │    ├── local var: resp (reference)
           │    │    └── local var: out (PrintWriter)
           │    └── Frame for useMe()
           │
           └── PC Register (which bytecode instruction is executing)
```

**Key rules:**

| Location | Who can see it | Thread safe? |
|----------|---------------|-------------|
| Heap (instance vars of Servlet) | All threads | NO — race condition |
| Thread Stack (local vars in doGet) | Only that thread | YES |
| Heap (request/response objects) | Only passed thread | YES (new object per request) |

**Why is instance variable sharing dangerous in Servlets?**

```java
// DANGEROUS — instance variable shared by all threads!
public class DangerousServlet extends HttpServlet {
    private int counter = 0;  // Shared! On the heap!

    public void doGet(HttpServletRequest req, HttpServletResponse resp) {
        counter++;  // Thread 1 reads 5, Thread 2 reads 5, both write 6
                    // Should be 7 (two increments) but is 6 → RACE CONDITION
        resp.getWriter().println("Requests: " + counter);
    }
}

// SAFE — local variable lives on thread's stack
public class SafeServlet extends HttpServlet {
    public void doGet(HttpServletRequest req, HttpServletResponse resp) {
        int localCounter = 0;  // Stack — private to this thread
        localCounter++;
        resp.getWriter().println("Count: " + localCounter);
    }
}
```

**Process vs Thread comparison (from your notes):**

| Aspect | Process | Thread |
|--------|---------|--------|
| Weight | Heavyweight | Lightweight |
| Memory | Own address space | Shared heap |
| Creation cost | High (fork syscall) | Low |
| Managed by | OS (PCB - Process Control Block) | JVM (ThreadScheduler) |
| Communication | IPC (pipes, sockets, shared memory) | Direct shared memory access |
| Failure impact | Only that process crashes | Can crash entire process |

**Benchmark (from your notes' code):**

```java
// Running 100 processes vs 100 threads
long start = System.currentTimeMillis();
for (int i = 0; i < 100; i++) {
    new ProcessBuilder("notepad").start();  // ~50,000ms = 50 seconds
    // vs
    new Thread(() -> {}).start();           // ~10ms total
}
long end = System.currentTimeMillis();
System.out.println("Time: " + (end - start) + "ms");
// Threads are ~5000x faster to create than processes
```

**Thread Safety Mechanisms:**

```java
// 1. synchronized — one thread at a time
synchronized (this) {
    counter++;
}

// 2. AtomicInteger — lock-free thread-safe increment
private AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet(); // Thread safe, no lock needed

// 3. volatile — ensures visibility across threads
private volatile boolean running = true;
// Thread 1 writes: running = false;
// Thread 2 reads the UPDATED value (not cached copy)

// 4. ThreadLocal — one copy per thread
private ThreadLocal<SimpleDateFormat> sdf =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
// Each thread gets its own SimpleDateFormat instance
```

**Deadlock — how it happens:**

```java
// Thread 1 holds lock A, waits for lock B
// Thread 2 holds lock B, waits for lock A → DEADLOCK

synchronized (lockA) {
    synchronized (lockB) { ... }  // Thread 1
}

synchronized (lockB) {
    synchronized (lockA) { ... }  // Thread 2 — deadlock!
}

// Solution: always acquire locks in the SAME order
```

---

## 9. Single Instance Multi-Threaded Model

### Level 1 — Beginner

This is the **most important architectural fact** about Servlets:

> **There is ONE Servlet object. Many threads use it simultaneously.**

```
1000 users hit /login simultaneously
         │
         ▼
  Tomcat Thread Pool picks 200 threads (maxThreads=200)
         │
         ▼
  All 200 threads call doGet() on THE SAME LoginServlet object
         │
         ▼
  Each thread gets its OWN request and response objects
  (Created fresh per request — thread safe)
         │
         ▼
  The Servlet OBJECT itself is shared — instance vars are dangerous
```

### Level 2 — Demonstrating the Model

```java
// This code (from your notes) proves Single Instance Multi-Thread:
public void doGet(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException {

    PrintWriter out = resp.getWriter();
    String threadName = Thread.currentThread().getName();

    out.println("<h1>Thread: " + threadName + "</h1>");
    out.println("<h3>Request object:  " + req          + "</h3>"); // DIFFERENT per request
    out.println("<h3>Response object: " + resp         + "</h3>"); // DIFFERENT per request
    out.println("<h2>Servlet object:  " + this.hashCode() + "</h2>"); // SAME always

    Thread.sleep(30000); // Hold the thread for 30 seconds
    // During this sleep, another request comes in with a DIFFERENT thread
    // but the SAME servlet hashcode (same object)
}
```

Output for 3 concurrent requests:
```
Thread: http-nio-9999-exec-1 | Request: RequestFacade@1a2b | Servlet: 12345678
Thread: http-nio-9999-exec-2 | Request: RequestFacade@3c4d | Servlet: 12345678
Thread: http-nio-9999-exec-3 | Request: RequestFacade@5e6f | Servlet: 12345678
                                        ^^^^^^^^^^^^^^^^^               ^^^^^^^^
                                        Different addresses              SAME address
```

**Memory layout with 1000 concurrent users:**

```
JVM Heap
  └── LoginServlet@12345678  (1 object, ~1KB)
       └── private DataSource ds  (shared — use thread-safe pool)
       └── NO instance variables that change per-request

Thread Stack (per thread, ~512KB each)
  Thread-1: doGet frame { req=RequestFacade@1a2b, resp=... }
  Thread-2: doGet frame { req=RequestFacade@3c4d, resp=... }
  ...
  Thread-200: doGet frame { req=RequestFacade@9z0x, resp=... }

Total memory:
  Servlet objects: ~10KB for 10 servlets
  Thread stacks:   ~100MB for 200 threads (200 × 512KB)
  Request objects: ~200KB for 200 requests
```

### Level 3 — Production

- **Uber (driver tracking)**: One `LocationServlet` instance handles GPS updates from 2 million active drivers. The servlet is stateless — it reads coordinates from the request body, calls a service layer, and returns 200 OK. No instance variables store state. The state lives in Redis and Cassandra.
- **Paytm**: One `PaymentServlet` processes thousands of transactions per second. The DB connection pool (HikariCP, 50 connections) is a `DataSource` stored in `init()`. Each request borrows a connection, uses it, and returns it. Thread-safe by design.

---

## 10. HttpServletRequest — The Request Object

### Level 1 — Beginner

`HttpServletRequest` is the object that represents the incoming HTTP request. Think of it as the **envelope** containing everything the browser sent: the URL, the form data, cookies, session info, browser type, etc.

```
Inheritance hierarchy:
  ServletRequest (I)
       └── HttpServletRequest (I)
                └── RequestFacade (C) ← Tomcat's concrete implementation
                    (passed to your servlet's doGet/doPost)
```

### Level 2 — Request Anatomy with Real API

**A. Request Line Information:**

```java
// Request: GET http://localhost:9999/FirstServletApp/user/login/salary?userName=sachin&userAge=53

String url         = request.getRequestURL().toString();
// = "http://localhost:9999/FirstServletApp/user/login/salary"

String uri         = request.getRequestURI();
// = "/FirstServletApp/user/login/salary"

String contextPath = request.getContextPath();
// = "/FirstServletApp"

String servletPath = request.getServletPath();
// = "/user/login"

String pathInfo    = request.getPathInfo();
// = "/salary"

String queryString = request.getQueryString();
// = "userName=sachin&userAge=53"

String method      = request.getMethod();
// = "GET"

String protocol    = request.getProtocol();
// = "HTTP/1.1"
```

**B. Reading Request Headers:**

```java
// Get a specific header
String userAgent   = request.getHeader("User-Agent");
String host        = request.getHeader("Host");
String contentType = request.getContentType();
int    contentLen  = request.getContentLength();

// Iterate all headers
Enumeration<String> headerNames = request.getHeaderNames();
while (headerNames.hasMoreElements()) {
    String name  = headerNames.nextElement();
    String value = request.getHeader(name);
    System.out.println(name + " --> " + value);
}

// Actual output from your notes:
// host---> localhost:9999
// user-agent---> Mozilla/5.0 (Windows NT 10.0; Win64; x64)...
// accept---> text/html,application/xhtml+xml...
// accept-language---> en-US,en;q=0.9
// cookie---> Webstorm-1e567889=70072c84-97cb...
```

**C. Reading Request Parameters (form data):**

```java
// HTML form: <form method="POST" action="./user">
//   <input name="userName" value="sachin">
//   <input name="courses" value="Java">
//   <input name="courses" value="Spring">
// </form>

// Single value
String name = request.getParameter("userName");     // "sachin"

// Multiple values (checkboxes, multi-select)
String[] courses = request.getParameterValues("courses");
// ["Java", "Spring"]

// Iterating all parameters
Enumeration<String> paramNames = request.getParameterNames();
while (paramNames.hasMoreElements()) {
    String key = paramNames.nextElement();
    System.out.println(key + " = " + request.getParameter(key));
}
```

**D. Type conversion (from your notes):**

```java
// getParameter() ALWAYS returns String
String numberStr = request.getParameter("mobileNumber");

// You must convert manually
long mobileNumber = Long.parseLong(numberStr);  // String → long

// In Spring MVC, this conversion is automatic:
// @RequestParam("mobileNumber") Long mobileNumber
```

**E. Working with Attributes (for inter-servlet communication):**

```java
// Servlet 1: set attribute
request.setAttribute("user", userObject);  // key=String, value=ANY type
request.setAttribute("orderId", 12345L);

// Servlet 2 (after forward): read attribute
User user = (User) request.getAttribute("user");  // must cast
Long orderId = (Long) request.getAttribute("orderId");

// Remove attribute
request.removeAttribute("user");
```

---

## 11. Request Parameters vs Attributes

### Full Comparison Table

| Aspect | Parameters | Attributes |
|--------|-----------|-----------|
| **Source** | Browser (form/URL) | Server-side code via `setAttribute()` |
| **Key type** | `String` | `String` |
| **Value type** | `String` only | `Object` (any Java type) |
| **Mode** | Read-only | Read, Write, Delete |
| **Scope** | Current request only | Current request + forwarded requests |
| **API to set** | Cannot set (browser sets) | `request.setAttribute(k, v)` |
| **API to get** | `request.getParameter(k)` | `request.getAttribute(k)` |
| **API to remove** | Cannot remove | `request.removeAttribute(k)` |
| **Survive redirect?** | Re-sent by browser if in URL | NO — new request object |
| **Survive forward?** | YES — same request object | YES — same request object |
| **Use case** | Read user input | Pass data between servlets/JSPs |
| **Spring equivalent** | `@RequestParam` | `Model.addAttribute()` |
| **Type safety** | No (always String) | No (Object, requires cast) |

**Visual flow:**

```
Browser → POST /register (name=Sachin, age=25)
              │
              ▼
        RegisterServlet.doPost()
          param: name = "sachin"    ← came from browser
          param: age  = "25"        ← came from browser (String!)

          // Validate, create User object
          User user = new User("sachin", 25);
          request.setAttribute("registeredUser", user);  // put in attribute
              │
              │ forward to /confirmation
              ▼
        ConfirmationServlet.doGet()
          // parameters still there:
          param: name = "sachin"
          // attribute also there:
          User u = (User) request.getAttribute("registeredUser");
          // u.getName() = "sachin", u.getAge() = 25 (Integer, not String!)
```

---

## 12. HttpServletResponse — The Response Object

### Level 1 — Beginner

`HttpServletResponse` is the object you use to write the response back to the browser. Think of it as the **blank paper** on which you write the answer.

```
Inheritance:
  ServletResponse (I)
       └── HttpServletResponse (I)
                └── ResponseFacade (C) ← Tomcat's implementation
```

### Level 2 — API Deep Dive

**A. Writing text response (HTML, JSON, text):**

```java
// Step 1: Set content type BEFORE getting writer
response.setContentType("text/html");
response.setCharacterEncoding("UTF-8");

// Step 2: Get the writer
PrintWriter out = response.getWriter();

// Step 3: Write
out.println("<html><body>");
out.println("<h1>Hello!</h1>");
out.println("</body></html>");

// Step 4: Close (flushes buffer to browser)
out.close();
```

**B. Writing binary response (images, PDFs, videos):**

```java
// From your notes — streaming a video file:
response.setContentType("video/mp4");

ServletOutputStream os = response.getOutputStream();

// Get real path on server filesystem
String path = getServletContext().getRealPath("video.mp4");
File file = new File(path);
FileInputStream fis = new FileInputStream(file);
byte[] buffer = new byte[(int) file.length()];
fis.read(buffer);

os.write(buffer);
os.flush();
// Don't use PrintWriter AND ServletOutputStream — pick one
```

**MIME Types:**

| Content | MIME Type |
|---------|-----------|
| HTML page | `text/html` |
| JSON API response | `application/json` |
| PDF | `application/pdf` |
| Word document | `application/msword` |
| Excel | `application/vnd.ms-excel` |
| JPEG image | `image/jpeg` |
| PNG image | `image/png` |
| MP4 video | `video/mp4` |
| Binary download | `application/octet-stream` |

**C. Setting response headers and status:**

```java
// Set status code
response.setStatus(HttpServletResponse.SC_OK);            // 200
response.setStatus(HttpServletResponse.SC_NOT_FOUND);     // 404
response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR); // 500

// Set a header
response.setHeader("Cache-Control", "no-cache, no-store");
response.setHeader("X-Custom-Header", "my-value");

// Add a header (allows multiple values)
response.addHeader("Set-Cookie", "theme=dark; Path=/");

// Send error with status + message
response.sendError(500, "Something broke on our end.");
```

**D. Redirect (from your notes):**

```java
// Method 1: Manual (low-level)
response.setStatus(302);
response.setHeader("Location", "http://localhost:9999/FirstServletApp/oracle");

// Method 2: Convenience method (recommended)
response.sendRedirect("http://localhost:9999/FirstServletApp/oracle");
response.sendRedirect("/FirstServletApp/oracle"); // relative URL
response.sendRedirect("https://www.google.com"); // absolute external URL
```

**PrintWriter vs ServletOutputStream:**

| | `PrintWriter` (getWriter) | `ServletOutputStream` (getOutputStream) |
|--|--------------------------|----------------------------------------|
| Use for | Text (HTML, JSON, XML) | Binary (images, PDFs, videos, bytes) |
| Method | `out.println(String)` | `os.write(byte[])` |
| Encoding | Respects charset setting | Raw bytes |
| Can use both? | **NO** — throws `IllegalStateException` |

---

## 13. ServletConfig vs ServletContext

### Level 1 — Beginner

Think of a school:
- **ServletConfig** = a teacher's personal desk drawer. Only that teacher can access it. Contains that teacher's specific materials.
- **ServletContext** = the school's common room. All teachers (servlets) can access it. Contains school-wide resources.

### Level 2 — Full Comparison

| Aspect | ServletConfig | ServletContext |
|--------|--------------|----------------|
| **Scope** | One per Servlet | One per Web Application |
| **Created by** | Container during Servlet `init()` | Container at application startup |
| **Destroyed when** | Servlet is destroyed | Application is undeployed |
| **Lives in** | JVM Heap (tied to Servlet object) | JVM Heap (tied to Context object) |
| **Access** | `getServletConfig()` inside servlet | `getServletContext()` inside servlet |
| **Data type** | Init parameters (String→String) | Attributes (String→Object) + Init params |
| **Mutability** | Read-only | Attributes: read/write/delete |
| **Thread safe?** | Yes (read-only after init) | Attributes: NO (use synchronization) |
| **Spring equivalent** | `@Value()` / `application.properties` | `@Autowired` ApplicationContext |
| **Annotation** | `@WebInitParam(name="k", value="v")` | Use `@WebListener` + `ServletContextListener` |

**ServletConfig in code:**

```java
@WebServlet(
    urlPatterns = "/dbtest",
    initParams = {
        @WebInitParam(name = "url",      value = "jdbc:mysql://localhost/mydb"),
        @WebInitParam(name = "username", value = "root"),
        @WebInitParam(name = "password", value = "secret")
    }
)
public class DBServlet extends HttpServlet {

    private String dbUrl;

    @Override
    public void init() throws ServletException {
        // getServletConfig() works because GenericServlet stored the config
        dbUrl = getServletConfig().getInitParameter("url");
        // or shorthand (GenericServlet delegates):
        dbUrl = getInitParameter("url");
    }
}
```

**ServletContext in code:**

```java
// Setting data (usually in a Listener at startup):
ServletContext ctx = getServletContext();
ctx.setAttribute("appVersion", "2.5.1");
ctx.setAttribute("dataSource", ds);          // share DS across all servlets

// Reading data (in any servlet):
String version = (String) ctx.getAttribute("appVersion");
DataSource ds  = (DataSource) ctx.getAttribute("dataSource");

// Reading context init params (from web.xml):
String env = ctx.getInitParameter("environment");  // "production"

// Getting real file path:
String realPath = ctx.getRealPath("/WEB-INF/config.properties");
```

**web.xml context init params:**

```xml
<web-app>
    <context-param>
        <param-name>environment</param-name>
        <param-value>production</param-value>
    </context-param>
</web-app>
```

**Lifecycle diagram:**

```
Server starts
     │
     ▼
ServletContext created ←─────────────── lives until app undeploy
     │
     ▼
Listeners notified (contextInitialized)
     │
     ▼
Filters initialized (eager)
     │
     ▼
Servlets with loadOnStartup >= 0 initialized
     │  (each gets its own ServletConfig)
     ▼
Application serving requests
     │
     ▼ (undeploy/shutdown)
     │
Servlets destroyed (destroy() called)
     │
Filters destroyed
     │
Listeners notified (contextDestroyed)
     │
ServletContext destroyed
```

---

## 14. Listeners

### Level 1 — Beginner

A **Listener** is a Java class that "listens" for events in the servlet lifecycle and reacts to them. Like a fire alarm — it sits quietly until the event (fire) occurs, then executes its logic.

**Three main event types:**

| Event | Listener Interface | When triggered |
|-------|-------------------|----------------|
| Application started/stopped | `ServletContextListener` | App deploy / undeploy |
| Request arrived/finished | `ServletRequestListener` | Every HTTP request |
| Session created/destroyed | `HttpSessionListener` | User session start/end |

### Level 2 — Implementation

**ServletContextListener (most important for production):**

```java
@WebListener
public class AppStartupListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        // App is starting — perfect time to:
        // 1. Set up connection pools
        // 2. Load application-wide config
        // 3. Start background threads

        ServletContext ctx = event.getServletContext();

        // Load config
        String env = ctx.getInitParameter("environment");

        // Set global attribute accessible by all servlets
        ctx.setAttribute("startTime", System.currentTimeMillis());
        ctx.setAttribute("appConfig", loadConfig(env));

        System.out.println("Application started in: " + env);
    }

    @Override
    public void contextDestroyed(ServletContextEvent event) {
        // App is stopping — clean up:
        // 1. Close connection pools
        // 2. Shutdown thread pools
        // 3. Flush logs

        ExecutorService pool = (ExecutorService)
            event.getServletContext().getAttribute("threadPool");
        if (pool != null) pool.shutdown();

        System.out.println("Application stopped.");
    }
}
```

**Why Listeners matter for Connection Pooling (from your notes):**

```
Problem:
  If we create a DB connection in each Servlet's init(),
  and we have 20 servlets, we get 20 separate connection pools.
  Wasteful. Uncoordinated.

Solution:
  Create ONE pool in the Listener.
  Store it in ServletContext.
  All Servlets look it up and share it.

Flow:
  Server starts
    → ServletContextListener.contextInitialized()
        → Create HikariCP pool (50 connections)
        → context.setAttribute("dataSource", pool)

  Request arrives → UserServlet.doGet()
    → DataSource ds = (DataSource) getServletContext().getAttribute("dataSource")
    → Connection conn = ds.getConnection()
    → // do work
    → conn.close() // returns to pool

  Server stops
    → ServletContextListener.contextDestroyed()
        → pool.close() // gracefully drain all connections
```

**HttpSessionListener:**

```java
@WebListener
public class SessionTracker implements HttpSessionListener {

    private static final AtomicInteger activeSessions = new AtomicInteger();

    @Override
    public void sessionCreated(HttpSessionEvent event) {
        int count = activeSessions.incrementAndGet();
        System.out.println("Session created. Active sessions: " + count);
        // Production use: track active users, enforce session limits
    }

    @Override
    public void sessionDestroyed(HttpSessionEvent event) {
        int count = activeSessions.decrementAndGet();
        System.out.println("Session destroyed. Active sessions: " + count);
    }
}
```

**Spring Boot equivalent:**

```java
@Component
public class AppStartupRunner implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        // Called when Spring context is refreshed (app started)
    }
}

// Or using @EventListener:
@Component
public class StartupListener {
    @EventListener(ApplicationReadyEvent.class)
    public void onReady() {
        System.out.println("Spring Boot app is ready!");
    }
}
```

---

## 15. JDBC & Database Connectivity

### Level 1 — Beginner

**JDBC (Java Database Connectivity)** is the Java API for connecting to and interacting with relational databases (MySQL, PostgreSQL, Oracle). It's the bridge between your Java code and the database.

**The 6 steps of JDBC:**

```java
// Step 1: Register the driver (not needed with JDBC 4.0+, auto-registered)
Class.forName("com.mysql.cj.jdbc.Driver");

// Step 2: Get a Connection
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/mydb", "root", "password");

// Step 3: Create a Statement
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM product WHERE price > ?");
pstmt.setDouble(1, 100.0);  // Set the ? parameter

// Step 4: Execute the query
ResultSet rs = pstmt.executeQuery();

// Step 5: Process results
while (rs.next()) {
    String name  = rs.getString("pname");
    double price = rs.getDouble("price");
    System.out.println(name + " -- " + price);
}

// Step 6: Close resources (in reverse order)
rs.close();
pstmt.close();
conn.close();
```

### Level 2 — Statement vs PreparedStatement vs CallableStatement

| | Statement | PreparedStatement | CallableStatement |
|--|-----------|------------------|------------------|
| Use for | Simple, no-param SQL | Parameterized SQL | Stored procedures |
| SQL injection safe? | NO | YES | YES |
| Pre-compiled? | No | YES (compiled once, run many) | YES |
| Performance | Slow (compiled each time) | Fast (compiled once) | Fast |
| Example | `executeQuery("SELECT * FROM t")` | `pstmt.setString(1, value)` | `{CALL proc(?)}` |

**SQL Injection — why PreparedStatement is mandatory:**

```java
// DANGEROUS — SQL Injection possible
String username = request.getParameter("username");
String sql = "SELECT * FROM users WHERE name = '" + username + "'";
// If username = "' OR '1'='1" → bypasses authentication!

// SAFE — PreparedStatement
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM users WHERE name = ?");
pstmt.setString(1, username);
// The ? is escaped — injection impossible
```

**Full JDBC in a Servlet (from your notes):**

```java
@WebServlet(urlPatterns = "/test")
public class TestDBServlet extends HttpServlet {
    private DataSource ds;

    @Override
    public void init() {
        try {
            InitialContext context = new InitialContext();
            ds = (DataSource) context.lookup("java:/comp/env/jdbc/EmployeeDB");
        } catch (NamingException e) {
            throw new RuntimeException("Cannot get DataSource", e);
        }
    }

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {

        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            conn  = ds.getConnection();
            pstmt = conn.prepareStatement("SELECT * FROM product");
            rs    = pstmt.executeQuery();

            resp.setContentType("text/html");
            PrintWriter out = resp.getWriter();
            out.println("<table border='1'>");
            while (rs.next()) {
                out.println("<tr><td>" + rs.getString("pname") +
                            "</td><td>" + rs.getString("price") + "</td></tr>");
            }
            out.println("</table>");

        } catch (SQLException e) {
            throw new ServletException("DB error", e);
        } finally {
            // Always close in finally — in reverse order
            try { if (rs    != null) rs.close();    } catch (SQLException ignore) {}
            try { if (pstmt != null) pstmt.close(); } catch (SQLException ignore) {}
            try { if (conn  != null) conn.close();  } catch (SQLException ignore) {}
            // conn.close() returns connection to pool — does NOT physically close it
        }
    }
}
```

**Transactions:**

```java
conn.setAutoCommit(false);  // Start transaction
try {
    pstmt1.executeUpdate();  // Debit account A
    pstmt2.executeUpdate();  // Credit account B
    conn.commit();           // Both succeed → commit
} catch (SQLException e) {
    conn.rollback();         // Either fails → undo both
}
```

---

## 16. Connection Pooling · DataSource · JNDI

### Level 1 — Beginner

**Why not create a new Connection per request?**

```
Problem (naive approach):
  Each request: DriverManager.getConnection() → opens socket to DB → authentication → ~200ms overhead
  1000 requests/second = 1000 × 200ms connection setups = disaster

Solution (Connection Pool):
  At startup: create 50 connections, keep them open
  Each request: borrow one → use → return
  Time to "get connection": ~0.1ms (just take from queue)
  Max concurrent DB operations: 50 (configurable)
```

### Level 2 — JNDI + DataSource Architecture

**JNDI (Java Naming and Directory Interface)** is a lookup service — like a phone book for Java objects. You register objects by name, and any code in the same JVM can look them up by name.

```
Context.xml configuration (from your notes):

<Resource
    name="jdbc/EmployeeDB"           ← JNDI name
    auth="Container"                  ← Tomcat manages it
    type="javax.sql.DataSource"       ← Type of object
    driverClassName="com.mysql.cj.jdbc.Driver"
    url="jdbc:mysql://localhost:3306/octbatch"
    username="root"
    password="root123"
    maxTotal="8"                      ← Max 8 connections in pool
    maxIdle="4"/>                     ← Max 4 idle connections
```

**How JNDI lookup works:**

```
Tomcat starts
    │
    ▼
Reads context.xml → creates DataSource (pool of 8 connections)
    │
    ▼
Registers it in JNDI under name "java:/comp/env/jdbc/EmployeeDB"

    JNDI Tree:
    java:
     └── comp:
          └── env:
               └── jdbc:
                    └── EmployeeDB: [DataSource object with 8 pooled connections]
```

**Servlet lookup (from your notes):**

```java
@Override
public void init() throws ServletException {
    try {
        InitialContext context = new InitialContext();
        // "java:/comp/env/" prefix is required for Tomcat's JNDI
        ds = (DataSource) context.lookup("java:/comp/env/jdbc/EmployeeDB");
    } catch (NamingException e) {
        e.printStackTrace();
    }
}

@Override
public void doGet(HttpServletRequest req, HttpServletResponse resp) {
    Connection conn = ds.getConnection();  // borrow from pool
    // ... do DB work ...
    conn.close();  // RETURN to pool (not actually closed!)
}
```

**HikariCP — the modern production standard:**

Tomcat's built-in pool (DBCP2) is fine for learning. Production systems use **HikariCP** — the fastest Java connection pool.

```xml
<!-- Spring Boot application.properties -->
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=root123
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
```

```java
// Spring Boot auto-configures HikariCP — just inject:
@Autowired
private DataSource dataSource;

// Or use JPA/Spring Data — no raw JDBC needed:
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByPriceGreaterThan(double price);
}
```

**Pool sizing rule of thumb:**

```
Optimal pool size ≠ Maximum pool size

Formula: pool size = (core count × 2) + effective_spindle_count
For a 4-core server with SSD: 4 × 2 + 1 = 9 connections

More connections ≠ more throughput
Too many connections = context switching overhead, DB server overload

Netflix uses ~10-20 connections per service instance
Each microservice pod has its own pool (not shared across pods)
```

### Level 3 — Production

- **Zomato**: 200 microservice pods, each with HikariCP pool of 10 connections = 2000 connections to MySQL. But MySQL max_connections is usually 500-1000. Solution: PgBouncer / ProxySQL as a connection proxy between services and DB.
- **Paytm**: Payment DB must NEVER run out of connections. Pool min=max=50 (no idle connections close). Dedicated read replicas for SELECT queries. Primary only for INSERT/UPDATE/DELETE.

---

## 17. RequestDispatcher — Forward & Include

### Level 1 — Beginner

`RequestDispatcher` lets one servlet hand off the request to another servlet (or JSP or HTML) — server-side, invisibly to the browser.

Two operations:
- **forward()**: "You handle this completely" — first servlet stops, second servlet takes over
- **include()**: "Add your content to mine" — first servlet resumes after second finishes

### Level 2 — Forward in Detail

**Getting a RequestDispatcher:**

```java
// Method 1: From Request object (relative or absolute path)
RequestDispatcher rd = request.getRequestDispatcher("/home");
RequestDispatcher rd = request.getRequestDispatcher("../other");  // relative

// Method 2: From ServletContext (absolute path only)
RequestDispatcher rd = getServletContext().getRequestDispatcher("/home");

// Method 3: Foreign Context (cross-application dispatch, from your notes)
ServletContext foreignCtx = getServletContext().getContext("/App2");
RequestDispatcher rd = foreignCtx.getRequestDispatcher("/user");
// Requires <Context crossContext="true"> in context.xml
```

**forward() execution flow:**

```
Browser ──────────────── GET /home ──────────────────────────────→ Servlet1.doGet()
                                                                        │
                                               ┌─────────────────────── │ ───────────────────┐
                                               │  request.setAttribute("data", value)          │
                                               │  RequestDispatcher rd = req.getRequestDispatcher("/user") │
                                               │  rd.forward(request, response)                │
                                               └───────────────────────────────────────────────┘
                                                                        │
                                                                        ▼
                                                               Servlet2.doGet()
                                                               (same req, same resp)
                                                               reads getAttribute("data")
                                                               writes full response
                                                                        │
                                                                        ▼
                                                               Returns to Servlet1
                                                               (remaining code runs
                                                                but output is IGNORED
                                                                if response was committed)
                                                                        │
Browser ←─────────────────────────────────── Servlet2's response ──────┘
(URL in browser still shows /home — not /user)
```

**Important forward() rules (from your notes as Cases):**

```java
// Case 1: Data sharing via attributes
request.setAttribute("userId", 42L);
rd.forward(request, response);
// Servlet2 can read getAttribute("userId")

// Case 2: Remaining code after forward() still executes
// BUT any output written after forward() is IGNORED by browser
rd.forward(request, response);
System.out.println("This still runs"); // Prints to server log
response.getWriter().println("This is IGNORED"); // Silently ignored

// Case 3: Exception after forward() doesn't affect browser response
rd.forward(request, response);
throw new RuntimeException("oops"); // Browser already has response

// Case 4: Flush BEFORE forward = IllegalStateException
PrintWriter out = response.getWriter();
out.println("some output");
out.flush();  // Commits the response!
rd.forward(request, response);  // THROWS IllegalStateException
// Never flush before forwarding

// Case 5: Auto-added attributes during forward
// javax.servlet.forward.request_uri  → "/FirstServletApp/home"
// javax.servlet.forward.context_path → "/FirstServletApp"
// javax.servlet.forward.servlet_path → "/home"
String originalUri = (String) request.getAttribute("javax.servlet.forward.request_uri");
```

**include() execution flow:**

```
Browser ──── GET /page ──────────────────────────────────────────→ Servlet1.doGet()
                                                                    │ writes part 1
                                                                    │ rd.include(req, resp)
                                                                    │         │
                                                                    │         ▼
                                                                    │    Servlet2.doGet()
                                                                    │    writes part 2
                                                                    │         │ (returns)
                                                                    │◀────────┘
                                                                    │ writes part 3
                                                                    │
Browser ←─────────────────────── part1 + part2 + part3 ───────────┘
(URL still shows /page)
```

```java
// Servlet1
out.println("<header>Navigation</header>");
RequestDispatcher rd = getServletContext().getRequestDispatcher("/user");
rd.include(request, response);  // Servlet2's output inserted here
out.println("<footer>Copyright 2026</footer>");
// Browser receives: header + user-servlet-output + footer
```

---

## 18. Forward vs Redirect — Master Comparison

| Aspect | `rd.forward(req, resp)` | `response.sendRedirect(url)` |
|--------|------------------------|------------------------------|
| **Who handles it** | Server-side | Client-side (browser) |
| **URL bar changes?** | NO — URL stays the same | YES — URL changes to new location |
| **Number of HTTP requests** | 1 (one round trip) | 2 (first → redirect → second) |
| **Request object** | SAME request object passed | NEW request object (old data lost) |
| **Response object** | SAME response object | NEW response object |
| **Data sharing** | YES — via `request.setAttribute()` | NO — data lost unless in session/URL |
| **Speed** | Faster (one network round trip) | Slower (two round trips) |
| **Works across apps?** | YES (with crossContext=true) | YES (any URL, even external) |
| **Can forward to external URL?** | NO — server resources only | YES — any URL |
| **Status code** | 200 (whatever target returns) | 302 (then 200 for second request) |
| **Use case** | MVC pattern, JSP rendering | After POST (PRG pattern), auth redirect |
| **Browser back button** | Shows forwarded content | Shows previous page before redirect |
| **Bookmark** | User bookmarks first URL | User bookmarks final URL |
| **Spring MVC** | `return "viewName"` | `return "redirect:/url"` |

**POST-Redirect-GET (PRG) Pattern — why redirect after POST:**

```
WITHOUT PRG:
  Browser → POST /order → Server processes → 200 OK (HTML)
  User hits F5 (refresh) → Browser re-sends POST → Duplicate order!

WITH PRG (Redirect After POST):
  Browser → POST /order → Server processes → 302 Redirect to /order/success
  Browser → GET /order/success → Server returns 200 OK (HTML)
  User hits F5 → Browser re-sends GET (harmless — idempotent)
  No duplicate order!
```

```java
// Spring MVC PRG:
@PostMapping("/order")
public String placeOrder(@ModelAttribute Order order) {
    orderService.save(order);
    return "redirect:/order/success";  // sendRedirect internally
}

@GetMapping("/order/success")
public String showSuccess() {
    return "success";  // forward to success.jsp
}
```

---

## 19. Resource Streaming

### Level 1 — Beginner

When a servlet needs to send a file (PDF, image, video), it reads the file bytes and writes them to the response's `OutputStream`. The browser uses the `Content-Type` header to know how to handle the bytes.

### Level 2 — Implementation Patterns

**Streaming a video (from your notes):**

```java
@WebServlet("/video")
public class VideoServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {

        resp.setContentType("video/mp4");

        String path = getServletContext().getRealPath("video.mp4");
        File file = new File(path);

        // Set Content-Length for browser progress bar
        resp.setContentLengthLong(file.length());

        ServletOutputStream os = resp.getOutputStream();
        try (FileInputStream fis = new FileInputStream(file)) {
            byte[] buffer = new byte[8192];  // 8KB chunks — don't load whole file!
            int bytesRead;
            while ((bytesRead = fis.read(buffer)) != -1) {
                os.write(buffer, 0, bytesRead);
            }
        }
        os.flush();
    }
}
```

**Streaming a PDF with download prompt:**

```java
resp.setContentType("application/pdf");
resp.setHeader("Content-Disposition", "attachment; filename=\"report.pdf\"");
//  "attachment" = download dialog     "inline" = display in browser
```

**Memory consideration:**

```
BAD (loads entire file into RAM):
  byte[] all = new byte[(int) file.length()];  // 500MB file → 500MB RAM
  fis.read(all);
  os.write(all);

GOOD (streaming — constant memory):
  byte[] chunk = new byte[8192];  // Always 8KB RAM regardless of file size
  while ((n = fis.read(chunk)) != -1) {
      os.write(chunk, 0, n);
  }
```

**HTTP Range Requests (for video seeking):**

Modern browsers send `Range: bytes=0-1000000` headers to request specific portions of a video (for seeking). Production video servers implement range support:

```java
String rangeHeader = req.getHeader("Range");
if (rangeHeader != null) {
    // Parse "bytes=start-end"
    // Set Content-Range: bytes start-end/total header
    // Set status 206 Partial Content
    // Stream only the requested bytes
}
```

Netflix, YouTube, and Hotstar all implement HTTP range requests. This is why you can drag the timeline to the middle of a video without downloading the first half.

---

## 20. Filters & FilterChain

### Level 1 — Beginner

A **Filter** is code that runs **before and after** a Servlet, for every matching request. Like a quality inspector on an assembly line — every product passes through before shipping.

**Use cases:**
- Authentication: Is the user logged in?
- Authorization: Does the user have permission?
- JWT validation: Is the token valid and not expired?
- Logging: Log every request/response
- Rate limiting: Max 100 requests/minute per IP
- CORS: Add cross-origin headers
- Compression: GZIP the response
- Request/Response modification

### Level 2 — Filter Lifecycle & Implementation

**Filter Lifecycle:**

```
Server starts (EAGER loading — always, no loadOnStartup needed)
  │
  ▼
Filter.init(FilterConfig)   ← called once

Request arrives
  │
  ▼
Filter.doFilter(req, resp, chain)  ← called per request
  │
  ├── [pre-processing logic]
  │
  ├── chain.doFilter(req, resp)    ← passes to next filter or servlet
  │
  └── [post-processing logic]      ← runs AFTER servlet completes

Server stops
  │
  ▼
Filter.destroy()   ← called once
```

**Full filter implementation (from your notes):**

```java
@WebFilter(
    urlPatterns = "/s1",             // Apply to /s1 only
    initParams = {@WebInitParam(name="logLevel", value="DEBUG")}
)
public class LoggingFilter implements Filter {

    static { System.out.println("Filter loading..."); }

    public LoggingFilter() { System.out.println("Filter Instantiation..."); }

    @Override
    public void init(FilterConfig config) throws ServletException {
        System.out.println("Filter Initialization...");
        String logLevel = config.getInitParameter("logLevel"); // "DEBUG"
    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
        throws IOException, ServletException {

        // PRE-PROCESSING
        System.out.println("Pre-processing: " + ((HttpServletRequest)req).getRequestURI());
        long start = System.currentTimeMillis();

        // Pass to next filter or servlet
        chain.doFilter(req, resp);

        // POST-PROCESSING (runs after servlet returns)
        long duration = System.currentTimeMillis() - start;
        System.out.println("Post-processing: request took " + duration + "ms");
    }

    @Override
    public void destroy() { System.out.println("Filter DeInstantiation..."); }
}
```

**Authentication Filter (production example):**

```java
@WebFilter("/*")
public class AuthFilter implements Filter {
    private static final Set<String> PUBLIC_PATHS = Set.of(
        "/login", "/register", "/public"
    );

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
        throws IOException, ServletException {

        HttpServletRequest  hReq  = (HttpServletRequest)  req;
        HttpServletResponse hResp = (HttpServletResponse) resp;

        String path = hReq.getServletPath();

        if (PUBLIC_PATHS.contains(path)) {
            chain.doFilter(req, resp);  // skip auth for public pages
            return;
        }

        HttpSession session = hReq.getSession(false);
        if (session != null && session.getAttribute("user") != null) {
            chain.doFilter(req, resp);  // authenticated — let through
        } else {
            hResp.sendRedirect("/login");  // not authenticated — redirect
        }
    }
}
```

**FilterConfig API:**

```java
public interface FilterConfig {
    String getFilterName();                          // Name of the filter
    ServletContext getServletContext();               // Application context
    String getInitParameter(String name);            // Init param value
    Enumeration<String> getInitParameterNames();     // All init param names
}
```

**How Spring Security Uses Filters:**

Spring Security is built entirely on servlet filters. It inserts a chain of filters via `DelegatingFilterProxy`:

```
Request
  ↓
SecurityContextPersistenceFilter    ← loads security context from session
  ↓
UsernamePasswordAuthenticationFilter ← handles /login POST
  ↓
JwtAuthenticationFilter             ← validates JWT token (custom)
  ↓
ExceptionTranslationFilter          ← converts auth exceptions to 401/403
  ↓
FilterSecurityInterceptor           ← checks URL-level permissions
  ↓
DispatcherServlet                   ← Spring MVC
  ↓
Your @Controller
```

From your notes:
```
AuthenticationFilter --AuthenticationException--> maps to /error (401)
AuthorizationFilter  --AuthorizationException --> maps to /login (302)
```

**Multiple Filters and ordering (FilterChain):**

```
Request → Filter1 → Filter2 → Filter3 → Servlet
                                            │
Response ← Filter1 ← Filter2 ← Filter3 ←──┘

// Filter1.doFilter:
  pre1();
  chain.doFilter(req, resp);  // invokes Filter2
  post1();

// Filter2.doFilter:
  pre2();
  chain.doFilter(req, resp);  // invokes Filter3
  post2();

// Filter3.doFilter:
  pre3();
  chain.doFilter(req, resp);  // invokes Servlet
  post3();
```

Filter execution order when using `@WebFilter`:
- Order is **unspecified** with annotation-based config
- For guaranteed order, use `web.xml` `<filter-mapping>` order
- Spring Boot: Use `@Order(1)`, `@Order(2)` or `FilterRegistrationBean.setOrder()`

---

## 21. Dispatcher Types

### Level 1 — Beginner

A request can reach a Servlet through different paths. `DispatcherType` tells a Filter **how** the request arrived:

```java
public enum DispatcherType {
    FORWARD,   // Arrived via rd.forward()
    INCLUDE,   // Arrived via rd.include()
    REQUEST,   // Direct from browser (DEFAULT)
    ASYNC,     // Via AsyncContext.dispatch()
    ERROR      // Via container error handling
}
```

By default, filters only intercept `REQUEST` type. You must explicitly configure other types.

### Level 2 — Each Type with Flow

**REQUEST (default) — direct browser request:**

```
Browser ──────────── GET /s1 ──────────────────────────→ Filter → Servlet-1
                     DispatcherType = REQUEST
```

**FORWARD — via rd.forward():**

```
Browser → Servlet-1 ──── rd.forward(req,resp) ──────────→ Filter → Servlet-2
                         DispatcherType = FORWARD
                         (only intercepted if filter has FORWARD type)
```

**INCLUDE — via rd.include():**

```
Browser → Servlet-1 ──── rd.include(req,resp) ──────────→ Filter → Servlet-2
                         DispatcherType = INCLUDE          returns to Servlet-1
```

**ERROR — via container error forwarding:**

```
Browser → Servlet-1 throws exception
               │
               ▼ container reads web.xml
              web.xml: <error-code>500</error-code> → <location>/error</location>
               │
               ▼ container forwards internally
              Filter (if ERROR type) → ErrorServlet
              DispatcherType = ERROR
```

**Full filter covering all types (from your notes):**

```java
@WebFilter(
    urlPatterns = {"/*"},
    dispatcherTypes = {
        DispatcherType.REQUEST,
        DispatcherType.FORWARD,
        DispatcherType.INCLUDE,
        DispatcherType.ERROR
    }
)
public class AllTypesFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
        throws IOException, ServletException {

        System.out.println("Filter executing for: " + req.getDispatcherType());
        chain.doFilter(req, resp);
    }
}
```

**Error handling with web.xml (from your notes):**

```xml
<web-app>
    <!-- Handle by status code -->
    <error-page>
        <error-code>500</error-code>
        <location>/error</location>
    </error-page>

    <!-- Handle by exception type -->
    <error-page>
        <exception-type>java.lang.ArithmeticException</exception-type>
        <location>/error</location>
    </error-page>

    <error-page>
        <error-code>404</error-code>
        <location>/notfound</location>
    </error-page>
</web-app>
```

**ErrorServlet — reading error details (from your notes):**

```java
@WebServlet("/error")
public class ErrorServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {

        // Container sets these attributes before forwarding to error servlet
        Throwable cause    = (Throwable) req.getAttribute(RequestDispatcher.ERROR_EXCEPTION);
        String servletName = (String)    req.getAttribute(RequestDispatcher.ERROR_SERVLET_NAME);
        Integer statusCode = (Integer)   req.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        String requestUri  = (String)    req.getAttribute(RequestDispatcher.ERROR_REQUEST_URI);

        PrintWriter out = resp.getWriter();
        if (cause != null) {
            out.println("Exception: " + cause.getClass().getName());
            out.println("Message:   " + cause.getMessage());
        } else {
            out.println("HTTP " + statusCode + " error.");
        }
    }
}
```

---

## 22. Spring MVC Flow

### Level 1 — Beginner

Spring MVC is a web framework built on top of the Servlet API. It simplifies servlet development by letting you write simple Java methods (`@Controller`) instead of classes that extend `HttpServlet`.

```
Pure Servlet:
  extend HttpServlet → override doGet/doPost → manually parse params → manually write HTML

Spring MVC:
  @Controller + @GetMapping → method params auto-bound → return view name → auto-rendered
```

### Level 2 — DispatcherServlet: The Front Controller

Spring MVC uses the **Front Controller Pattern**: one central servlet (`DispatcherServlet`) receives ALL requests and delegates to the right controller.

```
Browser ──────── ANY request ────────────────────────────────────────────────────────────→
                               ┌──────────────────────────────────────────────────────┐
                               │  DispatcherServlet (extends HttpServlet)              │
                               │  • Loaded via loadOnStartup=1                        │
                               │  • Maps to "/" (catches everything)                  │
                               │                                                      │
                               │  1. Consults HandlerMapping                          │
                               │     "Which controller handles this URL?"              │
                               │     GET /student/save → StudentController.save()     │
                               │                                                      │
                               │  2. Calls HandlerAdapter                             │
                               │     Adapts the controller method to servlet model    │
                               │     Binds request params to method parameters        │
                               │                                                      │
                               │  3. Calls the Controller method                      │
                               │     @GetMapping("/save")                             │
                               │     public String save(Student student, Model model) │
                               │     → business logic runs                            │
                               │     → returns "viewName" (e.g. "success")           │
                               │                                                      │
                               │  4. Consults ViewResolver                            │
                               │     "success" → /WEB-INF/view/success.jsp           │
                               │     prefix=/WEB-INF/view/ suffix=.jsp               │
                               │                                                      │
                               │  5. Renders the View (JSP/Thymeleaf)                │
                               │     JSP accesses Model attributes                   │
                               │     Returns HTML to DispatcherServlet                │
                               │                                                      │
                               │  6. Writes response to browser                      │
                               └──────────────────────────────────────────────────────┘
Browser ←─────── HTML response ─────────────────────────────────────────────────────────
```

**Spring MVC Controller (from your notes' diagram):**

```java
@Controller("/student")
public class StudentController {

    @PostMapping("/save")
    public String saveStudent(@ModelAttribute Student student, Model model) {
        // Student object auto-populated from request parameters
        // student.getName(), student.getAge() are already set

        studentService.save(student);

        model.addAttribute("message", "Saved: " + student.getName());
        return "success";  // ViewResolver → /WEB-INF/view/success.jsp
    }

    @GetMapping("/list")
    public String listStudents(Model model) {
        model.addAttribute("students", studentService.findAll());
        return "studentList";  // → /WEB-INF/view/studentList.jsp
    }
}
```

**ViewResolver configuration:**

```java
// Java config (Spring MVC)
@Bean
public InternalResourceViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/view/");
    resolver.setSuffix(".jsp");
    return resolver;
}

// application.properties (Spring Boot + Thymeleaf)
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

**Redirect in Spring MVC:**

```java
// Forward (default — return view name)
return "success";   // forward to /WEB-INF/view/success.jsp

// Redirect (return "redirect:..." prefix)
return "redirect:/student/list";  // sends 302 to /student/list
return "redirect:https://google.com";  // external redirect
```

**HandlerMapping types:**

| Type | How it maps |
|------|------------|
| `RequestMappingHandlerMapping` | `@RequestMapping`, `@GetMapping` annotations |
| `BeanNameUrlHandlerMapping` | Bean name matches URL (e.g., bean name "/login") |
| `SimpleUrlHandlerMapping` | Explicit URL-to-bean mapping in config |

**Spring MVC vs Pure Servlet — Comparison:**

| Task | Pure Servlet | Spring MVC |
|------|-------------|------------|
| Map URL to handler | `@WebServlet(urlPatterns="/login")` | `@GetMapping("/login")` |
| Read form param | `request.getParameter("name")` | `@RequestParam String name` |
| Read request body (JSON) | Read stream + JSON parse manually | `@RequestBody Student student` |
| Pass data to view | `request.setAttribute("user", u)` | `model.addAttribute("user", u)` |
| Forward to JSP | `rd.forward(req, resp)` | `return "viewName"` |
| Redirect | `resp.sendRedirect("/url")` | `return "redirect:/url"` |
| Validate input | Manual null/length checks | `@Valid` + Bean Validation |

**Spring Boot simplification:**

```java
// Spring Boot — no web.xml, no context.xml, no WAR
// Everything is auto-configured

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);  // Embeds Tomcat!
    }
}

@RestController  // = @Controller + @ResponseBody
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping
    public List<Product> getAll() {
        return productService.findAll();
        // Returns JSON automatically (Jackson converts List to JSON)
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product create(@RequestBody @Valid Product product) {
        return productService.save(product);
    }
}
```

---

## 23. HTTP Status Codes

### Complete Reference

**1xx — Informational**

| Code | Name | Meaning |
|------|------|---------|
| 100 | Continue | Server received headers, client should send body |
| 101 | Switching Protocols | Upgrading from HTTP to WebSocket |

**2xx — Success**

| Code | Name | Use |
|------|------|-----|
| 200 | OK | Standard success (GET, PUT, PATCH) |
| 201 | Created | Resource successfully created (POST) |
| 204 | No Content | Success but no response body (DELETE) |
| 206 | Partial Content | Range request for streaming media |

**3xx — Redirection**

| Code | Name | Use |
|------|------|-----|
| 301 | Moved Permanently | URL permanently changed — update bookmarks |
| 302 | Found (Temporary Redirect) | `sendRedirect()` — login, PRG pattern |
| 304 | Not Modified | Cached version is still valid |
| 307 | Temporary Redirect | Like 302 but preserves HTTP method |
| 308 | Permanent Redirect | Like 301 but preserves HTTP method |

**4xx — Client Errors**

| Code | Name | Use |
|------|------|-----|
| 400 | Bad Request | Invalid request syntax, validation failed |
| 401 | Unauthorized | Not authenticated (no/invalid credentials) |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | Using GET on a POST-only endpoint |
| 409 | Conflict | Duplicate resource (username taken) |
| 415 | Unsupported Media Type | Wrong Content-Type sent |
| 422 | Unprocessable Entity | Request understood but business rule violated |
| 429 | Too Many Requests | Rate limit exceeded |

**5xx — Server Errors**

| Code | Name | Use |
|------|------|-----|
| 500 | Internal Server Error | Unexpected exception in server code |
| 502 | Bad Gateway | Upstream server returned invalid response |
| 503 | Service Unavailable | Server overloaded or in maintenance |
| 504 | Gateway Timeout | Upstream server didn't respond in time |

**401 vs 403 — the classic interview question:**

```
401 Unauthorized → "I don't know WHO you are" → Login and try again
403 Forbidden    → "I know WHO you are, but you can't do THIS" → You're logged in but lack permission

Example:
  User not logged in, hits /admin → 401 (please log in)
  User logged in as ROLE_USER, hits /admin → 403 (you don't have ROLE_ADMIN)
```

**Using status codes in Servlets:**

```java
// Set a specific status
response.setStatus(HttpServletResponse.SC_CREATED);  // 201

// Send error with message
response.sendError(HttpServletResponse.SC_NOT_FOUND, "Product not found");  // 404

// Common constants
HttpServletResponse.SC_OK                    // 200
HttpServletResponse.SC_CREATED               // 201
HttpServletResponse.SC_NO_CONTENT            // 204
HttpServletResponse.SC_MOVED_PERMANENTLY     // 301
HttpServletResponse.SC_FOUND                 // 302
HttpServletResponse.SC_NOT_MODIFIED          // 304
HttpServletResponse.SC_BAD_REQUEST           // 400
HttpServletResponse.SC_UNAUTHORIZED          // 401
HttpServletResponse.SC_FORBIDDEN             // 403
HttpServletResponse.SC_NOT_FOUND             // 404
HttpServletResponse.SC_INTERNAL_SERVER_ERROR // 500
```

---

## 24. Interview Master Reference

### Topic: Servlet Interface

**Beginner (B):**
1. B: What is a Servlet? → A Java program that runs inside a web server to generate dynamic responses.
2. B: What is `servlet-api.jar` and where does it come from? → Contains Servlet interface; comes from Tomcat's `lib/` folder; needed for compilation but NOT packaged in WAR.
3. B: What does `@WebServlet(urlPatterns="/test")` do? → Maps the servlet to the URL `/test`. Tomcat reads this annotation at startup.
4. B: What is `WEB-INF`? → Protected directory. Browser cannot directly access it. Contains `.class` files, `web.xml`, and `lib/`.
5. B: What are the 5 methods of the `Servlet` interface? → `init(ServletConfig)`, `service(SR, SR)`, `destroy()`, `getServletConfig()`, `getServletInfo()`.

**Advanced (A):**
6. A: What is `WODA` in context of Servlets? → Write Once, Deploy Anywhere. The Servlet specification is vendor-neutral. A WAR file built to the spec runs on Tomcat, JBoss, WebLogic without changes.
7. A: What is the difference between `javax.servlet` and `jakarta.servlet`? → `javax.servlet` is pre-Jakarta EE 9 (Tomcat 9, Spring Boot 2). `jakarta.servlet` is Jakarta EE 9+ (Tomcat 10, Spring Boot 3). Package rename happened when Oracle donated Java EE to Eclipse Foundation.

---

### Topic: Servlet Lifecycle

**Tricky (T):**
1. T: Is it possible for `destroy()` to be called on a servlet that's currently handling a request? → Technically yes in a race condition. A well-behaved container waits for active requests to complete. Production code tracks active request count and waits in `destroy()`.
2. T: Can you have two instances of the same servlet class? → No, by default. But if you declare the same servlet class with two different `@WebServlet` URL patterns, the container creates two separate instances — two objects, two `init()` calls.
3. T: Does `loadOnStartup=0` mean higher priority than `loadOnStartup=1`? → Yes. Lower (non-negative) value = higher priority. 0 is highest. Negative = lazy loading (default -1).
4. T: If `init()` throws `UnavailableException(isPermanent=true)`, what happens? → Container permanently removes the servlet. Returns 404 for all future requests to that URL.

---

### Topic: Threading

**Intermediate (I):**
1. I: Why are local variables in `doGet()` thread-safe but instance variables are not? → Local variables live on the thread's private stack. Instance variables live on the shared heap, accessible by all threads.
2. I: How would you make a counter in a servlet thread-safe? → Use `AtomicInteger` or `synchronized`. Never use a plain `int` instance variable.
3. I: What is volatile and when do you use it in servlets? → `volatile` ensures a variable's value is always read from main memory, not a thread-local CPU cache. Use for a `boolean running` flag checked across threads.
4. I: What is a deadlock? Give a servlet example. → Two threads each holding a lock and waiting for the other's lock. In a servlet: Thread-1 locks Cart then Order; Thread-2 locks Order then Cart — both wait forever.

**Advanced:**
5. A: Tomcat's maxThreads is 200. What happens when request 201 arrives? → It goes into the accept queue (`acceptCount`). If the queue is also full, the new connection is rejected.

---

### Topic: Forward vs Redirect

**Tricky:**
1. T: If I call `rd.forward()` and then `response.sendRedirect()` after it, what happens? → `IllegalStateException` is thrown because the response was already committed by `forward()`.
2. T: After `rd.forward()`, the remaining code in the original servlet runs. But can it write to the response? → Technically the code runs, but if the forwarded servlet committed the response, any writes to `response.getWriter()` are silently ignored or throw `IllegalStateException`.
3. T: Can `sendRedirect()` be used after the response is committed? → No. Throws `IllegalStateException`. You must call `sendRedirect()` before any `out.println()` and `out.flush()`.

---

### Topic: Filters

**Intermediate:**
1. I: What is the difference between a Filter and a Servlet? → A Filter intercepts requests to one or more resources for cross-cutting concerns. A Servlet handles the request and generates the final response. Filters wrap servlets.
2. I: How do you prevent a filter from running for specific URLs? → Check `request.getServletPath()` inside `doFilter()` and conditionally call `chain.doFilter()` or skip it.
3. I: What is `FilterChain`? → A linked list of filters. `chain.doFilter(req, resp)` passes control to the next filter in chain or to the servlet if no more filters.
4. I: Is filter order guaranteed with `@WebFilter`? → No. Use `web.xml` `<filter-mapping>` ordering or Spring's `FilterRegistrationBean.setOrder()` for guaranteed order.

**Advanced:**
5. A: How does Spring Security's filter chain differ from Servlet filters? → Spring Security registers a single `DelegatingFilterProxy` as a servlet filter. This proxy delegates to a Spring-managed `FilterChainProxy` which contains Spring Security's own chain of filters (not servlet filters). This allows Spring-managed beans (with DI) to participate as security filters.

---

### Topic: Connection Pooling

**Tricky:**
1. T: If I call `connection.close()` after using a pooled connection, does it physically close the TCP socket to the database? → No. With connection pooling (HikariCP, DBCP2), `close()` returns the connection to the pool. The underlying socket stays open. Physical close only happens when the pool itself is shut down.
2. T: What happens if you forget to call `connection.close()` in a servlet? → The connection is never returned to the pool. Eventually the pool exhausts all connections. New `getConnection()` calls block (and eventually time out), causing the application to hang.
3. T: Why do we look up `DataSource` in `init()` and not in `doGet()`? → `InitialContext` lookup is an expensive JNDI tree traversal. Doing it once in `init()` and caching the `DataSource` is the correct pattern. `getConnection()` in `doGet()` is fast (just takes from pool).

---

### Quick Cheat Sheets

**Servlet Lifecycle Order:**
```
static {} → constructor → init(SC) → init() → service(SR,SR) → service(HSR,HSResp) → doXXX() → destroy()
```

**Filter Order vs Servlet Order:**
```
Filter: Eager (always loads at server start)
Servlet (with loadOnStartup): Eager (loads at server start)
Servlet (no loadOnStartup): Lazy (loads on first request)
```

**Forward vs Redirect one-liner:**
```
Forward  = server-side, same request, URL stays, faster
Redirect = client-side, new request, URL changes, slower (2 round trips)
```

**dispatcher types one-liner:**
```
REQUEST=browser, FORWARD=rd.forward, INCLUDE=rd.include, ERROR=error page, ASYNC=async dispatch
```

**Thread safety rule:**
```
Local variable in doGet() → SAFE (stack)
Instance variable of Servlet → DANGEROUS (heap, shared)
DataSource (initialized once in init) → SAFE if immutable after init
```

**Spring MVC return values:**
```
return "viewName"        → forward to WEB-INF/view/viewName.jsp
return "redirect:/url"   → 302 redirect
return "forward:/url"    → explicit forward
```

---

*End of Backend Engineering Handbook v1.0*  
*Based on classroom notes: 27 Apr – 25 May 2026*  
*Topics: HTTP · Servlet · GenericServlet · HttpServlet · Lifecycle · Tomcat · Threading · Request · Response · ServletConfig · ServletContext · Listeners · JDBC · Connection Pooling · JNDI · RequestDispatcher · Forward/Redirect · Streaming · Filters · DispatcherTypes · Spring MVC · HTTP Status Codes*
