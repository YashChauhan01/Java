# Complete XML Tutorial Notes

## Table of Contents
1. [Introduction to XML](#introduction-to-xml)
2. [XML vs HTML](#xml-vs-html)
3. [XML Tree Structure](#xml-tree-structure)
4. [XML Syntax Rules](#xml-syntax-rules)
5. [XML Elements](#xml-elements)
6. [XML Attributes](#xml-attributes)
7. [XML Namespaces](#xml-namespaces)
8. [Displaying XML](#displaying-xml)
9. [XML HTTP Request](#xml-http-request)
10. [XML Parser](#xml-parser)
11. [XML DOM](#xml-dom)
12. [XPath](#xpath)
13. [XSLT](#xslt)
14. [XQuery](#xquery)
15. [XLink and XPointer](#xlink-and-xpointer)

---

## Introduction to XML

### What is XML?
**XML** stands for **eXtensible Markup Language**

**Key Features:**
- Not a programming language - it's a markup language
- Designed for **storing and transporting data**
- Does NOT display or format data
- Not used for fancy output like CSS or JavaScript
- Self-descriptive language
- Platform and software independent

### Purpose of XML
- Store data in an organized, structured manner
- Make data readable by both humans and machines
- Provide metadata (data about data)
- Enable data sharing across different platforms and systems

### Basic XML Example
```xml
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
```

### Why Use XML?
1. **Simplifies data sharing** - Works across platforms and languages
2. **Simplifies data transport** - Easy to move between systems
3. **Simplifies platform changes** - Works on any operating system
4. **Simplifies data availability** - Accessible to various devices
5. **Plain text format** - Universal readability
6. **Future-proof** - Old XML files work with new systems

---

## XML vs HTML

### Key Differences

| Feature | XML | HTML |
|---------|-----|------|
| **Purpose** | Carry and store data | Display data |
| **Focus** | What data IS | How data LOOKS |
| **Tags** | User-defined (custom) | Predefined |
| **Structure** | Data storage | Data presentation |
| **Flexibility** | Highly extensible | Limited to predefined tags |

### XML Example
```xml
<book>
  <title>XML Guide</title>
  <author>John Doe</author>
  <price>29.99</price>
</book>
```

### HTML Example
```html
<h1>XML Guide</h1>
<p>Author: John Doe</p>
<p>Price: $29.99</p>
```

**Important:** XML and HTML can work together - XML stores the data, HTML displays it.

---

## XML Tree Structure

### Hierarchical Structure
XML documents form a **tree structure** starting from a root element and branching to child elements.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
  <book category="cooking">
    <title lang="en">Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <year>2005</year>
    <price>30.00</price>
  </book>
  <book category="children">
    <title lang="en">Harry Potter</title>
    <author>J.K. Rowling</author>
    <year>2005</year>
    <price>29.99</price>
  </book>
</bookstore>
```

### Tree Components
- **Root Element**: `<bookstore>` (parent of all elements)
- **Child Elements**: `<book>` (children of bookstore)
- **Sub-child Elements**: `<title>`, `<author>`, etc. (children of book)
- **Attributes**: `category="cooking"`, `lang="en"`
- **Text Content**: The actual data within tags

### Parent-Child Relationships
- **Parent**: Contains other elements
- **Child**: Contained within another element
- **Siblings**: Elements at the same level
- All elements can have text content and sub-elements

---

## XML Syntax Rules

### 1. XML Prologue (Optional but Recommended)
```xml
<?xml version="1.0" encoding="UTF-8"?>
```
- Must be the first line if included
- Specifies XML version and character encoding
- **UTF-8** is the default encoding

### 2. Root Element (Required)
**Every XML document MUST have ONE root element**
```xml
<root>
  <!-- All other elements go here -->
</root>
```

### 3. Closing Tags (Required)
**All tags MUST be closed**
```xml
<!-- Correct -->
<message>Hello</message>

<!-- Incorrect in XML -->
<message>Hello
```

### 4. Case Sensitivity
**XML tags are case-sensitive**
```xml
<!-- These are DIFFERENT tags -->
<Message>Text</Message>
<message>Text</message>
```

### 5. Proper Nesting
**Elements must be properly nested**
```xml
<!-- Correct -->
<b><i>Bold and Italic</i></b>

<!-- Incorrect -->
<b><i>Bold and Italic</b></i>
```

### 6. Attribute Values Must Be Quoted
```xml
<!-- Correct -->
<book category="fiction">

<!-- Incorrect -->
<book category=fiction>
```

### 7. Entity References for Special Characters

| Character | Entity Reference | Usage |
|-----------|-----------------|--------|
| < | `&lt;` | Less than |
| > | `&gt;` | Greater than |
| & | `&amp;` | Ampersand |
| ' | `&apos;` | Apostrophe |
| " | `&quot;` | Quotation mark |

**Example:**
```xml
<message>Salary &lt; 1000</message>
```

### 8. Comments
```xml
<!-- This is a comment -->
<!-- Comments can span
     multiple lines -->
```
**Note:** Cannot use `--` inside comments

### 9. White Space Preservation
XML preserves white spaces (unlike HTML)

---

## XML Elements

### Definition
An XML element includes everything from the start tag to the end tag.

### Element Structure
```xml
<elementName>Content</elementName>
```

### Types of Element Content
1. **Text content**
```xml
<title>XML Tutorial</title>
```

2. **Other elements**
```xml
<book>
  <title>XML Guide</title>
  <author>Jane Smith</author>
</book>
```

3. **Mixed content**
```xml
<description>This is <b>bold</b> text</description>
```

4. **Empty elements**
```xml
<linebreak/>
<!-- OR -->
<linebreak></linebreak>
```

### Naming Rules
1. Must start with a **letter** or **underscore**
2. Cannot start with "XML" (reserved)
3. Can contain letters, digits, hyphens, underscores, periods
4. **Cannot contain spaces**
5. Case-sensitive

### Best Naming Practices
✅ **Good:**
- `<book_title>`
- `<firstName>`
- `<date-of-birth>`

❌ **Avoid:**
- `<the-title-of-the-book>` (too long)
- `<first-name>` (minus sign can be confused with subtraction)
- `<first.name>` (period can be confused with object property)

### Extensibility
XML elements are extensible - you can add new elements without breaking existing code:
```xml
<!-- Original -->
<note>
  <to>Tove</to>
  <from>Jani</from>
  <body>Meeting at 3pm</body>
</note>

<!-- Extended (still compatible) -->
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <date>2025-01-15</date>
  <body>Meeting at 3pm</body>
</note>
```

---

## XML Attributes

### Definition
Attributes provide additional information about elements.

### Syntax
```xml
<element attribute="value">Content</element>
```

### Examples
```xml
<person gender="female">
  <firstname>Anna</firstname>
  <lastname>Smith</lastname>
</person>

<!-- Multiple attributes -->
<book category="children" language="en">
  <title>Harry Potter</title>
</book>
```

### Attribute vs Element
**Same information, two approaches:**

**Using Attribute:**
```xml
<person gender="female">
  <firstname>Anna</firstname>
  <lastname>Smith</lastname>
</person>
```

**Using Element:**
```xml
<person>
  <gender>female</gender>
  <firstname>Anna</firstname>
  <lastname>Smith</lastname>
</person>
```

**Using Expanded Elements (Best for complex data):**
```xml
<person>
  <gender>female</gender>
  <firstname>Anna</firstname>
  <lastname>Smith</lastname>
</person>
```

### When to Use Attributes vs Elements

**Use Attributes for:**
- Metadata (data about data)
- IDs and references
- Simple, single values

**Use Elements for:**
- Actual data content
- Data that might need to expand
- Multiple values
- Tree structures

### Attribute Rules
1. Values must be quoted (single or double quotes)
2. Cannot contain multiple values
3. Cannot contain tree structures
4. Not easily expandable

### Best Practice: Use Attributes for Metadata
```xml
<note id="501">
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget!</body>
</note>
```

Here, `id` is metadata - it identifies the note but isn't part of the actual note content.

---

## XML Namespaces

### Purpose
Avoid element name conflicts when mixing XML documents from different sources.

### Name Conflict Example
```xml
<!-- HTML Table -->
<table>
  <tr>
    <td>Apples</td>
    <td>Bananas</td>
  </tr>
</table>

<!-- Furniture Table -->
<table>
  <name>Coffee Table</name>
  <width>80</width>
  <length>120</length>
</table>
```
**Problem:** Both use `<table>` but mean different things!

### Solution: Using Prefixes
```xml
<h:table>
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
  </h:tr>
</h:table>

<f:table>
  <f:name>Coffee Table</f:name>
  <f:width>80</f:width>
  <f:length>120</f:length>
</f:table>
```

### Declaring Namespaces
**Syntax:**
```xml
xmlns:prefix="URI"
```

**Example:**
```xml
<root xmlns:h="http://www.w3.org/TR/html4/"
      xmlns:f="http://www.w3schools.com/furniture">
  
  <h:table>
    <h:tr>
      <h:td>Apples</h:td>
    </h:tr>
  </h:table>
  
  <f:table>
    <f:name>Coffee Table</f:name>
    <f:width>80</f:width>
  </f:table>
  
</root>
```

### Default Namespace
Avoid prefixes on all child elements:
```xml
<table xmlns="http://www.w3.org/TR/html4/">
  <tr>
    <td>Apples</td>
    <td>Bananas</td>
  </tr>
</table>
```

### Namespace URI
- The URI doesn't need to be a real web page
- It's just a unique identifier
- Companies often use their domain as namespace

---

## Displaying XML

### Browser Display
Raw XML files can be viewed in all major browsers, but:
- Displays as a tree structure
- Shows tags and content
- Color-coded elements
- Collapsible/expandable nodes

### Example Browser Output
```xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me!</body>
</note>
```

### Important Notes
- XML does NOT contain display information
- Browsers show the raw XML structure
- To display nicely, use CSS or XSLT
- Viewing source shows the complete XML

### Invalid XML Display
If XML has errors, browsers display error messages:
```
This page contains the following errors:
error on line 5 at column 3: Opening and ending tag mismatch
```

---

## XML HTTP Request

### What is XMLHttpRequest?
Built-in browser object that allows requesting data from a server without reloading the page.

### Why It's Important
- Update web pages without reloading
- Request data from server after page loads
- Receive data from server
- Send data to server in background

### Basic Example
```javascript
// Create XMLHttpRequest object
var xhttp = new XMLHttpRequest();

// Define callback function
xhttp.onreadystatechange = function() {
  if (this.readyState == 4 && this.status == 200) {
    document.getElementById("demo").innerHTML = this.responseText;
  }
};

// Open and send request
xhttp.open("GET", "filename.xml", true);
xhttp.send();
```

### Complete Example
```html
<!DOCTYPE html>
<html>
<body>

<div id="demo">
  <h2>Click to load content</h2>
</div>

<button onclick="loadXMLDoc()">Load XML</button>

<script>
function loadXMLDoc() {
  var xhttp = new XMLHttpRequest();
  
  xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
      document.getElementById("demo").innerHTML = this.responseText;
    }
  };
  
  xhttp.open("GET", "xmlinfo.txt", true);
  xhttp.send();
}
</script>

</body>
</html>
```

### ReadyState Values
- **0**: Request not initialized
- **1**: Server connection established
- **2**: Request received
- **3**: Processing request
- **4**: Request finished, response ready

### Status Codes
- **200**: "OK"
- **404**: Page not found
- **500**: Internal server error

---

## XML Parser

### What is an XML Parser?
Software that reads XML and converts it to a format programs can work with (XML DOM).

### DOMParser
Built-in JavaScript parser for converting text to XML DOM.

### Basic Usage
```javascript
// XML as text string
var text = `
<bookstore>
  <book>
    <title>Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <year>2005</year>
  </book>
</bookstore>`;

// Create parser
var parser = new DOMParser();

// Parse text to XML
var xmlDoc = parser.parseFromString(text, "text/xml");

// Extract data
var title = xmlDoc.getElementsByTagName("title")[0].childNodes[0].nodeValue;
console.log(title); // "Everyday Italian"
```

### Complete Example
```html
<p id="demo"></p>

<script>
var text = 
`<bookstore>
  <book>
    <title>Everyday Italian</title>
    <author>Giada De Laurentiis</author>
  </book>
</bookstore>`;

var parser = new DOMParser();
var xmlDoc = parser.parseFromString(text, "text/xml");

document.getElementById("demo").innerHTML =
  xmlDoc.getElementsByTagName("title")[0].childNodes[0].nodeValue;
</script>
```

### XMLHttpRequest with Built-in Parser
```javascript
var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
  if (this.readyState == 4 && this.status == 200) {
    myFunction(this);
  }
};
xhttp.open("GET", "cd_catalog.xml", true);
xhttp.send();

function myFunction(xml) {
  var xmlDoc = xml.responseXML; // Built-in parser
  var x = xmlDoc.getElementsByTagName("ARTIST");
  
  var txt = "";
  for (var i = 0; i < x.length; i++) {
    txt += x[i].childNodes[0].nodeValue + "<br>";
  }
  document.getElementById("demo").innerHTML = txt;
}
```

---

## XML DOM

### What is XML DOM?
**Document Object Model** - a standard for accessing and manipulating XML documents.

### DOM Structure
Presents XML as a tree of objects with properties and methods.

### HTML DOM vs XML DOM
- **HTML DOM**: Accesses HTML elements
- **XML DOM**: Accesses XML elements

### Accessing Elements

**HTML DOM Example:**
```javascript
document.getElementById("demo").innerHTML = "Hello World";
```

**XML DOM Example:**
```javascript
// Get element by tag name
var x = xmlDoc.getElementsByTagName("title")[0];

// Access child node value
var text = x.childNodes[0].nodeValue;
```

### Complete DOM Example
```javascript
var text = `
<bookstore>
  <book category="cooking">
    <title>Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <year>2005</year>
    <price>30.00</price>
  </book>
</bookstore>`;

var parser = new DOMParser();
var xmlDoc = parser.parseFromString(text, "text/xml");

// Access by tag name and index
var title = xmlDoc.getElementsByTagName("title")[0]
                  .childNodes[0].nodeValue;

console.log(title); // "Everyday Italian"
```

### DOM Navigation Methods
- `getElementsByTagName()` - Get elements by tag name
- `childNodes` - Access child nodes
- `nodeValue` - Get text content
- `getAttribute()` - Get attribute value

### Index Notation
```xml
<bookstore>
  <book>...</book>     <!-- Index 0 -->
  <book>...</book>     <!-- Index 1 -->
  <book>...</book>     <!-- Index 2 -->
</bookstore>
```

```javascript
// Get first book
var book1 = xmlDoc.getElementsByTagName("book")[0];

// Get second book
var book2 = xmlDoc.getElementsByTagName("book")[1];
```

---

## XPath

### What is XPath?
Language for navigating and selecting nodes in XML documents using path expressions.

### Why XPath?
- Navigate through XML elements and attributes
- Major element in XSLT
- Used in XQuery, XPointer, XLink
- W3C recommendation

### Basic Path Expressions

| Expression | Description | Example |
|------------|-------------|---------|
| `nodename` | Selects nodes with name | `bookstore` |
| `/` | Root element | `/bookstore` |
| `//` | Any descendant | `//book` |
| `.` | Current node | `.` |
| `..` | Parent node | `..` |
| `@` | Attribute | `@category` |

### Example XML
```xml
<bookstore>
  <book category="cooking">
    <title lang="en">Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <price>30.00</price>
  </book>
  <book category="children">
    <title lang="en">Harry Potter</title>
    <author>J.K. Rowling</author>
    <price>29.99</price>
  </book>
</bookstore>
```

### XPath Expressions

**Select first book:**
```xpath
/bookstore/book[1]
```

**Select last book:**
```xpath
/bookstore/book[last()]
```

**Select first two books:**
```xpath
/bookstore/book[position()<3]
```

**Select all titles with lang attribute:**
```xpath
//title[@lang]
```

**Select titles with lang='en':**
```xpath
//title[@lang='en']
```

**Select books with price > 35:**
```xpath
/bookstore/book[price>35]
```

**Select titles where book price > 35:**
```xpath
/bookstore/book[price>35]/title
```

### XPath Functions
- `last()` - Last element
- `position()` - Position of element
- `count()` - Count elements
- `sum()` - Sum values
- `name()` - Node name

---

## XSLT

### What is XSLT?
**eXtensible Stylesheet Language Transformations** - transforms XML into HTML or other formats.

### Purpose
- Convert XML to HTML for display
- Add/remove elements
- Rearrange and sort elements
- Perform conditional operations

### Basic XSLT Structure
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" 
  xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

<xsl:template match="/">
  <html>
    <body>
      <!-- XSLT code here -->
    </body>
  </html>
</xsl:template>

</xsl:stylesheet>
```

### Example: Transform XML to HTML

**XML Data:**
```xml
<breakfast_menu>
  <food>
    <name>Belgian Waffles</name>
    <price>$5.95</price>
    <description>Two waffles with syrup</description>
    <calories>650</calories>
  </food>
  <food>
    <name>French Toast</name>
    <price>$4.50</price>
    <description>Thick slices with syrup</description>
    <calories>600</calories>
  </food>
</breakfast_menu>
```

**XSLT Transformation:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
  xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

<xsl:template match="/">
  <html>
  <body>
    <h2>Breakfast Menu</h2>
    
    <xsl:for-each select="breakfast_menu/food">
      <div style="background-color:#eeeeee; padding:10px; margin:10px;">
        <span style="font-weight:bold;">
          <xsl:value-of select="name"/>
        </span> - 
        <xsl:value-of select="price"/>
        <br/>
        <xsl:value-of select="description"/>
        <br/>
        Calories: <xsl:value-of select="calories"/>
      </div>
    </xsl:for-each>
    
  </body>
  </html>
</xsl:template>

</xsl:stylesheet>
```

### XSLT Elements

**`<xsl:for-each>`** - Loop through elements
```xml
<xsl:for-each select="breakfast_menu/food">
  <!-- Process each food item -->
</xsl:for-each>
```

**`<xsl:value-of>`** - Extract element value
```xml
<xsl:value-of select="name"/>
```

**`<xsl:if>`** - Conditional
```xml
<xsl:if test="price &gt; 5">
  <p>Premium item</p>
</xsl:if>
```

**`<xsl:sort>`** - Sort elements
```xml
<xsl:for-each select="breakfast_menu/food">
  <xsl:sort select="price"/>
  <!-- Display sorted items -->
</xsl:for-each>
```

---

## XQuery

### What is XQuery?
Query language for XML - like SQL for databases.

### Purpose
- Extract data from XML
- Query XML documents
- Perform complex searches

### Basic XQuery Syntax
```xquery
for $x in doc("books.xml")/bookstore/book
where $x/price > 30
order by $x/title
return $x/title
```

### Components

**FOR** - Loop through elements
```xquery
for $x in doc("books.xml")/bookstore/book
```

**WHERE** - Filter results
```xquery
where $x/price > 30
```

**ORDER BY** - Sort results
```xquery
order by $x/title
```

**RETURN** - Specify output
```xquery
return $x/title
```

### Example: Query CD Catalog
```xquery
for $x in doc("cd_catalog.xml")/catalog/cd
where $x/price < 10
return $x/title
```

### XQuery with XPath
XQuery uses XPath expressions:
```xquery
doc("books.xml")//book[@category='web']/title
```

### Functions
- `doc()` - Open document
- `count()` - Count results
- `avg()` - Average value
- `sum()` - Sum values
- `distinct-values()` - Unique values

---

## XLink and XPointer

### XLink - Creating Hyperlinks in XML

### What is XLink?
Creates hyperlinks in XML documents (like `<a>` tag in HTML).

### Basic XLink Syntax
```xml
<homepage 
  xmlns:xlink="http://www.w3.org/1999/xlink"
  xlink:type="simple"
  xlink:href="https://www.w3schools.com">
  Visit W3Schools
</homepage>
```

### XLink Attributes

| Attribute | Description | Values |
|-----------|-------------|--------|
| `xlink:type` | Type of link | simple, extended, locator, arc |
| `xlink:href` | URL to link to | URI |
| `xlink:show` | How to show | new, replace, embed |
| `xlink:actuate` | When to show | onLoad, onRequest |

### Complete XLink Example
```xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore xmlns:xlink="http://www.w3.org/1999/xlink">
  
  <book>
    <title>Harry Potter</title>
    <description 
      xlink:type="simple" 
      xlink:href="https://example.com/harry-potter"
      xlink:show="new">
      Click for more info
    </description>
  </book>
  
  <book>
    <title>XQuery Kickstart</title>
    <description 
      xlink:type="simple" 
      xlink:href="https://example.com/xquery"
      xlink:show="new">
      Learn XQuery
    </description>
  </book>
  
</bookstore>
```

### XPointer - Pointing to Specific Parts

### What is XPointer?
Points to specific parts of an XML document using fragment identifiers.

### XPointer with IDs

**Target Document (dogs.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<dogs>
  <dog breed="Rottweiler" id="rottweiler">
    <name>Max</name>
    <owner>Peter</owner>
  </dog>
  <dog breed="Labrador" id="labrador">
    <name>Buddy</name>
    <owner>Sarah</owner>
  </dog>
</dogs>
```

**Source Document with XPointer:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<mydogs xmlns:xlink="http://www.w3.org/1999/xlink">
  
  <mydog>
    <description
      xlink:type="simple"
      xlink:href="dogs.xml#rottweiler">
      My Rottweiler
    </description>
  </mydog>
  
  <mydog>
    <description
      xlink:type="simple"
      xlink:href="dogs.xml#labrador">
      My Labrador
    </description>
  </mydog>
  
</mydogs>
```

### Key Points
- Use `#` followed by ID to point to specific element
- Works within same document or across documents
- Requires ID attributes on target elements

### Same Document XPointer
```xml
<section id="intro">Introduction</section>
<link xlink:href="#intro">Go to intro</link>
```

---

## Summary

### XML Quick Reference

**Core Concepts:**
- XML stores and transports data
- Self-descriptive with custom tags
- Must be well-formed (proper syntax)
- Tree/hierarchical structure
- Case-sensitive
- All tags must close

**Key Technologies:**
- **XML DOM** - Access/manipulate XML
- **XPath** - Navigate XML
- **XSLT** - Transform XML to HTML
- **XQuery** - Query XML data
- **XLink/XPointer** - Create links in XML

**Best Practices:**
1. Always use UTF-8 encoding
2. Include XML prologue
3. Use descriptive element names
4. Use elements for data, attributes for metadata
5. Properly nest all elements
6. Use entity references for special characters
7. Add comments for documentation
8. Declare namespaces to avoid conflicts

**Common Use Cases:**
- Configuration files
- Data exchange between systems
- Web services (SOAP, REST)
- Document storage
- RSS/Atom feeds
- Office document formats (DOCX, XLSX)

---

## Additional Resources

- W3C XML Specification: https://www.w3.org/XML/
- W3Schools XML Tutorial: https://www.w3schools.com/xml/
- XML Validator: https://www.xmlvalidation.com/
- XPath Tester: https://www.freeformatter.com/xpath-tester.html

---

*End of XML Tutorial Notes*
