# 8. Admin Panel Bypass

## Default credentials

```
admin:admin
admin:password
author:author
administrator:password
admin123:password
username:pass12345
... and many more — default-credential lists are huge.
```

## Bypass via SQL injection

Inject SQLi payloads into the username or password fields.

```
=> error-based
=> time-based
```

## Bypass via Cross-Site Scripting (XSS)

Inject XSS payloads into the username or password fields, including encoded variants:

```
=> URL-encoded
=> Base64-encoded
```

## Response manipulation

Change the response status code or body, e.g.:

```
200    => 302
failed => success
error  => success
403    => 200
403    => 302
false  => true
```

## Bypass via brute force

```
https://medium.com/@uttamgupta_/1-how-to-perform-login-brute-force-using-burp-suite-9d06b67fb53d
https://medium.com/@uttamgupta_/broken-brute-force-protection-ip-block-aae835895a74
```

## Bypass via directory fuzzing

Use this wordlist:

```
https://github.com/six2dez/OneListForAll
```

## Removing a parameter from the request

When you submit incorrect credentials, the application returns errors such as:

- "Username and password are incorrect / do not match"
- "Password is incorrect for this username"

Intercept the login request in Burp's proxy tab, **delete the `password` parameter entirely**, and forward the request. If the back-end only checks that the username exists when no password is sent, it may log you in. This happens when the server does not safely parse the request.

## Inspect JavaScript files referenced from the login page

JavaScript files often contain useful paths or even hardcoded credentials.

## Inspect HTML comments on the page

Comments occasionally contain credentials, hidden URLs, or notes left by developers.

## PHP comparison errors

```json
user[]=a&pwd=b , user=a&pwd[]=b , user[]=a&pwd[]=b
```

Switch the `Content-Type` to `application/json` and submit JSON values containing booleans (e.g. `true`):

If the response says POST is not supported, switch the method to `GET` while keeping `Content-Type: application/json`.

## Node.js parsing-error abuse

[Read this article](https://flattsecurity.medium.com/finding-an-unseen-sql-injection-by-bypassing-escape-functions-in-mysqljs-mysql-90b27f6542b4).

Node.js converts certain payloads into a query like the one below, which makes the password check always evaluate to true:

```
SELECT id, username, left(password, 8) AS snipped_password, email
FROM accounts
WHERE username='admin' AND password=password=1;
```

If you can submit a JSON object, try `"password":{"password": 1}` to bypass login. Remember that the username must already exist in the database.

Adding the option `stringifyObjects: true` when calling `mysql.createConnection` blocks this behaviour by refusing object parameters.
