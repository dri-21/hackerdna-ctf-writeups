# Hack the Cookie

## Category

Web Security — Cookies & Sessions

## Objective

Gain access to the admin functionality of the TechCorp Internal Portal by investigating how the application handles user sessions.

## What I Observed

I first opened the TechCorp Internal Portal and saw a normal employee login page.

Instead of trying to guess credentials, I opened **Chrome DevTools → Network** and inspected the requests made during login.

In the response headers of the login request, I found a `Set-Cookie` header containing a cookie named `user_session`.

The cookie looked like:

```text
user_session=eyJ1c2VyX2lkIjogMSwgInVzZXJuYW1lIjogImd1ZXN0IiwgInJvbGUiOiAiZ3Vlc3QiLCAiZW1haWwiOiAiZ3Vlc3RAdGVjaGNvcnAubG9jYWwifQ==
```

The value appeared to be Base64-encoded, which was an important clue.

## Approach

I decoded the Base64 value and found a JSON object containing information about the logged-in user:

```json
{
  "user_id": 1,
  "username": "guest",
  "role": "guest",
  "email": "guest@techcorp.local"
}
```

The important observation was that the user's **role was stored directly inside the client-side cookie**.

Since Base64 is only an encoding and not encryption, the contents could be decoded and modified.

## Exploitation

The challenge involved cookie poisoning, so I modified the role in the decoded JSON:

```text
"role": "guest"
```

to:

```
"role": "admin"
```

I kept the other values unchanged, encoded the modified JSON using Base64 again, and replaced the original `user_session` cookie with the modified value.

After refreshing the page, the server accepted the modified cookie and treated the session as an admin session.

This provided access to the admin-only functionality, allowing me to retrieve the flag.

## Flag

747c3d6b-9604-496b-a167-c9326c305dce

## What I Learned

* How to inspect HTTP requests and response headers using Chrome DevTools.
* How cookies can contain client-side session information.
* Base64 is an encoding scheme, not encryption.
* Client-controlled authorization information should not be trusted by the server.
* Session data should be properly protected and validated.
* Improperly validated session information can lead to privilege escalation.
