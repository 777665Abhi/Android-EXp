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

  -------------------------------------
12. `inline`, `noinline`, and `crossinline` functions.

- **`inline`**
  - Instructs the compiler to copy the function body **and** its lambda parameters directly into the call site at compile time, instead of allocating `Function` objects for the lambdas at runtime.
  - **Benefits:**
    - Eliminates heap allocation and call overhead for higher-order functions (e.g., `let`, `apply`, `repeat`).
    - Allows **non-local returns** (`return` inside the lambda exits the enclosing outer function).
    - Required to use `reified` generic type parameters.
  - **Caveat:** Avoid inlining large functions called from many places—it increases generated bytecode size (code bloat). Inlining functions with no lambda or `reified` parameters is usually unnecessary because the JVM JIT already optimizes regular calls.

- **`noinline`**
  - Applied to a specific lambda parameter inside an `inline` function to **opt it out** of inlining so it stays a normal `Function` object.
  - **When to use:** When you need to store the lambda in a variable/property, pass it to a non-inline function, or return it (an inlined lambda has no object representation at runtime).
  - **Effect:** Non-local `return` is not allowed inside a `noinline` lambda (use labeled returns like `return@fn`).

- **`crossinline`**
  - Applied to a lambda parameter inside an `inline` function to keep it **inlined** while **disallowing non-local `return`**.
  - **When to use:** When the lambda is invoked in a different execution context (e.g., inside a `Runnable`, listener/callback, local object, or another thread) where returning from the outer enclosing function is illegal.

Examples (Kotlin):

```kotlin
// 1. inline: body and lambda are copied directly to the call site
inline fun measureTime(block: () -> Unit) {
    val start = System.nanoTime()
    block()
    println("Elapsed: ${System.nanoTime() - start} ns")
}

fun demoInline() {
    measureTime {
        println("Running...")
        return // ✅ Non-local return: exits demoInline()
    }
}
```

```kotlin
// 2. noinline: opt a specific lambda out so it can be passed/stored as an object
fun registerCallback(cb: () -> Unit) { /* stores cb */ }

inline fun setup(
    immediate: () -> Unit,          // inlined
    noinline later: () -> Unit      // kept as a Function object
) {
    immediate()
    registerCallback(later)         // ✅ allowed because `later` is noinline
}
```

```kotlin
// 3. crossinline: still inlined, but safe to call from another execution context
inline fun runOnBackground(crossinline task: () -> Unit) {
    val runnable = Runnable {
        task() // ✅ allowed because `crossinline` forbids non-local return
    }
    Thread(runnable).start()
}

fun demoCrossinline() {
    runOnBackground {
        println("Background work")
        // return           // ❌ compiler error: non-local return not allowed
        return@runOnBackground // ✅ labeled local return is allowed
    }
}
```

Quick Comparison

| Modifier | Lambda Inlined? | Lambda Object Allocated? | Can Pass/Store Lambda? | Non-local `return`? |
| :--- | :--- | :--- | :--- | :--- |
| `inline` (default param) | Yes | No | No | Yes |
| `noinline` | No | Yes | Yes | No |
| `crossinline` | Yes | No (lambda body inlined into nested object/context) | No | No |

Practical guidance
- Use `inline` for small higher-order functions that take lambdas (or need `reified` types).
- Add `noinline` only to lambda parameters that must be stored or passed to non-inline functions.
- Add `crossinline` when an inlined lambda is called inside another object, nested lambda, or async callback.

   -------------------------------------
13. `reified` type parameters in inline functions.

