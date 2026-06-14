# 2. Authentication

## Authentication Bypass

### Initial checks

- Check whether authentication URLs are reachable directly without an active session.
- If a privileged URL can be stolen, guessed, or brute-forced, account takeover becomes possible.

### Captcha bypass — `X-Forwarded-For`

Inject a random value in the `X-Forwarded-For` header to bypass per-IP captcha enforcement.

### Missing password re-confirmation

Check whether the application asks for the password again before sensitive actions, especially:

- Email change
- Password change
- Account deletion
- Managing 2FA / two-factor authentication

### No rate limiting on a regular page

1. Open a page on the site through Burp's proxy and capture the request.
2. Send the request to Intruder.
3. Replay the same request 20–30 times.
4. Confirm that all requests are accepted without throttling.

### No rate limiting / no captcha on the login page

1. Visit the login page and send a deliberately failed login to Burp Intruder.
2. Set a wordlist of random passwords on the password parameter.
3. Confirm the response does not change after 20–30 attempts and the account is not locked.

### Username / Email enumeration

Visit the password-reset / login / signup pages — anywhere that accepts a username or email.

1. Enter a known existing username/email with a wrong password and note the error message.
2. Enter a clearly non-existent username/email (e.g. mash the keyboard) and note the error message.
3. If the messages differ — for example "incorrect password" vs "user does not exist" — the application is leaking which accounts exist. With user enumeration confirmed you can move on to brute force / credential stuffing.

### Weak password policy

Try registering with passwords that are:

- Numeric only
- Lowercase only
- Common dictionary passwords (`password`, `123456`, `qwerty`)
- Very short

If the application accepts these, the password policy is weak.

### Weak registration / authentication over HTTP

When registering or logging in, intercept the request in Burp and confirm that traffic is sent over HTTPS, not plain HTTP. If credentials cross the wire in plaintext, that is a finding on its own.
