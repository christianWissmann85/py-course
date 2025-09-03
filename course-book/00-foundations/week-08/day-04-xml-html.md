# Week 8, Day 4: XML & HTML Processing

## 🎯 Learning Objectives
- [ ] Understand the basic structure of XML documents.
- [ ] Use Python's built-in `xml.etree.ElementTree` module to parse and query XML data.
- [ ] Write basic XPath queries to find specific elements in an XML tree.
- [ ] Understand the difference between XML and HTML and know which tools to use for each.

## 📚 Concepts

### 1. XML Basics
XML (eXtensible Markup Language) is a markup language designed to store and transport data. Unlike HTML, it doesn't have predefined tags. You define your own tags to describe your data. It represents data in a tree structure.

```xml
<!-- A simple XML document -->
<config>
    <user>
        <name>Alice</name>
        <email>alice@example.com</email>
    </user>
    <settings>
        <theme>dark</theme>
        <notifications enabled="true" />
    </settings>
</config>
```
- **Tags**: `<tag>` and `</tag>` define elements.
- **Elements**: The content between tags (e.g., `<name>Alice</name>`).
- **Attributes**: Key-value pairs inside the opening tag (e.g., `enabled="true"`).

### 2. `xml.etree.ElementTree`
Python's standard library includes `ElementTree` for working with XML. It represents the entire XML document as a tree, and each element as a node.

- `ET.parse(filepath)`: Parses an XML document from a file.
- `ET.fromstring(xml_string)`: Parses an XML document from a string.
- `root.findall(path)`: Finds all matching sub-elements using a path.
- `element.text`: The text content of an element.
- `element.attrib`: A dictionary of the element's attributes.
- `element.get(key)`: Gets an attribute value by key.

```python
import xml.etree.ElementTree as ET

xml_string = """
<config>
    <user>
        <name>Alice</name>
    </user>
    <settings>
        <notifications enabled="true" />
    </settings>
</config>
"""

root = ET.fromstring(xml_string)

# Find the 'name' element and get its text
name_element = root.find('user/name')
if name_element is not None:
    print(f"User name: {name_element.text}")

# Find the 'notifications' element and get its attribute
notif_element = root.find('settings/notifications')
if notif_element is not None:
    print(f"Notifications enabled: {notif_element.get('enabled')}")
```

### 3. XPath Queries
The path string used in `.find()` and `.findall()` is a simplified version of **XPath**. XPath is a powerful language for navigating elements and attributes in an XML document.
- `.` selects the current node.
- `//` selects nodes from the current node that match the selection, no matter where they are.
- `[@attrib]` selects elements with a specific attribute.

```python
# Find all elements with an 'enabled' attribute, anywhere in the document
all_enabled = root.findall(".//*[@enabled]")
for elem in all_enabled:
    print(f"Found an element with 'enabled' attribute: {elem.tag}")
```
Full XPath support requires third-party libraries like `lxml`.

### 4. HTML Processing with BeautifulSoup
HTML looks like XML, but it's often "messy" and doesn't follow strict XML rules. Trying to parse real-world HTML with an XML parser will often fail.

The standard tool for parsing HTML in Python is a third-party library called **BeautifulSoup**. It's designed to be lenient and handle malformed HTML gracefully.

```python
# You need to install it: poetry add beautifulsoup4
from bs4 import BeautifulSoup

html_doc = """
<html><head><title>My Page</title></head>
<body><p class="title"><b>My Page</b></p>
<p class="story">Some text and a <a href="http://example.com/link" class="link" id="link1">Link</a>.</p>
</body></html>
"""

soup = BeautifulSoup(html_doc, 'html.parser')

# Find elements by tag name
print(soup.title.string) # My Page

# Find elements by CSS class
link = soup.find('a', class_='link')
if link:
    print(link['href']) # http://example.com/link
```
BeautifulSoup is essential for any web scraping task.

### 5. Generating XML
You can also use `ElementTree` to build XML documents programmatically.

