# Android-EXp



Part 1: Kotlin & Language Fundamentals (1–60)

1. Difference between `val` and `var`.

The core difference comes down to **reassignment** (mutability):

* **`val`** (Value): **Immutable** / Read-Only. Once assigned, you cannot reassign it to a new value.
* **`var`** (Variable): **Mutable**. You can reassign its value as many times as you like.

---

### Quick Comparison

```kotlin
fun main() {
    val name = "Alex"
    // name = "Taylor" // ❌ Compiler Error: Val cannot be reassigned

    var age = 25
    age = 26          //  Works perfectly
}

```

---

### Key Nuances to Keep in Mind

#### 1. `val` is not always fully "immutable"

While you can't reassign a `val` variable, if it points to a **mutable object** (like a `MutableList`), you can still modify the contents *inside* that object.

```kotlin
val numbers = mutableListOf(1, 2, 3)

//  Allowed: Modifying internal state
numbers.add(4) 

// ❌ Not allowed: Pointing 'numbers' to a completely new list
numbers = mutableListOf(5, 6, 7) 

```

#### 2. `val` vs. `const val`

* **`val`**: Can be assigned at **runtime** (e.g., set to the result of a function call or a database fetch).
* **`const val`**: Must be known at **compile-time** (primitives or Strings only, declared at top-level or inside an `object`).

```kotlin
val currentTime = System.currentTimeMillis() //  Evaluated when code runs

const val MAX_ATTEMPTS = 5                   //  Fixed before the app even runs

```

---

> **Rule of Thumb:** Default to **`val`** everywhere. Only switch to **`var`** when you explicitly know the value needs to change. This reduces bugs, makes code safer, and simplifies concurrency/threading.
2. Null safety in Kotlin (`?`, `?.`, `?:`, `!!`).

In many programming languages, accessing a reference that turns out to be `null` triggers a runtime crash (the dreaded `NullPointerException` or NPE).

Kotlin solves this at **compile time** by distinguishing between types that can hold `null` and types that can't.

---

## 1. Nullable Types (`?`)

By default, variables in Kotlin **cannot** be `null`. To allow a variable to hold `null`, you append a `?` to its type.

```kotlin
var name: String = "Alex"
// name = null // ❌ Compiler Error: Null cannot be a value of a non-null type

var nullableName: String? = "Alex"
nullableName = null //  Allowed

```

---

## 2. Safe Call Operator (`?.`)

If a variable can be `null`, the compiler won't let you access its properties directly. The Safe Call operator runs the right side **only if the variable is not null**; otherwise, it returns `null`.

```kotlin
val name: String? = null

// ❌ Compiler Error: Only safe (?.) or non-null asserted (!!.) calls are allowed
// val length = name.length 

val length: Int? = name?.length //  Returns null safely without crashing

```

---

## 3. Elvis Operator (`?:`)

