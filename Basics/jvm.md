# ☕ Java Basics Overview: JDK, JRE, JVM ☕

> A foundational guide to understanding the core components of the Java ecosystem, explaining how Java achieves its famed "write once, run anywhere" capability.

---

## 📑 Table of Contents

1. [What is Java?](#what-is-java)
2. [The 3 Main Components: JDK, JRE, JVM](#the-3-main-components-jdk-jre-jvm)
   - [JVM (Java Virtual Machine)](#1-jvm-java-virtual-machine)
   - [JRE (Java Runtime Environment)](#2-jre-java-runtime-environment)
   - [JDK (Java Development Kit)](#3-jdk-java-development-kit)
3. [First Java Program Example](#first-java-program-example)
4. [How to Download JDK](#how-to-download-jdk)
5. [Java Editions: JSE, JEE, JME](#java-editions-jse-jee-jme)

---

## 🤔 What is Java?

**Timestamp:** [[00:26](http://www.youtube.com/watch?v=IoireaKRRFo&t=26)]

Java is a widely popular, high-level, **platform-independent language** and a prominent **Object-Oriented Programming (OOP)** language.

### Platform Independent

**Timestamp:** [[01:32](http://www.youtube.com/watch?v=IoireaKRRFo&t=92)]

This is Java's most significant advantage, famously known as **WORA** (Write Once, Run Anywhere).

- It means you can write Java code on one operating system (like macOS) and run it on any other operating system (like Windows or Linux) without modification. [[01:54](http://www.youtube.com/watch?v=IoireaKRRFo&t=114)]

### OOP Language

Java adheres to OOP principles (Inheritance, Polymorphism, Encapsulation, Abstraction), which helps in creating modular, reusable, and maintainable code.

---

## 🧩 The 3 Main Components: JDK, JRE, JVM

**Timestamp:** [[02:37](http://www.youtube.com/watch?v=IoireaKRRFo&t=157)]

Understanding these three components is crucial to grasp how Java code is executed. They form a nested structure: **JDK** contains **JRE**, and **JRE** contains **JVM**.

### 1. JVM (Java Virtual Machine)

**Timestamp:** [[03:07](http://www.youtube.com/watch?v=IoireaKRRFo&t=187)]

#### What it is

JVM stands for **Java Virtual Machine**. It's an **abstract machine** [[03:17](http://www.youtube.com/watch?v=IoireaKRRFo&t=197)] (meaning it doesn't physically exist as hardware) that provides a runtime environment for executing Java bytecode.

#### Platform Dependent

Unlike Java code, the **JVM is platform-dependent**. [[05:10](http://www.youtube.com/watch?v=IoireaKRRFo&t=310)] Each operating system (Windows, macOS, Linux) has its specific JVM implementation. [[05:47](http://www.youtube.com/watch?v=IoireaKRRFo&t=347)]

#### Role in Execution

1. Your **Java Program** (`.java` file) is **compiled** by the Java compiler (`javac`). [[03:54](http://www.youtube.com/watch?v=IoireaKRRFo&t=234)]
2. This compilation converts the `.java` file into **Bytecode** (`.class` file). [[04:06](http://www.youtube.com/watch?v=IoireaKRRFo&t=246)]
3. The **JVM** takes this bytecode as input. [[04:15](http://www.youtube.com/watch?v=IoireaKRRFo&t=255)]
4. Inside the JVM, a **JIT (Just-In-Time) compiler** [[06:27](http://www.youtube.com/watch?v=IoireaKRRFo&t=387)] converts the bytecode into **Machine Code** (native code for the specific OS). [[04:21](http://www.youtube.com/watch?v=IoireaKRRFo&t=261)]
5. This machine code is then executed by the CPU, producing the output. [[04:24](http://www.youtube.com/watch?v=IoireaKRRFo&t=264)]

#### Achieving Portability

The JVM's ability to interpret bytecode is what enables Java's portability. You compile your `.java` code *once* to bytecode, and then *any* JVM (on any OS) can execute that bytecode. [[07:04](http://www.youtube.com/watch?v=IoireaKRRFo&t=424)]

---

### 2. JRE (Java Runtime Environment)

**Timestamp:** [[10:45](http://www.youtube.com/watch?v=IoireaKRRFo&t=645)]

#### What it is

JRE stands for **Java Runtime Environment**. It is an environment required to **run** Java applications.

#### Components

**Timestamp:** [[11:09](http://www.youtube.com/watch?v=IoireaKRRFo&t=669)]

1. **JVM:** The Java Virtual Machine.
2. **Class Libraries:** A set of essential Java API classes (e.g., `java.lang`, `java.util`). [[11:15](http://www.youtube.com/watch?v=IoireaKRRFo&t=675)] These libraries contain pre-written code for common functionalities (like `Math.abs()` or `Arrays.sort()` as shown in the video example). [[12:02](http://www.youtube.com/watch?v=IoireaKRRFo&t=722)]

#### Running Java Programs

If you only need to run a Java application (and not develop one), you only need JRE installed. It provides everything necessary to execute compiled Java bytecode.

#### Cannot Develop

You **cannot write or compile** Java code with just a JRE.

---

### 3. JDK (Java Development Kit)

**Timestamp:** [[15:20](http://www.youtube.com/watch?v=IoireaKRRFo&t=920)]

#### What it is

JDK stands for **Java Development Kit**. It's a complete software development environment for writing Java applications.

#### Components

**Timestamp:** [[16:26](http://www.youtube.com/watch?v=IoireaKRRFo&t=986)]

1. **JRE:** The Java Runtime Environment.
2. **Development Tools:** Tools necessary for Java development, including:
   - **Programming Language:** The Java language itself. [[15:35](http://www.youtube.com/watch?v=IoireaKRRFo&t=935)]
   - **Compiler (`javac`):** To compile `.java` source code into `.class` bytecode. [[15:56](http://www.youtube.com/watch?v=IoireaKRRFo&t=956)]
   - **Debugger (`jdb`):** For debugging Java applications. [[16:21](http://www.youtube.com/watch?v=IoireaKRRFo&t=981)]
   - And other utilities (e.g., `javadoc`, `jar`).

#### For Developers

If you are a Java developer, you need JDK installed as it contains all the tools for writing, compiling, and running Java programs.

#### Relationship

**JDK = JRE + Development Tools** (Programming Language, Compiler, Debugger, etc.) [[16:26](http://www.youtube.com/watch?v=IoireaKRRFo&t=986)]

---

## 💻 First Java Program Example

**Timestamp:** [[23:51](http://www.youtube.com/watch?v=IoireaKRRFo&t=1431)]

Let's walk through creating and running a simple Java program.

### 1. Create a Java Source File

Create a file named `Employee.java`. The **file name must match the public class name**.

```java
// Employee.java
public class Employee {
    // This is a comment line, compiler will ignore this
    /* This is also a comment
     * it can be in multiple lines also */
    public static void main(String args[]) {
        int a = -10;
        System.out.println("This is my first program and output of a is: " + a);
    }
}
```

#### Understanding the Code

- **`public class Employee`**: Defines a public class named `Employee`.
- **`public static void main(String args[])`**: This is the **main method**, the **starting point** of the program. [[26:25](http://www.youtube.com/watch?v=IoireaKRRFo&t=1585)]
  - **`public`**: Accessible from anywhere (JVM can call it). [[27:45](http://www.youtube.com/watch?v=IoireaKRRFo&t=1665)]
  - **`static`**: Can be called without creating an object of the `Employee` class. (JVM doesn't create an `Employee` object to start the program). [[28:46](http://www.youtube.com/watch?v=IoireaKRRFo&t=1726)]
  - **`void`**: The method does not return any value. [[27:38](http://www.youtube.com/watch?v=IoireaKRRFo&t=1658)]
  - **`main`**: The standard name JVM looks for to start execution. [[27:30](http://www.youtube.com/watch?v=IoireaKRRFo&t=1650)]
  - **`(String args[])`**: An array to accept command-line arguments.
- **`System.out.println(...)`**: Prints the given message to the console.

### 2. Compile the Java Program

**Timestamp:** [[30:57](http://www.youtube.com/watch?v=IoireaKRRFo&t=1857)]

Open your terminal or command prompt, navigate to the directory where `Employee.java` is saved, and compile it using `javac`.

```bash
javac Employee.java
```

Upon successful compilation, a **`Employee.class`** file (bytecode) will be generated in the same directory. [[31:07](http://www.youtube.com/watch?v=IoireaKRRFo&t=1867)]

### 3. Run the Java Program

**Timestamp:** [[33:48](http://www.youtube.com/watch?v=IoireaKRRFo&t=2028)]

Execute the compiled Java program using the `java` command.

```bash
java Employee
```

The JVM will load `Employee.class`, find the `main` method, and start executing the code within it.

### Output

```
This is my first program and output of a is: -10
```

---

## ⬇️ How to Download JDK

**Timestamp:** [[18:00](http://www.youtube.com/watch?v=IoireaKRRFo&t=1080)]

1. **Search on Google:** Search for "Java 8 download" (or any desired version, e.g., "Java 17 download"). [[18:40](http://www.youtube.com/watch?v=IoireaKRRFo&t=1120)]
2. **Oracle Downloads:** Navigate to the official Oracle website for Java SE Development Kit (JDK) archives. [[18:51](http://www.youtube.com/watch?v=IoireaKRRFo&t=1131)]
3. **Select Version & Platform:** Choose the specific JDK version you need (e.g., JDK 8u202) and download the appropriate installer for your operating system (Windows x64, macOS x64, Linux x64). [[19:00](http://www.youtube.com/watch?v=IoireaKRRFo&t=1140)]
4. **Install:** Run the installer and follow the on-screen instructions.
5. **Verify Installation:** After installation, open your terminal and run `java -version`. [[19:35](http://www.youtube.com/watch?v=IoireaKRRFo&t=1175)] This will display the installed Java version, confirming successful installation.

---

## 🌐 Java Editions: JSE, JEE, JME

**Timestamp:** [[20:13](http://www.youtube.com/watch?v=IoireaKRRFo&t=1213)]

Java comes in different editions tailored for various development needs:

### JSE (Java Standard Edition)

**Timestamp:** [[21:04](http://www.youtube.com/watch?v=IoireaKRRFo&t=1264)]

- **What it is:** The **core Java** platform.
- **Purpose:** Used for developing **desktop applications**, applets, and command-line utilities. It provides the fundamental APIs for basic programming.
- **Example:** Anything we learn about core Java (classes, objects, threads, basic I/O) falls under JSE.

### JEE (Java Enterprise Edition) / Jakarta EE

**Timestamp:** [[21:28](http://www.youtube.com/watch?v=IoireaKRRFo&t=1288)]

- **What it is:** Built on top of JSE, adding APIs for enterprise-level development. (Note: JEE is now known as Jakarta EE).
- **Purpose:** Used for building **large-scale, distributed, multi-tier, and web-based enterprise applications**.
- **Features:** Includes APIs for transactional capabilities (rollback, commit), [[21:42](http://www.youtube.com/watch?v=IoireaKRRFo&t=1302)] persistence (managing relational databases), servlets, JSP (JavaServer Pages), and more.
- **Example:** E-commerce platforms, banking systems, and complex web services often use JEE/Jakarta EE.

### JME (Java Micro Edition)

**Timestamp:** [[22:39](http://www.youtube.com/watch?v=IoireaKRRFo&t=1359)]

- **What it is:** A subset of Java SE, optimized for resource-constrained devices.
- **Purpose:** Used for developing applications for **mobile devices**, embedded systems, and other limited-resource environments.
- **Features:** Provides APIs specifically designed for mobile applications. [[22:45](http://www.youtube.com/watch?v=IoireaKRRFo&t=1365)]
- **Example:** Older feature phone games or embedded device software might use JME.

---

## 📚 Summary

This guide covered the fundamental components of Java:

- **Java** is a platform-independent, object-oriented programming language
- **JVM** executes Java bytecode and is platform-specific
- **JRE** provides the runtime environment (JVM + libraries) to run Java applications
- **JDK** is the complete development kit (JRE + development tools) for Java developers
- Java comes in different editions (JSE, JEE, JME) for different use cases

With this foundation, you're ready to start your Java development journey! ☕

---

*Note: All timestamps link to the original video tutorial for further reference.*
