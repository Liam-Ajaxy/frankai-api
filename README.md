# frankai-examples

Copy-paste examples for the **Frank AI API** and the `frankai` terminal package.

Frank AI API is a free AI service made for junior developers. With one key and plain HTTPS requests you can add a chat assistant, picture generation and a voice to your app. There is nothing to install, and it works from a web page or from your own backend.

**It is free, and it never asks for a credit card or any payment.** Signing up, getting a key and using the API do not involve payment details of any kind.

- **Base URL:** `https://api.frankai.workers.dev/v1`
- **Developer portal (get a key):** https://developer.frankapi.workers.dev
- **Hosted docs:** https://api.frankai.workers.dev/docs/
- **OpenAPI file:** https://api.frankai.workers.dev/openapi.json
- **Status:** https://api.frankai.workers.dev/health

> This README is a copy of the hosted documentation, written so you can read it next to the examples. If the two ever disagree, the [hosted docs](https://api.frankai.workers.dev/docs/) are the source of truth.

## Contents

- [What you can build](#what-you-can-build)
- [Quick start](#quick-start)
- [Getting started](#getting-started)
- [Authentication](#authentication)
- **Endpoints:** [Chat](#chat) · [Pictures](#pictures) · [Voice](#voice) · [Respond](#respond) · [Files](#files) · [Your key](#your-key)
- **Guides:** [Browser apps](#browser-apps) · [Backend apps](#backend-apps)
- [Identity](#identity)
- [Limits](#limits)
- [Errors](#errors)
- [FAQ](#faq)
- [Rules, terms and privacy](#rules-terms-and-privacy)
- [The frankai terminal package](#the-frankai-terminal-package)
- [Machine-readable docs](#machine-readable-docs)
- [Examples in this repo](#examples-in-this-repo)
- [License](#license)

## What you can build

| You want to                                    | Call                | Section                 |
| ---------------------------------------------- | ------------------- | ----------------------- |
| Chat with an assistant                         | `POST /v1/chat`     | [Chat](#chat)           |
| Make a picture from a text description         | `POST /v1/image`    | [Pictures](#pictures)   |
| Turn text into speech                          | `POST /v1/tts`      | [Voice](#voice)         |
| Get a reply, a voice and a picture in one call | `POST /v1/respond`  | [Respond](#respond)     |
| Open a picture or audio file you made          | `GET /files/{name}` | [Files](#files)         |
| Check your key and what is left today          | `GET /v1/me`        | [Your key](#your-key)   |

## Quick start

1. **Get a key.** Sign up in the [developer portal](https://developer.frankapi.workers.dev), wait for approval, then create a key. Details in [Getting started](#getting-started).
2. **Send a request.** The key goes in the `Authorization` header:

   ```bash
   curl https://api.frankai.workers.dev/v1/chat \
     -H "Authorization: Bearer YOUR_SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"message": "Explain what an API is in one sentence."}'
   ```

3. **Read the answer.** Every successful call returns JSON:

   ```json
   {
     "reply": "An API is a way for one program to ask another program for something and get an answer back.",
     "usage": { "tokens": 58, "remaining_today": 199942 }
   }
   ```

### How it works

- **Three kinds of key.** A public key (`fk_pub_`) is safe to put in browser code and only works from the sites you list. A secret key (`fk_sec_`) stays on your backend and can also make pictures and voices. A Terminal key (`fk_cli_`) is for the command line.
- **A daily budget, not a bill.** Every key has a number of tokens it can use each day, and the budget resets at 00:00 UTC. There are no paid plans and nothing to pay.
- **Reliable answers.** Frank AI tries several AI providers behind the scenes and switches by itself when one is busy. If every provider is busy you get a clear `ai_busy` error, and that call costs nothing.
- **One identity.** The assistant is called Frank AI by default. You can give each key its own assistant, model and company name.

> **Note:** Replies arrive whole, in one response. Streaming is not available yet.

## Getting started

This section takes you from nothing to your first reply from the assistant.

### 1. Create a developer account

Open the [developer portal](https://developer.frankapi.workers.dev) and sign up with your name, an email address or phone number, and a password of at least 10 characters. The portal never asks for a credit card or any payment details.

New accounts are approved by the Frank AI team. Each person may have one account, and you accept the [Usage rules](https://api.frankai.workers.dev/docs/rules) when you sign up. Until your account is approved, your keys answer `pending_approval`. They start working by themselves as soon as you are approved, and you do not need to create new ones. If sign-ups are closed or full, the portal says so.

### 2. Create a key

In the portal, create a key and choose its type:

- **Public key** (`fk_pub_...`). For browser code. Add the address of your site, for example `https://my-site.vercel.app`. Sites must use `https`. You can list up to 5 sites. `http://localhost` (any port) is allowed by default so you can test on your own computer.
- **Secret key** (`fk_sec_...`). For your backend. Secret keys need approval: send a "secret key request" from the portal and explain what you are building.
- **Terminal key** (`fk_cli_...`). Created for you when you sign in from the `frankai` command line. It has a smaller budget and cannot be used from a browser.

Copy the key right away. The portal shows it once and cannot show it again. You can have up to 5 active keys. If a key leaks, revoke it in the portal and create a new one.

### 3. Send your first request

Pick the one that matches your key.

**In a web page (public key).** Run this from a page served on an allowed address, such as `http://localhost:5500`:

```js
const res = await fetch("https://api.frankai.workers.dev/v1/chat", {
  method: "POST",
  headers: {
    "Authorization": "Bearer YOUR_PUBLIC_KEY",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ message: "Give me one tip for learning JavaScript." }),
});
const data = await res.json();
if (!res.ok) throw new Error(data.error.message);
console.log(data.reply);
```

**From a backend (secret key), in Node.js 18 or newer:**

```js
const res = await fetch("https://api.frankai.workers.dev/v1/chat", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${process.env.FRANK_AI_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ message: "Give me one tip for learning Node.js." }),
});
const data = await res.json();
if (!res.ok) throw new Error(data.error.message);
console.log(data.reply);
```

**From a backend, in Python:**

```python
import os
import requests

res = requests.post(
    "https://api.frankai.workers.dev/v1/chat",
    headers={"Authorization": "Bearer " + os.environ["FRANK_AI_KEY"]},
    json={"message": "Give me one tip for learning Python."},
    timeout=60,
)
data = res.json()
if not res.ok:
    raise RuntimeError(data["error"]["message"])
print(data["reply"])
```

**From a terminal (secret or Terminal key):**

```bash
curl https://api.frankai.workers.dev/v1/chat \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{"message": "Give me one tip for learning to code."}'
```

> **Warning:** Never put a secret key in browser code, in a mobile app, or in a public GitHub repository. Anyone who finds it can spend your budget. Keep it in an environment variable on your backend.

### 4. Check that your key works

`GET /v1/me` shows the key's type, what it may do and how many tokens are left today. It costs nothing.

```bash
curl https://api.frankai.workers.dev/v1/me -H "Authorization: Bearer YOUR_SECRET_KEY"
```

See [Your key](#your-key) for the answer. If you get an error instead, look up its `code` in [Errors](#errors).

## Authentication

Every call to `/v1` needs a key. Send it in the `Authorization` header:

```http
Authorization: Bearer YOUR_KEY
```

Requests are JSON. Send `Content-Type: application/json` on every `POST`.

### The three key types

|                              | Public key     | Secret key                     | Terminal key                   |
| ---------------------------- | -------------- | ------------------------------ | ------------------------------ |
| Starts with                  | `fk_pub_`      | `fk_sec_`                      | `fk_cli_`                      |
| Use it in                    | Browser code   | Your backend                   | The `frankai` command line     |
| Endpoints                    | chat, respond  | chat, respond, voice, pictures | chat, respond, voice, pictures |
| Requests per minute          | 30             | 60                             | 20                             |
| Tokens per day               | 30,000         | 200,000                        | 40,000                         |
| Largest reply (`max_tokens`) | 512            | 1,024                          | 1,024                          |

- **Public keys** are made for browser code, so it is fine that visitors can see them. They only work from the sites you list on the key, they can only chat, and they have smaller limits. Each visitor also gets their own small budget.
- **Secret keys** belong on your backend. They can chat, make pictures and voices, and use the larger limits.
- **Terminal keys** are created when you sign in from the `frankai` command line. They have a smaller budget and cannot be used from a browser.

You create and revoke keys in the [developer portal](https://developer.frankapi.workers.dev). The portal shows a new key once, so copy it right away.

### Where each key may be used

The service looks at the `Origin` header, which browsers add to every cross-site request and your backend normally does not send.

- A **secret or Terminal key** with an `Origin` header is refused with `secret_key_in_browser`. This protects you if you paste a secret key into browser code by mistake.
- A **public key** without an `Origin` header is refused with `origin_required`. A public key only works from a browser page.
- A **public key** from a site that is not on its list is refused with `origin_not_allowed`. Add the site to the key in the portal. A page opened as a file (`file://`) has no usable origin, so serve it from `http://localhost` while you develop.

### Account status

Your developer account must be approved. While it is waiting, every call answers `pending_approval`. If an account is suspended, calls answer `developer_suspended`.

### Telling your own users apart (secret keys)

If your backend serves many users, send your user's id in the `X-User-Id` header, up to 100 characters. Each user then has their own daily budget of 10,000 tokens, so one heavy user cannot use up the whole key. When a user is out of budget the answer is `daily_budget_exceeded`, and your other users are not affected.

```bash
curl https://api.frankai.workers.dev/v1/chat \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "X-User-Id: user-4821" \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'
```

Public keys do not use this header. Each visitor is told apart by their own address instead.

### Wrong keys are limited

If more than 30 invalid keys come from one address in a minute, further attempts are answered `too_many_attempts` with a `Retry-After` of 60 seconds. Calls with a valid key are never slowed by this.

### Keep keys safe

- Keep secret keys in environment variables, never in your source code and never on GitHub.
- Use a public key in the browser, even if it feels less powerful. That is what it is for.
- If a key leaks, revoke it in the portal and create a new one. The portal can also rotate a key: it makes a new key with the same settings and revokes the old one.
- One key per project makes it easy to revoke only the one that leaked.

## Endpoints

### Chat

`POST https://api.frankai.workers.dev/v1/chat` sends a message, or a whole conversation, to the assistant and returns its reply.

**Allowed keys:** public, secret and Terminal.

#### Request

Send a JSON body with `message` or `messages`.

| Field        | Type   | Required       | What it does                                                                                                                                                                                                                     |
| ------------ | ------ | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `message`    | string | one of the two | One message from the user, 1 to 4,000 characters.                                                                                                                                                                                |
| `messages`   | array  | one of the two | A conversation, oldest message first. Up to 20 items, each `{ "role": ..., "content": ... }`. `role` is `system`, `user` or `assistant`. `content` is 1 to 4,000 characters. If you send both, `messages` is used.                |
| `max_tokens` | number | no             | The longest reply you will accept. The default is 512. It is capped at 512 for public keys and 1,024 for secret and Terminal keys.                                                                                              |

The whole request body can be up to 100 KB.

**A single message**

```bash
curl https://api.frankai.workers.dev/v1/chat \
  -H "Authorization: Bearer YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"message": "Explain what an API is in one sentence."}'
```

**A conversation**

The API does not remember earlier requests. To continue a conversation, send the messages so far each time, with the assistant's earlier replies as `assistant` messages:

```js
const messages = [
  { role: "user", content: "What is a variable?" },
  { role: "assistant", content: "A variable is a named box that holds a value." },
  { role: "user", content: "Show me one in JavaScript." },
];

const res = await fetch("https://api.frankai.workers.dev/v1/chat", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_KEY", "Content-Type": "application/json" },
  body: JSON.stringify({ messages, max_tokens: 300 }),
});
const data = await res.json();
if (!res.ok) throw new Error(data.error.message);
messages.push({ role: "assistant", content: data.reply }); // keep the history for the next turn
console.log(data.reply);
```

```python
import os
import requests

messages = [
    {"role": "user", "content": "What is a variable?"},
    {"role": "assistant", "content": "A variable is a named box that holds a value."},
    {"role": "user", "content": "Show me one in Python."},
]
res = requests.post(
    "https://api.frankai.workers.dev/v1/chat",
    headers={"Authorization": "Bearer " + os.environ["FRANK_AI_KEY"]},
    json={"messages": messages, "max_tokens": 300},
    timeout=60,
)
data = res.json()
if not res.ok:
    raise RuntimeError(data["error"]["message"])
print(data["reply"])
```

**Instructions for the assistant**

A message with the role `system` tells the assistant how to behave, for example `{ "role": "system", "content": "Answer in at most two sentences." }`. Your instructions can change the tone, length and topic of the answers. They cannot change who the assistant says it is. See [Identity](#identity).

#### Response

A successful call returns status 200:

```json
{
  "reply": "A variable is a named box that stores a value so you can use it later.",
  "usage": { "tokens": 74, "remaining_today": 199926 }
}
```

| Field                   | What it holds                                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `reply`                 | The assistant's answer, as text.                                                                                       |
| `usage.tokens`          | The tokens this call used, counted against your daily budget.                                                          |
| `usage.remaining_today` | The tokens left today in your tightest budget (the key's, or the end user's when you send `X-User-Id`).               |

#### Good to know

- The reply arrives whole. Streaming is not available yet.
- A call that fails is not charged.

**Common errors:** `bad_request` (read the `message`), `invalid_json`, `rate_limited`, `daily_budget_exceeded` and `ai_busy` (try again in a moment). The full list is in [Errors](#errors).

### Pictures

`POST https://api.frankai.workers.dev/v1/image` makes a picture from a text description and returns a link to it.

**Allowed keys:** secret and Terminal. Public keys cannot make pictures, so call this from your backend and send the link to your page.

**Cost:** 1,000 tokens from your daily budget, only when the picture was made.

#### Request

| Field    | Type   | Required | What it does                                          |
| -------- | ------ | -------- | ----------------------------------------------------- |
| `prompt` | string | yes      | What the picture should show, 1 to 1,000 characters.  |

```bash
curl https://api.frankai.workers.dev/v1/image \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A red bicycle on a green hill in Kigali at sunrise"}'
```

```js
const res = await fetch("https://api.frankai.workers.dev/v1/image", {
  method: "POST",
  headers: { "Authorization": `Bearer ${process.env.FRANK_AI_KEY}`, "Content-Type": "application/json" },
  body: JSON.stringify({ prompt: "A red bicycle on a green hill in Kigali at sunrise" }),
});
const data = await res.json();
if (!res.ok) throw new Error(data.error.message);
console.log(data.url); // open this address, or use it in an <img> tag
```

```python
import os
import requests

res = requests.post(
    "https://api.frankai.workers.dev/v1/image",
    headers={"Authorization": "Bearer " + os.environ["FRANK_AI_KEY"]},
    json={"prompt": "A red bicycle on a green hill in Kigali at sunrise"},
    timeout=120,
)
data = res.json()
if not res.ok:
    raise RuntimeError(data["error"]["message"])
print(data["url"])
```

#### Response

```json
{
  "url": "https://api.frankai.workers.dev/files/0f1e2d3c4b5a69788796a5b4c3d2e1f0.png",
  "type": "image/png",
  "expires_in_hours": 24,
  "usage": { "tokens": 1000, "remaining_today": 199000 }
}
```

| Field              | What it holds                                                                                                |
| ------------------ | ------------------------------------------------------------------------------------------------------------ |
| `url`              | A public link to the picture. Use it in an `<img>` tag or open it in a browser.                              |
| `type`             | The file type: `image/png`, `image/jpeg` or `image/webp`. It depends on which provider made the picture.     |
| `expires_in_hours` | How long the file stays online. After that the link answers `not_found`.                                     |
| `usage`            | The tokens this call used, and what is left today.                                                           |

#### Good to know

- **Save what you want to keep.** Files are deleted after 24 hours. Download the picture to your own storage if you need it longer. See [Files](#files).
- **Pictures can take a while.** Set a generous timeout, 60 to 120 seconds.
- **Links are public.** Anyone with the link can open the picture until it expires. The names are random and cannot be guessed.
- **A failed call costs nothing.** If every provider is busy you get `ai_busy`. If the file could not be saved you get `storage_unavailable`. Try again in a moment.

**Common errors:** `bad_request` (the prompt is missing or too long), `scope_not_allowed` (public key), `ai_busy`, `service_not_configured`, `storage_unavailable`, `rate_limited`, `daily_budget_exceeded`.

### Voice

`POST https://api.frankai.workers.dev/v1/tts` turns a short text into speech and returns a link to the audio file.

**Allowed keys:** secret and Terminal. Public keys cannot make voices, so call this from your backend and send the link to your page.

**Cost:** 300 tokens from your daily budget, only when the audio was made.

#### Request

| Field  | Type   | Required | What it does                              |
| ------ | ------ | -------- | ----------------------------------------- |
| `text` | string | yes      | The words to speak, 1 to 200 characters.  |

```bash
curl https://api.frankai.workers.dev/v1/tts \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello, welcome to Frank AI."}'
```

```js
const res = await fetch("https://api.frankai.workers.dev/v1/tts", {
  method: "POST",
  headers: { "Authorization": `Bearer ${process.env.FRANK_AI_KEY}`, "Content-Type": "application/json" },
  body: JSON.stringify({ text: "Hello, welcome to Frank AI." }),
});
const data = await res.json();
if (!res.ok) throw new Error(data.error.message);
console.log(data.url); // use it as the src of an <audio> tag
```

```python
import os
import requests

res = requests.post(
    "https://api.frankai.workers.dev/v1/tts",
    headers={"Authorization": "Bearer " + os.environ["FRANK_AI_KEY"]},
    json={"text": "Hello, welcome to Frank AI."},
    timeout=60,
)
data = res.json()
if not res.ok:
    raise RuntimeError(data["error"]["message"])
print(data["url"])
```

#### Response

```json
{
  "url": "https://api.frankai.workers.dev/files/9a8b7c6d5e4f30211203f4e5d6c7b8a9.wav",
  "type": "audio/wav",
  "expires_in_hours": 24,
  "usage": { "tokens": 300, "remaining_today": 199700 }
}
```

| Field              | What it holds                                                                                             |
| ------------------ | --------------------------------------------------------------------------------------------------------- |
| `url`              | A public link to the audio file.                                                                          |
| `type`             | `audio/wav` or `audio/mpeg` (mp3). It depends on which provider made the voice, so do not assume one.     |
| `expires_in_hours` | How long the file stays online.                                                                           |
| `usage`            | The tokens this call used, and what is left today.                                                        |

Play it in a page like this:

```html
<audio controls src="https://api.frankai.workers.dev/files/YOUR_FILE_NAME.wav"></audio>
```

#### Good to know

- The limit is 200 characters per call. To read a longer chat reply aloud, use [Respond](#respond): it cuts the text at the end of a sentence for you.
- Write plain sentences. Markdown marks and code are not spoken well.
- Files are deleted after 24 hours. See [Files](#files).
- A failed call costs nothing.

**Common errors:** `bad_request` (the text is empty or too long), `scope_not_allowed` (public key), `ai_busy`, `service_not_configured`, `storage_unavailable`, `rate_limited`, `daily_budget_exceeded`.

### Respond

`POST https://api.frankai.workers.dev/v1/respond` works like [Chat](#chat), and it can also read the reply aloud and draw a picture, all in one call. It is the easiest way to build a talking assistant.

**Allowed keys:** public, secret and Terminal for a plain chat reply. Asking for `speak` or `image` needs a secret or Terminal key, because those make files.

#### Request

| Field                   | Type              | Required       | What it does                                                                                                        |
| ----------------------- | ----------------- | -------------- | ------------------------------------------------------------------------------------------------------------------- |
| `message` or `messages` | string or array   | one of the two | The same as in [Chat](#chat).                                                                                       |
| `max_tokens`            | number            | no             | The longest reply you will accept. The default is 512, capped by your key type.                                     |
| `speak`                 | boolean           | no             | `true` also turns the reply into an audio file.                                                                     |
| `image`                 | boolean or string | no             | `true` draws a picture from your last user message. A string is used as the picture description (up to 1,000 characters). |

When `speak` is `true`, the assistant is asked for a short plain-text reply that fits the voice limit of 200 characters, so it can be read aloud in full.

```bash
curl https://api.frankai.workers.dev/v1/respond \
  -H "Authorization: Bearer YOUR_SECRET_KEY" \
  -H "Content-Type: application/json" \
  -d '{"message": "Tell me a short story about a cow.", "speak": true, "image": "A friendly cow in a green field"}'
```

```js
const res = await fetch("https://api.frankai.workers.dev/v1/respond", {
  method: "POST",
  headers: { "Authorization": `Bearer ${process.env.FRANK_AI_KEY}`, "Content-Type": "application/json" },
  body: JSON.stringify({
    message: "Tell me a short story about a cow.",
    speak: true,
    image: "A friendly cow in a green field",
  }),
});
const data = await res.json();
if (!res.ok) throw new Error(data.error.message);
console.log(data.reply, data.audio?.url, data.image?.url);
if (data.errors) console.warn("Some parts failed:", data.errors);
```

```python
import os
import requests

res = requests.post(
    "https://api.frankai.workers.dev/v1/respond",
    headers={"Authorization": "Bearer " + os.environ["FRANK_AI_KEY"]},
    json={"message": "Tell me a short story about a cow.", "speak": True, "image": "A friendly cow in a green field"},
    timeout=120,
)
data = res.json()
if not res.ok:
    raise RuntimeError(data["error"]["message"])
print(data["reply"], data.get("audio", {}).get("url"), data.get("image", {}).get("url"))
```

#### Response

```json
{
  "reply": "Once upon a time, a cow named Inka learned to sing at dawn.",
  "audio": { "url": "https://api.frankai.workers.dev/files/9a8b7c6d5e4f30211203f4e5d6c7b8a9.wav", "type": "audio/wav" },
  "image": { "url": "https://api.frankai.workers.dev/files/0f1e2d3c4b5a69788796a5b4c3d2e1f0.png", "type": "image/png" },
  "usage": { "tokens": 1390, "remaining_today": 198610 }
}
```

| Field    | What it holds                                                                                                                                                                             |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `reply`  | The assistant's answer. Always present on success.                                                                                                                                        |
| `audio`  | Present when you asked for `speak` and it worked: `url` and `type`. If the reply was longer than the voice limit, only the first part was spoken and `audio.truncated` is `true`.        |
| `image`  | Present when you asked for `image` and it worked: `url` and `type`.                                                                                                                       |
| `errors` | Present only when the voice or the picture failed: `errors.audio` or `errors.image`, each with a `code` and a `message`.                                                                  |
| `usage`  | The tokens this call used, and what is left today.                                                                                                                                        |

#### When the voice or the picture fails

The chat reply comes first. If the chat succeeds but the voice or the picture fails, you still get `200` with the `reply`, and the failure is listed under `errors`. Only the parts that worked are charged. The reply is the part you should always show.

If the chat itself fails (for example `ai_busy`), the whole call fails with an error status and costs nothing.

#### Cost

The chat tokens, plus 1,000 for a picture and 300 for a voice when they worked. See [Limits](#limits).

**Common errors:** `bad_request` (`speak` or `image` has the wrong type, or there is nothing to draw), `scope_not_allowed` (a public key asked for `speak` or `image`), `ai_busy`, `rate_limited`, `daily_budget_exceeded`, and inside `errors`: `nothing_to_speak`, `storage_unavailable`.

### Files

`GET https://api.frankai.workers.dev/files/{name}` opens a picture or an audio file that you made with [Pictures](#pictures), [Voice](#voice) or [Respond](#respond). You never build this address yourself: use the `url` from the answer.

**No key is needed.** The link is public, so you can use it directly in an `<img>` or `<audio>` tag.

#### The address

```text
https://api.frankai.workers.dev/files/0f1e2d3c4b5a69788796a5b4c3d2e1f0.png
```

The name is 32 random letters and digits (`a` to `f` and `0` to `9`), a dot, and the file type: `png`, `jpg`, `webp`, `mp3` or `wav`. It cannot be guessed.

#### What the answer looks like

| Case                                           | Status | Notes                                                                                                                                  |
| ---------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| The file exists                                | `200`  | The file itself, with its `Content-Type` (for example `image/png` or `audio/wav`).                                                     |
| A part of the file was asked for with `Range`  | `206`  | Needed by Safari to play audio. Seeking in audio and video uses it too.                                                                |
| The range is not valid                         | `416`  |                                                                                                                                        |
| The name is wrong, or the file has expired     | `404`  | `{ "error": { "code": "not_found", "message": "File not found or expired." } }`                                                        |

`HEAD` requests work too. Answers carry `Access-Control-Allow-Origin: *`, so pages on any site can show the files, and `Cache-Control: public, max-age=3600`.

#### Good to know

- **Files are deleted after 24 hours.** The `expires_in_hours` field in the answer tells you. Download a file to your own storage if you need to keep it.
- **Anyone with the link can open it** until it expires. Do not generate anything private.
- **There is a daily limit on new files**, shared by all developers. When it is used up, picture and voice calls answer `storage_unavailable` and are not charged. It resets every day.
- Opening a file costs no tokens.

#### Download a file in Node.js

```js
import { writeFile } from "node:fs/promises";

const file = await fetch(data.url); // data.url comes from /v1/image or /v1/tts
await writeFile("picture.png", Buffer.from(await file.arrayBuffer()));
```

### Your key

`GET https://api.frankai.workers.dev/v1/me` tells you about the key you used: its type, what it may do, its limits, and how many tokens are left today.

**Allowed keys:** all of them. **Cost:** nothing. The call still works when today's budget is used up, and it never returns the key itself.

```bash
curl https://api.frankai.workers.dev/v1/me -H "Authorization: Bearer YOUR_KEY"
```

```js
const res = await fetch("https://api.frankai.workers.dev/v1/me", { headers: { "Authorization": "Bearer YOUR_KEY" } });
const me = await res.json();
console.log(me.usage.remaining_today, "tokens left today");
```

#### Response

```json
{
  "key": { "type": "secret", "label": "My backend", "owner": "Your name" },
  "assistant": "Frank AI",
  "limits": {
    "scopes": ["chat", "tts", "image", "storage"],
    "daily_tokens": 200000,
    "requests_per_minute": 60,
    "max_tokens_per_request": 1024
  },
  "usage": { "remaining_today": 200000, "resets_at": "00:00 UTC" }
}
```

| Field                           | What it holds                                                                                                      |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `key.type`                      | `public`, `secret` or `cli` (Terminal).                                                                            |
| `key.label`                     | The name you gave the key in the portal, or `null`.                                                                |
| `key.owner`                     | The developer account the key belongs to.                                                                          |
| `assistant`                     | The assistant's name for this key. It is Frank AI unless you set your own. See [Identity](#identity).              |
| `limits.scopes`                 | What the key may use: `chat`, `tts` (voice), `image` (pictures), `storage`.                                        |
| `limits.daily_tokens`           | The key's daily token budget.                                                                                      |
| `limits.requests_per_minute`    | How many requests per minute the key may send.                                                                     |
| `limits.max_tokens_per_request` | The largest `max_tokens` you can ask for.                                                                          |
| `usage.remaining_today`         | Tokens left today in your tightest budget.                                                                         |
| `usage.resets_at`               | When the daily budget resets.                                                                                      |

The numbers above are the secret key's. Public and Terminal keys have their own, shown in [Limits](#limits). With a secret key and an `X-User-Id` header, `remaining_today` is that user's budget.

A good use: call it once when your app starts, to check that the key works before showing the chat box.

## Browser apps

You can call Frank AI straight from the JavaScript of a web page with a **public key** (`fk_pub_...`). A public key is meant to be visible in your page. It is safe because it only works from the sites you list on it, and it can only chat.

### Set up

1. Create a public key in the [developer portal](https://developer.frankapi.workers.dev).
2. List the addresses of your site, for example `https://my-site.vercel.app`. Addresses must use `https`. You can list up to 5.
3. `http://localhost` with any port is allowed by default, so you can develop on your own computer. Turn it off in the portal if you want.
4. Put the key in your JavaScript and call `/v1/chat` or `/v1/respond`.

No CORS setup is needed. The API answers browser preflight requests and allows the `Authorization` and `Content-Type` headers from any site. The key's own list of sites is what decides who may use it.

### A complete chat page

Save this as `index.html`, put your public key in it and open it from a local server such as the VS Code Live Server extension (`http://localhost:5500`). Opening the file directly will not work.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Ask Frank AI</title>
</head>
<body>
  <form id="form">
    <input id="question" placeholder="Ask something" required maxlength="500">
    <button>Send</button>
  </form>
  <p id="answer"></p>

  <script>
    const KEY = "YOUR_PUBLIC_KEY"; // a public key is meant to be in browser code

    document.getElementById("form").addEventListener("submit", async (event) => {
      event.preventDefault();
      const answer = document.getElementById("answer");
      answer.textContent = "Thinking...";
      try {
        const res = await fetch("https://api.frankai.workers.dev/v1/chat", {
          method: "POST",
          headers: { "Authorization": "Bearer " + KEY, "Content-Type": "application/json" },
          body: JSON.stringify({ message: document.getElementById("question").value }),
        });
        const data = await res.json();
        answer.textContent = res.ok ? data.reply : data.error.message;
      } catch (err) {
        answer.textContent = "Could not reach the server. Check your connection.";
      }
    });
  </script>
</body>
</html>
```

The page shows the reply with `textContent`, not `innerHTML`, so nothing the assistant writes can run as code in your page. Keep it that way.

### What public keys cannot do

- They cannot make pictures or voices. Public keys can chat and use `/v1/respond` without `speak` or `image`. For anything else, call your backend, and let your backend call Frank AI with a secret key. See [Backend apps](#backend-apps).
- They have smaller limits (30,000 tokens a day for the whole key) and each visitor has their own budget of 5,000 tokens a day and 6 requests a minute. Visitors are told apart by their network address, so people on the same Wi-Fi or mobile network share one visitor budget. See [Limits](#limits).

### Fix common errors

| You see                                                  | Why                                                     | What to do                                                                                                                       |
| -------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `origin_not_allowed` and "opened as a file"              | The page was opened with `file://`.                     | Serve it from `http://localhost:5500` (or another local server), which the key allows by default.                                |
| `origin_not_allowed` and "not allowed on https://..."    | The page's address is not in the key's list.            | Add the exact address, including `https://`, in the portal. Addresses must match exactly, so `www.` and no `www.` are different. |
| `secret_key_in_browser`                                  | You used a secret or Terminal key in browser code.      | Revoke that key (it is no longer private) and use a public key in the page.                                                      |
| `origin_required`                                        | The public key was used without a browser page, for example from curl. | Call from the page, or from a backend with a secret key. For a quick test see below.                              |
| `daily_budget_exceeded`                                  | The visitor or the key used today's budget.             | Show a friendly message. The budget resets at 00:00 UTC.                                                                         |
| A network error and no answer                            | The device is offline, or an extension blocked the request. | Show a message and offer to try again.                                                                                       |

### Test a public key from the terminal

A public key needs an `Origin` header that is on its list. In a terminal you can add one yourself, which is handy for testing:

```bash
curl https://api.frankai.workers.dev/v1/chat \
  -H "Authorization: Bearer YOUR_PUBLIC_KEY" \
  -H "Origin: http://localhost:5500" \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'
```

This only works with an address that is on the key's list, so it does not let anyone borrow your key from another site.

## Backend apps

A **secret key** (`fk_sec_...`) is for code you control: a Node.js, Python, PHP or any other server. With it you can chat, make pictures and voices, and get the larger limits. A secret key must never reach a browser or a mobile app.

The usual pattern is: your page or app calls **your** backend, and your backend calls Frank AI.

```text
Browser or mobile app  ->  your backend (holds the secret key)  ->  Frank AI API
```

This hides your key, lets you decide who may ask, and lets you use pictures and voices. Mobile apps always follow this pattern, because a key put inside an app can be read by anyone.

### Keep the key in an environment variable

```bash
# .env  (add this file to .gitignore and never commit it)
FRANK_AI_KEY=fk_sec_your_key_here
```

Read it with `process.env.FRANK_AI_KEY` in Node.js or `os.environ["FRANK_AI_KEY"]` in Python.

### Node.js example (Express)

```js
import express from "express";

const app = express();
app.use(express.json());

app.post("/api/ask", async (req, res) => {
  const userId = req.ip; // use your own user id here if your app has logins
  const upstream = await fetch("https://api.frankai.workers.dev/v1/chat", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.FRANK_AI_KEY}`,
      "Content-Type": "application/json",
      "X-User-Id": String(userId).slice(0, 100),
    },
    body: JSON.stringify({ message: String(req.body.message ?? "").slice(0, 4000) }),
  });
  const data = await upstream.json();
  res.status(upstream.status).json(data);
});

app.listen(3000);
```

### Python example (Flask)

```python
import os
import requests
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.post("/api/ask")
def ask():
    user_id = request.remote_addr  # use your own user id if your app has logins
    upstream = requests.post(
        "https://api.frankai.workers.dev/v1/chat",
        headers={
            "Authorization": "Bearer " + os.environ["FRANK_AI_KEY"],
            "X-User-Id": str(user_id)[:100],
        },
        json={"message": str((request.get_json(silent=True) or {}).get("message", ""))[:4000]},
        timeout=60,
    )
    return jsonify(upstream.json()), upstream.status_code
```

### Give every user their own budget

Send your user's id in the `X-User-Id` header (up to 100 characters). Each user then has a daily budget of 10,000 tokens, so one heavy user cannot use up the whole key. Without the header, all your users share the key's budget of 200,000 tokens.

### Handle errors well

- **`rate_limited` and `daily_budget_exceeded`** (status 429): wait for the `Retry-After` seconds, or tell the user to come back tomorrow.
- **`ai_busy`, `temporarily_unavailable`, `server_error`** (503 and 500): these are temporary. Try again a couple of times with a short wait. These calls are not charged.
- **Other 4xx errors**: your request needs a change. Read the `message`, fix the cause, and do not retry the same request.

```js
async function askFrankAI(message, tries = 3) {
  for (let attempt = 1; attempt <= tries; attempt++) {
    const res = await fetch("https://api.frankai.workers.dev/v1/chat", {
      method: "POST",
      headers: { "Authorization": `Bearer ${process.env.FRANK_AI_KEY}`, "Content-Type": "application/json" },
      body: JSON.stringify({ message }),
    });
    const data = await res.json();
    if (res.ok) return data.reply;

    const temporary = ["ai_busy", "temporarily_unavailable", "server_error"].includes(data.error.code);
    if (!temporary || attempt === tries) throw new Error(data.error.message);
    await new Promise((done) => setTimeout(done, attempt * 1000)); // wait 1 s, then 2 s
  }
}
```

### Pictures and voices

Call [`/v1/image`](#pictures), [`/v1/tts`](#voice) or [`/v1/respond`](#respond) from your backend, then send the `url` to your page. The links are public and expire after 24 hours, so download the file to your own storage if you want to keep it. See [Files](#files).

### Checklist before you go live

- The key is in an environment variable, and `.env` is in `.gitignore`.
- Your backend limits message length and does its own login or rate limiting.
- Errors show a friendly message, and `Retry-After` is respected.
- You send `X-User-Id` so one user cannot use the whole budget.

## Identity

Every answer comes from one assistant with one identity. By default it is **Frank AI**, running on **Frank AI Sprinta 2**, built by Frank (Frank Nabasa) at Islle.

### When the assistant talks about itself

The assistant states its name, model and maker **only when the user asks**, for example "Who are you?" or "Who made you?". Otherwise it just answers the question: it does not introduce itself, and it does not open replies with a greeting that names it.

When it is asked, it answers with its identity and says it does not share technical internals. It never says or suggests that it is another AI, or that another company or AI provider made it or runs it. If a user tries to make it claim otherwise, or asks it to reveal or ignore its rules, it politely declines.

As an extra safeguard, a check on the answers rewrites any sentence in which the assistant claims to be another AI or to be made by another company. The check reads English text.

Frank AI routes each request across several AI providers and switches between them automatically. The providers are not named, and the assistant does not discuss them.

### Give a key its own identity

Each key can have its own assistant, model and company name. This is useful when you build an app of your own and want the assistant to appear under your product's name. Set them in the key's settings in the [developer portal](https://developer.frankapi.workers.dev).

| Setting        | Default             | Notes                                           |
| -------------- | ------------------- | ----------------------------------------------- |
| Assistant name | Frank AI            | What the assistant calls itself.                |
| Model name     | Frank AI Sprinta 2  | The name it gives for the model it runs on.     |
| Company name   | Islle               | Who it says it is from.                         |

- Leave a field blank to use the default.
- Each name is 2 to 40 characters: letters, numbers, spaces and `. , & ' + -`.
- Names of other AI companies and models, such as ChatGPT, Llama or Gemini, are refused.
- With a custom company name, the assistant says it is from your company, and the Frank AI background story is left out.
- `GET /v1/me` shows the assistant name for a key. See [Your key](#your-key).

### Your own instructions

A `system` message in a [chat](#chat) request can change the assistant's tone, length and topic, for example "Answer in at most two sentences" or "Only talk about cooking". It cannot change the assistant's name, model, company or maker. The identity rules always win.

### Good to know

- The identity applies to every endpoint that writes text: [Chat](#chat) and [Respond](#respond).
- Pictures and voices have no identity.

## Limits

Frank AI API is free and shared by many developers, so every key has limits. There are no paid plans, and nothing to buy or upgrade: the limits keep the service fair for everyone.

### Limits per key type

| Limit                            | Public key           | Secret key                         | Terminal key   |
| -------------------------------- | -------------------- | ---------------------------------- | -------------- |
| Requests per minute, per key     | 30                   | 60                                 | 20             |
| Requests per minute, per visitor | 6                    | not applicable                     | not applicable |
| Tokens per day, per key          | 30,000               | 200,000                            | 40,000         |
| Tokens per day, per end user     | 5,000 (per visitor)  | 10,000 (when you send `X-User-Id`) | not applicable |
| Largest reply (`max_tokens`)     | 512                  | 1,024                              | 1,024          |

- **Requests per minute** is how fast you may send. If you go faster you get `rate_limited` with a `Retry-After` header. Per-minute limits are approximate, because the service runs in several copies.
- **Tokens per day** is your daily budget. It resets at 00:00 UTC. When it is used up you get `daily_budget_exceeded`, and `Retry-After` says how many seconds remain.
- **Per end user** budgets stop one heavy user from using the whole key. For secret keys they apply when you send the `X-User-Id` header (see [Authentication](#authentication)). For public keys each visitor is told apart by network address, so people on the same Wi-Fi or mobile network share one visitor budget.
- **`max_tokens`** is the longest reply you can ask for.

### What a call costs

| Call                         | Tokens used from the daily budget                                                              |
| ---------------------------- | ---------------------------------------------------------------------------------------------- |
| `POST /v1/chat`              | What the reply really used (input and output), as reported by the AI provider or estimated     |
| `POST /v1/image`             | 1,000                                                                                          |
| `POST /v1/tts`               | 300                                                                                            |
| `POST /v1/respond`           | The chat tokens, plus the picture and the voice if you asked for them and they worked          |
| `GET /v1/me`, `GET /files/...` | 0                                                                                            |

A token is a small piece of text, roughly three quarters of a word. A call that fails is not charged, and neither is an `ai_busy` answer.

### Size limits

| What                                         | Limit        |
| -------------------------------------------- | ------------ |
| Request body                                 | 100 KB       |
| Messages in one conversation                 | 20           |
| One message                                  | 4,000 characters |
| Picture description (`prompt`)               | 1,000 characters |
| Text for a voice (`text`)                    | 200 characters   |
| How long a picture or voice file stays online | 24 hours    |

### Check what is left

`GET /v1/me` shows your remaining tokens and costs nothing. See [Your key](#your-key). Every answer from chat, pictures, voice and respond also carries `usage.remaining_today`.

### Stay inside the limits

- Keep conversations short: send only the messages the assistant needs. Every message you send counts as input.
- Set `max_tokens` to what you really need.
- Cache answers you will need again, for example a daily tip.
- Show a friendly message on `daily_budget_exceeded` instead of retrying. Retrying does not help until the budget resets.
- Wait for `Retry-After` on every 429 before you try again.
- Use one secret key per project and send `X-User-Id`, so you can see which user is using the most.
- Do not make extra accounts, or spread one project over extra keys, to get a bigger budget. It is against the [Usage rules](https://api.frankai.workers.dev/docs/rules) and can get an account suspended.

### Pictures and voices share a daily file limit

Every new picture or voice file is saved for 24 hours, and the service can save a limited number of new files each day, shared by all developers. When that is used up, picture and voice calls answer `storage_unavailable` and are not charged. Try again later: it resets every day.

## Errors

When a call fails, the API answers with an HTTP error status and a JSON body in one fixed shape:

```json
{
  "error": {
    "code": "daily_budget_exceeded",
    "message": "This key has used today's free token budget. It resets at 00:00 UTC."
  }
}
```

- **`code`** is stable. Test it in your code.
- **`message`** is a sentence for people. It can change, so show it to users or log it, but do not test it in code.

### Every error code

| Code                      | HTTP status             | What it means                                                                                                      | What to do                                                                                            |
| ------------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| `missing_key`             | 401                     | No key was sent.                                                                                                   | Send the header `Authorization: Bearer YOUR_KEY`.                                                     |
| `invalid_key`             | 401                     | The key does not exist or was revoked.                                                                             | Check the key for typing mistakes or spaces. Create a new key in the developer portal if needed.      |
| `too_many_attempts`       | 429                     | Too many invalid keys were sent from your address in a minute.                                                     | Fix the key, then wait the number of seconds in the `Retry-After` header (60).                        |
| `pending_approval`        | 403                     | Your developer account has not been approved yet.                                                                  | Wait for the Frank AI team to approve it. The key starts working by itself afterwards.                |
| `developer_suspended`     | 403                     | Your developer account is not active.                                                                              | Contact the Frank AI team.                                                                            |
| `scope_not_allowed`       | 403                     | This key type cannot use this endpoint or option, for example a public key asking for a picture or a voice.        | Call it from your backend with a secret key.                                                          |
| `secret_key_in_browser`   | 403                     | A secret or Terminal key was sent from a browser (the request carried an `Origin` header).                         | Use a public key in browser code. Keep secret keys on your backend.                                   |
| `origin_required`         | 403                     | A public key was used without a browser page behind it (no `Origin` header).                                       | Call from a web page on an allowed site, or call from your backend with a secret key.                 |
| `origin_not_allowed`      | 403                     | The page's address is not on the key's list of allowed sites, or the page was opened as a file (`file://`).        | Add the site to the key in the developer portal, or serve your page from `http://localhost`.          |
| `rate_limited`            | 429                     | Too many requests in a minute for this key (or from this visitor, for a public key).                               | Wait the number of seconds in the `Retry-After` header, then try again.                               |
| `daily_budget_exceeded`   | 429                     | The key (or the end user) has used today's free token budget.                                                      | Wait until 00:00 UTC. `Retry-After` tells you how many seconds are left.                              |
| `invalid_json`            | 400                     | The request body is missing or is not a JSON object.                                                               | Send valid JSON and the header `Content-Type: application/json`.                                      |
| `bad_request`             | 400                     | A field is missing, has the wrong type or is out of range.                                                         | Read the `message`: it says what to change.                                                           |
| `payload_too_large`       | 413                     | The request body is larger than 100 KB.                                                                            | Send less text.                                                                                       |
| `request_rejected`        | 400                     | The AI could not process this request.                                                                             | Try different wording.                                                                                |
| `ai_busy`                 | 503                     | All AI providers are busy or out of quota right now.                                                               | Try again in a moment, with a short wait between tries.                                               |
| `service_not_configured`  | 503                     | No provider is available for this service at the moment.                                                           | Try again later. If it keeps happening, contact the Frank AI team.                                    |
| `storage_unavailable`     | 503                     | The picture or voice file could not be saved. The service saves a limited number of new files each day.            | Try again later. The limit resets every day.                                                          |
| `temporarily_unavailable` | 503                     | Your key could not be checked right now.                                                                           | Try again in a moment.                                                                                |
| `not_found`               | 404                     | The route does not exist, or the file name is wrong or the file has expired.                                       | Check the URL. Files expire after 24 hours.                                                           |
| `server_error`            | 500                     | Something went wrong on our side.                                                                                  | Try again. If it keeps happening, contact the Frank AI team.                                          |
| `nothing_to_speak`        | 200 (inside the answer) | Only inside `errors.audio` of a `/v1/respond` answer: the reply had no text that can be spoken.                    | Ask for a reply in plain sentences.                                                                   |

### Which errors to retry

| Kind                       | Codes                                                                                                                                                       | What to do                                                                                  |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Temporary, safe to retry   | `ai_busy`, `temporarily_unavailable`, `service_not_configured`, `storage_unavailable`, `server_error`                                                       | Wait a second or two and try again, up to a few times. These calls are not charged.         |
| Wait, then retry           | `rate_limited`, `too_many_attempts`                                                                                                                         | Wait the seconds in the `Retry-After` header.                                               |
| Retry tomorrow             | `daily_budget_exceeded`                                                                                                                                     | Do not retry now. Show a friendly message. The budget resets at 00:00 UTC.                  |
| Fix your request           | `invalid_json`, `bad_request`, `payload_too_large`, `request_rejected`                                                                                      | Read the `message`, change the request, send it again.                                      |
| Fix your key or your setup | `missing_key`, `invalid_key`, `scope_not_allowed`, `secret_key_in_browser`, `origin_required`, `origin_not_allowed`, `pending_approval`, `developer_suspended` | Retrying will not help. See [Authentication](#authentication).                              |

### Handle errors in JavaScript

```js
async function ask(message) {
  const res = await fetch("https://api.frankai.workers.dev/v1/chat", {
    method: "POST",
    headers: { "Authorization": "Bearer YOUR_KEY", "Content-Type": "application/json" },
    body: JSON.stringify({ message }),
  });
  const data = await res.json();
  if (res.ok) return data.reply;

  const { code, message: text } = data.error;
  if (code === "daily_budget_exceeded") {
    throw new Error("The free budget is used up for today. Please come back tomorrow.");
  }
  if (code === "rate_limited") {
    const wait = Number(res.headers.get("Retry-After")) || 10;
    throw new Error(`Too many requests. Try again in ${wait} seconds.`);
  }
  throw new Error(text);
}
```

`Retry-After` can be read from browser code because the API lists it in `Access-Control-Expose-Headers`.

### Handle errors in Python

```python
import os
import requests

res = requests.post(
    "https://api.frankai.workers.dev/v1/chat",
    headers={"Authorization": "Bearer " + os.environ["FRANK_AI_KEY"]},
    json={"message": "Hello"},
    timeout=60,
)
data = res.json()
if not res.ok:
    code = data["error"]["code"]
    if code in ("rate_limited", "too_many_attempts"):
        wait = int(res.headers.get("Retry-After", "10"))
        print("Wait", wait, "seconds")
    else:
        print(code, data["error"]["message"])
```

### Good to know

- Every error answer is JSON, including `404` and `500`.
- A call that returns an error is not charged.
- Still stuck? Look at the [FAQ](#faq).

## FAQ

### About the service

**Is Frank AI API really free? Do I need a credit card?**
It is free, and it never asks for a credit card or any payment. Signing up, creating keys and using the API do not involve payment details, and there are no paid plans. Each key has a daily budget of tokens instead, so the service stays fair for everyone. See [Limits](#limits).

**Who is it for?**
Junior developers who want to add AI to a project: a chat box on a portfolio site, a study helper, a voice reader, a picture maker. It is built to be simple, with plain HTTPS requests and JSON.

**Which AI model is behind it?**
The assistant is Frank AI, running on Frank AI Sprinta 2. Behind the scenes the service uses several AI providers and switches between them automatically, so answers keep coming when one is busy. The providers are not named. See [Identity](#identity).

**Does it support streaming?**
No. Each reply arrives whole, in one response.

**Does the API remember my conversation?**
No. Each request stands alone. To continue a conversation, send the earlier messages again in the `messages` list. See [Chat](#chat).

**Which programming languages can I use?**
Any language that can send an HTTPS request with a header and a JSON body: JavaScript, Python, PHP, Java, C#, Go, Dart and more. There is no SDK to install.

### Privacy

**Are my messages stored?**
The request log records only details about each call: when it happened, which key, which endpoint, the status, the time it took and the tokens used. It does not include the text of your messages or of the replies. Your text is sent to AI providers to produce the answer, so do not send passwords, ID numbers or other private information. Everything the service keeps is listed in the [Privacy policy](https://api.frankai.workers.dev/docs/privacy).

### Keys and access

**How do I get a key?**
Sign up in the [developer portal](https://developer.frankapi.workers.dev), wait for approval, then create a key. See [Getting started](#getting-started).

**Why do I get `pending_approval`?**
Your developer account has not been approved yet. Your keys start working by themselves after approval.

**Which key do I use where?**
A **public key** in browser code, a **secret key** on your backend, a **Terminal key** in the `frankai` command line. See [Authentication](#authentication).

**Can I use it in a mobile app?**
Put the call behind your own backend and keep the secret key there. A key inside a mobile app can be read by anyone. See [Backend apps](#backend-apps).

**I get `secret_key_in_browser`. What do I do?**
You used a secret or Terminal key in browser code. Revoke it in the portal, because it is no longer private, and use a public key in the page instead.

**I get `origin_not_allowed` on my own computer.**
If your page was opened as a file (`file://`), serve it from a local server such as `http://localhost:5500` instead. If the error names an address, add that exact address to the key in the portal. Addresses must use `https`, except `localhost`. See [Browser apps](#browser-apps).

**Can I make pictures or voices from a web page?**
Not directly. Public keys can only chat. Call your backend, and let it call Frank AI with a secret key. See [Backend apps](#backend-apps).

### Limits and errors

**I get `daily_budget_exceeded`. How do I get more tokens?**
Wait until 00:00 UTC, when the budget resets. Limits are fixed per key type. Secret keys have larger budgets than public keys, and you can send `X-User-Id` so users do not use each other's share. See [Limits](#limits).

**Can I make more than one account to get more tokens?**
No. Each person may have one developer account. Extra accounts made to get more tokens, or to get past a limit, are against the [Usage rules](https://api.frankai.workers.dev/docs/rules) and can lead to a suspension or a permanent ban. If a public key is too small for your project, you can ask for a secret key in the portal and explain what you are building.

**What happens if I break the rules?**
It depends on how serious it is: a note in the portal, revoked keys, a suspended account (its keys answer `developer_suspended`) or a permanent ban. See the [Usage rules](https://api.frankai.workers.dev/docs/rules#what-happens-if-the-rules-are-broken).

**I get `ai_busy`.**
Every AI provider was busy at that moment. Wait a second or two and try again. The call was not charged. See [Errors](#errors).

**Why did my picture link stop working?**
Files are deleted after 24 hours. Download a picture or an audio file to your own storage if you want to keep it. See [Files](#files).

**Is the service up?**
`GET https://api.frankai.workers.dev/health` answers `{ "ok": true }` when it is running.

## Rules, terms and privacy

Frank AI API is free and many developers share it. By signing up for a developer account or using a key, you accept the full texts, which are kept up to date on the hosted docs:

- [Usage rules](https://api.frankai.workers.dev/docs/rules)
- [Terms of use](https://api.frankai.workers.dev/docs/terms)
- [Privacy policy](https://api.frankai.workers.dev/docs/privacy)

The short version:

- **One person, one developer account.** Do not make extra accounts, spread one project over several keys, or hide who you are to get around the limits.
- **Keep your keys private.** Never put a secret or Terminal key in a web page, a public repository or a mobile app. You are responsible for everything done with your keys.
- **Do not use the service for harm.** That means nothing illegal, nothing sexual involving children, no harassment or threats, no scams or spam, no malware, and nothing that collects or exposes other people's private information.
- **Rules can be enforced.** Depending on how serious the problem is: a note in the portal, revoked keys, a suspended account, or a permanent ban.
- **The service is shared and given as it is.** It can be slow, busy or unavailable, and it can change. AI replies can be wrong, so check them before you rely on them, and do not use them alone for medical, legal, money or safety decisions.
- **Privacy.** The service does not store the text of your messages or of the replies. Picture and voice files are kept for 24 hours, usage counters for 7 days and the request log for 30 days. There are no ads, no cookies on the docs, and no payment details. Your text is sent to AI providers to produce the answer, so do not send passwords, ID numbers or other private information.

## The frankai terminal package

A Terminal key (`fk_cli_...`) is created for you when you sign in from the `frankai` command line. It has a smaller budget (20 requests per minute, 40,000 tokens per day), can use chat, respond, voice and pictures, and cannot be used from a browser.

<!-- TODO: add install and usage instructions for the frankai terminal package (package name, install command, sign-in, commands). The hosted API docs do not cover them. -->

## Machine-readable docs

| Resource            | Address                                                  | Use it for                                              |
| ------------------- | -------------------------------------------------------- | ------------------------------------------------------- |
| OpenAPI 3.1         | https://api.frankai.workers.dev/openapi.json             | Endpoints, fields, limits and errors                    |
| llms.txt            | https://api.frankai.workers.dev/llms.txt                 | A short summary and a list of every page                |
| llms-full.txt       | https://api.frankai.workers.dev/llms-full.txt            | All documentation in one Markdown file                  |
| Markdown pages      | `https://api.frankai.workers.dev/docs/<page>.md`         | Any single page as clean Markdown                       |
| Sitemap             | https://api.frankai.workers.dev/sitemap.xml              | Every documentation page with its last update date      |
| Status              | https://api.frankai.workers.dev/health                   | `{ "ok": true }` when the service is running            |

If you use an AI assistant to write code against this API, point it at `llms-full.txt` or the OpenAPI file so it uses the real endpoints, fields and limits.

## Examples in this repo

Browse the [`examples`](./examples) folder for runnable, copy-paste examples. Most examples read your key from an environment variable:

```bash
export FRANK_AI_KEY=fk_sec_your_key_here
```

Never commit a real key to this repository. Use a placeholder such as `YOUR_SECRET_KEY` in anything you push.

## License

[MIT](./LICENSE)

---

Frank AI API is free for junior developers. It never asks for a credit card or any payment. Built by Frank (Frank Nabasa) at Islle.