The Elvis operator provides a **default value** if the expression on its left evaluates to `null`. (Turn your head sideways — `?:` looks like Elvis's hair and eyes!)

```kotlin
val name: String? = null

// If 'name' is null, fallback to "Guest"
val displayName: String = name ?: "Guest" 

// Often combined with safe call:
val length: Int = name?.length ?: 0 // If length is null, returns 0

```

---

## 4. Non-Null Assertion Operator (`!!`)

The `!!` operator forcefully converts any value to a non-null type. You are telling the compiler: *"Trust me, this is NOT null."*

> **Warning:** If the value turns out to be `null` at runtime, it will throw a `NullPointerException`.

```kotlin
val name: String? = null

val length: Int = name!!.length // 💥 Throws NullPointerException at runtime!

```

---

### Quick Cheat Sheet

| Operator | Name | What it does | Example | Result if `x` is `null` |
| --- | --- | --- | --- | --- |
| **`Type?`** | Nullable Type | Marks variable as allowed to hold `null` | `val x: String?` | N/A |
| **`?.`** | Safe Call | Executes call only if non-null | `x?.length` | `null` |
| **`?:`** | Elvis Operator | Provides a fallback value if left side is `null` | `x ?: "Default"` | `"Default"` |
| **`!!`** | Non-Null Assertion | Forces non-null conversion (risks NPE) | `x!!.length` | 💥 `NullPointerException` |

> **Best Practice:** Favor `?.` and `?:` everywhere. Reserve `!!` only for scenarios where you are 100% sure a value isn't null, or in tests where you *want* it to fail fast if null.
-----------------------------
3. `lateinit` vs `lazy` initialization.
Both `lateinit` and `lazy` are used in Kotlin to delay initializing a property until it is actually needed. This saves memory and speeds up initial execution, but they are designed for completely different use cases.

---

### Core Differences

| Feature | `lateinit` | `lazy` |
| --- | --- | --- |
| **Mutability** | Must be used with **`var`** | Must be used with **`val`** |
| **Type Support** | Non-null, non-primitive types only | Any type (primitives, nullable, custom types) |
| **Initialization** | Manual (you assign it later in code) | Automatic (evaluated on first access via a lambda) |
| **Thread Safety** | Not thread-safe by default | Thread-safe by default |
| **Primary Use Case** | Dependency Injection, Android Lifecycle (`onCreate`) | Heavy computational properties, setup on first access |

---

### 1. `lateinit` (Late Initialization)

`lateinit` promises the compiler: *"I won't assign this variable right now, but I guarantee I will initialize it before accessing it."*

```kotlin
class UserActivity {
    // Must be 'var' and a non-primitive, non-nullable type
    private lateinit var userApi: UserApiService 

    fun onCreate() {
        // Initialized later manually
        userApi = UserApiService() 
    }

    fun fetchData() {
        // You can check if it has been initialized to avoid errors
        if (::userApi.isInitialized) {
            userApi.getUsers()
        }
    }
}

```

> 💥 **Warning:** If you access a `lateinit var` before assigning it, Kotlin throws an `UninitializedPropertyAccessException`.

---

### 2. `lazy` Initialization

`by lazy` delegates property initialization to a lambda block. The code inside the block runs **only once**, the very first time the property is accessed. Subsequent calls return the cached result.

```kotlin
class DataLoader {
    // Must be 'val'
    val databaseConnection: Connection by lazy {
        println("Connecting to DB...") // Runs ONLY on first access
        Connection.open()
    }
}

fun main() {
    val loader = DataLoader()
    
    // DB is NOT connected yet here
    
    println(loader.databaseConnection) // Prints "Connecting to DB..." and returns connection
    println(loader.databaseConnection) // Simply returns cached connection (no print)
}

```

---

### When to use which?

* Use **`lateinit var`** when you cannot initialize a property in the constructor because its lifecycle is driven by a framework (e.g., Dependency Injection with Dagger/Hilt, or Android Activity lifecycle method `onCreate()`).
* Use **`val by lazy`** for properties that are expensive to compute or fetch, but whose initialization logic is known upfront and shouldn't run until the property is first read.
-----------------
5. Kotlin `data class` and generated methods (`copy`, `equals`, `hashCode`, `toString`, `componentN`).

  In Kotlin, prefixing a class with the `data` keyword tells the compiler that this class exists primarily to **hold data**.

When you do this, Kotlin automatically generates a set of boilerplate methods under the hood based on the properties defined in the **primary constructor**.

```kotlin
data class User(val name: String, val age: Int)

```

---

### The Generated Methods Breakdown

#### 1. `equals()` and `hashCode()`

In standard Java/Kotlin classes, `equals()` checks **structural identity** (whether two references point to the exact same memory location).

For a `data class`, the compiler overrides `equals()` to check **value equality** across all properties in the primary constructor. `hashCode()` is updated accordingly to keep hashed collections like `HashSet` or `HashMap` working correctly.

```kotlin
val user1 = User("Alex", 25)
val user2 = User("Alex", 25)

println(user1 == user2)        // true (Structural equality: values match)
println(user1 === user2)       // false (Referential equality: different memory instances)

```

---

#### 2. `toString()`

Instead of printing an ugly memory reference like `User@45ee92a1`, `data class` automatically synthesizes a clean, readable string representation showing all property names and values.

```kotlin
val user = User("Alex", 25)
println(user) // Prints: User(name=Alex, age=25)

```

---

#### 3. `copy()`

The `copy()` method lets you duplicate an instance while selectively altering only specific properties. This makes maintaining **immutability** (`val`) effortless.

```kotlin
val originalUser = User("Alex", 25)

// Create a new instance with the same name, but updated age
val birthdayUser = originalUser.copy(age = 26)

println(birthdayUser) // User(name=Alex, age=26)

```

---

#### 4. `componentN()` functions (Destructuring Declarations)

The compiler auto-generates `component1()`, `component2()`, ..., `componentN()` functions corresponding to the order of properties in the primary constructor. This powers **destructuring declarations**.

```kotlin
val user = User("Alex", 25)

// Calls user.component1() and user.component2() under the hood
val (userName, userAge) = user 

println(userName) // "Alex"
println(userAge)  // 25

```

---

### Critical Rules to Remember

* **Primary Constructor Only:** The auto-generated methods *only* use properties declared in the primary constructor. Any properties declared inside the class body are ignored by `equals()`, `hashCode()`, `toString()`, and `copy()`.
* **Constructor Requirements:** A `data class` primary constructor must have at least one parameter, and all parameters must be explicitly marked as `val` or `var`.
* **Class Inheritance:** `data class` cannot be marked as `open`, `abstract`, or `sealed`. They are `final` by default and cannot be extended.

   -------------------------------------
7. `open`, `final`, `abstract`, and `sealed` class modifiers.

- final
  - Means "no subclasses / no overrides allowed".
  - In Kotlin everything is final by default (you don't need to write final most of the time).
  - Use final to explicitly forbid further extension/overriding (commonly applied to members after they were open in a base class).

- open
  - Marks a class or member as extendable/overridable.
  - Without open you cannot inherit from a class or override a member.
  - Use open only when you intend consumers to subclass or override.

- abstract
  - Declares a class that cannot be instantiated directly.
  - May contain abstract members (no implementation) that subclasses must implement, and regular members with implementations.
  - Subclasses must implement all abstract members unless they are also abstract.

- sealed
  - Restricts subclassing to a known, fixed set of types: all direct subclasses must be declared in the same file as the sealed class.
  - Creates a closed type hierarchy useful for exhaustive when-expressions and modeling finite ADTs (algebraic data types).
  - Subclasses can be classes, data classes, or objects.

Examples (Kotlin):

```kotlin
name=open_final.kt
```kotlin
// open and final
open class Animal {
    open fun sound(): String = "some noise"
}

class Dog : Animal() {
    override fun sound() = "Woof"
}

// By default classes are final, this is redundant but valid:
final class Cat : Animal() {
    override fun sound() = "Meow"
}

// This would be illegal because Cat is final:
// class PersianCat : Cat() { ... } // ❌ compiler error
```

```kotlin
name=abstract_example.kt
```kotlin
// abstract
abstract class Shape {
    abstract fun area(): Double   // must be implemented by subclasses
    fun describe() = "I am a shape" // concrete member is allowed
}

class Circle(private val r: Double) : Shape() {
    override fun area() = Math.PI * r * r
}

val c = Circle(2.0)
// val s = Shape() // ❌ cannot instantiate abstract class
```

```kotlin
name=sealed_example.kt
```kotlin
// sealed
sealed class Result {
    data class Success(val data: String) : Result()
    object Failure : Result()
}

// Exhaustive when — no `else` needed because compiler knows all subclasses
fun handle(r: Result) = when (r) {
    is Result.Success -> "Got ${r.data}"
    is Result.Failure -> "Failure"
}
```

When to use each (guidance)
- Prefer final (the default): make classes final unless you have a concrete reason to allow subclassing.
- Use open sparingly when designing APIs meant for extension.
- Use abstract when you want a base class with shared code plus required methods that concrete subclasses must supply.
- Use sealed for closed hierarchies (e.g., Result/Response states) where exhaustive handling is desirable.

   ------------------------------------
9. `companion object` vs `object` declaration.

- Use object when you want a standalone singleton (module-level or nested helper). 
- Use companion object when you want a class-scoped singleton that acts like the class’s “static” area (one companion per class, can be called via ClassName and can access the class’s private members).

Explanation and important differences

- object (object declaration)
  - Declares a singleton instance. There can be many object declarations (top-level, nested, or local).
  - Initialized lazily on first access, thread-safe by default.
  - Good for singletons, utility holders, or named instances (e.g., object Logger).
  - If nested inside a class, it behaves like a static nested object and does NOT get the special “companion” semantics.
  - Can implement interfaces and hold state.

- companion object
  - A special object declaration tied to a containing class. Each class can have at most one companion object.
  - Acts like the class’s static area: from Kotlin you can call members directly as ClassName.member (compiler lets you omit Companion).
  - Also an object (lazy init, thread-safe) and can implement interfaces.
  - Can access private members of the containing class (special privilege that plain nested objects don’t have).
  - For Java interop: companion members are compiled to a nested class OuterClass.Companion; annotate members with @JvmStatic to expose them as true static methods/fields on OuterClass for Java callers.

Short examples

```kotlin name=companion_example.kt
class MyClass(private val secret: String) {
    companion object { // can be unnamed (Companion) or named: companion object Factory { ... }
        fun createWithDefault(): MyClass = MyClass("default")
    }

    fun reveal() = secret
}

// Kotlin usage:
val m = MyClass.createWithDefault()
```

```kotlin name=object_example.kt
object Logger {
    fun d(msg: String) = println("DEBUG: $msg")
}

// Kotlin usage:
Logger.d("hi")
```

Java interop note (quick)
- Without @JvmStatic:
  - Companion members are invoked from Java as OuterClass.Companion.method(...).
- With @JvmStatic on a function/property inside companion:
  - Java can call OuterClass.method(...) as a real static.

```kotlin name=interop_example.kt
class Util {
    companion object {
        @JvmStatic fun fromJavaStatic() = "static for Java"
        fun fromJavaViaCompanion() = "needs Companion access"
    }
}
```

Practical guidance / when to pick which
- Use object for standalone singletons (e.g., Logger, Cache, ServiceLocator).
- Use companion object when you need per-class “static-like” members, factory methods, constants, or when you want to expose behavior attached to the class (and optionally support Java-friendly static methods via @JvmStatic).
- Prefer companion object when members need access to private constructors or private members of the class.
- Don’t confuse companion object with static fields in Java — companion is an object instance; Java interop differs unless you add @JvmStatic.

  -------------------------------------
10. Higher-order functions and lambdas.

## Higher-Order Functions and Lambdas

### **What is a Lambda?**

A **lambda** is an anonymous function — a compact way to define a function without declaring a full named function. Lambdas are lightweight, inline functions that can be passed around like values.

```kotlin
// Basic lambda syntax:
val sum = { a: Int, b: Int -> a + b }
val result = sum(5, 3) // result = 8

// Lambda with single parameter (it is implicit)
val doubled = { x: Int -> x * 2 }
val result = doubled(5) // result = 10
```

### **What is a Higher-Order Function?**

A **higher-order function** is a function that either:
- **Takes one or more functions as parameters**, OR
- **Returns a function as a result**

```kotlin
// Higher-order function that takes a lambda as a parameter
fun applyOperation(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// Usage:
val result = applyOperation(5, 3, { x, y -> x + y }) // result = 8
```

### **Function Types Syntax**

In Kotlin, function types follow this pattern:
```kotlin
(InputType1, InputType2, ...) -> ReturnType
```

Examples:
```kotlin
(Int, Int) -> Int           // Function taking two Ints, returns Int
(String) -> Boolean         // Function taking a String, returns Boolean
(List<Int>) -> Unit         // Function taking a List, returns nothing
() -> String                // Function taking no parameters, returns String
```

### **Practical Examples**

#### 1. **Functions as Parameters**

```kotlin
fun executeOperation(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

// Call with lambda
val sum = executeOperation(10, 5, { x, y -> x + y })         // 15
val product = executeOperation(10, 5, { x, y -> x * y })    // 50

// Call with function reference
fun subtract(x: Int, y: Int) = x - y
val difference = executeOperation(10, 5, ::subtract)         // 5
```

#### 2. **Functions Returning Functions**

```kotlin
// Function that returns a function
fun getMultiplier(factor: Int): (Int) -> Int {
    return { number -> number * factor }
}

// Usage
val triple = getMultiplier(3)
println(triple(5))  // 15
println(triple(10)) // 30
```

#### 3. **Common Kotlin Standard Library Higher-Order Functions**

```kotlin
// map: transform each element
val numbers = listOf(1, 2, 3, 4)
val doubled = numbers.map { it * 2 }  // [2, 4, 6, 8]

// filter: keep elements that satisfy a condition
val evens = numbers.filter { it % 2 == 0 }  // [2, 4]

// forEach: execute an action on each element
numbers.forEach { println(it) }

// fold: accumulate values
val sum = numbers.fold(0) { acc, value -> acc + value }  // 10

// sortBy: sort by a property
data class Person(val name: String, val age: Int)
val people = listOf(Person("Alice", 30), Person("Bob", 25))
val sorted = people.sortBy { it.age }

// any / all: check conditions
val hasEven = numbers.any { it % 2 == 0 }     // true
val allPositive = numbers.all { it > 0 }      // true
```

### **Lambda Syntax Variations**

```kotlin
// Explicit types
val add: (Int, Int) -> Int = { a, b -> a + b }

// Type inference (compiler deduces types)
val add = { a: Int, b: Int -> a + b }

// Single parameter with implicit 'it'
val double = { it * 2 }  // 'it' is the single parameter

// Multi-line lambda
val calculate = { a: Int, b: Int ->
    val sum = a + b
    val product = a * b
    sum + product  // last expression is returned
}

// No parameters
val greeting: () -> String = { "Hello, Kotlin!" }
println(greeting())  // "Hello, Kotlin!"
```

### **Trailing Lambda Syntax** (Idiomatic Kotlin)

When a function's last parameter is a lambda, you can move it **outside the parentheses**:

```kotlin
// Standard call
val result = applyOperation(5, 3, { x, y -> x + y })

// Trailing lambda syntax (more readable)
val result = applyOperation(5, 3) { x, y -> x + y }

// Real-world example with Kotlin collections
val names = listOf("Alice", "Bob", "Charlie")
names.forEach { name ->
    println("Hello, $name!")
}
```

### **Practical Real-World Example**

```kotlin
// Repository pattern with higher-order functions
class UserRepository {
    fun fetchUsers(
        onSuccess: (List<User>) -> Unit,
        onError: (Exception) -> Unit
    ) {
        try {
            val users = listOf(User("Alice"), User("Bob"))
            onSuccess(users)  // Execute the success callback
        } catch (e: Exception) {
            onError(e)  // Execute the error callback
        }
    }
}

// Usage
val repo = UserRepository()
repo.fetchUsers(
    onSuccess = { users ->
        println("Got ${users.size} users")
        users.forEach { println(it.name) }
    },
    onError = { error ->
        println("Error: ${error.message}")
    }
)
```

### **Key Takeaways**

| Concept | Description |
| --- | --- |
| **Lambda** | Anonymous function; compact syntax for simple functions |
| **Higher-Order Function** | Function that takes or returns another function |
| **Function Type** | Syntax: `(ParamType) -> ReturnType` |
| **Trailing Lambda** | Move last lambda parameter outside parentheses for readability |
| **`it` keyword** | Implicit parameter name for single-parameter lambdas |
| **Function Reference** | Use `::functionName` to pass a named function as a lambda |

---

-------------------------------------------
13. `inline`, `noinline`, and `crossinline` functions.
14. `reified` type parameters in inline functions.
15. Extension functions and property implementation under the hood.
16. `map` vs `flatMap` vs `compactMap` equivalents.
17. Collection mutability (`List` vs `MutableList`).
18. Scope functions (`let`, `run`, `with`, `apply`, `also`) selection criteria.
19. Elvis operator short-circuiting mechanics.
20. `infix` notation requirements and usage.
21. Destructuring declarations (`component1()`, `component2()`).
22. `const val` vs `val`.
23. Delegated properties (`by`) and custom delegates (`ReadOnlyProperty`, `ReadWriteProperty`).
24. Backing fields (`field`) vs backing properties.
25. `sealed class` vs `sealed interface`.
26. Operator overloading syntax and common use cases.
27. Unchecked vs checked exceptions in Kotlin vs Java.
28. `@JvmStatic`, `@JvmOverloads`, `@JvmField`, and `@JvmName` annotations.
29. Primary vs secondary constructors and init blocks.
30. `enum class` vs `sealed class`.
31. Value classes (`@JvmInline value class`).
32. `typealias` keyword usage and limitations.
33. Functional interfaces (`fun interface` / SAM conversion).
34. Generics variance: covariance (`out`), contravariance (`in`), invariance.
35. Star projection (`*`) in generics.
36. Type erasure in Kotlin/Java generics.
37. `Nothing` type vs `Unit` vs `Any`.
38. Structural equality (`==`) vs Referential equality (`===`).
39. Range expressions and progression loops (`step`, `downTo`, `until`).
40. `Sequence` vs `Iterable` processing (lazy vs eager evaluation).
41. Smart casts and `is` / `!is` operators.
42. Custom getters and setters implementation rules.
43. Kotlin reflection (`KClass`, `KProperty`, `KFunction`).
44. Non-local returns in lambdas and `inline` interaction.
45. Top-level functions and properties compiled representation.
46. Standard library functions: `takeIf`, `takeUnless`.
47. Array types in Kotlin (`IntArray`, `Array<Int>`) performance differences.
48. `@Volatile` and `@Transient` annotations.
49. String templates and raw multiline strings.
50. `tailrec` modifier and tail recursion optimization.
51. Contracts API (`contract { ... }`).
52. How Kotlin handles default arguments in Java interop.
53. Package-private visibility equivalent in Kotlin (`internal`).
54. Exception handling with `runCatching` and `Result<T>`.
55. Multiplatform Kotlin (KMP) basics: `expect` / `actual` declaration.
56. Immutable collections vs read-only collections.
57. `associate`, `associateBy`, and `groupBy` operators.
58. `fold` vs `reduce` collection methods.
59. Primitive type wrapping and boxing in Kotlin.
60. Destructuring custom classes using `componentN()` functions.
61. Overriding properties in subclasses.
62. Interface default method implementations in Kotlin bytecode.
63. Operator function conventions for collections (`get`, `set`, `contains`).
64. `checkNotNull` and `require` precondition checks.
65. Explicit API mode in Kotlin libraries.

---

## Part 2: Core Android Framework & Lifecycle (61–125)

61. Four primary Android application components.
62. Activity lifecycle methods and execution flow.
63. Fragment lifecycle methods vs Activity lifecycle.
64. `Context` types (`ApplicationContext`, `ActivityContext`, `ServiceContext`).
65. Structure and purpose of `AndroidManifest.xml`.
66. Explicit vs Implicit Intents.
67. `PendingIntent` security and flags (`FLAG_IMMUTABLE`, `FLAG_MUTABLE`).
68. Intent flags: `FLAG_ACTIVITY_NEW_TASK`, `FLAG_ACTIVITY_CLEAR_TOP`, `FLAG_ACTIVITY_SINGLE_TOP`.
69. Activity launch modes: `standard`, `singleTop`, `singleTask`, `singleInstance`, `singleInstancePerTask`.
70. Configuration changes handling (orientation, locale, dark mode).
71. `onSaveInstanceState()` vs `onRestoreInstanceState()`.
72. `SavedStateHandle` in ViewModels vs Bundle state saving.
73. Static vs Dynamic `BroadcastReceiver` registration.
74. Ordered vs Normal Broadcasts.
75. `ContentProvider` structure, URI matching, and CRUD operations.
76. `Parcelable` vs `Serializable` performance comparison.
77. `Application` subclass lifecycle and scope.
78. What causes ANR (Application Not Responding) and diagnosis tools.
79. Service types: Foreground, Background, Bound.
80. Service lifecycle (`onStartCommand`, `onBind`, `onDestroy`).
81. `onStartCommand` return values (`START_STICKY`, `START_NOT_STICKY`, `START_REDELIVER_INTENT`).
82. Why `IntentService` was deprecated and its modern replacements.
83. Differences between `minSdkVersion`, `targetSdkVersion`, and `compileSdkVersion`.
84. Runtime permission model (Android 6.0+) and permission groups.
85. Task back stack management and clear tasks.
86. `FragmentFactory` purpose and custom dependency injection.
87. Fragment transactions (`add` vs `replace`, `addToBackStack`).
88. Fragment communication methods (`setFragmentResult`, Shared ViewModel, Interfaces).
89. `android:exported` manifest declaration rules (Android 12+).
90. Android Runtime (ART) vs Dalvik VM.
91. Ahead-Of-Time (AOT), Just-In-Time (JIT), and Profile-Guided Compilation.
92. `Zygote` process role in application cold start.
93. Application startup phases: Cold Start, Warm Start, Hot Start.
94. Splash Screen API integration (Android 12+).
95. Deep linking: Standard Deep Links vs App Links vs Web Links.
96. AssetManager vs Resources (`res/raw` vs `assets/`).
97. Vector Drawables and Vector Animation (`AnimatedVectorDrawable`).
98. Resource qualifiers (`sw600dp`, `land`, `night`, `v31`).
99. Dark Theme implementation and `Force Dark` configuration.
100. Multi-window mode and Picture-in-Picture (PiP) support.
101. Keyboard visibility listeners and `WindowInsetsCompat`.
102. Edge-to-edge drawing (`enableEdgeToEdge()`).
103. `HandlerThread`, `Looper`, `Handler`, and `MessageQueue` architecture.
104. Main UI Thread event loop.
105. `AsyncTask` deprecation causes and alternatives.
106. Notification Channels, Importance levels, and Groups.
107. Foreground service notifications and user notification permissions (Android 13+).
108. App Standby Buckets and battery optimization effects.
109. Exact alarm permissions (`SCHEDULE_EXACT_ALARM`) and power restrictions.
110. Storage Access Framework (SAF) vs Scoped Storage.
111. Photo Picker API (`ActivityResultContracts.PickVisualMedia`).
112. Activity Result API (`registerForActivityResult`) replacing `onActivityResult`.
113. Android Backup Service and `android:allowBackup`.
114. Locale Preferences and Per-App Language Preferences (Android 13+).
115. Predictive Back Gesture API (Android 13/14+).
116. `LocalBroadcastManager` deprecation reasons.
117. Memory Pressure callbacks (`onTrimMemory`, `onLowMemory`).
118. App process priority hierarchy (Foreground, Visible, Service, Cached).
119. Process death vs Activity destruction.
120. Multi-process application configuration (`android:process`).
121. Inter-Process Communication (IPC) options: Binder, AIDL, Messenger, ContentProvider.
122. AIDL (Android Interface Definition Language) workflow.
123. Binder transaction limit (1MB buffer) and `TransactionTooLargeException`.
124. Font resource loading and downloadable fonts.
125. Adaptive Icons structure (Foreground, Background, Monochromatic).

---

## Part 3: Asynchronous Programming, Coroutines & Flow (126–175)

126. What is a Kotlin Coroutine?
127. Coroutines vs Threads execution model.
128. `suspend` keyword mechanics under the hood (Continuation-Passing Style).
129. `CoroutineScope` vs `CoroutineContext`.
130. Standard Dispatchers (`Dispatchers.Main`, `Dispatchers.IO`, `Dispatchers.Default`, `Dispatchers.Unconfined`).
131. `launch` vs `async` coroutine builders.
132. Structured Concurrency principles.
133. `Job` lifecycle states (`New`, `Active`, `Completing`, `Completed`, `Cancelling`, `Cancelled`).
134. Coroutine cancellation cooperativeness and `ensureActive()`, `yield()`.
135. `withContext` usage and context switching efficiency.
136. `SupervisorJob` vs standard `Job` failure propagation.
137. `CoroutineExceptionHandler` mechanics and scope boundary rules.
138. Exception handling in `async` vs `launch`.
139. Kotlin `Flow` API definition and reactive stream paradigm.
140. Cold Streams vs Hot Streams.
141. `StateFlow` vs `SharedFlow` vs `LiveData`.
142. Lifecycle-aware Flow collection (`repeatOnLifecycle`, `flowWithLifecycle`).
143. `Channel` primitive and buffer strategies (`RENDEZVOUS`, `UNBUFFERED`, `CONFLATED`, `BUFFERED`).
144. Flow operators: `map`, `filter`, `transform`, `onEach`.
145. Intermediate vs Terminal Flow operators.
146. `combine` vs `zip` operators.
147. `flatMapConcat` vs `flatMapMerge` vs `flatMapLatest`.
148. Backpressure management strategies in Flow (`buffer`, `conflate`, `collectLatest`).
149. `collect` vs `collectLatest`.
150. Converting cold flows to hot flows using `stateIn` and `shareIn`.
151. Concurrency control using `Mutex` in coroutines.
152. Bridging callback APIs using `suspendCancellableCoroutine` and `callbackFlow`.
153. `delay()` vs `Thread.sleep()`.
154. `coroutineScope { }` vs `supervisorScope { }`.
155. `NonCancellable` context usage.
156. Testing coroutines with `TestDispatcher`, `StandardTestDispatcher`, and `UnconfinedTestDispatcher`.
157. `runTest` scope in unit testing.
158. `MainDispatcherRule` in Android unit tests.
159. Testing Flows using the `Turbine` library.
160. `actor` and channel-based state management (deprecated vs modern alternatives).
161. `GlobalScope` usage risks.
162. `viewModelScope` and `lifecycleScope` implementation details.
163. Thread confinement in coroutines (`newSingleThreadContext`).
164. Volatile variables vs Atomic references in concurrent Kotlin.
165. Shared mutable state concurrency solutions in Kotlin.
166. `FlowOn` operator and context preservation rule.
167. Catching errors in Flow using `catch` operator.
168. Retrying Flow collection using `retry` and `retryWhen`.
169. `onStart`, `onCompletion`, and `onEmpty` Flow lifecycle operators.
170. Select expression (`select { ... }`) for multi-coroutine racing.
171. Custom Flow builders (`flow { ... }`).
172. Debouncing user input with `debounce` operator.
173. Throttling strategies with Flow.
174. Infinite Flow handling and resource cleanup.
175. Coroutine trace logging and debugging tools (`-Dkotlinx.coroutines.debug`).

---

## Part 4: Jetpack Compose & Declarative UI (176–240)

176. Declarative UI vs Imperative View system model.
177. `@Composable` annotation function properties.
178. Recomposition process in Compose.
179. Three phases of Compose rendering: Composition, Layout, Drawing.
180. `remember` vs `rememberSaveable`.
181. `mutableStateOf` and state snapshot tracking system.
182. State Hoisting pattern.
183. `Modifier` order chaining significance.
184. `LazyColumn` and `LazyRow` internal recycling mechanics.
185. Key usage in `items()` of Lazy lists.
186. `LaunchedEffect` usage and key parameters.
187. `DisposableEffect` cleanup lifecycle.
188. `SideEffect` purpose and execution timing.
189. `derivedStateOf` calculation caching and optimization.
190. `rememberCoroutineScope` usage in Composables.
191. `CompositionLocal` (`compositionLocalOf` vs `staticCompositionLocalOf`).
192. Compose layout engine and node tree (`LayoutNode`).
193. `Scaffold` layout slot structure.
194. `ConstraintLayout` in Compose vs XML ConstraintLayout.
195. Navigation in Compose (`NavHost`, `composable`, Type-Safe Navigation in Compose 2.8+).
196. Animation APIs: `animateAsState`, `updateTransition`, `AnimatedVisibility`, `Crossfade`.
197. Performance annotations: `@Stable` and `@Immutable`.
198. Observing `Flow`, `StateFlow`, and `LiveData` in Compose (`collectAsStateWithLifecycle`).
199. Canvas rendering in Compose (`Canvas` composable, `DrawScope`).
200. `SubcomposeLayout` purpose and examples (`BoxWithConstraints`).
201. Interoperability: Hosting XML Views inside Compose (`AndroidView`).
202. Interoperability: Hosting Composables in XML (`ComposeView`).
203. Intrinsic measurements (`IntrinsicSize.Min`, `IntrinsicSize.Max`).
204. Theme configuration (`MaterialTheme`, Typography, ColorScheme, Shapes).
205. Testing Compose UI (`ComposeTestRule`, `onNodeWithText`, `performClick`).
206. Debugging recomposition counts (Layout Inspector, Recomposition Counters).
207. `rememberUpdatedState` pattern for long-lived callbacks.
208. `produceState` converter.
209. Custom Modifiers implementation (`Modifier.Node`, `Modifier.composed`).
210. Custom Layout implementation using `Layout` composable.
211. `Paging 3` integration with Compose (`collectAsLazyPagingItems`).
212. Text input handling (`TextField`, `BasicTextField`, `TextFieldValue`).
213. Keyboard actions and IME options in Compose.
214. Focus management (`FocusRequester`, `FocusManager`).
215. Image loading in Compose using Coil (`AsyncImage`).
216. Window Size Classes (`calculateWindowSizeClass`) for adaptive layouts.
217. Pull-to-refresh implementation in Compose.
218. Swipeable layouts and AnchoredDraggable API.
219. Dialogs and Popups in Compose (`Dialog`, `Popup`).
220. Semantics tree and Accessibility (`semantics`, `clearAndSetSemantics`).
221. Test Tags usage (`Modifier.testTag`).
222. Compose compiler metrics output generation and interpretation.
223. Restartable vs Non-restartable Composable functions.
224. Skippable Composable functions criteria.
225. Strong Skipping Mode in Compose compiler.
226. Slot API design pattern.
227. `LazyVerticalGrid` and `LazyHorizontalGrid` usage.
228. Sticky Headers in Lazy Lists (`stickyHeader`).
229. Scrolling state control (`LazyListState`, `rememberLazyListState`).
230. Nested scrolling in Compose (`NestedScrollConnection`).
231. Graphics Layer transformations (`Modifier.graphicsLayer`).
232. Blend Modes and ColorFilters in Compose.
233. Font loading and custom Font Families in Compose.
234. MotionLayout in Compose.
235. Dynamic Color (Material You) integration in Compose.
236. Multi-preview annotations (`@PreviewScreenSizes`, `@PreviewFontScales`).
237. CompositionLocal default fallback handling.
238. Snapshot state system (`Snapshot.takeSnapshot`, `SnapshotStateList`).
239. Back Handler composable (`BackHandler`).
240. Bottom Sheets implementation (`ModalBottomSheet`).

---

## Part 5: Legacy View System & Custom Views (241–270)

241. `View` vs `ViewGroup` taxonomy.
242. View rendering pipeline steps: `onMeasure()`, `onLayout()`, `onDraw()`.
243. `MeasureSpec` modes (`EXACTLY`, `AT_MOST`, `UNSPECIFIED`).
244. `RecyclerView` architecture: `Adapter`, `LayoutManager`, `ViewHolder`, `ItemAnimator`.
245. `DiffUtil` algorithm and `AsyncListDiffer` performance.
246. `RecyclerView` view pool sharing (`RecycledViewPool`).
247. `ItemDecoration` drawing canvas ordering.
248. `ItemTouchHelper` drag-and-drop and swipe-to-delete logic.
249. `View.GONE` vs `View.INVISIBLE` vs `View.VISIBLE`.
250. ViewBinding vs DataBinding.
251. Two-way DataBinding implementation (`@={variable}`).
252. Custom View creation extending `View` vs `LinearLayout`.
253. Custom Attributes handling via `declare-styleable` and `obtainStyledAttributes`.
254. Hardware Acceleration flags and incompatible drawing operations.
255. `Canvas`, `Paint`, `Path`, and `Matrix` graphics operations.
256. `SurfaceView` vs `TextureView` vs standard `View`.
257. `CoordinatorLayout` and `Behavior` interactions.
258. Collapsing Toolbar pattern (`AppBarLayout`, `CollapsingToolbarLayout`).
259. Touch Event dispatching hierarchy (`dispatchTouchEvent`, `onInterceptTouchEvent`, `onTouchEvent`).
260. View invalidate (`invalidate()`) vs Request Layout (`requestLayout()`).
261. `ViewTreeObserver` listeners (`onGlobalLayoutListener`, `onPreDrawListener`).
262. Flat layout hierarchies benefits (`ConstraintLayout` vs nested `LinearLayout`).
263. `ConstraintLayout` barriers, guidelines, chains, and groups.
264. `LayoutInflater` process and factory set.
265. `ViewStub` lazy inflation mechanism.
266. `Include` vs `Merge` XML tags.
267. Custom touch handling with `GestureDetector` and `ScaleGestureDetector`.
268. ScrollView vs NestedScrollView.
269. Auto-sizing TextViews.
270. Transition Framework and Scene transitions.

---

## Part 6: Architecture, Jetpack Components & Patterns (271–320)

271. Architectural Patterns comparison: MVC vs MVP vs MVVM vs MVI.
272. Clean Architecture principles: Presentation, Domain, Data layers.
273. SOLID principles application in Android codebases.
274. Domain layer necessity and Use Case / Interactor pattern.
275. `ViewModel` state retention across configuration changes mechanism.
276. `ViewModelProvider.Factory` purpose.
277. `ViewModel` vs `AndroidViewModel` context leak risks.
278. `LiveData` lifecycle observation mechanism.
279. `LiveData` transformation methods (`map`, `switchMap`).
280. `MediatorLiveData` combining multiple sources.
281. `SingleLiveEvent` pattern and why Flow replacement is preferred.
282. Room Database components: Entity, DAO, RoomDatabase.
283. Room TypeConverters for complex objects.
284. Database relations in Room (`@Relation`, `@Embedded`, Junction tables).
285. `@Transaction` annotation necessity in Room operations.
286. Manual vs Automatic Room migrations (`Migration`, `AutoMigration`).
287. Room with Coroutines Flow for reactive database queries.
288. `WorkManager` purpose vs AlarmManager vs Foreground Services.
289. WorkManager task persistence and execution guarantees.
290. WorkManager constraints (`NetworkType`, `RequiresCharging`, `RequiresBatteryNotLow`).
291. `OneTimeWorkRequest` vs `PeriodicWorkRequest`.
292. Work chaining and parallel execution in WorkManager.
293. Passing data to/from WorkManager using `Data` objects.
294. `ListenableWorker` vs `CoroutineWorker`.
295. `DataStore` (Preferences DataStore & Proto DataStore) vs SharedPreferences.
296. Migration path from SharedPreferences to DataStore.
297. SharedPreferences synchronous `commit()` vs asynchronous `apply()`.
298. `Paging 3` architecture: `PagingSource`, `RemoteMediator`, `PagingData`.
299. Invalidation and refreshing strategies in Paging 3.
300. Dependency Injection (DI) concept and inversion of control.
301. Compile-time DI (Dagger/Hilt) vs Service Locator (Koin).
302. Hilt setup: `@HiltAndroidApp`, `@AndroidEntryPoint`.
303. Hilt modules and bindings (`@Module`, `@InstallIn`, `@Binds`, `@Provides`).
304. Hilt scopes (`@Singleton`, `@ActivityRetainedScoped`, `@ViewModelScoped`).
305. Qualifier annotations (`@Qualifier`, `@Named`) in Hilt.
306. Custom Hilt EntryPoints for non-Android-managed classes.
307. Single Source of Truth (SSOT) pattern implementation.
308. Repository pattern responsibility boundaries.
309. Unidirectional Data Flow (UDF) pattern.
310. State vs Event differentiation in UI architecture.
311. Handling one-off events in UI (Channels vs SharedFlow vs State).
312. Navigation Component architecture: NavGraph, NavHost, NavController.
313. Safe Args plugin mechanism.
314. Modularization strategies: Layer-based vs Feature-based vs Dynamic Feature modules.
315. Circular dependencies handling in modular projects.
316. Base ViewModel abstraction pros and cons.
317. Offline-First application architecture design.
318. Cache invalidation strategies in repository layer.
319. Event Bus pattern risks and modern alternatives.
320. Feature flags architectural integration.

---

## Part 7: Networking, Data Handling & Security (321–370)

321. Retrofit architecture and dynamic interface proxies.
322. OkHttp library role in network stack.
323. OkHttp Interceptors: Application Interceptors vs Network Interceptors.
324. HTTP Caching mechanisms in OkHttp (`Cache`, `CacheControl`).
325. Certificate Pinning setup (`CertificatePinner`) and MITM protection.
326. OAuth2 flow implementation and automatic token refresh using OkHttp `Authenticator`.
327. Image loading pipeline architecture: Caching (Memory + Disk), Downsampling, Decoding.
328. Coil vs Glide vs Picasso comparison.
329. SQLite engine vs Room abstraction overhead.
330. Encrypted DataStore and EncryptedSharedPreferences implementation.
331. Android KeyStore system, hardware-backed keys, and TEE (Trusted Execution Environment).
332. Biometric Authentication API (`BiometricPrompt`) implementation.
333. Scoped Storage rules (Android 10+) and MediaStore API usage.
334. Network Security Configuration XML (`network_security_config.xml`).
335. HTTPS/TLS enforcement and cleartext traffic policy.
336. Firebase Cloud Messaging (FCM) architecture: Data Messages vs Notification Messages.
337. Handling background vs foreground push notifications.
338. WebSockets for real-time streaming in Android.
339. Server-Sent Events (SSE) handling.
340. REST vs GraphQL consumption libraries (Apollo GraphQL Client).
341. Serialization options comparison: `kotlinx.serialization` vs Moshi vs Gson.
342. Polymorphic serialization with `kotlinx.serialization`.
343. Pagination techniques: Offset-based vs Cursor-based.
344. Handling large file uploads using Multipart requests.
345. DownloadManager system service usage.
346. Connection state monitoring (`ConnectivityManager`, `NetworkCallback`).
347. Data compression over network (GZIP response handling).
348. Content Provider security (`android:permission`, `readPermission`, `writePermission`).
349. SQL Injection prevention in raw SQLite queries (selection args).
350. Cross-Site Scripting (XSS) risks in WebView.
351. WebView security best practices (`setJavaScriptEnabled`, JavascriptInterface risks).
352. Secure data wiping techniques in memory/storage.
353. Cryptographic primitives: Symmetric (AES) vs Asymmetric (RSA) encryption.
354. Key derivation functions (PBKDF2) for user passwords.
355. Android Keystore key invalidation on new biometric enrollment (`setInvalidatedByBiometricEnrollment`).
356. Android Keystore entry aliases and key rotation.
357. Certificate Authority (CA) custom certificate trust setup.
358. Detecting mock location providers.
359. Play Integrity API (formerly SafetyNet Attestation) integration.
360. App Sandboxing and Process Isolation security model.
361. File Provider setup (`<provider>`, `file_paths.xml`) for secure file sharing.
362. Custom URI schemes security vulnerabilities.
363. Dynamic feature downloading security validation.
364. Tapjacking protection (`setFilterTouchesWhenObscured`).
365. Screen capture and screenshot prevention (`FLAG_SECURE`).
366. Clipboard data security risks and sensitive data flags.
367. Data sanitization before logging.
368. Reverse engineering defense strategies.
369. Checking app signature hash at runtime.
370. Data Backup encryption standards.

---

## Part 8: Memory Management, Performance & System Internals (371–425)

371. Garbage Collection algorithms in ART (Generational Concurrent GC).
372. Memory Leak definition and lifecycle references.
373. Common Android memory leak scenarios (Static Context, Unregistered Listeners, Inner Classes, Anonymous Callbacks).
374. LeakCanary working mechanism under the hood.
375. Reference types: Strong, Soft, Weak, Phantom references.
376. `SparseArray` family vs standard `HashMap` memory footprint.
377. Android Studio Memory Profiler usage (Heap Dump, Allocation Tracking).
378. Android Studio CPU Profiler usage (System Trace, Call Chart, Flame Chart).
379. Android Studio Network Profiler usage.
380. `StrictMode` configuration for disk/network detection on UI thread.
381. App Startup Jetpack library initialization optimization.
382. Baseline Profiles creation and performance benefits (JIT to AOT compilation guidance).
383. Macrobenchmark library usage.
384. APK and AAB size optimization techniques.
385. R8 compiler vs ProGuard differences.
386. Code Obfuscation, Shrinking, and Optimization with R8.
387. Writing custom Keep Rules (`proguard-rules.pro`).
388. Native C/C++ libraries usage via NDK and JNI interface.
389. Hiding secret keys using C/C++ native implementation vs `local.properties`.
390. Root detection strategies (Su binary check, Build tags, SafetyNet/Play Integrity).
391. Doze Mode phases and App Standby battery management optimizations.
392. Overdraw definition and GPU Overdraw debugging tool.
393. Frame drops (Jank) causes and 60fps/120fps rendering budgets (16ms / 8ms).
394. RenderThread role in Android UI pipeline.
395. ChoreoGrapher role in frame rendering synchronization.
396. Bitmaps memory allocation (Native Heap vs Java Heap across Android versions).
397. Bitmap in-memory optimization (`inSampleSize`, `inBitmap`, `RGB_565` vs `ARGB_8888`).
398. Bitmap memory pooling techniques.
399. ANR tracing log file interpretation (`traces.txt`).
400. Battery Historian tool usage for power consumption analysis.
401. WakeLocks usage risks and battery drain mitigation.
402. WorkManager power consumption efficiency.
403. Object allocation during frame rendering avoidance strategies.
404. String concatenation overhead and `StringBuilder` usage in tight loops.
405. Autoboxing performance impacts in critical code paths.
406. Optimizing RecyclerView layout rendering depth.
407. Flat UI hierarchy rendering performance benefits.
408. Prefetching data in RecyclerView (`setItemViewCacheSize`).
409. Thermal Throttling detection and mitigation.
410. Memory fragmentation issues on long-running processes.
411. Low Memory Killer (LMK) daemon mechanism and `oom_adj` scores.
412. Deadlock conditions identification and prevention in multi-threaded code.
413. Thread pool configuration (`ThreadPoolExecutor`) tuning for IO vs CPU bound tasks.
414. `LruCache` class internal implementation (`LinkedHashMap` with access order).
415. DiskLruCache usage for network caching.
416. Measuring startup time: `reportFullyDrawn()`.
417. Trace marker APIs (`Trace.beginSection()`, `Trace.endSection()`).
418. Systrace / Perfetto tool usage for deep system profiling.
419. Memory footprint of Kotlin lambdas and SAM conversion objects.
420. `inline` class efficiency in avoiding allocations.
421. Profile-Guided Optimization (PGO) in Android builds.
422. App Bundle Dynamic Delivery mechanics (Split APKs).
423. On-Demand Dynamic Feature Module downloading API.
424. Unused resource stripping (`shrinkResources true`).
425. Native memory leaks detection (`HWASan`, `AddressSanitizer`).

---

## Part 9: Testing, Build Systems & Continuous Integration (426–465)

426. Testing Pyramid concept: Unit Tests, Integration Tests, UI/E2E Tests ratios.
427. Unit Tests vs Instrumented Tests execution environments.
428. JUnit 4 vs JUnit 5 framework features.
429. Mocking frameworks comparison: Mockito vs MockK.
430. Mocking vs Spying vs Stubbing vs Faking vs Dummy objects.
431. Testing ViewModels: Coroutines testing rules and state verification.
432. Testing ViewModel state events with Turbine library.
433. Robolectric framework principles and fast local JVM testing.
434. Espresso UI testing framework core concepts (`onView`, `withText`, `perform`, `check`).
435. Espresso `IdlingResource` for asynchronous operation synchronization.
436. UI Automator framework usage for cross-app UI testing.
437. MockWebServer for testing Retrofit API calls.
438. Gradle build system architecture and execution phases (Initialization, Configuration, Execution).
439. Root `build.gradle` vs Module `build.gradle` responsibilities.
440. Gradle Version Catalogs (`libs.versions.toml`) structure and setup.
441. Dependency configuration scopes: `implementation` vs `api` vs `compileOnly` vs `runtimeOnly`.
442. `kapt` vs `ksp` (Kotlin Symbol Processing) performance and mechanics.
443. Build Types vs Product Flavors vs Build Variants.
444. `dimension` usage in Gradle product flavors.
445. Custom Gradle Tasks creation in Kotlin DSL (`build.gradle.kts`).
446. Groovy DSL vs Kotlin DSL (`.gradle` vs `.gradle.kts`) in Gradle.
447. Gradle build cache, configuration cache, and incremental compilation optimizations.
448. APK vs Android App Bundle (AAB) deployment architecture.
449. Play Feature Delivery and Dynamic Modules.
450. CI/CD pipeline integration for Android apps (GitHub Actions, Bitrise, GitLab CI).
451. Automated APK signing configuration in Gradle (`signingConfigs`).
452. Managing keystore passwords securely in CI/CD environments.
453. Static code analysis tools setup: Detekt, ktlint, Android Lint, SonarQube.
454. Running custom Android Lint checks.
455. Code Coverage reporting tools: JaCoCo, Kover.
456. Screenshot testing tools and visual regression testing (Paparazzi, Roborazzi, Shot).
457. Test Driven Development (TDD) workflow in Android.
458. Dependency graph validation tests.
459. Test fixtures in Gradle.
460. Running instrumented tests on Firebase Test Lab or Matrix devices.
461. Flaky tests root causes and mitigation techniques.
462. Parameterized unit testing in JUnit.
463. Fake repository creation for clean integration testing.
464. Hermetic testing environment setup.
465. Automated app deployment to Google Play Console via Fastlane.

---

## Part 10: Advanced Scenarios, Hardware, Design & System Architecture (466–500)

466. Designing an Offline-First News Reader Application architecture.
467. Designing a Real-Time Chat Application architecture (WebSockets + Room + FCM).
468. Designing an Image/Video Feed with infinite scrolling and memory caching.
469. Handling Bluetooth Low Energy (BLE) scanning, connection, and GATT services in Android.
470. Location Tracking application design (Background Location, Geofencing, Foreground Service).
471. Location permissions changes across Android versions (Approximate vs Precise, Background Location).
472. CameraX API implementation: Image Capture, Image Analysis, Video Capture.
473. Camera2 API low-level control vs CameraX high-level abstraction.
474. Audio recording and playback pipelines (`MediaPlayer`, `ExoPlayer` / `Media3`).
475. Media3 Transformer API for media processing.
476. Custom Android Keyboard (IME) development fundamentals.
477. Home Screen Widgets implementation using Jetpack Glance (`GlanceAppWidget`).
478. Android Wear OS development basics and tile architecture.
479. Android Auto / Automotive OS application design guidelines.
480. TV App development with Leanback / Compose for TV.
481. Accessibility (a11y) implementation: TalkBack compatibility, content descriptions, touch targets.
482. Localization (i18n) and Internationalization best practices (RTL layouts, plural rules).
483. In-App Purchases and Subscription model using Google Play Billing Library.
484. In-App Updates API (Immediate vs Flexible update UI flow).
485. In-App Reviews API integration timing rules.
486. Google Pay integration via Payments API.
487. Health Connect API integration for health data sharing.
488. NFC (Near Field Communication) reading and card emulation (HCE).
489. Wi-Fi Direct and Direct P2P Communication.
490. Managing multi-display and foldable device layouts (`WindowTracker` API, hinge postures).
491. Sensors API usage (Accelerometer, Gyroscope) and power management.
492. Cross-Device Connectivity API (Google Nearby Connections).
493. Enterprise App Deployment and Device Admin / Enterprise EMM APIs.
494. Custom ROMs / AOSP modification basics and system app privileges.
495. Google Play Policy compliance (Data Safety section, Target API requirements, Prominent Disclosure).
496. Design Patterns in Android codebase: Singleton, Factory, Builder, Observer, Adapter, Strategy, Decorator.
497. Migrating legacy Java/XML Android codebases to 100% Kotlin/Compose strategy.
498. Zero-downtime database schema migration strategy in production.
499. Crash reporting architecture setup (Crashlytics, custom unhandled exception handlers).
500. Remote Config and AB Testing integration architecture.

