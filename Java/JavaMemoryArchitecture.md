# Java Memory Architecture

## Java Architecture

```
+------------------------------------------------------+
|                      JDK                             |
|         (Java Development Kit)                       |
|                                                      |
|  +-----------------------------------------------+   |
|  |                    JRE                        |   |
|  |         (Java Runtime Environment)            |   |
|  |                                               |   |
|  |   +--------------------------------------+    |   |
|  |   |                JVM                   |    |   |
|  |   |       (Java Virtual Machine)         |    |   |
|  |   +--------------------------------------+    |   |
|  |                                               |   |
|  |   Core Libraries, Runtime Components          |   |
|  +-----------------------------------------------+   |
|                                                      |
|  javac, javadoc, jar, jdb and other tools            |
+------------------------------------------------------+
```

### 1. JDK (Java Development Kit)

JDK is used for developing Java applications.

It contains:

- JRE
- JVM
- Development Tools

Common tools:

- javac   -> Java Compiler
- java    -> Runs Java programs
- javadoc -> Generates documentation
- jar     -> Creates JAR files
- jdb     -> Java Debugger

Example:

`JDK = JRE + Development Tools`

A developer needs JDK because code must be compiled before execution.

### 2. JRE (Java Runtime Environment)

JRE provides everything required to run a Java application.

It contains:

- JVM
- Java Class Libraries
- Runtime Components

A JRE can execute Java programs but cannot compile them.

Example:

`JRE = JVM + Runtime Libraries`

### 3. JVM (Java Virtual Machine)

The JVM is a virtual machine responsible for executing Java bytecode.

Responsibilities:

Loads classes into memory
Manages memory (Heap, Stack, Metaspace)
Executes bytecode
Performs Garbage Collection
Provides platform independence

Example:
```
.java
   ↓
.class (Bytecode)
   ↓
  JVM
   ↓
Machine Code
```


If a user only wants to run a Java application, JRE is sufficient.

## The JVM memory architecture is divided into five major runtime data areas:

###  1. **Heap**: 
Heap used to store mainly objects and instance variables. Garbage collectre manages and cleans unused objects from heap.
### 2. **Stack**: 
Stack used to store local varables, method calls, method parameters, and references to objects.
### 3. **Method area (Metaspace)**: 
Method Area (Metaspace in Java 8+) stores class metadata, static variables, runtime constant pool, and method information.
### 4. **Program Counter (PC) Register**: 
PC Register stores the address of the current instruction being executed by a thread.
### 5. **Native Method Stack**: 
Native Method Stack is used for the execution of native methods written in languages such as C and C++ through JNI (Java Native Interface).


## JVM Memory Architecture

```text
                JVM Memory

        +-------------------+
        |   Method Area     |
        |   (Metaspace)     |
        +-------------------+

        +-------------------+
        |       Heap        |
        |      Objects      |
        +-------------------+

Thread 1               Thread 2
+---------+           +---------+
|  Stack  |           |  Stack  |
+---------+           +---------+
|   PC    |           |   PC    |
+---------+           +---------+
| Native  |           | Native  |
| Stack   |           | Stack   |
+---------+           +---------+
```

<br>

## What Happens When We Run a Java Program?

When a Java program is executed, the following steps take place:

1. The `.java` source file is compiled by the `javac` compiler into `.class` files containing bytecode.
2. The Class Loader loads the required classes into JVM memory.
3. The Runtime Data Areas (Heap, Stack, Method Area, etc.) are created.
4. The Execution Engine executes the bytecode.
5. The Interpreter executes bytecode instruction by instruction.
6. Frequently executed code is identified and compiled into native machine code by the JIT (Just-In-Time) Compiler.
7. The Garbage Collector periodically removes unused objects from the heap.

