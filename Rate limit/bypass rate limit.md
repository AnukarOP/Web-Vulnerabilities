# 19. Rate Limit Bypass

## Where to look for these bugs

```
- Login
- Password reset
- 2FA / OTP verification
- Confirmation / verification codes
- Sign up
```

## Use null / control characters

```
%00, %0d%0a, %09, %0C, %20, %0
```

If you brute-forced `abc@xyz.com` and got blocked, try `abc@xyz.com%00`. The application sees a different value but the back-end often normalizes it to the same email.

## Host header injection

- `Host: www.newsite.com`
- `Host: localhost`
- `Host: 127.0.0.1`

## Rotate cookies / sessions

For example, if you get blocked after 15 requests, change the session cookie on the 14th request and continue — the rate limit counter is often tied to the session, not the user.

## Spoof originating IP via headers

```
X-Forwarded: <IP>
X-Forwarded-For: <IP>
X-Forwarded-Host: <IP>
X-Client-IP: <IP>
X-Remote-IP: <IP>
X-Remote-Addr: <IP>
X-Host: <IP>
X-Originating-IP: <IP>
```

## Double X-Forwarded-For

Add two `X-Forwarded-For` headers — some back-ends only inspect the first / last and the WAF inspects the other:

```
X-Forwarded-For: <attacker IP>
X-Forwarded-For: 198.168.43.1
```