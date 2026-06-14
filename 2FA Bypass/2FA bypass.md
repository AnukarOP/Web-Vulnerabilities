# 7. 2FA Bypass

## Incomplete 2FA verification

Two-factor authentication is "incomplete" when, after the first login step, the website does not properly verify that the **same** user is the one completing the second step. Example — first step:

```
POST /login-steps/first HTTP/1.1
Host: vulnerable-website.com
...
username=carlos&password=qwerty
```

Before reaching the second step, a cookie tied to the account is issued:

```
HTTP/1.1 200 OK
Set-Cookie: account=carlos

GET /login-steps/second HTTP/1.1
Cookie: account=carlos
```

When the verification code is submitted, the request relies on this cookie to know which account is being logged into:

```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos
...
verification-code=123456
```

An attacker can log in with their own credentials, then change the value of the `account` cookie to **any victim username** while sending the verification code:

```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=victim-user
...
verification-code=123456
```

## Clickjacking on the disable-2FA page

Try to iframe the page that lets a user disable 2FA. If the iframe loads, set up a social-engineering / clickjacking attack to trick the victim into disabling their own 2FA.

## Response manipulation

Inspect the 2FA response. If you see something like `"Success":false`, change it to `"Success":true` and observe whether 2FA is bypassed.

## Status-code manipulation

If the 2FA response returns a `4xx` status (e.g. `401`, `402`), change it to `200 OK` and observe whether 2FA is bypassed.

## 2FA code reuse

1. Request a 2FA code and use it.
2. Try to use the **same** code again — if it works, that's a finding.

Also try:

- Request several 2FA codes in a row and check whether previously issued codes still work (they should be invalidated).
- Wait a long time (e.g. 1 day) and try a previously used code. If it still works, an attacker has more than enough time to brute force a 6-digit code.

## CSRF on the disable-2FA endpoint

1. Create two accounts: attacker and victim.
2. Log into the attacker account, intercept the "Disable 2FA" request in Burp, generate a CSRF PoC, and save it as an HTML file.
3. In a different browser (or private tab) log in as the victim and open the CSRF PoC. If 2FA is disabled, you have a 2FA bypass via CSRF.

## Backup-code abuse

The same techniques used against the OTP code (response manipulation, status-code swap, brute force, …) also apply to **backup recovery codes** that the application provides. Use them to bypass / reset / disable 2FA.

## Enabling 2FA does not expire previous sessions

1. Log into the same account on two different browsers.
2. Enable 2FA on the first browser.
3. From the second browser (where you were already logged in before 2FA was enabled), reload the page. If the second session is **not** invalidated, an attacker who hijacked a pre-2FA session can keep using it without ever providing 2FA.

## 2FA Referer-check bypass

Try going directly to the page that loads after the 2FA step, or to any other authenticated page. If that fails, set the `Referer` header to the URL of the 2FA page — the application may believe the request comes after a successful 2FA challenge.

## 2FA code leakage in the response

Capture the request that **issues** the 2FA code and inspect the response. The OTP itself is sometimes returned in the response body or in a header.

## JavaScript file analysis

When the 2FA code is requested, analyse all JS files referenced in the response. They occasionally contain information that helps bypass the OTP step (validation logic, secret prefixes, etc.).

## Lack of brute-force protection

This is a security-misconfiguration class of issue: no rate limit, no brute-force protection, etc.

1. Request a 2FA code and capture the verification request.
2. Replay the verification request 100–200 times. If there is no throttling, the OTP can be brute-forced.
3. On the verify-2FA page, try brute-forcing the code outright (especially when it is only 4 or 5 digits).

If old OTPs do not expire when new ones are issued, you can request many OTPs and brute force them in parallel — chances of a hit go up dramatically and the attack often finishes faster.

## Password reset / email change disables 2FA

If you can change the victim's email or trigger a password reset for them (or socially engineer them into doing it), check whether 2FA is automatically disabled afterwards. For some organizations this is a serious finding; for others it is by design — depends on context.

## Missing 2FA-code integrity validation

1. Request a 2FA code from the attacker account.
2. Use that valid attacker code to complete 2FA on the **victim's** session and see whether the application notices that the code does not belong to the victim.

## Direct request

Browse straight to the post-2FA page (or any other authenticated page). See whether the 2FA gate can be bypassed entirely. Combine with the `Referer`-spoofing trick if needed.

## Reusing tokens

You may be able to reuse a token that was already accepted by the application during a previous authentication.

## Sharing unused tokens

Try issuing a token in your own account and using it to bypass 2FA on a different account.

## Leaked token

Is the 2FA token leaked anywhere in the response (body, headers, set-cookies, etc.)?

## Session permission elevation

1. Using a single session, start the login flow for both your own account and the victim's account.
2. When both reach the 2FA step, complete 2FA only with your own account — but do not advance to the next step.
3. Instead try to advance to the next step **as the victim**.
4. If the back-end only sets a boolean like `2fa_passed=true` on your session, you can ride that flag to bypass the victim's 2FA.

## Password-reset auto-login

Almost all web applications auto-login the user after a successful password reset. Check whether the reset-link email can be **reused** multiple times, even after the victim later changes their email — that pattern frequently survives audits.

## Lack of rate limiting on OTP entry

Is there a limit on the number of codes you can submit? Watch out for "silent" rate limits — always submit a few wrong codes followed by the genuine one to confirm whether the rate limit just stops verifying, instead of returning an error.

## Flow rate limit but no per-attempt rate limit

Some applications throttle the speed of the brute-force flow but never lock you out (you must brute force slowly — one thread, with a wait between attempts). Given enough time, the OTP can still be cracked.

## Resend-code resets the limit

There may be a rate limit on attempts, but issuing a new code (or "resend code") resets the counter — and the app sometimes resends the **same** code. Loop: resend → brute force a few attempts → resend → brute force → ... until the OTP is hit.

## [Client-side rate-limit bypass](../Rate%20limit/bypass%20rate%20limit.md)

## No rate limit on in-account actions

Some applications enforce 2FA on account-internal actions (change email, change password, etc.). Even when there is rate limiting on **login**, there is often **none** on the in-account 2FA steps.

## No rate limit on resending the code via SMS

You may not be able to bypass 2FA, but you can burn through the company's SMS budget. If the account is not blocked, you can request as many codes as you want.

## Infinite OTP regeneration

If you can keep generating new OTPs forever, and each new OTP allows 4–5 wrong attempts before invalidation, and the OTP is short (e.g. 4 digits), you can pin a fixed candidate set and keep trying it across regenerations until it lands. With short OTPs the same code is sometimes re-issued multiple times, so include duplicate-detection in your tooling.

## Guessable "remember me" cookie

If "Remember me" relies on a new cookie that contains a guessable value, try to predict it.

## IP-bound "remember me"

If "Remember me" is bound to the user's IP, find the victim's IP (recon, leak, OSINT) and spoof it via the `X-Forwarded-For` header.
