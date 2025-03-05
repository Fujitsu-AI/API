# PrivateGPT API v1 Definition

Python API Examples for Private GPT

### To access the instance of Private GPT you need:

* User and password to access the proxy
* User and password for the PGPT instance
* You will receive both from our AI team


The following information outlines the API endpoints and their usage for the PrivateGPT API v1.

---

## Table of Contents

1. [Authentication](#authentication)
    - [Login](#login)
    - [Logout](#logout)
2. [Chats (Conversations)](#chats-conversations)
    - [New Chat](#new-chat)
    - [Continue Existing Chat](#continue-existing-chat)
3. [Sources](#sources)
    - [Store a New Source](#store-a-new-source)
    - [Get Information About a Source](#get-information-about-a-source)
    - [Edit an Existing Source](#edit-an-existing-source)
    - [Delete a Source](#delete-a-source)
4. [Groups](#groups)
    - [Get Available Groups](#get-available-groups)
    - [Store a New Group](#store-a-new-group)
    - [Delete a Group](#delete-a-group)
5. [Users](#users)
    - [Store a New User](#store-a-new-user)
    - [Edit a User](#edit-a-user)
    - [Delete a User](#delete-a-user)

---

## Authentication

### Login

**Endpoint:** `{base-url}/api/v1/login`  
**Method:** `POST`  
**Headers:**  
- `Accept: application/json`

**Request Body:**
```json
{
    "email": "{user-email}",
    "password": "{user-password}"
}
```

**Response:**
```json
{
    "data": {
        "token": "1|example-token"
    },
    "message": "success",
    "status": 200
}
```

The returned token will be used for authenticated API calls.

### Logout

**Endpoint:** `{base-url}/api/v1/logout`  
**Method:** `DELETE`  
**Headers:**  
- `Accept: application/json`  
- `Authentication: Bearer {api-token}`

**Response:**
```json
{
    "data": {},
    "message": "success",
    "status": 200
}
```

---

## Chats (Conversations)

### New Chat

**Endpoint:** `{base-url}/api/v1/chats`  
**Method:** `POST`  
**Headers:**  
- `Accept: application/json`  
- `Authentication: Bearer {api-token}`

**Request Body:**
```json
{
    "language": "en",
    "question": "Hello World?",
    "usePublic": true,
    "groups": [] // Optional
}
```

**Response:**
```json
{
    "data": {
        "chatId": "example-chat-id",
        "answer": "Hello! How can I assist you today?",
        "sources": ["source-id-1", "source-id-2"]
    },
    "message": "success",
    "status": 200
}
```

### Continue Existing Chat

**Endpoint:** `{base-url}/api/v1/chats/{chatId}`  
**Method:** `POST`  
**Headers:**  
- `Accept: application/json`  
- `Authentication: Bearer {api-token}`

**Request Body:**
```json
{
    "question": "What is PrivateGPT?"
}
```

**Response:** Same as New Chat.

---

## Sources

### Store a New Source

**Endpoint:** `{base-url}/api/v1/sources`  
**Method:** `POST`  
**Headers:**  
- `Accept: application/json`  
- `Authentication: Bearer {api-token}`

**Request Body:**
```json
{
    "title": "My new Source",
    "groups": ["Group A"], // Optional
    "content": "This is my content in Markdown format"
}
```

**Response:**
```json
{
    "data": {
        "sourceId": "new-source-id"
    },
    "message": "success",
    "status": 200
}
```

---

## Groups

### Get Available Groups

**Endpoint:** `{base-url}/api/v1/groups`  
**Method:** `GET`  
**Headers:**  
- `Accept: application/json`  
- `Authentication: Bearer {api-token}`

**Response:**
```json
{
    "data": {
        "personalGroups": ["group1", "group2"],
        "assignableGroups": ["group1", "group2", "group3"]
    },
    "message": "success",
    "status": 200
}
```

### Store a New Group

**Endpoint:** `{base-url}/api/v1/groups`  
**Method:** `POST`  
**Headers:**  
- `Accept: application/json`  
- `Authentication: Bearer {api-token}`

**Request Body:**
```json
{
    "groupName": "New Group"
}
```

**Response:**
```json
{
    "data": {},
    "message": "success",
    "status": 200
}
```

---

## Users

### Store a New User

**Endpoint:** `{base-url}/api/v1/users`  
**Method:** `POST`  
**Headers:**  
- `Accept: application/json`  
- `Authentication: Bearer {api-token}`

**Request Body:**
```json
{
    "name": "New User",
    "email": "newuser@example.com",
    "language": "en",
    "timezone": "UTC",
    "password": "StrongPassword123!",
    "usePublic": true,
    "groups": ["Group A"],
    "roles": ["Sources"]
}
```

**Response:**
```json
{
    "data": {},
    "message": "success",
    "status": 200
}
```

---

Please refer to the full documentation for additional details on editing and deleting users, groups, sources, and chats.