```text
.java
   |
   v
javac Compiler
   |
   v
.class (Bytecode)
   |
   v
Class Loader
   |
   v
JVM Memory
   |
   v
Execution Engine
   |
   +--> Interpreter
   |
   +--> JIT Compiler
   |
   v
Machine Code
   |
   v
Output
```

<br>

## Class loader

A Class Loader is a JVM component responsible for loading Java classes into memory at runtime. 
Java follows a lazy loading approach, which means classes are loaded only when they are first required instead of loading all classes at application startup.

There are 3 main class loaders

### 1. Bootstrap Class Loader

The Bootstrap Class Loader is the parent of all class loaders. It loads core Java classes from the Java Runtime Environment such as:

- java.lang.String
- java.lang.Object
- java.util.ArrayList

### 2. Platform Class Loader

The Platform Class Loader (called Extension Class Loader before Java 9) loads platform-specific Java libraries.

### 3. Application Class Loader

The Application Class Loader loads classes that are present in the application's classpath.


### Class Loader - Parent Delegation Model

when class loader loads classes it uses `parent delegation model` where it checkes if class to load is core class so first Bootstrap class loader comes into play, the Extension class loader and then Application class loader
It prevents core Java classes from being overridden by application classes and ensures classes are loaded only once.

Example: 

Suppose someone creates:

```
package java.lang; 

public class String {

}
```
Without parent delegation, JVM might load this fake String class instead of the original one.
Parent delegation prevents this.

```
Request to load class
        |
        v
Application Loader
        |
        v
Platform Loader
        |
        v
Bootstrap Loader
        |
        |
  Found ? -----> Load
        |
       No
        |
        v
Return to child loader
```
<br>

## Interview Questions

### What is a Class Loader?

A Class Loader is a JVM component responsible for loading classes into memory at runtime.

### How many class loaders are present in Java?

There are three major class loaders:
- Bootstrap Class Loader
- Platform Class Loader
- Application Class Loader

### What is Parent Delegation Model?

Parent Delegation Model is a mechanism where a class loader first delegates the class loading request to its parent before attempting to load the class itself.

### Why is Parent Delegation Model important?

- Prevents duplicate class loading.
- Protects core Java classes from being overridden.
- Improves security and consistency.



<br>

## Class loader Life cycle

There are 3 main stages in the Class Loading Lifecycle

```
1. Loading: 
     ↓
2. Linking
     ↓
   Verification
     ↓
   Preparation
     ↓
   Resolution
     ↓
3. Initialization
```

1. `Loading`: 
      - JVM checks whether the class has already been loaded.
      - If not, the Class Loader locates and reads the corresponding .class file.
      - The class metadata is stored in the Method Area (Metaspace in Java 8+).
      - A java.lang.Class object representing the class is created in the heap.
2. `Linking`: 
      - `Verification`: Ensure bytecode is valid
      - `Preperation`: Allocates memory for static variables and assign default values
      - `Resolution`: Converts symbolic references to direct references
3.  `Initialization`: 
      - Static variable assignments are executed.
      - Static initialization blocks are executed.
      - The class becomes ready for use.

Example: 

```
class Test {
    static int count = 10;
}

// During preperation ->  count = 0;
// During Initialization ->  count = 10;
```


<br>

## Execution Engine

The Execution Engine is a component of the JVM responsible for executing the bytecode loaded into memory.

It consists mainly of:

1. Interpreter
2. JIT (Just-In-Time) Compiler
3. Garbage Collector

### Interpreter

The Interpreter executes bytecode instruction by instruction and converts it into machine code at runtime.

#### Advantages

- Fast startup time
- Less compilation overhead

#### Disadvantages

- Slower execution for frequently executed code
- Same bytecode may be interpreted repeatedly

### JIT Compiler

The JIT (Just-In-Time) Compiler improves performance by identifying frequently executed code, known as Hot Spots, and compiling it into native machine code.

Once compiled, the native code can be executed directly without repeatedly interpreting the bytecode.

#### Advantages

- Better runtime performance
- Reduces repeated interpretation
- Optimizes frequently executed code



