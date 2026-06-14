# 9. Business Logic Flaws

The flaws below appear when application features are implemented without thinking about how they can be abused. Each section describes a real, paid scenario.

## 1. Price manipulation on checkout

When checking out, capture the request and tamper with the `price`, `total`, `amount`, or `qty` parameter. Try negative values, zero, decimal points (e.g. `1.00` → `1.0000001`), very large values, and currency-symbol injection. Buying for `$0` or a negative amount that increases account balance is a real bug pattern.

## 2. Profile editing — change another user's profile

Edit your own profile but swap the `user_id`/`username` for the victim's. Check whether the application validates ownership.

## 3. Cart abuse — adding items to another user's cart

Capture the "Add to cart" request and replace the user/cart ID with the victim's. Or apply a coupon to the victim's cart, exhausting their single-use coupon.

## 4. Review functionality

- Review on a product you never bought.
- Edit / delete reviews left by other users.
- Use HTML / XSS in review content.
- Bypass the rate limit (multiple reviews on a single product, abusing rating averages).

## 5. Coupon-code abuse

- Use the same one-time coupon multiple times via a race condition.
- Combine coupons that should be mutually exclusive.
- Apply a vendor-only coupon to a different vendor's product.
- Use a coupon on items that are excluded.
- Stack discount codes by making parallel requests.

## 6. Delivery / shipping charge abuse

Tamper with the shipping cost in the checkout request. Try setting it to `0` or a negative value.

## 7. Currency arbitrage

If the site supports multiple currencies, try paying in one currency while billing in another (e.g. send `IRR` value but flip to `USD` at the last step). Look for missing conversion checks.

## 8. Premium-feature abuse

- Access premium features by tampering with `is_premium`, `tier`, `subscription` cookies/parameters.
- Cancel a paid plan but keep premium features active.
- Activate a free trial repeatedly.

## 9. Refund abuse

- Refund a purchase you never made.
- Refund the same purchase twice (race condition).
- Get refunded **and** keep the digital product.

## 10. Cart / wishlist abuse

- Add items to another user's cart or wishlist.
- Empty another user's cart.
- Move products between carts.

## 11. Comment functionality

- Comment as another user (IDOR on `user_id` field).
- HTML / XSS in comments.
- Comment on resources you should not have access to.
- Bypass length, frequency, or content limits.

## 12. Parameter tampering

Look for hidden parameters in HTML forms, JavaScript, and Burp's history. Try removing them, duplicating them, or changing their values to bypass server-side validation.

## 13. Exam / quiz / test result manipulation

- Change `score` or `is_correct` parameters in the answer-submission request.
- Submit answers after the timer has expired.
- Submit answers for an exam you are not enrolled in.
- Replay a "submit" request to score multiple times.

## 14. ACL / privilege escalation

- Modify `role`, `is_admin`, `permissions` parameters during signup or profile edit (mass assignment).
- Access admin-only endpoints with a regular user's session.
- Manipulate group / organization membership parameters.

## 15. Order / process workflow skipping

Many multi-step flows (registration, KYC, checkout) assume each step is performed in order. Try jumping straight to step 5 without doing steps 1–4. The application may grant access without validating prior steps.

## 16. Negative-quantity abuse

Submit a negative quantity in cart / payment / withdrawal requests. The server may credit instead of debit, or transfer funds in the wrong direction.

## 17. Loyalty-points / referral abuse

- Refer yourself.
- Submit the same referral code multiple times.
- Combine multiple referrals to push your account above thresholds.
- Get points for an action you never performed (cancelled order, returned product, etc.).

## 18. Two-factor / KYC bypass

- Skip the 2FA / KYC step by going directly to the post-2FA URL.
- Tamper with the response (`success: false` → `success: true`).
- Reuse old 2FA tokens.
