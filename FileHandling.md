# Java File Handling - Complete Course Notes

## Table of Contents
1. [Introduction to Streams](#introduction-to-streams)
2. [Types of Streams](#types-of-streams)
3. [Stream Classes Hierarchy](#stream-classes-hierarchy)
4. [Predefined Streams](#predefined-streams)
5. [Input Operations](#input-operations)
6. [Output Operations](#output-operations)
7. [File Class](#file-class)
8. [IOException](#ioexception)
9. [Best Practices](#best-practices)

---

## Introduction to Streams

### What is a Stream?

**Definition**: A stream is a sequence of data in Java

**Data Types in Streams**:
1. **Byte values** - For binary data (images, PDFs, audio files)
2. **Character values** - For Unicode characters (text, strings)

### Purpose of Streams

- Java performs input/output operations through streams
- Streams are an **abstraction** provided by Java
- Hides complexity of I/O operations
- Linked to physical devices (keyboard, file system, network)

### Implementation

- Streams are implemented within class hierarchies
- Located in **`java.io`** package
- IO = Input/Output

---

## Types of Streams

### 1. Byte Stream

**Purpose**: Handle input and output of bytes (binary data)

**Use Cases**:
- Reading/writing images
- Reading/writing PDF files
- Reading/writing audio files
- Any binary data operations
- Opening images in text editors

**Characteristics**:
- Works with binary data
- Two main abstract classes

**Class Hierarchy**:

```
Object
├── InputStream (for reading byte data)
│   ├── AudioInputStream
│   ├── ByteArrayInputStream
│   ├── FileInputStream
│   ├── FilterInputStream
│   ├── ObjectInputStream
│   └── StringBufferInputStream
│
└── OutputStream (for writing byte data)
    ├── ByteArrayOutputStream
    ├── FileOutputStream
    ├── FilterOutputStream
    └── ObjectOutputStream
```

**Naming Convention**: 
- Classes ending with **`InputStream`** or **`OutputStream`** are for byte data

---

### 2. Character Stream

**Purpose**: Handle input and output of Unicode characters

**Use Cases**:
- Reading/writing text files
- Working with internationalized content (Hindi, Chinese, etc.)
- Generally more efficient than byte streams for text

**Characteristics**:
- Works with Unicode characters
- Can handle international characters
- Two main abstract classes

**Class Hierarchy**:

```
Object
├── Reader (for reading character data)
│   ├── BufferedReader
│   ├── CharArrayReader
│   ├── InputStreamReader
│   │   └── FileReader
│   └── StringReader
│
└── Writer (for writing character data)
    ├── BufferedWriter
    ├── CharArrayWriter
    ├── OutputStreamWriter
    │   └── FileWriter
    └── StringWriter
```

**Naming Convention**: 
- Classes ending with **`Reader`** or **`Writer`** are for character data

---

## Stream Classes Hierarchy

### Abstract Classes

All stream classes extend from four main abstract classes:

1. **InputStream** - Reading byte data
2. **OutputStream** - Writing byte data
3. **Reader** - Reading character data
4. **Writer** - Writing character data

### Key Methods

**InputStream/Reader**:
- `read()` - Read data from the stream

**OutputStream/Writer**:
- `write()` - Write data to the stream
- `flush()` - Flush the stream
- `close()` - Close the stream and release resources

---

## Predefined Streams

Java provides three predefined streams in the `System` class:

### 1. System.in

```java
public static final InputStream in
```

- **Type**: `InputStream` (byte stream)
- **Purpose**: Standard input stream
- **Default**: Keyboard input

### 2. System.out

```java
public static final PrintStream out
```

- **Type**: `PrintStream` (byte stream)
- **Purpose**: Standard output stream
- **Default**: Console output

### 3. System.err

```java
public static final PrintStream err
```

- **Type**: `PrintStream` (byte stream)
- **Purpose**: Standard error stream
- **Default**: Console output (for errors)

---

## Input Operations

### 1. InputStreamReader

**Purpose**: Bridge from byte streams to character streams

**Constructor**:
```java
InputStreamReader(InputStream in)
```

**Example - Reading from Keyboard**:

```java
import java.io.*;

public class InputStreamReaderExample {
    public static void main(String[] args) {
        try {
            InputStreamReader isr = new InputStreamReader(System.in);
            
            System.out.println("Enter some letters:");
            
            int letters;
            while (isr.ready()) {
                letters = isr.read();
                System.out.println((char) letters);
            }
            
            isr.close();
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Key Points**:
- Converts byte stream (`System.in`) to character stream
- `read()` returns an integer (Unicode value)
- Cast to `char` to get the actual character
- Must close the stream after use

---

### 2. FileReader

**Purpose**: Reading character files

**Extends**: `InputStreamReader`

**Constructor**:
```java
FileReader(String fileName)
FileReader(File file)
```

**Example - Reading from File**:

```java
import java.io.*;

public class FileReaderExample {
    public static void main(String[] args) {
        try (FileReader fr = new FileReader("note.txt")) {
            
            int letters;
            while ((letters = fr.read()) != -1) {
                System.out.print((char) letters);
            }
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Key Points**:
- Reads one character at a time
- `read()` returns -1 when end of file is reached
- Try-with-resources automatically closes the stream

---

### 3. BufferedReader

**Purpose**: Reads text from character input stream efficiently using buffering

**Constructor**:
```java
BufferedReader(Reader in)
```

**Key Method**:
- `readLine()` - Reads an entire line of text

**Example - Reading Lines from Keyboard**:

```java
import java.io.*;

public class BufferedReaderExample {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(System.in))) {
            
            System.out.println("Enter text:");
            System.out.println("You typed: " + br.readLine());
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Example - Reading from File**:

```java
import java.io.*;

public class BufferedReaderFileExample {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(
                new FileReader("note.txt"))) {
            
            while (br.ready()) {
                System.out.println(br.readLine());
            }
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Fast Input for Competitive Programming**:

```java
// For fast input in coding competitions
BufferedReader br = new BufferedReader(
    new InputStreamReader(System.in));
```

**Why It's Faster**:
- Uses buffering for efficiency
- Character streams are generally faster than byte streams
- Can read entire lines at once

---

### 4. Scanner Class

**Note**: Scanner is NOT part of the stream hierarchy

**Purpose**: Simple text scanner for parsing primitives and strings

**Constructor**:
```java
Scanner(InputStream source)
```

**Example**:

```java
import java.util.Scanner;

Scanner scanner = new Scanner(System.in);
int number = scanner.nextInt();
String text = scanner.nextLine();
```

**Key Point**: 
- You tell Scanner where to get data from (e.g., `System.in` for keyboard)
- Scanner has convenient methods like `nextInt()`, `nextLine()`, etc.

---

## Output Operations

### 1. OutputStreamWriter

**Purpose**: Bridge from character streams to byte streams

**Constructor**:
```java
OutputStreamWriter(OutputStream out)
```

**Public Methods**:
1. `close()` - Close the stream
2. `flush()` - Flush the stream
3. `getEncoding()` - Get character encoding
4. `write()` - Write data (3 overloaded versions)

**Write Method Variations**:

```java
// 1. Write single character
write(int c)

// 2. Write portion of character array
write(char[] cbuf, int off, int len)

// 3. Write portion of string
write(String str, int off, int len)
```

**Example**:

```java
import java.io.*;

public class OutputStreamWriterExample {
    public static void main(String[] args) {
        try (OutputStreamWriter osw = new OutputStreamWriter(System.out)) {
            
            osw.write("Hello World\n");
            osw.write(97); // Writes 'a'
            osw.write(10); // Writes newline
            
            char[] arr = "Hello World".toCharArray();
            osw.write(arr);
            
            osw.flush(); // Important: flush to ensure data is written
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Output**:
```
Hello World
a
Hello World
```

**Key Points**:
- Converts character stream to byte stream
- Must call `flush()` to ensure data is written
- Can write strings, characters, or character arrays

---

### 2. FileWriter

**Purpose**: Writing characters to files

**Extends**: `OutputStreamWriter`

**Constructor**:
```java
FileWriter(String fileName)
FileWriter(String fileName, boolean append)
FileWriter(File file)
```

**Example - Overwrite File**:

```java
import java.io.*;

public class FileWriterExample {
    public static void main(String[] args) {
        try (FileWriter fw = new FileWriter("note.txt")) {
            
            fw.write("Hello World");
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Example - Append to File**:

```java
import java.io.*;

public class FileWriterAppendExample {
    public static void main(String[] args) {
        try (FileWriter fw = new FileWriter("note.txt", true)) {
            
            fw.write("\nThis should be appended");
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Key Points**:
- Default behavior: **overwrites** the file
- Use `new FileWriter(filename, true)` to **append** instead
- Inherits all methods from `OutputStreamWriter`
- Supports Unicode characters (can write in any language)

---

### 3. BufferedWriter

**Purpose**: Writes text to character output stream efficiently using buffering

**Constructor**:
```java
BufferedWriter(Writer out)
```

**Key Methods**:
- All `write()` methods from `Writer`
- `newLine()` - Writes a line separator

**Example**:

```java
import java.io.*;

public class BufferedWriterExample {
    public static void main(String[] args) {
        try (BufferedWriter bw = new BufferedWriter(
                new FileWriter("note.txt"))) {
            
            bw.write("Hare Krishna");
            bw.newLine(); // Platform-independent newline
            bw.write("Second line");
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Advantages**:
- More efficient than direct writing
- `newLine()` is platform-independent (handles Windows/Unix differences)
- Reduces the number of I/O operations

---

## File Class

**Package**: `java.io.File`

**Purpose**: Represents file and directory pathnames

### Creating a File

**Constructor**:
```java
File(String pathname)
```

**Note**: Constructor creates a File **instance**, not an actual file

**Example - Create New File**:

```java
import java.io.*;

public class FileExample {
    public static void main(String[] args) {
        try {
            // Create File instance
            File fileObject = new File("newfile.txt");
            
            // Actually create the file
            if (fileObject.createNewFile()) {
                System.out.println("File created: " + fileObject.getName());
            } else {
                System.out.println("File already exists");
            }
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

---

### Writing to a File

**Example - Complete Workflow**:

```java
import java.io.*;

public class CompleteFileExample {
    public static void main(String[] args) {
        try {
            // Step 1: Create the file
            File fileObject = new File("newfile.txt");
            fileObject.createNewFile();
            
            // Step 2: Write to the file (supports Unicode)
            FileWriter fw = new FileWriter("newfile.txt");
            fw.write("सर्वधर्मान्परित्यज्य मामेकं शरणं व्रज");
            fw.close();
            
            System.out.println("File created and written successfully");
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

**Output in File** (Sanskrit text):
```
सर्वधर्मान्परित्यज्य मामेकं शरणं व्रज
```

---

### Reading from a File

**Example**:

```java
import java.io.*;

public class ReadFileExample {
    public static void main(String[] args) {
        try (FileReader fr = new FileReader("newfile.txt")) {
            
            int letters;
            while ((letters = fr.read()) != -1) {
                System.out.print((char) letters);
            }
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

---

### Deleting a File

**Method**: `delete()`

**Returns**: `boolean` (true if deleted, false otherwise)

**Example**:

```java
import java.io.*;

public class DeleteFileExample {
    public static void main(String[] args) {
        try {
            File fileObject = new File("random.txt");
            fileObject.createNewFile();
            
            // Delete the file
            if (fileObject.delete()) {
                System.out.println("Deleted: " + fileObject.getName());
            } else {
                System.out.println("Failed to delete file");
            }
            
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

---

### Useful File Methods

```java
File file = new File("example.txt");

// Check if file exists
boolean exists = file.exists();

// Get file name
String name = file.getName();

// Get absolute path
String path = file.getAbsolutePath();

// Check if it's a directory
boolean isDir = file.isDirectory();

// Check if it's a file
boolean isFile = file.isFile();

// Get file size in bytes
long size = file.length();

// Check if readable
boolean canRead = file.canRead();

// Check if writable
boolean canWrite = file.canWrite();
```

---

## IOException

**Package**: `java.io.IOException`

**Definition**: Exception thrown when unexpected I/O problems occur

**Common Causes**:
1. **File not found** - Attempting to read non-existent file
2. **File corrupted** - File data is damaged
3. **Unable to read** - Permission issues or file in use
4. **Disk full** - No space to write
5. **Network issues** - For network I/O operations

**Usage Pattern**:

```java
try {
    // File operations
} catch (IOException e) {
    System.out.println("Error: " + e.getMessage());
}
```

---

## Auto-Closable Interface

### What is Auto-Closable?

**Definition**: Any class that implements `AutoCloseable` is considered a **resource**

**Why It Matters**:
- Resources hold system resources (file handles, socket connections)
- Must be closed to release resources
- Prevents resource leaks

**Interface**:
```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

### Try-with-Resources

**Syntax**:
```java
try (Resource resource = new Resource()) {
    // Use resource
} catch (Exception e) {
    // Handle exception
}
// Resource automatically closed
```

**Example - Manual Close**:
```java
FileReader fr = null;
try {
    fr = new FileReader("file.txt");
    // Use fr
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fr != null) {
        try {
            fr.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**Example - Try-with-Resources** (Recommended):
```java
try (FileReader fr = new FileReader("file.txt")) {
    // Use fr
} catch (IOException e) {
    e.printStackTrace();
}
// fr automatically closed
```

**Advantages**:
- Cleaner code
- Automatic resource management
- No need for explicit `close()` or `finally` block
- Multiple resources can be declared
- Available from Java 7+

**Multiple Resources**:
```java
try (FileReader fr = new FileReader("input.txt");
     FileWriter fw = new FileWriter("output.txt")) {
    // Use both resources
} catch (IOException e) {
    e.printStackTrace();
}
// Both automatically closed
```

---

## Best Practices

### 1. Always Close Streams

**Why**: To release system resources

**How**: Use try-with-resources (recommended)

```java
// ✅ Good - Automatic closing
try (FileReader fr = new FileReader("file.txt")) {
    // Use fr
} catch (IOException e) {
    e.printStackTrace();
}

// ❌ Bad - Manual closing (error-prone)
FileReader fr = new FileReader("file.txt");
// ... use fr ...
fr.close(); // Might not execute if exception occurs
```

---

### 2. Choose Appropriate Stream Type

**For Text Files**:
- Use `Reader`/`Writer` classes (character streams)
- More efficient for text
- Supports Unicode

**For Binary Files**:
- Use `InputStream`/`OutputStream` classes (byte streams)
- For images, PDFs, audio, video

**Quick Reference**:

| Task | Use This |
|------|----------|
| Read text file | `FileReader` or `BufferedReader` |
| Write text file | `FileWriter` or `BufferedWriter` |
| Read binary file | `FileInputStream` |
| Write binary file | `FileOutputStream` |
| Fast keyboard input | `BufferedReader` with `InputStreamReader` |
| Parse input | `Scanner` |

---

### 3. Use Buffered Streams for Efficiency

**Why**: Reduces number of I/O operations

```java
// ✅ Good - Buffered (efficient)
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}

// ❌ Less efficient - Unbuffered
try (FileReader fr = new FileReader("file.txt")) {
    int c;
    while ((c = fr.read()) != -1) {
        System.out.print((char) c);
    }
}
```

---

### 4. Handle Exceptions Properly

```java
try (FileReader fr = new FileReader("file.txt")) {
    // Use fr
} catch (FileNotFoundException e) {
    System.out.println("File not found: " + e.getMessage());
} catch (IOException e) {
    System.out.println("I/O error: " + e.getMessage());
}
```

---

### 5. Use Append Mode Carefully

```java
// Overwrite mode (default)
FileWriter fw = new FileWriter("file.txt");

// Append mode
FileWriter fw = new FileWriter("file.txt", true);
```

---

## Complete Examples

### Example 1: Copy File Content

```java
import java.io.*;

public class CopyFile {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("source.txt"));
             BufferedWriter bw = new BufferedWriter(new FileWriter("destination.txt"))) {
            
            String line;
            while ((line = br.readLine()) != null) {
                bw.write(line);
                bw.newLine();
            }
            
            System.out.println("File copied successfully");
            
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

---

### Example 2: Count Lines in File

```java
import java.io.*;

public class CountLines {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
            
            int count = 0;
            while (br.readLine() != null) {
                count++;
            }
            
            System.out.println("Total lines: " + count);
            
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

---

### Example 3: Read User Input and Write to File

```java
import java.io.*;

public class UserInputToFile {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(System.in));
             BufferedWriter bw = new BufferedWriter(
                new FileWriter("output.txt"))) {
            
            System.out.println("Enter text (type 'exit' to quit):");
            
            String line;
            while (!(line = br.readLine()).equals("exit")) {
                bw.write(line);
                bw.newLine();
            }
            
            System.out.println("Content saved to output.txt");
            
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

---

## Quick Reference Chart

### Stream Hierarchy Summary

```
BYTE STREAMS (ends with Stream)
├── Input (Reading)
│   └── InputStream → FileInputStream, ByteArrayInputStream, etc.
└── Output (Writing)
    └── OutputStream → FileOutputStream, ByteArrayOutputStream, etc.

CHARACTER STREAMS (ends with Reader/Writer)
├── Input (Reading)
│   └── Reader → FileReader, BufferedReader, InputStreamReader
└── Output (Writing)
    └── Writer → FileWriter, BufferedWriter, OutputStreamWriter
```

### Common Operations

| Operation | Class to Use | Example |
|-----------|--------------|---------|
| Read text file | `FileReader` | `new FileReader("file.txt")` |
| Write text file | `FileWriter` | `new FileWriter("file.txt")` |
| Read text efficiently | `BufferedReader` | `new BufferedReader(new FileReader("file.txt"))` |
| Write text efficiently | `BufferedWriter` | `new BufferedWriter(new FileWriter("file.txt"))` |
| Read from keyboard | `InputStreamReader` | `new InputStreamReader(System.in)` |
| Write to console | `OutputStreamWriter` | `new OutputStreamWriter(System.out)` |
| Create/delete files | `File` | `new File("file.txt")` |
| Fast competitive input | `BufferedReader` | `new BufferedReader(new InputStreamReader(System.in))` |

---

## Summary

### Key Concepts

1. **Streams** are sequences of data (bytes or characters)
2. **Byte Streams** (InputStream/OutputStream) handle binary data
3. **Character Streams** (Reader/Writer) handle text data
4. **Naming Convention**: 
   - Ends with `Stream` → Byte data
   - Ends with `Reader`/`Writer` → Character data
5. **Always close streams** to release resources (use try-with-resources)
6. **Buffered streams** are more efficient than unbuffered ones
7. **File class** is used to create/delete files and get file information

### Four Main Abstract Classes

- `InputStream` - Read bytes
- `OutputStream` - Write bytes
- `Reader` - Read characters
- `Writer` - Write characters

### Common Exceptions

- `IOException` - General I/O problems
- `FileNotFoundException` - File doesn't exist

### Best Practices

✅ Use try-with-resources for automatic resource management  
✅ Choose appropriate stream type (byte vs character)  
✅ Use buffered streams for better performance  
✅ Handle exceptions properly  
✅ Close streams after use  
✅ Use append mode when needed (`new FileWriter(file, true)`)  

---

## Practice Projects

1. **Image Editor**: Read an image file, manipulate RGB values, write modified image
2. **Text Analyzer**: Read a text file, count words, lines, characters
3. **Log File Manager**: Append log entries with timestamps
4. **File Encryption**: Read file, encrypt content, write to new file
5. **CSV Parser**: Read CSV file, parse data, perform operations
6. **File Backup**: Copy files from one directory to another

---

## Additional Resources

- Oracle Java Documentation: [java.io package](https://docs.oracle.com/javase/8/docs/api/java/io/package-summary.html)
- Explore all subclasses of InputStream, OutputStream, Reader, and Writer
- Check out NIO (New I/O) package for advanced file operations

---

**Note**: This course covered the traditional I/O (java.io). For modern applications, also explore:
- **NIO.2** (java.nio.file) - More powerful file operations
- **Path** and **Files** classes - Better alternative to File class