```python
import xml.etree.ElementTree as ET

# Create the root element
root = ET.Element("data")

# Create a sub-element
item = ET.SubElement(root, "item")
item.set("name", "Laptop") # Add an attribute
item.text = "A powerful laptop" # Set text content

# Convert the tree to a string
xml_string = ET.tostring(root, encoding="unicode")
print(xml_string)
```

## 🔹 Quick Exercise

Complete the `parse_config` function below. It should parse an XML string and extract the `name` and `theme` values.

```python
import xml.etree.ElementTree as ET
from typing import Dict, Optional

def parse_config(xml_string: str) -> dict[str, str | None]:
    """Parses an XML config string and returns a dictionary of settings."""
    root = ET.fromstring(xml_string)

    name_element = root.find('user/name')
    name = name_element.text if name_element is not None else None

    theme_element = root.find('settings/theme')
    theme = theme_element.text if theme_element is not None else None

    return {"name": name, "theme": theme}

# --- Test ---
config_xml = """
<config>
    <user>
        <name>Alice</name>
        <email>alice@example.com</email>
    </user>
    <settings>
        <theme>dark</theme>
    </settings>
</config>
"""
config = parse_config(config_xml)
assert config == {"name": "Alice", "theme": "dark"}
print("XML parsing successful!")
```

## 📝 Daily Assignment
**Goal**: Build a tool that reads data from one XML format and writes it to another.

1.  **Create Project Files**:
    -   `my_first_poetry_app/xml_transformer.py`
    -   A sample input file, `input_data.xml`.
2.  **Define Input XML Format**: Create `input_data.xml` with a list of products.
    ```xml
    <products>
        <product id="p1">
            <name>Laptop</name>
            <price currency="USD">1200.00</price>
        </product>
        <product id="p2">
            <name>Mouse</name>
            <price currency="USD">25.00</price>
        </product>
    </products>
    ```
3.  **Define Output XML Format**: Your goal is to transform the input into this format:
    ```xml
    <inventory>
        <item product-id="p1" name="Laptop" price-usd="1200.00" />
        <item product-id="p2" name="Mouse" price-usd="25.00" />
    </inventory>
    ```
4.  **Implement the Transformer**:
    -   Write a function `transform_xml(input_path: str, output_path: str) -> None`.
    -   **Parse**: Use `ET.parse()` to read and parse `input_data.xml`.
    -   **Build**: Create a new XML tree in the target format.
        -   Create the `<inventory>` root element.
        -   Loop through each `<product>` in the source tree.
        -   For each product, create a new `<item>` element.
        -   Extract the `id`, `name`, and `price` from the source product and set them as *attributes* on the new `<item>` element.
    -   **Write**: Use `ET.ElementTree(new_root).write(output_path)` to save the new XML file.
5.  **`main()` function**:
    -   Call your `transform_xml` function.
    -   Read the generated output file and print its content to verify it's correct.
6.  **Verify**: Run the script, type checker, and linter.

## ⚠️ Common Mistakes
- **Using XML parsers for HTML**: This will often fail on real-world websites. HTML can be messy, and parsers like BeautifulSoup are designed to handle it. Use the right tool for the job.
- **Ignoring namespaces**: Real-world XML often uses namespaces (e.g., `<ns1:tag>`). This complicates XPath queries. You have to register the namespace with a prefix to search correctly. This is an advanced topic but a common stumbling block.
- **Forgetting `.text`**: `root.find('name')` returns the *element* object. To get the content inside, you need to access its `.text` attribute.
- **XPath complexity**: The simplified XPath in `ElementTree` is limited. For complex queries (e.g., finding elements based on their text content), you'll need to use a more powerful library like `lxml`.

## 📖 Further Reading
- [Python Docs: `xml.etree.ElementTree`](https://docs.python.org/3/library/xml.etree.elementtree.html)
- [Real Python: Processing XML in Python](https://realpython.com/python-xml-parser/)
- [Beautiful Soup Documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
