# 22. Cookie Attacks

## Sensitive data inside cookies

Inspect the cookie content. Some sites embed sensitive data such as personally identifiable information directly inside the cookie. The data may be encoded with one of the following:

```
=> base64
=> base32
=> hex
=> URL-encode (the same value may be encoded several times)
```

## Buffer overflow in cookie parsing

Provide a deliberately oversized cookie (e.g. send 32 KB or 64 KB through Burp Intruder) and check the response for crashes, stack traces, or `5xx` errors.

## Arbitrary cookie injection

Try to inject **new** cookies that the victim's browser will accept. There are two common entry points:

- A page that reflects user input into a `Set-Cookie` response header.
- An open-redirect / login flow that lets you choose the cookie's value.

If you can plant cookies on the victim's browser you may be able to:

- Pre-authenticate the victim into your account (session fixation).
- Inject a cookie value that triggers another vulnerability (CRLF, XSS, server-side parsing bug).

## Mass assignment via cookies

Some applications populate the user model by reading values directly from cookies. Try to overwrite fields like `is_admin`, `role`, `userId`, etc.

```
Cookie: is_admin=true; role=admin; user_id=1
```

## Cookie bomb (Denial of Service)

Many web servers reject requests once the cookie size exceeds a threshold (often 4 KB or 8 KB). If you can plant a large cookie on the victim's browser (via XSS, subdomain compromise, or cookie injection), every subsequent request to the target will be rejected — effectively denying the victim access until they clear the cookie manually.

## SQL injection in cookies

Cookies are often passed straight into SQL queries. Try the standard SQLi payloads inside cookie values:

```
Cookie: session=' OR '1'='1' --
Cookie: user_id=1 UNION SELECT password FROM users --
```

## Cookie parameter pollution

Send the same cookie name twice with different values:

```
Cookie: role=user; role=admin
Cookie: user_id=1; user_id=2
```

The server may use the first value while the WAF / auth filter checks the second (or vice versa).

## Authentication bypass

If the cookie contains the username or user ID directly, simply change the value:

```
Cookie: user=admin
Cookie: user_id=1
Cookie: role=admin
```

## Cookie XSS

Whenever a cookie value is reflected anywhere on the page, try injecting an XSS payload inside the cookie.

## Session management

- The session cookie should be invalidated after logout.
- The session cookie should be regenerated after privilege change (login, password change, role change).
- The session cookie should expire after a reasonable amount of inactivity.
- The session cookie should have the `HttpOnly`, `Secure`, and `SameSite` attributes set.

## Privilege escalation (horizontal / vertical)

Inspect cookies that look like role indicators:

```
Cookie: role=user      ->  role=admin
Cookie: privilege=low  ->  privilege=high
Cookie: tier=free      ->  tier=premium
```

## Session puzzling

Some applications place sensitive values (account ID, role, plan tier, etc.) into cookies that are reused across different flows (login, password reset, email change). If the same cookie is read in different contexts, you may be able to set it during one flow and ride it into another.

## Python-flavoured code injection

If the application is written in Python and uses `pickle` or `eval` on cookie values, you can achieve RCE by serializing a malicious payload into the cookie. Look for cookies that look like base64-encoded pickle blobs and try the standard `pickle.loads` gadget.
