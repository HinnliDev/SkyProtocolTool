<div align="center">

# Sky Protocol Tool (SPT)
[![Русский](https://img.shields.io/badge/lang-Русский-blue)](./README_ru.md)
[![License MIT](https://badgen.net/github/license/HinnliDev/SkyProtocolTool)](https://github.com/HinnliDev/SkyProtocolTool/blob/main/LICENSE)

Sky Protocol Tool (SPT) is a plugin for the Android version of Sky: Children of the Light, loaded via Canvas. It provides a convenient interface for analyzing in-game HTTP traffic and working with requests.
</div>

---

## Features

- Intercept and display HTTP requests and responses
- Console with request/response grouping
- Log filtering and search
- Generate commands from selected requests
- Manual HTTP request sending
- Intercept commands via in-game chat
- Data copying

---

## How it works

SPT hooks into the game's internal HTTP functions and:
- intercepts outgoing requests
- analyzes incoming responses
- displays everything in the UI

It also uses real request templates to send custom commands.

---

## Interface

The main interface includes:
- log console
- filters and search
- request inspection
- manual input field

Traffic types:
- `C->S` / `SPT->S` — client → server
- `C<-S` / `SPT<-S` — server → client

---

## Manual Requests

Format:

```
/post /endpoint -c key:value
```

Examples:

```
/post /account/info -c user:123
/post /service/ping -c enabled:true
/post /example -c {"key":"value"}
```

### Chat Commands

If a message contains a `/post` command, it is intercepted and executed by SPT.  
> Nothing is sent to the chat and other players will not see it.

---

## Important

This project is intended for technical analysis and is not a general-purpose library.  
This project is distributed under the MIT License. See the [LICENSE](https://github.com/HinnliDev/SkyProtocolTool/blob/main/LICENSE) file for details.