### Why does Java have both Interpreter and JIT?

The Interpreter provides fast startup because it can execute bytecode immediately.

The JIT Compiler improves long-running performance by compiling frequently used code into machine code.

This combination gives Java both portability and performance.

### Why is Java slower during startup?

Initially, the JVM uses the Interpreter to execute bytecode.

As the application runs, the JIT Compiler identifies frequently executed code and compiles it into native machine code.

Therefore Java applications may start slower but become faster over time.


<br>

## Garbage Collection

Garbage Collection in JVM is a process of cleaning up heap memory by removing unreachable objects and making the memory available for other functionalities

Garbage Collector performs `reachability analysis` to determine whether an object is reachable from GC Roots. Objects that are not reachable are considered eligible for garbage collection.

### GC Roots

Reachability Analysis starts from GC Roots.

Examples of GC Roots:

1. Local variables in stack frames
2. Active threads
3. Static variables
4. JNI references

There are 4 types of references in this GC process

1. `Strong Reference`: A Strong Reference is the default type of reference created using the new keyword. Objects with strong references are not eligible for garbage collection as long as they are reachable.
2. `Weak Reference`: Initialization using `WeakReference<>`, can be GC'ed whenever GC runs even though memory still available. Objects referenced only through Weak References may be collected during the next GC cycle.
3. `Soft Reference`:  Initialization using `SoftReference<>`, Soft References are typically used for caches. The referenced object remains in memory until the JVM requires additional memory.
4. `Phantom Reference`: Initialization using `PhantomReference<>`, A Phantom Reference is used to receive notification after an object becomes unreachable but before its memory is reclaimed. It is commonly used for resource cleanup and advanced memory management.

```
GC Root
   |
   v
Object A
   |
   v
Object B

Object C

A and B are reachable.
C is unreachable.
```

### Generational Heap

The Heap is divided into:

1. Young Generation
   - Most Java objects have a very short lifespan and become unreachable shortly after creation. Examples include temporary objects, method-local objects, and intermediate calculation results.
   - `Eden Space`: Most newly created objects are initially allocated in the Eden Space.
   - `Survivor S0`: Objects that survive a Minor GC are moved from Eden Space to a Survivor Space.
   - `Survivor S1`: Objects continue to move between Survivor S0 and Survivor S1 after each Minor GC while their age increases.
2. Old Generation
   - Objects that survive multiple Minor GC cycles are eventually promoted to the Old Generation. The promotion threshold is JVM-dependent and is commonly around 15 survivor ages.
   - Example: Cache, Config, Singleton Objects, and Long-lived Collections

```
Heap
│
├── Young Generation
│   ├── Eden Space
│   ├── Survivor S0
│   └── Survivor S1
│
└── Old Generation
```

### Types of Garbage Collection

#### Minor GC

Occurs in the Young Generation.

#### Major GC

Major GC occurs in the Old Generation.

It removes unreachable objects from the Old Generation and is generally slower than Minor GC because it processes long-lived objects.

#### Full GC

Full GC operates on the entire heap, including both Young Generation and Old Generation.

It is the most expensive garbage collection operation and may cause noticeable application pauses.

```
1. Object created
      ↓
2. Stored in Eden
      ↓
3. Minor GC runs
      ↓
4. Surviving objects move to Survivor
      ↓
5. Survive multiple GC cycles
      ↓
6. Promoted to Old Generation
      ↓
7. Major GC cleans Old Generation
      ↓
8. Full GC cleans entire Heap when necessary
```

### Why should we avoid Full GC?

Because Full GC scans the entire heap and can cause application pauses, reducing performance.

### Why are there two Survivor Spaces?

The JVM uses two Survivor Spaces (S0 and S1) to efficiently copy surviving objects between GC cycles.

At any point, one Survivor Space acts as the source and the other acts as the destination. After each Minor GC, their roles are swapped.

