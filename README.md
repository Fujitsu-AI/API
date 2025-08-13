# PrivateGPT API v1.4 — README

A concise, developer‑friendly guide to the **PrivateGPT API v1** (release **1.4**). It covers authentication, conversations (chats), sources, groups, users, and available options. The API is JSON‑only.

> **Important safety note**: PrivateGPT is **not designed or intended for any high‑risk uses** as defined by the EU AI Act (Art. 6 and Annex III). You must assess and manage risks for your specific use case.

---

## Table of Contents

- [Overview](#overview)
- [Version & Changes](#version--changes)
- [Authentication](#authentication)
  - [Login (get API token)](#login-get-api-token)
  - [Logout (invalidate token)](#logout-invalidate-token)
- [Conversations (Chats)](#conversations-chats)
  - [Start a new chat](#start-a-new-chat)
  - [Continue an existing chat](#continue-an-existing-chat)
  - [Get chat info](#get-chat-info)
  - [Delete a chat](#delete-a-chat)
  - [Flush all chats for current user](#flush-all-chats-for-current-user)
  - [Notes & limitations](#notes--limitations-chats)
- [Sources](#sources)
  - [Create a source (Markdown only)](#create-a-source-markdown-only)
  - [Get source info](#get-source-info)
  - [Edit a source](#edit-a-source)
  - [List sources by group](#list-sources-by-group)
  - [Delete a source](#delete-a-source)
  - [Notes & limitations](#notes--limitations-sources)
- [Groups](#groups)
  - [List groups](#list-groups)
  - [Create a group](#create-a-group)
  - [Delete a group](#delete-a-group)
- [Users](#users)
  - [Create a user](#create-a-user)
  - [Edit a user](#edit-a-user)
  - [Delete a user](#delete-a-user)
  - [Reactivate a user](#reactivate-a-user)
- [Options](#options)
  - [Languages](#languages)
  - [Roles](#roles)
- [Security Fixes in v1.4](#security-fixes-in-v14)
- [Document Status Reporting Fix](#document-status-reporting-fix)
- [Implementation Notes](#implementation-notes)
- [FAQ](#faq)
- [License & Compliance](#license--compliance)

---

## Overview

- **Base URL**: `{base_url}`
- **Content Type**: `application/json` for requests and responses.
- **Authentication**: Bearer token (see [Authentication](#authentication)).
- **RAG behavior**: To use document knowledge (RAG), pass `usePublic: true` and/or provide `groups`. If `usePublic` is `false` and no `groups` are provided, the LLM will answer from its **general knowledge** only.

---

## Version & Changes

- **Current version**: **1.4**
- **Publication date**: **2025-03-27**
- **Compatibility**: Built for v1 API; features supersede 1.3 where noted.

**Highlights in 1.4**

- *Security*: Proper enforcement of permissions on API resources (documents, groups, users).
- *Correct document status reporting* via `GET /api/v1/sources/{sourceId}` (no longer always `vectorized`; true states are returned such as `error`, `processing`, etc.).
- *Conversations with document results*: Returned `page` is always `0` (see [Notes & limitations](#notes--limitations-chats)).
- *Single Document Chats*: Items uploaded **inside** a chat are not accessible via the `/chats` or `/sources` API (see [Notes & limitations](#notes--limitations-chats)).

---

## Authentication

### Login (get API token)

**POST** `{base_url}/api/v1/login`

**Body**

```json
{
  "email": "{user-email}",
  "password": "{user-password}"
}
```

**Response**

```json
{
  "data": { "token": "1|h2Y8cWpKwsGI8..." },
  "message": "success",
  "status": 200
}
```

> The token **does not expire**. Use it as `Authorization: Bearer {api-token}` in subsequent calls.

### Logout (invalidate token)

**DELETE** `{base_url}/api/v1/logout`

Headers: `Authorization: Bearer {api-token}`

---

## Conversations (Chats)

### Start a new chat

**POST** `{base_url}/api/v1/chats`

**Body**

```json
{
  "language": "en",
  "question": "Hello World?",
  "usePublic": true,
  "groups": []
}
```

**Response**

```json
{
  "data": {
    "chatId": "9d29bfb6-m763-92ne-4f10-c9c28e4d94fa",
    "answer": "Hello! How can I assist you today?",
    "sources": []
  },
  "message": "success",
  "status": 200,
  "notice": "PrivateGPT provides automated responses and can make mistakes. Verify critical..."
}
```

> **RAG control**: If `usePublic` is `false` and `groups` is empty/omitted, RAG is **disabled**. To use RAG, set `usePublic: true` and/or provide `groups` available to the user.

### Continue an existing chat

**PATCH** `{base_url}/api/v1/chats/{chatId}`

**Body**

```json
{ "question": "Hello World?" }
```

**Response**

```json
{
  "data": {
    "chatId": "9d29bfb6-m763-92ne-4f10-c9c28e4d94fa",
    "answer": "Hello! How can I assist you today?",
    "sources": []
  },
  "message": "success",
  "status": 200,
  "notice": "PrivateGPT provides automated responses and can make mistakes. Verify critical..."
}
```

### Get chat info

**GET** `{base_url}/api/v1/chats/{chatId}`

**Response**

```json
{
  "data": {
    "chatId": "chat-uuid",
    "title": "My Chat",
    "language": "en",
    "groups": ["Group A", "Internals"],
    "messages": [
      { "question": "What is PrivateGPT?", "answer": "PrivateGPT is mindblowing!", "language": "en" }
    ]
  },
  "message": "success",
  "status": 200
}
```

### Delete a chat

**DELETE** `{base_url}/api/v1/chats/{chatId}`

### Flush all chats for current user

**DELETE** `{base_url}/api/v1/chats/flush`

### Notes & limitations (Chats)

- From **v1.4**, chats that include document results **always return** a `page` value of `0`. In the frontend this page is hidden; the API keeps the field for compatibility.
- **Single Document Chats**: Documents uploaded directly *in a chat* are **not** accessible via `/chats` or `/sources`.
- A chat uses RAG only if `usePublic` and/or `groups` are provided as described above.

---

## Sources

> **Upload format**: **Markdown only** via API (no other formats supported at the moment).  
> **Permissions**: Users can only upload to groups **already assigned** to their user profile (differs from WebUI where admins can upload to any group).

### Create a source (Markdown only)

**POST** `{base_url}/api/v1/sources`

**Body**

```json
{
  "name": "My new Source",
  "groups": ["Group A"],
  "content": "This is my content in Markdown format"
}
```

**Response**

```json
{ "data": { "documentId": "9d29bfb6-m763-92ne-4f10-c9c28e4d94fa" }, "message": "success", "status": 200 }
```

### Get source info

**GET** `{base_url}/api/v1/sources/{sourceId}`

**Response (example)**

```json
{
  "data": {
    "sourceId": "446f198f-9d45-423b-95d1-624c4cdbfd1c",
    "title": "Another amazing source",
    "groups": ["Internals"],
    "isPublic": false,
    "state": "vectorized",
    "creator": { "email": "roy@acme.com", "name": "Roy Trenneman" },
    "createdAt": "2024-10-01T13:37:42Z",
    "updatedAt": "2024-10-26T09:11:13Z"
  },
  "message": "success",
  "status": 200
}
```

> In v1.4 the `state` correctly reflects the **true processing state**, e.g. `error`, `processing`, etc.

### Edit a source

**PATCH** `{base_url}/api/v1/sources/{sourceId}`

**Body** (all fields optional; omitted fields remain unchanged)

```json
{
  "name": "My amazing source",
  "groups": ["Group A"],
  "content": "This is my content in Markdown format"
}
```

**Response**

```json
{ "data": { "documentId": "9d29bfb6-m763-92ne-4f10-c9c28e4d94fa" }, "message": "success", "status": 200 }
```

### List sources by group

**POST** `{base_url}/api/v1/sources/groups`

**Body**

```json
{ "groupName": "Group A" }
```

**Response**

```json
{
  "data": { "sources": ["9d29...", "fe8a1...", "bcd69...", "c2df5...", "5fa13..."] },
  "message": "success",
  "status": 200
}
```

### Delete a source

**DELETE** `{base_url}/api/v1/sources/{sourceId}`

---

## Groups

### List groups

**GET** `{base_url}/api/v1/groups`

**Response**

```json
{
  "data": {
    "personalGroups": ["alpha", "beta"],
    "assignableGroups": ["alpha", "beta", "internal", "finance"]
  },
  "message": "success",
  "status": 200
}
```

> `personalGroups` are usable for chat. `assignableGroups` are available for creating/editing users and sources.

### Create a group

**POST** `{base_url}/api/v1/groups`

**Body**

```json
{ "groupName": "Group A" }
```

### Delete a group

**DELETE** `{base_url}/api/v1/groups`

**Body**

```json
{ "groupName": "Group A" }
```

---

## Users

> Users are identified by **email**. Timezone defaults to `Europe/Berlin`; language defaults to `en` unless provided otherwise.

### Create a user

**POST** `{base_url}/api/v1/users`

**Body**

```json
{
  "name": "John Doe",
  "email": "john.doe@pgpt.local",
  "language": "de",
  "timezone": "UTC",
  "password": "SuperSecret42!",
  "usePublic": true,
  "groups": ["Group A"],
  "roles": ["system"],
  "activateFtp": true,
  "ftpPassword": "myFTP-Password1337"
}
```

### Edit a user

**PATCH** `{base_url}/api/v1/users`

**Body** (email **required**; all other fields optional)

```json
{
  "email": "roy@acme.com",
  "name": "Roy Trenneman",
  "language": "de",
  "timezone": "UTC",
  "password": "SuperSecret42!",
  "publicUpload": true,
  "groups": ["Group A"],
  "roles": ["system"],
  "activateFtp": true,
  "ftpPassword": "myFTP-Password1337"
}
```

### Delete a user

**DELETE** `{base_url}/api/v1/users`

**Body**

```json
{ "email": "roy@acme.com" }
```

### Reactivate a user

**POST** `{base_url}/api/v1/users/reactivate`

**Body**

```json
{ "email": "roy@acme.com" }
```

---

## Options

### Languages

Supported languages include (not exhaustive): `en`, `de`, `it`, `fr`, `es`, `ar`, `bg`, `cs`, `eo`, `et`, `el`, `fi`, `fa`, `he`, `hi`, `hu`, `id`, `ja`, `lt`, `lv`, `nl`, `no`, `pl`, `pt`, `pt-br`, `ro`, `ru`, `sk`, `sv`, `tr`, `uk`, `zh`.

### Roles

Representative roles (match web app roles/capabilities): `ad`, `analytics`, `documents`, `confluence-documents`, `smtp`, `system`, `users`.

---

## Security Fixes in v1.4

- **Access control enforcement**: Only users with appropriate permissions can view/update/delete documents, groups, or users; permission assignment via API is likewise restricted. Unauthorized attempts return appropriate error responses.

---

## Document Status Reporting Fix

- `GET /api/v1/sources/{sourceId}` now returns the **actual** status (e.g., `error`, `processing`), not always `vectorized`.

---

## Implementation Notes

- **Tokens**: Treat the bearer token like a password. Rotate if compromised.
- **RAG**: Decide between public vs. group‑scoped documents; remember that API uploads are limited to **Markdown** (for now).
- **Groups & Users**: A user must already be **assigned** to a group to upload sources to that group via API.
- **Timestamps**: ISO‑8601 strings (UTC) in responses.

---

## FAQ

**Q: Why do some chat responses include `page: 0` for sources?**  
A: In v1.4 a new context‑finding service doesn’t provide specific page numbers. The API keeps the `page` field for compatibility but always sets `0`. Frontends typically hide it.

**Q: Can I access documents uploaded directly inside a chat?**  
A: No. “Single document chat” uploads are **not** exposed via `/chats` or `/sources` yet.

**Q: Does the API token expire?**  
A: No. It has no expiration date.

---

## License & Compliance

- **High‑risk usage**: The software **must not** be used for high‑risk AI uses as per EU AI Act definitions.
- You are responsible for compliance, security, and data protection in your deployments.

---

### cURL Quick Reference

```bash
# Login
curl -s -X POST "{base_url}/api/v1/login" -H "Accept: application/json"   -d '{ "email":"me@example.com", "password":"secret" }'

# New chat
curl -s -X POST "{base_url}/api/v1/chats" -H "Accept: application/json"   -H "Authorization: Bearer ${API_TOKEN}"   -d '{ "language":"en", "question":"Hello?", "usePublic":true, "groups":[] }'

# Continue chat
curl -s -X PATCH "{base_url}/api/v1/chats/${CHAT_ID}" -H "Accept: application/json"   -H "Authorization: Bearer ${API_TOKEN}"   -d '{ "question":"Next question" }'

# Create Markdown source
curl -s -X POST "{base_url}/api/v1/sources" -H "Accept: application/json"   -H "Authorization: Bearer ${API_TOKEN}"   -d '{ "name":"My Source", "groups":["Group A"], "content":"# Title\nMy markdown." }'
```

---

> Replace `{base_url}` with your actual API root and adjust examples as needed.
