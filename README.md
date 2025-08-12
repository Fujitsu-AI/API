# PrivateGPT API (v1.4)

> Offizielle, in Markdown aufbereitete Referenz der **PrivateGPT** REST-API – zur Veröffentlichung auf GitHub. Basierend auf der internen Spezifikation (Version 1.4, veröffentlicht am 27.03.2025).

---

## Inhaltsverzeichnis

- [Überblick](#überblick)
- [Änderungen & Fixes](#änderungen--fixes)
- [Schnellstart](#schnellstart)
- [Authentifizierung](#authentifizierung)
  - [Login – Token anfordern](#login--token-anfordern)
  - [Logout – Token invalidieren](#logout--token-invalidieren)
- [Chats (Conversations)](#chats-conversations)
  - [Neuen Chat starten](#neuen-chat-starten)
  - [Chat fortsetzen](#chat-fortsetzen)
  - [Chat-Details abrufen](#chat-details-abrufen)
  - [Chat löschen](#chat-löschen)
  - [Alle Chats löschen](#alle-chats-löschen)
- [Sources (Dokumentenquellen)](#sources-dokumentenquellen)
  - [Neue Source als Markdown anlegen](#neue-source-als-markdown-anlegen)
  - [Source-Details abrufen](#source-details-abrufen)
  - [Source bearbeiten](#source-bearbeiten)
  - [Alle Sources einer Gruppe auflisten](#alle-sources-einer-gruppe-auflisten)
  - [Source löschen](#source-löschen)
- [Groups (Gruppen)](#groups-gruppen)
  - [Verfügbare Gruppen listen](#verfügbare-gruppen-listen)
  - [Neue Gruppe anlegen](#neue-gruppe-anlegen)
  - [Gruppe löschen](#gruppe-löschen)
- [Users (Benutzer)](#users-benutzer)
  - [Benutzer anlegen](#benutzer-anlegen)
  - [Benutzer bearbeiten](#benutzer-bearbeiten)
  - [Benutzer löschen](#benutzer-löschen)
  - [Benutzer reaktivieren](#benutzer-reaktivieren)
- [Optionen (Sprachen & Rollen)](#optionen-sprachen--rollen)
- [Einschränkungen & Hinweise](#einschränkungen--hinweise)
- [Fehlerbehandlung](#fehlerbehandlung)
- [Beispiele (cURL)](#beispiele-curl)
- [Lizenz & Haftungsausschluss](#lizenz--haftungsausschluss)

---

## Überblick

Die **PrivateGPT API** stellt Endpunkte bereit, um Chats zu führen (mit optionalem Dokumentenkontext/RAG), Dokumentenquellen zu verwalten, Gruppen und Benutzer zu administrieren.

- **Basis-URL:** `{base_url}` (z. B. `https://pgpt.example.com`)
- **API-Version:** `v1`
- **Format:** Alle Requests/Responses verwenden `application/json`.
- **Auth:** Bearer Token (siehe [Authentifizierung](#authentifizierung))

---

## Änderungen & Fixes

### Änderungen (v1.4)

- *Delete an existing chat*
- *Delete all chats from the authorized user*
- *Added sources to conversations*
- *Reactivate deleted user*

### Fixes (v1.4)

- **Security Fix:** Korrekte Durchsetzung von Berechtigungen auf allen relevanten API-Ressourcen (Dokumente, Gruppen, Benutzer, Rechtevergabe).
- **Korrekte Dokumentstatus-Rückgabe:** `GET /api/v1/sources/{sourceId}` liefert den tatsächlichen Status (z. B. `error`, `processing`, `vectorized`).

**Versionskompatibilität:** 1.3 → 1.4 (Publikation: 27.03.2025)

---

## Schnellstart

```bash
# 1) Login – API-Token erhalten
curl -sS -X POST \
  -H 'Accept: application/json' \
  -d '{"email":"USER","password":"PASS"}' \
  {base_url}/api/v1/login

# 2) Beispiel-Chat ohne Gruppen (nur allgemeines LLM-Wissen)
curl -sS -X POST \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {api-token}' \
  -d '{"language":"en","question":"Hello World?","usePublic":true}' \
  {base_url}/api/v1/chats
```

---

## Authentifizierung

### Login – Token anfordern

**POST** `{base_url}/api/v1/login`

**Request**

```json
{
  "email": "{user-email}",
  "password": "{user-password}"
}
```

**Response**

```json
{
  "data": { "token": "1|h2Y8cWpK..." },
  "message": "success",
  "status": 200
}
```

> Das Token hat aktuell kein Ablaufdatum und wird in Folge-Calls als `Authorization: Bearer {api-token}` genutzt.

### Logout – Token invalidieren

**DELETE** `{base_url}/api/v1/logout`

**Headers:** `Accept: application/json`, `Authorization: Bearer {api-token}`

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

---

## Chats (Conversations)

> Um RAG (Dokumentenwissen) zu nutzen, müssen **entweder** `usePublic: true` gesetzt **oder** eine/n oder mehrere `groups` angegeben werden. Andernfalls nutzt das LLM nur sein allgemeines Wissen.

### Neuen Chat starten

**POST** `{base_url}/api/v1/chats`

**Request**

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
  "notice": "PrivateGPT provides automated responses and can make mistakes. Verify critical"
}
```

**Hinweis zu Quellen in Chat-Antworten (seit v1.4):** Enthaltene Dokumenttreffer liefern aktuell stets `"page": 0`, da die neue Kontextfindung keine Seitenzahlen bereitstellt. Im Frontend wird die Seite daher weggelassen.

### Chat fortsetzen

**PATCH** `{base_url}/api/v1/chats/{chatId}`

**Request**

```json
{ "question": "Hello World?" }
```

**Response** *(analog wie beim Starten)*

### Chat-Details abrufen

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
      {
        "question": "What is PrivateGPT?",
        "answer": "PrivateGPT is mindblowing!",
        "language": "en"
      }
    ]
  },
  "message": "success",
  "status": 200
}
```

### Chat löschen

**DELETE** `{base_url}/api/v1/chats/{chatId}`

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

### Alle Chats löschen

**DELETE** `{base_url}/api/v1/chats/flush`

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

---

## Sources (Dokumentenquellen)

> Upload via API ist derzeit auf **Markdown** beschränkt. Für Uploads in Gruppen muss die Gruppe **vorher dem Benutzer** zugeordnet sein.

### Neue Source als Markdown anlegen

**POST** `{base_url}/api/v1/sources`

**Request**

```json
{
  "name": "My new Source",
  "groups": ["Group A"],
  "content": "# Markdown\nThis is my content in Markdown format"
}
```

**Response**

```json
{ "data": { "documentId": "9d29bfb6-m763-92ne-4f10-c9c28e4d94fa" }, "message": "success", "status": 200 }
```

### Source-Details abrufen

**GET** `{base_url}/api/v1/sources/{sourceId}`

**Response**

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

### Source bearbeiten

**PATCH** `{base_url}/api/v1/sources/{sourceId}`

**Request** *(alle Felder optional; nicht übergebene Felder bleiben unverändert)*

```json
{
  "name": "My amazing source",
  "groups": ["Group A"],
  "content": "Updated Markdown content"
}
```

**Response**

```json
{ "data": { "documentId": "9d29bfb6-m763-92ne-4f10-c9c28e4d94fa" }, "message": "success", "status": 200 }
```

### Alle Sources einer Gruppe auflisten

**POST** `{base_url}/api/v1/sources/groups`

**Request**

```json
{ "groupName": "Group A" }
```

**Response**

```json
{
  "data": { "sources": [
    "9d29bfb6-m763-92ne-4f10-c9c28e4d94fa",
    "fe8a1336-510b-4baf-8d2b-47f20c33e7d6",
    "bcd69bc4-e3fc-4771-8526-e7d894574f13"
  ]},
  "message": "success",
  "status": 200
}
```

### Source löschen

**DELETE** `{base_url}/api/v1/sources/{sourceId}`

**Response**

```json
{ "data": [], "message": "success", "status": 200 }
```

---

## Groups (Gruppen)

### Verfügbare Gruppen listen

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

### Neue Gruppe anlegen

**POST** `{base_url}/api/v1/groups`

**Request**

```json
{ "groupName": "Group A" }
```

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

### Gruppe löschen

**DELETE** `{base_url}/api/v1/groups`

**Request**

```json
{ "groupName": "Group A" }
```

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

---

## Users (Benutzer)

> Die Benutzer-Identifikation erfolgt über die **E-Mail-Adresse**.

### Benutzer anlegen

**POST** `{base_url}/api/v1/users`

**Request**

```json
{
  "name": "John Doe",
  "email": "john.doe@pgpt.local",
  "language": "de",
  "timezone": "Europe/Berlin",
  "password": "SuperSecret42!",
  "usePublic": true,
  "groups": ["Group A"],
  "roles": ["system"],
  "activateFtp": true,
  "ftpPassword": "myFTP-Password1337"
}
```

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

### Benutzer bearbeiten

**PATCH** `{base_url}/api/v1/users`

**Request** *(Felder optional; E-Mail ist Pflicht und muss existieren)*

```json
{
  "email": "roy@acme.com",
  "name": "Roy Trenneman",
  "language": "de",
  "timezone": "Europe/Berlin",
  "password": "SuperSecret42!",
  "publicUpload": true,
  "groups": ["Group A"],
  "roles": ["system"],
  "activateFtp": true,
  "ftpPassword": "myFTP-Password1337"
}
```

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

### Benutzer löschen

**DELETE** `{base_url}/api/v1/users`

**Request**

```json
{ "email": "roy@acme.com" }
```

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

### Benutzer reaktivieren

**POST** `{base_url}/api/v1/users/reactivate`

**Request**

```json
{ "email": "roy@acme.com" }
```

**Response**

```json
{ "data": {}, "message": "success", "status": 200 }
```

---

## Optionen (Sprachen & Rollen)

### Sprachen (`language`)

Unterstützt u. a.: `en`, `de`, `it`, `fr`, `es`, `ar`, `bg`, `cs`, `eo`, `et`, `el`, `fi`, `fa`, `he`, `hi`, `hu`, `id`, `ja`, `lt`, `lv`, `nl`, `no`, `pl`, `pt`, `pt-br`, `ro`, `ru`, `sk`, `sv`, `tr`, `uk`, `zh`.

### Rollen (`roles`)

Beispiele: `ad`, `analytics`, `documents`, `confluence-documents`, `smtp`, `system`, `users` – entsprechend den Rollen der Webanwendung.

---

## Einschränkungen & Hinweise

- **Single-Document-Chats:** Während eines Chats hochgeladene Einzeldokumente sind derzeit **nicht** über die API verfügbar (`/chats` & `/sources`).
- **Upload-Formate:** Über die API können aktuell nur **Markdown**-Quellen erstellt werden.
- **Berechtigungen:** Zugriffskontrollen werden strikt durchgesetzt; unautorisierte Zugriffe werden mit passenden Fehlern blockiert.
- **Seitenzahlen in Quellenhinweisen:** Aktuell immer `0` (siehe Hinweis in [Chats](#chats-conversations)).

---

## Fehlerbehandlung

- Responses enthalten i. d. R. Felder `message` und `status`.
- Bei nicht ausreichenden Berechtigungen werden Fehler mit entsprechenden HTTP-Statuscodes (z. B. `401/403`) geliefert.

---

## Beispiele (cURL)

> Ersetze `{base_url}` und `{api-token}` entsprechend deiner Umgebung.

**Login**

```bash
curl -X POST \
  -H 'Accept: application/json' \
  -d '{"email":"user@example.com","password":"pass"}' \
  {base_url}/api/v1/login
```

**Neuen Chat mit Public-Dokumenten**

```bash
curl -X POST \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {api-token}' \
  -d '{"language":"en","question":"Hello?","usePublic":true}' \
  {base_url}/api/v1/chats
```

**Source als Markdown anlegen**

```bash
curl -X POST \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {api-token}' \
  -d '{"name":"Doc","groups":["alpha"],"content":"# Title\\nText"}' \
  {base_url}/api/v1/sources
```

**Gruppen listen**

```bash
curl -X GET \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer {api-token}' \
  {base_url}/api/v1/groups
```

---

## Lizenz & Haftungsausschluss

Die PrivateGPT-Software ist **nicht für Hochrisikoanwendungen** (gemäß EU AI Act, Art. 6 und Anhang III) vorgesehen. Nutzung nur, sofern keine erheblichen Risiken für Gesundheit, Sicherheit oder Grundrechte zu erwarten sind und Entscheidungsfindungen nicht materiell beeinflusst werden. Die Bewertung und Handhabung solcher Risiken liegt allein in der Verantwortung der Nutzer.

---

> **Stand:** v1.4 • Diese README dient als GitHub-kompatible Referenz und kann nach Bedarf erweitert (z. B. mit OpenAPI/Swagger-Datei) und versioniert werden.

