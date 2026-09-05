---
title: "Building Desktop Apps with Kotlin/Native: When You Have to Build Every Tool Yourself"
date: 2026-04-28 10:00:00 -0300
categories: [projects, tooling]
tags: [projects, tooling]
---

My background has long been rooted in the JVM. Over the last several years, Kotlin became my absolute favorite programming language—its conciseness, null-safety, and expressive type system make writing software genuinely enjoyable. 

Naturally, when I started building small desktop utilities and background tools (like system tray apps, helper daemons, or MIDI tools for my guitar setup), my initial instinct was to stay within the Kotlin world.

However, the JVM is not always the best fit for small, unobtrusive background utilities. Even with modern optimizations, baseline memory overhead and startup latency are heavier than what you want for a lightweight daemon meant to sit quietly in the system tray. 

That led me down an ambitious rabbit hole: **Kotlin/Native for Desktop**.

What followed was an intense, educational, and ultimately bittersweet journey into building a standalone native desktop stack from scratch under the [kmupla](https://github.com/kmupla) project.

---

## The Promise vs. Reality of Kotlin/Native for Desktop

Kotlin Multiplatform (KMP) is often praised for its versatility, but when developers talk about "KMP on Desktop," they almost always mean **Compose Multiplatform on the JVM**. 

Pure Kotlin/Native—compiling Kotlin directly to native machine code via LLVM without bundling a JVM—is heavily focused on iOS and shared mobile logic. On desktop operating systems (macOS, Linux, Windows), Kotlin/Native is a niche inside a niche. 

There were practically no mature native desktop GUI frameworks, no ready-to-use system tray bindings, no standard asset bundling mechanism, and very few persistence libraries that fit my workflow.

If I wanted to build native desktop applications in Kotlin without the JVM, I had to build the foundation myself.

---

## 1. The UI Layer: WebViews, System Tray, and C-Interop

Without a ready-made native GUI toolkit, the most practical approach was embedding a lightweight WebView backed by the operating system's built-in engine (WebKit on macOS/Linux, WebView2 on Windows) and combining it with a native system tray icon.

To do this in Kotlin/Native, you have to use **C Interop** (`cinterop`) to bind against low-level C libraries. 

Because standard build systems like Maven lack deep integration with Kotlin/Native's C-interop toolchain, Gradle was mandatory. To keep the setup modular and reusable, I created two standalone bindings:

- [**`webview-kt`**](https://github.com/kmupla/webview-kt): A Kotlin/Native wrapper around lightweight C WebView libraries that interface directly with OS-provided browser engines.
- [**`tray-kt`**](https://github.com/kmupla/tray-kt): A native system tray wrapper allowing menu items, status updates, and background window control.

```kotlin
// Example of orchestrating native tray and webview lifecycle
val tray = SystemTray(title = "App Monitor", icon = "tray_icon.png")
tray.onMenuItemClick("Open Dashboard") {
    webView.show()
}
```

This gave me a functional, low-memory UI surface. But taking the WebView route immediately introduced the next major obstacle: **resource bundling**.

---

## 2. Resource Bundling: The Absence of the Classpath

For anyone coming from the JVM, loading static assets (HTML, CSS, JavaScript, icons) is second nature:

```java
InputStream is = getClass().getClassLoader().getResourceAsStream("ui/index.html");
```

In a compiled native binary, **there is no classpath**. 

Native executables run directly on the OS, meaning assets must either be distributed as external loose files (which is fragile and complicates installation) or embedded directly into the binary's data segments, much like C++ resource compilers (`rc.exe`, `xxd`) or Go's `//go:embed`.

To solve this for Kotlin/Native, I developed the [**`resource-generator`**](https://github.com/kmupla/resource-generator) Gradle plugin.

### How `resource-generator` Works

The plugin inspects asset directories at compile time and transforms raw files into type-safe Kotlin objects containing embedded byte arrays.

```kotlin
// Consuming generated assets in Kotlin/Native
val htmlContent = GeneratedResources.WebUi.indexHtml.readText()
val iconBytes = GeneratedResources.Icons.appLogo.readBytes()
```

Building this plugin required solving several low-level engineering challenges:
1. **Compression**: Embedding raw web assets would inflate the binary size. The plugin compresses assets during code generation and decompresses them on demand in memory.
2. **Byte Chunking & Class Splitting**: Compiling massive byte arrays into a single generated Kotlin source file quickly triggers compiler limits (or huge intermediate object files). The generator splits large assets across multiple modular classes to keep the compiler and memory footprint well-behaved.

Detailed documentation and usage patterns can be found on the [resource-generator documentation site](https://kmupla.github.io/resource-generator/).

---

## 3. Persistence: Building a Code-First Native ORM (KIST ORM)

The next missing piece was local data storage.

In the Java EE / Spring world, Spring Data JPA provides clean, declarative repositories. On Android, Room offers a great annotation-driven abstraction over SQLite. 

In Kotlin Multiplatform, the dominant database tool is **SQLDelight**. While SQLDelight is a fantastic project, it enforces a *SQL-first* philosophy: you write raw `.sq` files and the plugin generates Kotlin code. 

Personally, I prefer a *code-first / entity-driven* approach where I define entity data classes and repository interfaces in Kotlin, while retaining full control over custom SQL when necessary.

To bridge this gap, I created [**`kist-orm`**](https://github.com/kmupla/kist-orm) (Kotlin Interface-based SQLite Toolkit).

### Architecture of Kist ORM

Kist ORM is a lightweight ORM designed specifically for native SQLite drivers:

- **`kist-api`**: Provides core annotations (`@Entity`, `@Dao`, `@Query`, `@Repository`).
- **`kist-ksp`**: A compile-time symbol processor built on **KSP (Kotlin Symbol Processing)** that generates concrete, type-safe SQLite access code without any runtime reflection overhead.

```kotlin
@Entity(tableName = "projects")
data class Project(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val name: String,
    val active: Boolean
)

@Dao
interface ProjectDao {
    @Query("SELECT * FROM projects WHERE active = :active")
    fun findByActive(active: Boolean): List<Project>

    @Insert
    fun insert(project: Project): Long
}
```

By generating all binding logic at compile time via KSP, Kist ORM achieves near zero runtime overhead while maintaining clean repository patterns. More details are available on the [kist-orm documentation site](https://kmupla.github.io/kist-orm/).

---

## The Turning Point: Knowing When to Stop

At this stage, I had a working stack:
- Native GUI & system tray ([`webview-kt`](https://github.com/kmupla/webview-kt), [`tray-kt`](https://github.com/kmupla/tray-kt))
- Compile-time asset embedding ([`resource-generator`](https://github.com/kmupla/resource-generator))
- Type-safe SQLite persistence ([`kist-orm`](https://github.com/kmupla/kist-orm))

Yet, stepping back, an unmistakable reality became clear:

> **When you find yourself spending 80% of your time building foundational infrastructure—C-interop wrappers, asset embedders, custom ORMs, and build plugins—just to write a small utility app, that is a clear signal from the ecosystem.**

Kotlin is an exceptional language, and Kotlin/Native is remarkable engineering. But the ecosystem for **native desktop applications** simply isn't there yet. When community adoption is minimal, every upgrade, edge case, and platform quirk becomes your personal maintenance burden.

### Lessons Learned

I don't regret this exploration at all. Digging into Kotlin/Native gave me invaluable experience:
- Understanding low-level C memory models and pointer interop from a high-level language.
- Working deeply with KSP and compile-time code generation pipelines.
- Managing native asset lifecycle, byte streams, and binary footprints.

### Moving Forward with Swift

Engineering is fundamentally about pragmatism and choosing the right tool for the job. 

For the goal of building lean, native desktop applications that integrate smoothly with system APIs without fighting the tooling, I decided to park the Kotlin/Native desktop experiment for now. Today, I'm focusing my desktop efforts on **Swift** and native macOS/desktop toolchains, where the ecosystem, UI frameworks, and tooling align naturally with the platform.

All the code and plugins remain open source under [kmupla on GitHub](https://github.com/kmupla) for anyone exploring native Kotlin desktop development!
