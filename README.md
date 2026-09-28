# CXone + Cognigy chat test harness

A single-page, GOV.UK-styled chat front end built on the NiCE CXone Chat Web SDK
(`@nice-devone/nice-cxone-chat-web-sdk`). You can use it to check that Cognigy
xApps and Adaptive Cards work when a chat starts from a custom front end.

## Run it

- **GitHub Pages:** commit `index.html` to the repo root and turn on Pages.
- **Locally:** `npx serve .` (or any static server), then open the page.

Enter the Brand ID, Channel ID and environment under **Connection settings**.
You can also pass them in the URL:

```
index.html?brandId=1234&channelId=chat_xxxx&env=UK1
index.html?demo=1        # offline demo, no CXone needed
```

The page's origin (for example `https://you.github.io`) must be allowed on the
CXone chat channel. If it isn't, the connection fails.

## What gets rendered

| From Cognigy / CXone | Rendered as |
|---|---|
| `TEXT` | Message bubble, with links made clickable |
| `QUICK_REPLIES`, `LIST_PICKER` | GOV.UK secondary buttons, sent back with `sendPostbackMessage` |
| `RICH_LINK` | Link card |
| `ADAPTIVE_CARD` | Rendered with Microsoft `adaptivecards` v3 and GOV.UK styling |
| `PLUGIN` or unknown types, or Adaptive Card JSON sent as text | Deep-scanned for `"type":"AdaptiveCard"` and rendered if one is found |
| Any URL matching the xApp pattern | "Open xApp" button, plus an embedded iframe panel (opens automatically by default) |
| Streamed (GenAI) replies | Bubble that grows as the text arrives |

When someone submits an Adaptive Card, the card's data (including input values) is sent back
as a postback: the JSON string goes in the postback and the button title goes in the text. You can
switch this to a plain JSON text message in the settings, to match what your
Cognigy flow reads (`input.data` / `input.text`).

Open **Event log (developer)** to see every raw SDK event. Turn on **Show raw
message JSON** to see the exact payload that Cognigy sent through CXone.