- **Problem (Type Erasure):** On the JVM, generic type arguments (like `T`) are erased at runtime, so inside a normal generic function you cannot write `value is T` or `T::class.java`.
- **How `reified` solves it:** Because an `inline` function's bytecode is copied directly into the call site, the compiler knows the concrete type argument at each call site and replaces `T` with that real type.
- **Rules & Limitations:**
  - Can **only** be used with `inline` functions.
  - Cannot be called directly from Java (since Java doesn't inline Kotlin functions).
  - Common in Android for `startActivity<DetailActivity>()`, `Gson`/`Moshi` deserialization, and filtering lists by type (`filterIsInstance<T>()`).

```kotlin
inline fun <reified T> List<Any>.filterByType(): List<T> {
    val result = mutableListOf<T>()
    for (item in this) {
        if (item is T) result.add(item) // ✅ works because T is reified
    }
    return result
}

inline fun <reified T : Any> printClassName() {
    println(T::class.simpleName) // ✅ T::class accessible at runtime
}

val mixed = listOf(1, "Kotlin", 2, "Android")
val strings = mixed.filterByType<String>() // ["Kotlin", "Android"]
```

   -------------------------------------
14. Extension functions and property implementation under the hood.

- **No class modification:** Extension functions and properties do **not** add members to the target class.
- **Compiled bytecode:** A top-level extension function in `StringExt.kt` compiles into a Java `static` method inside `StringExtKt` whose **first parameter is the receiver** (`this`).
- **Static dispatch (not polymorphic):** Extensions are resolved **statically** based on the variable's declared compile-time type, not its runtime subtype. Also, if a class already has a member function with the exact same signature, the **member function always wins**.
- **Extension properties:** Compile into static getter (and setter for `var`) methods. Because no field is added to the class, extension properties **cannot have backing fields** (`field`) or initializers.

```kotlin
// Kotlin
fun String.firstWord(): String = substringBefore(" ")

val String.lastChar: Char
    get() = this[length - 1] // no backing field allowed

// Compiled Java equivalent:
// public final class StringExtKt {
//     public static String firstWord(String $this$firstWord) { ... }
//     public static char getLastChar(String $this$lastChar) { ... }
// }

// Static dispatch example:
open class Parent
class Child : Parent()
fun Parent.greet() = "Parent"
fun Child.greet() = "Child"

val obj: Parent = Child()
println(obj.greet()) // Prints "Parent" (resolved by declared type, not runtime type)
```

   -------------------------------------
15. `map` vs `flatMap` vs `compactMap` equivalents.

- **`map`**: Transforms every element **1-to-1** into another value. Output list has the **exact same size** as the input.
- **`flatMap`**: Transforms each element into an `Iterable` (0, 1, or many elements) and **flattens** all resulting collections into a single list.
- **`compactMap` (Swift) → `mapNotNull` (Kotlin)**: Transforms each element into a nullable value and **automatically filters out all `null`s**, returning a non-null `List<R>`.

```kotlin
val numbers = listOf(1, 2, 3)

// 1. map (1-to-1)
val doubled = numbers.map { it * 2 }
// [2, 4, 6]

// 2. flatMap (1-to-many + flatten)
val sentences = listOf("hello world", "kotlin rocks")
val words = sentences.flatMap { it.split(" ") }
// ["hello", "world", "kotlin", "rocks"]

// 3. mapNotNull (Kotlin equivalent of Swift's compactMap)
val raw = listOf("10", "abc", "20", null)
val parsedInts: List<Int> = raw.mapNotNull { it?.toIntOrNull() }
// [10, 20]
```

   -------------------------------------
16. Collection mutability (`List` vs `MutableList`).

- Kotlin splits collections into **read-only interfaces** (`List`, `Set`, `Map`) and **mutable interfaces** (`MutableList`, `MutableSet`, `MutableMap`).
- **`List<out E>`** exposes only read operations (`get`, `size`, `contains`) and is **covariant** (`List<Dog>` is a `List<Animal>`).
- **`MutableList<E>`** extends `List<E>` and adds write operations (`add`, `remove`, `set`, `clear`). It is **invariant** (`MutableList<Dog>` is *not* a `MutableList<Animal>`).
- **Important nuance:** `List` is *read-only*, not deeply *immutable*—if another reference still points to the same underlying `MutableList`, changes made through the mutable reference will show up in the `List` view.

```kotlin
val mutableNames: MutableList<String> = mutableListOf("Alice", "Bob")
val readOnlyView: List<String> = mutableNames // upcast to read-only view

// readOnlyView.add("Charlie") // ❌ compiler error
mutableNames.add("Charlie")
println(readOnlyView) // ["Alice", "Bob", "Charlie"] — view reflects underlying mutation!

// Defensive snapshot copy if you need isolation:
val snapshot: List<String> = mutableNames.toList()
```

   -------------------------------------
17. Scope functions (`let`, `run`, `with`, `apply`, `also`) selection criteria.

Scope functions execute a code block in the context of an object. Pick the right one using two axes: **context object reference** (`this` vs `it`) and **return value** (lambda result vs the object itself).

| Function | Object ref | Return value | Extension? | Best Use Case |
| :--- | :--- | :--- | :--- | :--- |
| `let` | `it` | Lambda result | Yes | Null-safety (`x?.let { ... }`), transforming a value, limiting variable scope |
| `run` | `this` | Lambda result | Yes (also non-ext) | Configuring an object **and** computing a result |
| `with` | `this` | Lambda result | No | Grouping calls on a known non-null object ("with this object, do…") |
| `apply` | `this` | Context object | Yes | Initializing / configuring object properties (builder style) |
| `also` | `it` | Context object | Yes | Side effects (logging, validation, debugging) without breaking a chain |

```kotlin
data class User(var name: String = "", var age: Int = 0)

// apply: configure object, returns User
val user = User().apply {
    name = "Alex"
    age = 25
}

// also: side effect, returns User
val loggedUser = user.also { println("Created: $it") }

// let: null check + transform, returns lambda result
val nullableName: String? = "Alex"
val length: Int = nullableName?.let { it.trim().length } ?: 0

// run: configure + compute result
val summary = user.run { "$name ($age)" }

// with: group calls on non-null object
val text = with(StringBuilder()) {
    append("Hello ")
    append(user.name)
    toString()
}
```

   -------------------------------------
18. Elvis operator short-circuiting mechanics.

- Syntax: `left ?: right`.
- **Short-circuiting:** Kotlin evaluates `left` **once**. If `left != null`, it immediately returns `left` and **never evaluates `right`** (no wasted computation or side effects).
- If `left == null`, `right` is evaluated and returned.
- Because `return` and `throw` are expressions of type `Nothing` in Kotlin, you can use them directly on the right-hand side for clean early exits and precondition checks.

```kotlin
fun computeExpensiveDefault(): String {
    println("Expensive computation ran!")
    return "Default"
}

val cached: String? = "CachedValue"
val result = cached ?: computeExpensiveDefault()
// Prints NOTHING — computeExpensiveDefault() is never called (short-circuited)

// Early return / throw on null:
fun greet(name: String?) {
    val validName = name ?: return // exits function if null
    val nonEmpty = validName.takeIf { it.isNotBlank() }
        ?: throw IllegalArgumentException("Name cannot be blank")
    println("Hello, $nonEmpty")
}
```

   -------------------------------------
19. `infix` notation requirements and usage.

- Allows calling a function without dots or parentheses, making DSLs and helper expressions read like natural English (e.g., `1 to "one"`, `a shl 2`, `list has item`).
- **Strict compiler requirements:**
  1. Must be a **member function** or an **extension function**.
  2. Must have **exactly one** parameter.
  3. The parameter must **not** accept `vararg` and must **not** have a default value.
- **Precedence:** Infix calls have lower precedence than arithmetic operators (`+`, `*`) and range (`..`), but higher precedence than boolean operators (`&&`, `||`, `is`, `in`).

```kotlin
infix fun Int.timesRepeat(text: String): String = text.repeat(this)

class Permissions {
    private val granted = mutableSetOf<String>()
    infix fun grant(permission: String) {
        granted.add(permission)
    }
}

val laugh = 3 timesRepeat "Ha" // "HaHaHa"
val perms = Permissions()
perms grant "CAMERA"           // same as perms.grant("CAMERA")
val pair = "key" to 42         // standard library infix function
```

   -------------------------------------
20. Destructuring declarations (`component1()`, `component2()`).

- Unpacks a single object into multiple local variables in one statement: `val (a, b) = obj`.
- Under the hood, Kotlin compiles this to:
  ```kotlin
  val a = obj.component1()
  val b = obj.component2()
  ```
- **Positional, not by name:** `componentN()` maps strictly by declaration order (e.g., primary constructor property order in a `data class`), NOT by variable name—so swapping variable names silently swaps values!
- Use `_` to skip unused components (the skipped `componentN()` is not even called). Works in `for ((k, v) in map)` loops and lambda parameters too.

```kotlin
data class Coordinates(val lat: Double, val lng: Double, val altitude: Double)

val loc = Coordinates(28.61, 77.20, 216.0)
val (lat, _, alt) = loc // calls component1() and component3() only

// In loops & lambdas:
val scores = mapOf("Alice" to 95, "Bob" to 88)
for ((name, score) in scores) {
    println("$name -> $score")
}
```

   -------------------------------------
21. `const val` vs `val`.

- **`val` (runtime read-only):** Value is assigned once at runtime (can come from a function call, constructor parameter, or expression). Accessing it goes through a getter method (`getX()`).
- **`const val` (compile-time constant):** Value is known at compile time and **inlined directly** at every usage site in bytecode (like Java `public static final` constants).
- **Rules for `const val`:**
  - Only allowed at **top-level**, inside an **`object`**, or inside a **`companion object`**.
  - Only **primitive types** (`Int`, `Long`, `Boolean`, etc.) and **`String`** are allowed.
  - Cannot have a custom getter.
- **Library warning:** Because `const val` values are copied into caller bytecode at compile time, changing a `const val` in a library requires recompiling callers to see the new value.

```kotlin
object ApiConfig {
    const val BASE_URL = "https://api.example.com" // compile-time constant (inlined)
    const val TIMEOUT_MS = 5_000L

    val startupTime = System.currentTimeMillis()   // runtime val (cannot be const)
}

// Required in annotations (only compile-time constants allowed):
@Deprecated(ApiConfig.BASE_URL)
fun oldEndpoint() {}
```

   -------------------------------------
22. Delegated properties (`by`) and custom delegates (`ReadOnlyProperty`, `ReadWriteProperty`).

- Property delegation (`val/var prop by Delegate()`) forwards `get()` (and `set()`) calls to a separate delegate object via `getValue(...)` / `setValue(...)`, keeping reusable property logic out of classes.
- **Built-in standard delegates:**
  - `by lazy { ... }` — computes once on first access (thread-safe `SYNCHRONIZED` by default).
  - `Delegates.observable(initial) { prop, old, new -> ... }` — runs callback on every change.
  - `Delegates.vetoable(initial) { prop, old, new -> Boolean }` — rejects updates when lambda returns `false`.
  - `by map` — reads property values from a `Map<String, Any?>`.
- **Custom delegates:** Implement `ReadOnlyProperty<Owner, T>` (for `val`) or `ReadWriteProperty<Owner, T>` (for `var`).

```kotlin
import kotlin.properties.ReadWriteProperty
import kotlin.reflect.KProperty

class TrimmedStringDelegate : ReadWriteProperty<Any?, String> {
    private var stored = ""

    override fun getValue(thisRef: Any?, property: KProperty<*>): String = stored

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        stored = value.trim()
    }
}

class Profile {
    var username: String by TrimmedStringDelegate()
    val heavyResource: String by lazy { "Loaded once" }
}

val p = Profile()
p.username = "   kotlin_dev   "
println(p.username) // "kotlin_dev"
```

   -------------------------------------
23. Backing fields (`field`) vs backing properties.

- **Backing field (`field`):**
  - Automatically generated by Kotlin when a property uses the default getter/setter or references the special `field` identifier inside a custom getter/setter.
  - Prevents infinite recursion (writing `prop = value` inside `set(value)` calls the setter recursively until `StackOverflowError`; writing `field = value` writes directly to memory).
  - No backing field is generated if a property is purely computed (e.g., `val isEmpty get() = size == 0`).
- **Backing property:**
  - A manually declared `private` property (commonly prefixed with `_`) that holds mutable/internal state, paired with a public read-only property exposing a narrower type (very common in Android ViewModels with `StateFlow`/`LiveData`).

```kotlin
class CounterViewModel {
    // 1. Backing field example:
    var maxLimit: Int = 10
        set(value) {
            field = value.coerceAtLeast(0) // uses `field`, NOT `maxLimit`
        }

    // 2. Backing property example:
    private val _items = mutableListOf<String>() // private mutable
    val items: List<String>                      // public read-only
        get() = _items
}
```

   -------------------------------------
24. `sealed class` vs `sealed interface`.

- Both define **restricted hierarchies**: all direct subclasses/implementations must be in the **same package and same module**, allowing the compiler to enforce **exhaustive `when`** expressions without an `else` branch.
- **`sealed class`:**
  - Can hold **constructor parameters and state** shared by all subclasses.
  - A subclass can inherit from only **one** class (single inheritance).
  - Can have `init` blocks and `protected` members.
- **`sealed interface`:**
  - Supports **multiple inheritance** (a class can belong to multiple sealed hierarchies at once).
  - Can be implemented by `enum class`es and `value class`es (which cannot extend a `sealed class`).
  - No constructor state (only abstract properties/methods or default methods).

```kotlin
// sealed interface: allows grouping across hierarchies & enums
sealed interface UiError
sealed interface NetworkError : UiError
sealed interface AuthError : UiError

data object NoInternet : NetworkError
data object TokenExpired : NetworkError, AuthError // ✅ multiple sealed interfaces!

// sealed class: when subclasses share common constructor state
sealed class LoadState(val timestamp: Long = System.currentTimeMillis()) {
    data object Loading : LoadState()
    data class Success(val data: String) : LoadState()
    data class Failure(val error: UiError) : LoadState()
}
```

   -------------------------------------
25. Operator overloading syntax and common use cases.

- Kotlin maps standard operators (`+`, `-`, `*`, `[]`, `()`, `in`, `<`, `+=`) to fixed function names marked with the **`operator`** modifier.
- You can define operator functions as either **member functions** or **extension functions**.
- **Common mappings:**
  - `a + b` → `a.plus(b)`, `a - b` → `a.minus(b)`
  - `a[i]` → `a.get(i)`, `a[i] = v` → `a.set(i, v)`
  - `a(x)` → `a.invoke(x)` (great for UseCase classes in Clean Architecture!)
  - `a < b` → `a.compareTo(b) < 0`
  - `x in a` → `a.contains(x)`

```kotlin
data class Money(val cents: Int) : Comparable<Money> {
    operator fun plus(other: Money) = Money(cents + other.cents)
    operator fun times(multiplier: Int) = Money(cents * multiplier)
    override operator fun compareTo(other: Money) = cents.compareTo(other.cents)
}

class GetUserUseCase {
    operator fun invoke(id: Int): String = "User#$id"
}

val total = Money(500) + Money(250) * 2 // Money(1000)
val useCase = GetUserUseCase()
println(useCase(7)) // Calls useCase.invoke(7) -> "User#7"
```

   -------------------------------------
26. Unchecked vs checked exceptions in Kotlin vs Java.

- **In Java:**
  - **Checked exceptions** (subclasses of `Exception`, excluding `RuntimeException`, e.g., `IOException`, `SQLException`) **must** be either caught in a `try/catch` or declared in the method signature with `throws`.
  - **Unchecked exceptions** (`RuntimeException` and `Error`, e.g., `NullPointerException`, `IllegalStateException`) do not require declaration or catching.
- **In Kotlin:**
  - **All exceptions are unchecked.** Kotlin intentionally dropped checked exceptions because in practice they often lead to empty catch blocks, noisy boilerplate, and poor composability with lambdas/higher-order functions (`map`, `filter`, etc.).
  - **Java interop (`@Throws`):** If Java code calls a Kotlin function that throws a checked Java exception (like `IOException`), annotate the Kotlin function with `@Throws(IOException::class)` so the Java compiler allows Java callers to catch it.

```kotlin
import java.io.File
import java.io.IOException

// No `throws IOException` needed in Kotlin; @Throws is only for Java callers
@Throws(IOException::class)
fun readConfig(path: String): String {
    return File(path).readText() // may throw IOException
}

fun caller() {
    // Kotlin does NOT force try/catch, though you can still handle it explicitly:
    val content = try {
        readConfig("config.json")
    } catch (e: IOException) {
        "{}"
    }
}
```

   -------------------------------------
27. `@JvmStatic`, `@JvmOverloads`, `@JvmField`, and `@JvmName` annotations.

These annotations control how Kotlin declarations are exposed in JVM bytecode for **Java interoperability**:

- **`@JvmStatic`**: Exposes a method/property inside a `companion object` or `object` as a true Java `static` member (`Util.parse()` instead of `Util.Companion.parse()`).
- **`@JvmOverloads`**: Generates multiple overloaded Java methods for a Kotlin function/constructor that uses default parameter values (since Java lacks default parameters).
- **`@JvmField`**: Exposes a Kotlin property as a direct public JVM field **without** generating `getX()` / `setX()` accessors (cannot be used on `private`, `override`, `open`, `const`, or delegated properties).
- **`@JvmName`**: Changes the compiled JVM method name or file class name—useful to resolve type-erasure signature clashes (e.g., `List<Int>.sum()` vs `List<String>.sum()`) or make cleaner Java names.

```kotlin
@file:JvmName("StringHelpers") // Java calls StringHelpers.foo() instead of FileNameKt.foo()

class ApiClient @JvmOverloads constructor(
    val baseUrl: String,
    val timeoutMs: Long = 5_000L,
    val retries: Int = 3
) {
    @JvmField
    val version = "1.0" // Java reads client.version directly (no getVersion())

    companion object {
        @JvmStatic
        fun createDefault() = ApiClient("https://api.example.com")
        // Java calls ApiClient.createDefault()
    }
}

// Avoiding JVM signature clash caused by generic type erasure:
@JvmName("filterStrings")
fun List<String>.filterValid(): List<String> = filter { it.isNotBlank() }

@JvmName("filterInts")
fun List<Int>.filterValid(): List<Int> = filter { it > 0 }
```

   -------------------------------------
28. Primary vs secondary constructors and init blocks.

- **Primary constructor:** Declared in the class header (`class User(val name: String, age: Int)`). It cannot contain executable code directly, but its parameters can initialize properties and be used inside `init` blocks.
- **`init` blocks:** Initialization code executed as part of the primary constructor. If a class has multiple property initializers and `init` blocks, they run **strictly in top-to-bottom order** in the class body.
- **Secondary constructors:** Declared inside the class body with `constructor(...)`.
  - If the class has a primary constructor, every secondary constructor **must delegate** to it directly (`: this(...)`) or indirectly via another secondary constructor.
  - **Execution order:** Primary constructor + property initializers & `init` blocks run **first**, and the secondary constructor's body runs **after** them.
  - In idiomatic Kotlin, default parameter values often replace the need for secondary constructors.

```kotlin
class Order(val id: String, val amount: Double) {
    val label = "Order-$id".also { println("1. Property initializer") }

    init {
        println("2. init block: id=$id, amount=$amount")
        require(amount >= 0) { "Amount must be non-negative" }
    }

    // Secondary constructor delegates to primary constructor
    constructor(id: String) : this(id, 0.0) {
        println("3. Secondary constructor body")
    }
}

// Order("A1") prints:
// 1. Property initializer
// 2. init block: id=A1, amount=0.0
// 3. Secondary constructor body
```

   -------------------------------------
29. `enum class` vs `sealed class`.

Both restrict a type to a fixed, compile-time-known set of possibilities (supporting exhaustive `when` expressions), but they model different things:

| Feature | `enum class` | `sealed class` |
| :--- | :--- | :--- |
| Instances per entry | **Single constant instance** (singleton per entry) | **Multiple instances** with different data (`data class`) or singletons (`data object`) |
| Per-variant properties | Every enum entry must have the **same** constructor shape | Each subclass can have **completely different** properties & methods |
| Built-in helpers | `entries`, `valueOf()`, `name`, `ordinal` | No built-in `entries` / `ordinal` |
| Best for | Fixed constants (days, log levels, HTTP methods, tabs) | Rich states & ADTs (UI state: `Loading`, `Success(items)`, `Error(msg, code)`) |

```kotlin
// enum class: fixed singletons with identical shape
enum class HttpMethod(val hasBody: Boolean) {
    GET(false), POST(true), DELETE(false)
}

// sealed class: different data shapes & multiple instances per state
sealed class UiState {
    data object Loading : UiState()
    data class Success(val items: List<String>) : UiState()
    data class Error(val code: Int, val message: String) : UiState()
}
```

   -------------------------------------
30. Value classes (`@JvmInline value class`).

- A **value class** wraps a **single** `val` property to give you **domain-level type safety** without paying the heap-allocation cost of creating a wrapper object at runtime.
- At compile time, the Kotlin compiler treats `UserId` and `OrderId` as completely separate types (preventing accidental argument swaps).
- At runtime on the JVM, the compiler **inlines (unboxes)** it back to the underlying type (`String` / `Long`) wherever possible.
- **When boxing still happens:** When used as a nullable type (`UserId?`), as a generic type argument (`List<UserId>`), or as an interface/supertype (`Any`).
- **Rules:** Must have `@JvmInline` on JVM, exactly one `val` property in primary constructor, cannot have backing fields on extra properties (computed properties & methods are allowed), and cannot inherit from classes (can implement interfaces).

```kotlin
@JvmInline
value class UserId(val raw: String) {
    init {
        require(raw.isNotBlank()) { "UserId cannot be blank" }
    }
}

@JvmInline
value class OrderId(val raw: String)

fun fetchOrder(userId: UserId, orderId: OrderId) {
    println("Fetching ${orderId.raw} for ${userId.raw}")
}

// fetchOrder(OrderId("o1"), UserId("u1")) // ❌ compile error — type safe!
fetchOrder(UserId("u1"), OrderId("o1"))    // ✅ compiles to fetchOrder(String, String) under the hood
```

   -------------------------------------
31. `typealias` keyword usage and limitations.

- Gives an existing type a shorter, more readable name—especially useful for complex function types (`(Int, String) -> Boolean`) or nested generic types (`Map<String, List<User>>`).
- **Important limitations:**
  - **No new type is created:** `typealias` is purely a compile-time synonym. The compiler treats `UserId` and `String` as identical, so it provides **zero** extra type safety (use `@JvmInline value class` when you need type safety).
  - **Top-level only:** Cannot be declared inside a class, object, or function.
  - Cannot add new methods/properties or `init` validation (though extension functions on the alias actually extend the underlying type).

```kotlin
typealias ClickHandler = (position: Int, id: String) -> Unit
typealias UserMap = Map<String, List<String>>
typealias Email = String

fun sendEmail(to: Email) = println("Sending to $to")

val plainText: String = "not-even-checked"
sendEmail(plainText) // ✅ compiles! typealias gives readability, NOT type safety
```

   -------------------------------------
32. Functional interfaces (`fun interface` / SAM conversion).

- A **functional interface** (also called a **SAM — Single Abstract Method** interface) has **exactly one** abstract method (it may have any number of non-abstract/default methods).
- Marking a Kotlin interface with `fun interface` enables **SAM conversion**: callers can pass a concise lambda `{ ... }` instead of writing a verbose anonymous `object : MyInterface { ... }`.
- Note: Java SAM interfaces automatically support SAM conversion in Kotlin without `fun`, whereas Kotlin interfaces **require** the `fun` keyword to enable it.

```kotlin
fun interface Validator {
    fun isValid(input: String): Boolean
}

// Without SAM conversion:
val oldWay = object : Validator {
    override fun isValid(input: String) = input.length >= 6
}

// With `fun interface` SAM conversion:
val minLengthValidator = Validator { it.length >= 6 }

fun checkInput(text: String, validator: Validator): Boolean = validator.isValid(text)

val ok = checkInput("kotlin123") { it.isNotBlank() } // pass lambda directly!
```

   -------------------------------------
33. Generics variance: covariance (`out`), contravariance (`in`), invariance.

Variance controls how subtyping of generic types relates to subtyping of their type arguments (e.g., if `Dog` is a subtype of `Animal`, is `Box<Dog>` a subtype of `Box<Animal>`?):

- **Invariance (`class Box<T>`) — default:**
  - `Box<Dog>` is **neither** a subtype nor a supertype of `Box<Animal>`.
  - Needed when `T` is both read and written (e.g., `MutableList<T>`). If `MutableList<Dog>` were treated as `MutableList<Animal>`, you could insert a `Cat` into a list of `Dog`s!
- **Covariance (`out T` — Producer):**
  - `Producer<Dog>` **is** a subtype of `Producer<Animal>` (preserves direction).
  - `T` can only appear in **`out` (return)** positions—you can *read/produce* `T`, never *consume/write* `T` (e.g., `List<out E>`).
- **Contravariance (`in T` — Consumer):**
  - `Consumer<Animal>` **is** a subtype of `Consumer<Dog>` (reverses direction).
  - `T` can only appear in **`in` (parameter)** positions—you can *consume/accept* `T`, never *return* `T` (e.g., `Comparable<in T>`).
- **Memory trick (PECS):** **P**roducer = `out` (`extends`), **C**onsumer = `in` (`super`).

```kotlin
open class Animal
class Dog : Animal()

// Covariant (Producer): only returns T
interface Shelter<out T> {
    fun adopt(): T
}

// Contravariant (Consumer): only takes T
interface Vet<in T> {
    fun treat(patient: T)
}

val dogShelter: Shelter<Dog> = object : Shelter<Dog> { override fun adopt() = Dog() }
val animalShelter: Shelter<Animal> = dogShelter // ✅ covariant (`out`)

val animalVet: Vet<Animal> = object : Vet<Animal> { override fun treat(patient: Animal) {} }
val dogVet: Vet<Dog> = animalVet                // ✅ contravariant (`in`)
```

   -------------------------------------
34. Star projection (`*`) in generics.

- Used when you know nothing (or don't care) about a generic type argument, similar to Java's raw/wildcard type `<?>`, while remaining **type-safe**.
- **How `*` behaves depending on variance:**
  - For `Foo<out T : UpperBound>`: `Foo<*>` acts like `Foo<out UpperBound>` (safe to read as `UpperBound`, or `Any?` if unbounded).
  - For `Foo<in T>`: `Foo<*>` acts like `Foo<in Nothing>` (you cannot safely write anything into it).
  - For invariant `Foo<T : UpperBound>`: `Foo<*>` reads as `UpperBound` (`Any?`) and writes as `Nothing` (read-only safely).
- Commonly used in `is` checks (`if (obj is List<*>)`) because of type erasure.

```kotlin
fun printAll(items: List<*>) {
    for (item: Any? in items) { // safe to read as Any?
        println(item)
    }
}

fun inspect(data: Any) {
    if (data is MutableList<*>) {
        println("Size: ${data.size}")
        // data.add("hello") // ❌ compile error: cannot write to MutableList<*>
    }
}
```

   -------------------------------------
35. Type erasure in Kotlin/Java generics.

- Generic type arguments are enforced by the compiler at **compile time** and **erased at runtime** on the JVM.
- At runtime, `List<String>` and `List<Int>` are both just raw `List`—the JVM has no idea what type argument was originally supplied.
- **Practical consequences:**
  - You cannot do `if (value is List<String>)` at runtime (use `if (value is List<*>)` instead).
  - Casting `value as List<String>` produces an **unchecked cast** warning and only fails later when an element of the wrong type is accessed (`ClassCastException`).
  - Two overloads differing only by generic type (`fun f(a: List<Int>)` and `fun f(a: List<String>)`) clash in bytecode unless disambiguated with `@JvmName`.
- **Workaround:** Use `inline fun <reified T>` when you need the actual type `T` at runtime.

```kotlin
fun process(input: Any) {
    // if (input is List<String>) {} // ❌ Cannot check for instance of erased type
    if (input is List<*>) {          // ✅ Star projection works at runtime
        println("It's a list of ${input.size} elements")
    }
}
```

   -------------------------------------
36. `Nothing` type vs `Unit` vs `Any`.

- **`Any` (Top of the non-null hierarchy):**
  - Every non-nullable type in Kotlin inherits from `Any` (analogous to Java's `Object`, providing `equals`, `hashCode`, `toString`).
  - `Any?` is the true root of **all** types (including nullable ones).
- **`Unit` (Normal return with no useful value):**
  - Equivalent to Java's `void`, except `Unit` is a **real singleton object** (`object Unit`).
  - Because it's a real type, generics like `() -> Unit` or `Result<Unit>` work cleanly without needing a special `Void` wrapper.
- **`Nothing` (Bottom of the hierarchy — "never returns"):**
  - Has **no instances**. A function returning `Nothing` never completes normally (it always throws an exception or loops infinitely).
  - Because `Nothing` is a **subtype of every type**, expressions like `throw` and `return` can appear anywhere (e.g., on the right side of `?:` or in a `when` branch) without breaking type inference.
  - `Nothing?` has exactly one valid value: `null` (`val x = null` has type `Nothing?`).

```kotlin
fun logMessage(msg: String): Unit { // returns normally, no meaningful value
    println(msg)
}

fun fail(reason: String): Nothing { // never returns normally
    throw IllegalStateException(reason)
}

fun getUsername(input: String?): String {
    // Works because `fail(...)` has type `Nothing`, which is a subtype of `String`
    return input ?: fail("Username missing")
}
```

   -------------------------------------
37. Structural equality (`==`) vs Referential equality (`===`).

- **Structural equality (`==` and `!=`):**
  - Compares **values/contents** by calling `a?.equals(b) ?: (b === null)` (null-safe!).
  - For `data class` and `String`, `equals()` compares property values / characters. For plain classes that don't override `equals()`, it falls back to reference equality.
- **Referential equality (`===` and `!==`):**
  - Checks whether two references point to the **exact same object in memory** (like Java's `==` for objects).
  - Note: For primitive values like `Int`, `===` compares values (unless boxed—and small `Int`s from `-128..127` are cached by the JVM!).

```kotlin
data class Point(val x: Int, val y: Int)

val p1 = Point(1, 2)
val p2 = Point(1, 2)
val p3 = p1

println(p1 == p2)  // true  (same values)
println(p1 === p2) // false (two distinct objects in memory)
println(p1 === p3) // true  (same reference)

val nullPoint: Point? = null
println(nullPoint == p1) // false (null-safe — no NullPointerException!)
```

   -------------------------------------
38. Range expressions and progression loops (`step`, `downTo`, `until`).

- **Ranges (`..`, `..<`, `until`):** Represent a closed or half-open interval of `Comparable` values (numbers, `Char`, etc.).
- **Progressions (`step`, `downTo`):** Sequences with a start, end, and step size that can be iterated over cleanly.
  - `1..5` → `1, 2, 3, 4, 5` (inclusive of both ends).
  - `1 until 5` / `1..<5` (Kotlin 1.9+) → `1, 2, 3, 4` (excludes upper bound; ideal for array/list indices `0..<list.size`).
  - `5 downTo 1` → `5, 4, 3, 2, 1` (descending; note `5..1` is empty!).
  - `step n` → jumps by `n` (`n` must always be positive, even with `downTo`).
- **Performance tip:** Simple `for (i in 0..<n)` loops are compiled by Kotlin into standard zero-allocation JVM `for` loops (no `IntRange` object created).

```kotlin
for (i in 1..5) print("$i ")              // 1 2 3 4 5
for (i in 1..<5) print("$i ")             // 1 2 3 4 (modern replacement for `until`)
for (i in 10 downTo 2 step 2) print("$i ") // 10 8 6 4 2

val letter = 'k'
if (letter in 'a'..'z') println("Lowercase letter") // fast range membership check
```

   -------------------------------------
39. `Sequence` vs `Iterable` processing (lazy vs eager evaluation).

- **`Iterable` (`List`, `Set`) — Eager (horizontal, step-by-step):**
  - Each operation (`filter`, `map`) runs across **all elements immediately** and allocates a brand-new intermediate `List` before passing it to the next step.
  - Best for small collections or single operations (less per-element abstraction overhead, and operations like `filter`/`map` are `inline`).
- **`Sequence` — Lazy (vertical, element-by-element):**
  - Intermediate operations (`filter`, `map`) do **nothing** until a terminal operation (`toList()`, `first()`, `count()`, `sum()`) is called.
  - Each element flows through the entire chain one by one with **zero intermediate collections**, and stops early as soon as the terminal condition is satisfied.
  - Best for **large collections**, multi-step pipelines, infinite streams (`generateSequence`), or short-circuiting (`first`, `take`, `any`).

```kotlin
val numbers = (1..1_000_000).toList()

// Eager (Iterable): filters ALL 1,000,000 items, maps ALL matches, then takes first
val eagerFirst = numbers
    .filter { it % 2 == 0 }
    .map { it * 10 }
    .first()

// Lazy (Sequence): checks 1 (fails), checks 2 (passes -> 20 -> done immediately!)
val lazyFirst = numbers.asSequence()
    .filter { it % 2 == 0 }
    .map { it * 10 }
    .first() // Only inspects elements 1 and 2!
```

   -------------------------------------
40. Smart casts and `is` / `!is` operators.

- `is` checks if an object is an instance of a type (like Java's `instanceof`); `!is` checks the opposite.
- **Smart cast:** Once the compiler verifies a type check (`if (x is String)` or early exit `if (x !is String) return`), it automatically casts `x` to `String` within that scope—no manual `as String` needed.
- **When smart cast fails:** The compiler must guarantee the value **cannot change** between the check and its use. Smart casts do **not** work on:
  - Mutable class properties (`var` properties — another thread could modify them).
  - Properties with custom getters or delegated properties (`val` with `get()` can return a different object each call).
  - `open val` properties in non-final classes (a subclass could override the getter).
  - Local `var` variables captured and mutated inside a lambda.
- **Workaround:** Copy the property into a local `val` (or use `?.let`) before checking.

```kotlin
fun describe(obj: Any?) {
    if (obj !is String) return
    // Smart-cast to non-null String from here on:
    println("Length is ${obj.length}")
}

class Holder(var data: Any) {
    fun test() {
        // if (data is String) println(data.length) // ❌ cannot smart-cast mutable property
        val local = data
        if (local is String) println(local.length)  // ✅ local val can be smart-cast
    }
}
```

   -------------------------------------
41. Custom getters and setters implementation rules.

- Every `val` has a getter (`get()`); every `var` has both a getter (`get()`) and a setter (`set(value)`). You can customize either right below the property declaration.
- **Key rules:**
  1. Use **`field`** inside custom accessors to access the backing storage—referring to the property's own name inside its getter/setter causes infinite recursion (`StackOverflowError`).
  2. A **`val`** cannot have a setter.
  3. **Getter visibility** must match the property's visibility; **setter visibility** can be **narrowed** (e.g., `private set`), which is great for encapsulating state.
  4. If a custom getter computes a value without referencing `field`, **no backing field is generated** (zero extra memory per instance), and you cannot provide an initializer (`= ...`).

```kotlin
class Rectangle(val width: Int, val height: Int) {
    // Computed property — no backing field generated
    val area: Int
        get() = width * height

    // Custom setter with validation using `field`
    var label: String = "Untitled"
        set(value) {
            field = value.trim().ifEmpty { "Untitled" }
        }

    // Public read, private write
    var renderCount: Int = 0
        private set

    fun render() {
        renderCount++
    }
}
```

   -------------------------------------
42. Kotlin reflection (`KClass`, `KProperty`, `KFunction`).

- Reflection allows inspecting and invoking classes, properties, and functions **dynamically at runtime** (requires `kotlin-reflect` dependency for full features beyond basic `KClass`).
- **Core references:**
  - `KClass<T>` (`User::class` or `obj::class`): inspect class name, constructors, properties, annotations, sealed subclasses.
  - `KProperty` (`User::name` or `user::name`): read/write property values dynamically via `.get()` / `.set()`, inspect name & annotations.
  - `KFunction` (`::greet` or `String::uppercase`): reference a function as a value and call it via `.invoke()` / `.call()`.
- **Trade-off:** Powerful for serialization, DI frameworks, and logging, but **slower** than direct calls and increases APK size (`kotlin-reflect` adds ~2.5 MB before R8 shrinking)—prefer compile-time codegen (KSP) in Android when possible.

```kotlin
data class User(val id: Int, var name: String)

fun isEven(n: Int) = n % 2 == 0

val kClass = User::class
println(kClass.simpleName) // "User"

val user = User(1, "Alex")
val nameProp = User::name
println(nameProp.get(user)) // "Alex"
nameProp.set(user, "Jordan")

val predicate = ::isEven // KFunction1<Int, Boolean>
println(listOf(1, 2, 3, 4).filter(predicate)) // [2, 4]
```

   -------------------------------------
43. Non-local returns in lambdas and `inline` interaction.

- A **local return** (`return@label`) exits only the lambda itself.
- A **non-local return** (bare `return`) exits the **nearest enclosing named or anonymous function**.
- **Why `inline` is required:**
  - A regular (non-inline) lambda is compiled into a separate `Function` object whose `invoke()` runs in its own stack frame—so a bare `return` inside it has no way to return from the outer caller, and Kotlin forbids it.
  - An **`inline`** function copies the lambda body directly into the outer function's bytecode, making a bare `return` completely valid and natural (this is why `forEach { if (it == 0) return }` exits the enclosing function).
- If an `inline` function passes a lambda into a nested execution context (like `Runnable`), mark it `crossinline` to forbid non-local returns.

```kotlin
fun findFirstNegative(nums: List<Int>): Int? {
    // forEach is `inline`, so bare `return` exits findFirstNegative()
    nums.forEach { n ->
        if (n < 0) return n // ✅ non-local return
    }
    return null
}

fun skipZeros(nums: List<Int>) {
    nums.forEach { n ->
        if (n == 0) return@forEach // ✅ local return (acts like `continue`)
        println(n)
    }
}
```

   -------------------------------------
44. Top-level functions and properties compiled representation.

- Kotlin lets you declare functions and properties directly in a `.kt` file without wrapping them in a utility class (`MathUtils`).
- **Under the hood (JVM bytecode):** The Kotlin compiler generates a `public final class` named `<FileName>Kt` (e.g., `DateUtils.kt` → `DateUtilsKt`) and puts all top-level functions and properties inside it as `static` methods and fields.
- **Customizing the generated class:**
  - `@file:JvmName("DateUtils")` changes the Java class name from `DateUtilsKt` to `DateUtils`.
  - `@file:JvmMultifileClass` lets multiple `.kt` files merge their top-level members into a single Java facade class.

```kotlin
// File: MathUtils.kt
@file:JvmName("MathUtils")

const val PI_APPROX = 3.14159
fun square(x: Int): Int = x * x

// From Kotlin:
// val s = square(5)

// From Java (thanks to @file:JvmName):
// int s = MathUtils.square(5);
// (Without @file:JvmName, Java would call MathUtilsKt.square(5))
```

   -------------------------------------
45. Standard library functions: `takeIf`, `takeUnless`.

- Extension functions on any type `T` that act like a single-value conditional filter, making nullable chains clean and expressive:
  - **`obj.takeIf { predicate }`**: Returns `obj` if `predicate(obj)` is **`true`**, otherwise returns **`null`**.
  - **`obj.takeUnless { predicate }`**: Returns `obj` if `predicate(obj)` is **`false`**, otherwise returns **`null`**.
- Remember that both return a **nullable `T?`**, so pair them with `?.` or `?:`.

```kotlin
fun normalizeQuery(raw: String): String {
    return raw.trim()
        .takeIf { it.isNotEmpty() } // returns trimmed string if non-empty, else null
        ?: "default"
}

val age = 20
val validAdultAge: Int? = age.takeUnless { it < 18 } // 20 (since 20 < 18 is false)

// Great in safe-call chains:
val displayName = "  Alice  ".trim().takeIf { it.length >= 3 }?.uppercase() // "ALICE"
```

   -------------------------------------
46. Array types in Kotlin (`IntArray`, `Array<Int>`) performance differences.

- **`Array<Int>`** compiles to Java's `Integer[]`—every single element is a **boxed `Integer` object** on the heap (16 bytes per object + 4/8 byte reference), causing extra memory usage, pointer chasing, and GC pressure.
- **`IntArray`** compiles directly to Java's primitive **`int[]`**—values sit contiguously in memory (just 4 bytes per `Int`), giving dramatically better CPU cache locality and zero boxing overhead.
- Kotlin provides primitive array classes for all primitive types: `ByteArray`, `ShortArray`, `IntArray`, `LongArray`, `FloatArray`, `DoubleArray`, `BooleanArray`, `CharArray`.
- **Rule of thumb:** Always prefer primitive arrays (`IntArray`, `ByteArray`, etc.) over `Array<Int>` for numeric/binary data, image pixels, buffers, and performance-sensitive loops.

```kotlin
val boxed: Array<Int> = arrayOf(1, 2, 3)    // JVM: Integer[] (heap objects)
val primitive: IntArray = intArrayOf(1, 2, 3) // JVM: int[] (contiguous primitives)

// Constructing large arrays efficiently:
val squares = IntArray(10_000) { i -> i * i }
```

   -------------------------------------
47. `@Volatile` and `@Transient` annotations.

Kotlin provides JVM annotations corresponding to Java's `volatile` and `transient` keywords:

- **`@Volatile` (Thread visibility):**
  - Applied to a `var` property (backed by a field).
  - Guarantees that writes to the field are flushed immediately to **main memory** and reads always fetch the latest value from main memory (not a stale CPU core cache), plus prevents instruction reordering around it.
  - **Important caveat:** `@Volatile` ensures **visibility**, NOT **atomicity**! Compound operations like `count++` (read-modify-write) are still NOT thread-safe—use `AtomicInteger`, `Mutex`, or `synchronized` for atomic updates.
- **`@Transient` (Skip serialization):**
  - Marks a property so Java's default `Serializable` mechanism skips it during serialization/deserialization (also respected by Gson; for `kotlinx.serialization`, use `@kotlinx.serialization.Transient`).

```kotlin
import java.io.Serializable

class SessionManager : Serializable {
    @Volatile
    var isRunning: Boolean = true // safely visible across threads (e.g., stop flag)

    @Transient
    var cachedSocket: Any? = null // ignored when serialized to disk/bundle
}
```

   -------------------------------------
48. String templates and raw multiline strings.

- **String templates:** Embed variables (`$varName`) or arbitrary expressions (`${expr}`) directly inside strings—cleaner and safer than string concatenation (compiles to `StringBuilder` / `invokedynamic` `StringConcatFactory` on modern JVM).
- **Raw (multiline) strings (`"""..."""`):**
  - Preserve newlines and special characters (`\n`, `\`, `"`) literally without escape sequences.
  - Still support string templates (`$x`). To print a literal `$` inside a raw string, use `${'$'}` (or `$$"""` multi-dollar interpolation in Kotlin 2.1+).
  - Pair with **`.trimIndent()`** (strips common leading whitespace) or **`.trimMargin()`** (strips up to `|` prefix).

```kotlin
val user = "Alex"
val items = listOf("A", "B")
println("Hello $user, you have ${items.size} items.") // String template

// Raw multiline string (great for SQL, JSON, regex):
val json = """
    {
        "user": "$user",
        "count": ${items.size},
        "regex": "\d+\s+\w+"
    }
""".trimIndent()
```

   -------------------------------------
49. `tailrec` modifier and tail recursion optimization.

- Deep recursion normally grows the call stack with every recursive call until it crashes with **`StackOverflowError`**.
- When a function is **tail-recursive**—meaning the recursive call is the **very last operation** it performs (no extra multiplication, addition, or `try/catch` wrapping it)—adding **`tailrec`** instructs the Kotlin compiler to rewrite it into a flat, zero-stack **`while` loop** in bytecode.
- **Rules:**
  - Recursive call must be in tail position (not inside `try`/`catch`/`finally`, and not on an `open` function).
  - If the function isn't truly tail-recursive, the compiler emits a warning and skips optimization.

```kotlin
// ❌ NOT tail-recursive (multiplication happens AFTER the recursive call returns):
// fun factBad(n: Long): Long = if (n <= 1) 1 else n * factBad(n - 1)

// ✅ Tail-recursive using an accumulator — compiled into a fast loop!
tailrec fun factorial(n: Long, acc: Long = 1L): Long {
    return if (n <= 1L) acc else factorial(n - 1L, acc * n)
}

tailrec fun gcd(a: Int, b: Int): Int {
    return if (b == 0) a else gcd(b, a % b)
}
```

   -------------------------------------
50. Contracts API (`contract { ... }`).

- Lets a function explicitly **describe its runtime guarantees to the Kotlin compiler** so the compiler can perform smarter flow analysis at call sites.
- **Two main contract capabilities:**
  1. **`returns(value) implies (condition)`**: Tells the compiler about nullability/type checks (e.g., Kotlin stdlib's `isNullOrEmpty()` promises that if it returns `false`, the receiver is non-null).
  2. **`callsInPlace(lambda, InvocationKind)`**: Tells the compiler how many times a lambda executes (`EXACTLY_ONCE`, `AT_MOST_ONCE`, `AT_LEAST_ONCE`), enabling definite initialization of `val` variables inside lambdas (used by `let`, `run`, `apply`, `synchronized`).
- **Caution:** The compiler **trusts** your contract without verifying it—writing a false contract can cause runtime `NullPointerException` or `ClassCastException`!

```kotlin
import kotlin.contracts.ExperimentalContracts
import kotlin.contracts.InvocationKind
import kotlin.contracts.contract

@OptIn(ExperimentalContracts::class)
fun String?.isValidEmail(): Boolean {
    contract {
        returns(true) implies (this@isValidEmail != null)
    }
    return this != null && contains("@")
}

@OptIn(ExperimentalContracts::class)
inline fun runOnce(block: () -> Unit) {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
}

fun demo(input: String?) {
    if (input.isValidEmail()) {
        println(input.length) // ✅ smart-cast to non-null String thanks to contract!
    }
    val x: Int
    runOnce { x = 42 }        // ✅ compiler knows `x` is initialized exactly once
    println(x)
}
```

   -------------------------------------
51. How Kotlin handles default arguments in Java interop.

- Kotlin supports default parameter values (`fun connect(host: String, port: Int = 80, tls: Boolean = true)`).
- Under the hood, Kotlin generates **one** method with all parameters plus a synthetic `connect$default(..., int mask, Object marker)` method that uses a **bitmask** to know which arguments were omitted.
- **Problem for Java:** Java doesn't know about Kotlin's default-argument bitmask, so Java callers are forced to pass **every single parameter** explicitly.
- **Solution (`@JvmOverloads`):** Instructs Kotlin to generate overloaded Java-friendly methods by dropping parameters with defaults **from right to left**:
  - `connect(String host)`
  - `connect(String host, int port)`
  - `connect(String host, int port, boolean tls)`
- **Note:** `@JvmOverloads` drops trailing defaults in order—it doesn't generate every combination if an earlier parameter has a default while a later one doesn't.

```kotlin
class HttpClient {
    @JvmOverloads
    fun request(
        url: String,
        timeoutMs: Int = 5_000,
        followRedirects: Boolean = true
    ) { /* ... */ }
}

// Java can now call any of:
// client.request("https://example.com");
// client.request("https://example.com", 10_000);
// client.request("https://example.com", 10_000, false);
```

   -------------------------------------
52. Package-private visibility equivalent in Kotlin (`internal`).

- Kotlin has **no `package-private`** visibility (Java's default when you omit a modifier). Instead, Kotlin's default visibility is `public`, and its module-level visibility is **`internal`**.
- **`internal`** means visible **anywhere inside the same compilation module** (e.g., the same Gradle module / Maven project / IntelliJ module), and completely hidden from other modules that depend on it.
- **Why Kotlin chose `internal` over package-private:** In Java, anyone in another library could create a class with the same package name and gain access to package-private members. `internal` enforces true library/module encapsulation.
- **Java interop nuance:** Because JVM bytecode has no `internal` modifier, Kotlin compiles `internal` declarations as `public` in bytecode and **mangles** their method names (e.g., `helper$my_module`) to discourage accidental calls from Java.

```kotlin
// Inside `:feature-auth` Gradle module:
internal class TokenStorage {
    internal fun saveToken(token: String) { /* ... */ }
}

// Accessible from any package inside `:feature-auth`,
// but NOT accessible from `:app` or `:feature-home` modules!
```

   -------------------------------------
53. Exception handling with `runCatching` and `Result<T>`.

- `runCatching { ... }` executes a block and wraps either its return value or any thrown `Throwable` inside a **`Result<T>`** value class (zero-overhead wrapper when unboxed).
- Enables functional, chainable error handling without nested `try/catch` blocks:
  - `.getOrNull()`, `.getOrDefault(fallback)`, `.getOrElse { ex -> ... }`, `.getOrThrow()`
  - `.map { ... }`, `.mapCatching { ... }`, `.recover { ... }`, `.fold(onSuccess, onFailure)`
  - `.onSuccess { ... }`, `.onFailure { ... }` (side effects that return the same `Result`)
- **Critical Android / Coroutines warning:** `runCatching` catches **`Throwable`**, which includes **`CancellationException`**! Inside a `suspend` function, swallowing `CancellationException` breaks structured concurrency—always rethrow it or write a `suspendRunCatching` helper.

```kotlin
fun parsePort(raw: String): Result<Int> = runCatching {
    val port = raw.toInt()
    require(port in 1..65535) { "Port out of range" }
    port
}

val port = parsePort("8080")
    .onSuccess { println("Valid port: $it") }
    .onFailure { println("Parse error: ${it.message}") }
    .getOrDefault(80)

// Safe wrapper for coroutines (rethrows CancellationException):
inline fun <T> suspendRunCatching(block: () -> T): Result<T> = try {
    Result.success(block())
} catch (c: kotlinx.coroutines.CancellationException) {
    throw c
} catch (e: Throwable) {
    Result.failure(e)
}
```

   -------------------------------------
54. Multiplatform Kotlin (KMP) basics: `expect` / `actual` declaration.

- Kotlin Multiplatform (KMP) lets you share business logic, networking, and models in `commonMain` across Android, iOS, Desktop, and Web while still accessing platform-specific APIs (e.g., `UUID`, `Context`, `NSDate`, SQLite drivers).
- **`expect`** (in `commonMain`): Declares a contract (function, class, property, object, or annotation) that every target platform promises to provide—contains **no implementation**.
- **`actual`** (in `androidMain`, `iosMain`, etc.): Provides the matching platform-specific implementation with the exact same package and signature.
- **Best practice:** Use `expect`/`actual` for small platform glue or top-level factories; prefer standard **interfaces + Dependency Injection** for larger components so they remain easy to mock and unit test in `commonTest`.

```kotlin
// commonMain/Platform.kt
expect fun platformName(): String
expect fun currentTimeMillis(): Long

// androidMain/Platform.android.kt
actual fun platformName(): String = "Android ${android.os.Build.VERSION.SDK_INT}"
actual fun currentTimeMillis(): Long = System.currentTimeMillis()

// iosMain/Platform.ios.kt
// actual fun platformName(): String = UIDevice.currentDevice.systemName()
// actual fun currentTimeMillis(): Long = (NSDate().timeIntervalSince1970 * 1000).toLong()
```

   -------------------------------------
55. Immutable collections vs read-only collections.

- **Read-only (`kotlin.collections.List`, `Set`, `Map`):**
  - Only means *this interface* has no mutating methods (`add`, `remove`).
  - Does **not** guarantee the underlying data won't change—another holder of the backing `MutableList` can still mutate it, or a caller can unsafe-cast `(list as MutableList).add(...)`.
- **Truly Immutable (`kotlinx.collections.immutable.ImmutableList` / `PersistentList`):**
  - Guarantees the collection can **never** be mutated after creation.
  - "Modification" operations (`add`, `remove`) return a **new** persistent collection sharing most of the structure efficiently (structural sharing).
- **Why this matters in Jetpack Compose:** Because Kotlin's standard `List<T>` is only read-only (not guaranteed immutable), the Compose compiler historically treated `List<T>` parameters as **unstable** (triggering unnecessary recompositions unless strong skipping is enabled or you use `ImmutableList<T>` / `@Immutable`).

```kotlin
// Read-only != Immutable
val backing = mutableListOf(1, 2)
val readOnly: List<Int> = backing
backing.add(3)
println(readOnly) // [1, 2, 3] — changed under your feet!

// Truly immutable (with kotlinx.collections.immutable):
// val immutable: ImmutableList<Int> = persistentListOf(1, 2)
// val next: ImmutableList<Int> = immutable.add(3) // original `immutable` stays [1, 2]
```

   -------------------------------------
56. `associate`, `associateBy`, and `groupBy` operators.

All three convert a collection into a `Map`, differing in how keys, values, and duplicate keys are handled:

- **`associate { item -> key to value }`**: You construct **both** the key and value as a `Pair`. Duplicate keys **overwrite** earlier entries (last wins).
- **`associateBy { keySelector }`**: The **element itself** becomes the value (`Map<K, T>`). Optionally takes a second `valueTransform` lambda. Duplicate keys **overwrite** earlier entries (last wins).
- **`associateWith { valueSelector }`**: The **element itself** becomes the key (`Map<T, V>`).
- **`groupBy { keySelector }`**: Groups **all** elements with the same key into a `List` (`Map<K, List<T>>`)—**no data is lost** on duplicate keys.

```kotlin
data class User(val id: Int, val name: String, val role: String)

val users = listOf(
    User(1, "Alice", "Admin"),
    User(2, "Bob", "User"),
    User(3, "Cara", "Admin")
)

// 1. associate: custom key AND custom value -> {1="Alice", 2="Bob", 3="Cara"}
val idToName: Map<Int, String> = users.associate { it.id to it.name }

// 2. associateBy: key selector, element as value -> {1=User(1,...), 2=User(2,...), 3=User(3,...)}
val byId: Map<Int, User> = users.associateBy { it.id }

// 3. groupBy: preserves ALL duplicates in a List -> {"Admin"=[Alice, Cara], "User"=[Bob]}
val byRole: Map<String, List<User>> = users.groupBy { it.role }
```

   -------------------------------------
57. `fold` vs `reduce` collection methods.

Both accumulate elements of a collection left-to-right into a single value, using `(acc, element) -> newAcc`:

| Feature | `fold(initial) { acc, item -> ... }` | `reduce { acc, item -> ... }` |
| :--- | :--- | :--- |
| Initial value | **Required** (`initial`) | **None** — starts with the 1st element |
| Empty collection | Safe — returns `initial` | Throws **`UnsupportedOperationException`** (use `reduceOrNull` to avoid crash) |
| Return type | Can be **any type `R`** (different from element type `T`) | Must be the **same type `T`** (or a supertype of `T`) |

```kotlin
val nums = listOf(1, 2, 3, 4)

// reduce: starts with acc = 1, then +2, +3, +4 -> 10
val sumReduce = nums.reduce { acc, n -> acc + n }

// fold: starts with acc = 100 -> 110 (also safe on emptyList())
val sumFold = nums.fold(100) { acc, n -> acc + n }

// fold can return a completely different type (List<Int> -> String):
val csv: String = nums.fold("Values:") { acc, n -> "$acc $n" } // "Values: 1 2 3 4"

// Empty list safety:
val emptySum = emptyList<Int>().fold(0) { acc, n -> acc + n } // 0 ✅
val emptyReduce = emptyList<Int>().reduceOrNull { acc, n -> acc + n } // null ✅
```

   -------------------------------------
58. Primitive type wrapping and boxing in Kotlin.

- Kotlin unifies primitive and wrapper types—you always write `Int`, `Long`, `Boolean`, `Double` (there is no `int` vs `Integer` syntax split).
- **Zero-cost primitives by default:** Non-nullable `Int` compiles directly to JVM primitive `int`.
- **When Kotlin must box (`int` → `java.lang.Integer`):**
  1. **Nullable types:** `Int?` must be boxed because primitives cannot hold `null`.
  2. **Generics:** `List<Int>`, `Map<String, Int>`, `Box<Int>` box elements because JVM generics only work with reference types.
  3. **Upcasting to `Any` or an interface:** `val a: Any = 42` boxes `42`.
- **Performance pitfall:** `Array<Int>` boxes every element; use `IntArray` for primitive `int[]`. Also avoid unnecessary `Int?` inside hot loops.

```kotlin
val primitive: Int = 42          // JVM: int (primitive, stack)
val boxedNullable: Int? = 42     // JVM: java.lang.Integer (heap object)
val boxedGeneric: List<Int> = listOf(1, 2, 3) // JVM: List<Integer>

// Subtle boxing gotcha with === (JVM caches Integer -128..127):
val a: Int? = 1000
val b: Int? = 1000
println(a == b)  // true  (structural equality)
println(a === b) // false (two separate boxed Integer objects!)
```

   -------------------------------------
59. Destructuring custom classes using `componentN()` functions.

- While `data class` generates `component1()`, `component2()`, ... automatically, **any** regular class can support destructuring (`val (a, b) = obj`) by defining **`operator fun component1()`**, **`operator fun component2()`**, etc.
- You can add `componentN()` either directly inside the class or as **extension functions** (even for third-party or Java classes like Android's `Point` or `Rect`!).
- Just remember: destructuring is strictly **positional** (`component1()` → 1st variable, `component2()` → 2nd variable).

```kotlin
// Regular (non-data) class with custom destructuring:
class HttpResponse(val statusCode: Int, val body: String, val headers: Map<String, String>) {
    operator fun component1(): Int = statusCode
    operator fun component2(): String = body
}

val response = HttpResponse(200, """{"ok":true}""", emptyMap())
val (code, body) = response // calls component1() and component2()
println("HTTP $code: $body")

// Extension componentN() for a class you don't own:
operator fun String.component1(): String = substringBefore(":")
operator fun String.component2(): String = substringAfter(":")

val (host, port) = "localhost:8080" // host="localhost", port="8080"
```

   -------------------------------------
60. Overriding properties in subclasses.

- Properties in Kotlin are `final` by default. To allow overriding in a subclass, mark the property `open` (or `abstract`, or declare it in an `interface`) and use the **`override`** keyword in the subclass.
- **Mutability rule:**
  - You **can** override a `val` with a `var` (a `val` guarantees a getter; `var` adds a setter while keeping the getter).
  - You **cannot** override a `var` with a `val` (that would break the parent's contract of having a setter).
- **Covariant return types:** You can narrow a `val`'s type in a subclass (e.g., `open val animal: Animal` → `override val animal: Dog`).
- **Pitfall:** Never access an `open` property inside a base class's `init` block—the subclass's property initializer hasn't run yet, so you'll read a default `null`/`0` value!

```kotlin
interface Shape {
    val name: String
    val color: String
}

open class Polygon : Shape {
    override val name: String = "Polygon"
    override var color: String = "Black" // ✅ overriding `val` with `var` is allowed
}

class Square(override val name: String = "Square") : Polygon() // ✅ override in primary constructor
```

   -------------------------------------
61. Interface default method implementations in Kotlin bytecode.

- Unlike pre-Java-8 interfaces, Kotlin interfaces can declare methods with default implementations (and properties with default getters).
- **How they compile to JVM bytecode:**
  - **Legacy mode (Java 6 compatibility):** Kotlin generated a hidden nested static helper class `MyInterface$DefaultImpls` containing static methods, and implementing classes got synthetic forwarding methods delegating to `DefaultImpls`.
  - **Modern JVM 8+ mode (`-Xjvm-default=all` / default in Kotlin 2.2+ with `-jvm-default=enable`):** Kotlin compiles interface methods directly into native **JVM `default` interface methods**, eliminating the extra `DefaultImpls` classes and letting Java classes inherit default implementations directly without overriding them!
- **Diamond conflict resolution:** If a class implements two interfaces with the same default method, you must override it and disambiguate using `super<InterfaceName>.method()`.

```kotlin
interface Clickable {
    fun showOff() = println("I'm clickable")
}

interface Focusable {
    fun showOff() = println("I'm focusable")
}

class Button : Clickable, Focusable {
    override fun showOff() {
        super<Clickable>.showOff() // disambiguate diamond conflict
        super<Focusable>.showOff()
    }
}
```

   -------------------------------------
62. Operator function conventions for collections (`get`, `set`, `contains`).

Kotlin's clean collection syntax (`list[i]`, `map[k] = v`, `x in col`) is powered by standard **`operator`** conventions that you can also implement on your own custom classes:

- **`a[i]`** → `a.get(i)` (supports multiple indices too: `matrix[row, col]` → `matrix.get(row, col)`)
- **`a[i] = v`** → `a.set(i, v)`
- **`x in a`** → `a.contains(x)` (and `x !in a` → `!a.contains(x)`)
- **`a += x`** → `a.plusAssign(x)` (for mutable collections) or `a = a.plus(x)` (for read-only `var`)

```kotlin
class Grid(private val cols: Int, private val data: IntArray) {
    operator fun get(row: Int, col: Int): Int = data[row * cols + col]
    operator fun set(row: Int, col: Int, value: Int) {
        data[row * cols + col] = value
    }
    operator fun contains(value: Int): Boolean = value in data
}

val grid = Grid(3, IntArray(9))
grid[1, 2] = 42              // calls grid.set(1, 2, 42)
println(grid[1, 2])          // calls grid.get(1, 2) -> 42
println(42 in grid)          // calls grid.contains(42) -> true
```

   -------------------------------------
63. `checkNotNull` and `require` precondition checks.

Kotlin's standard library provides clear, expressive precondition helpers that fail fast with the right semantic exception type (and use **Contracts** to smart-cast non-null values!):

- **Caller's fault (bad arguments) → `IllegalArgumentException`:**
  - `require(condition) { "lazy error message" }`
  - `requireNotNull(value) { "lazy error message" }` → returns non-null value
- **Object's internal state is invalid → `IllegalStateException`:**
  - `check(condition) { "lazy error message" }`
  - `checkNotNull(value) { "lazy error message" }` → returns non-null value
- **Why lazy message lambdas?** The `{ "..." }` string is **only built if the condition fails**, avoiding unnecessary string allocations on the happy path.

```kotlin
class BankAccount {
    private var isOpen = false
    private var owner: String? = null

    fun deposit(amount: Double) {
        // Invalid caller argument -> IllegalArgumentException
        require(amount > 0) { "Deposit amount must be positive, got $amount" }

        // Invalid object state -> IllegalStateException
        check(isOpen) { "Cannot deposit into a closed account" }

        // Smart-casts & returns non-null String (throws IllegalStateException if null)
        val validOwner: String = checkNotNull(owner) { "Account owner not initialized" }
        println("Deposited $amount for $validOwner")
    }
}
```

   -------------------------------------
64. Explicit API mode in Kotlin libraries.

- A compiler mode designed for **library authors** (`explicitApi()` in Gradle) that enforces stricter, self-documenting public APIs so you don't accidentally expose internals or silently change a return type via type inference.
- **What Explicit API mode enforces:**
  1. **Explicit visibility modifiers** are required on all declarations exposed as `public` (you must explicitly write `public`, `internal`, or `private`—no implicit default `public`).
  2. **Explicit return types** are required on all `public` functions and properties (prevents accidental binary/source-breaking API changes if an implementation's inferred type changes).
- **Two levels:**
  - `explicitApi()` (strict): violations cause **compile errors**.
  - `explicitApiWarning()`: violations emit **compiler warnings** (great for gradually migrating an existing library).

```kotlin
// In build.gradle.kts:
// kotlin {
//     explicitApi() // or explicitApiWarning()
// }

// With Explicit API mode enabled:
// ❌ Error: missing visibility modifier and explicit return type
// fun calculateTotal(a: Int, b: Int) = a + b

// ✅ Valid: explicit `public` visibility and explicit `: Int` return type
public fun calculateTotal(a: Int, b: Int): Int = a + b

// ✅ Internal/private members don't require explicit return types
internal fun helper(x: Int) = x * 2
```

   -------------------------------------

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

