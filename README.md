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
| `TEXT` | Markdown/HTML rendered (sanitised with DOMPurify), links clickable |
| `QUICK_REPLIES`, `LIST_PICKER` | GOV.UK secondary buttons. A click sends the button's value as the text (see below) |
| `RICH_LINK` | Link card |
| `ADAPTIVE_CARD` | Rendered with Microsoft `adaptivecards` v3 and GOV.UK styling |
| `PLUGIN` or unknown types, or Adaptive Card JSON sent as text | Deep-scanned for `"type":"AdaptiveCard"` and rendered if one is found |
| Any URL matching the xApp pattern | "Open xApp" button, plus an embedded iframe panel (opens automatically by default) |
| Cognigy's own output formats (`_quickReplies`, `_buttons`, `_gallery`, `_list`, `_image`, webchat templates) found anywhere in a message, including JSON sent as text | Buttons, gallery cards, list items, images |
| Streamed (GenAI) replies | Bubble that grows as the text arrives |

## Adaptive Card submits

When someone submits an Adaptive Card, the page collects every input on the card that has
an `id`. It sends them as JSON in the **message text**, with the button title shown in the chat:

```json
{"_cardSubmit":true,"_label":"Submit","utr":"1234567890","cardNumber":"AB1234567C"}
```

Text is the default under **Connection settings → Adaptive Card submit sends**. Don't use
**Postback** with Cognigy: CXone passes only the message text on to Cognigy, so the values
in a postback are lost and Cognigy receives just the button title.

## Button and list clicks

For the same reason, clicking a list-picker option, quick reply or Cognigy button sends the
button's **value** (its postback/payload) as the message text, as Cognigy webchat does. The chat
still shows the button title. You can switch back under **Connection settings → Button and list
clicks send**.

### Cognigy setup

In the NiCE CXone endpoint, replace the default `handleInput` with the one in
`cognigy-cxone-transformer-complete.ts` (comment out or delete the whole default
function), then switch on the transformer and the input transformer in
**Transformer Functions**. A card submit then reaches the Flow as:

- `input.data` – the card values, e.g. `{ utr: "1234567890", cardNumber: "AB1234567C" }`
- `input.text` – the button title

Other messages pass through unchanged.

For rich content from Cognigy, use the **NiCE CXone** output in your nodes (custom JSON),
for example an Adaptive Card. Give every card input an `id`, and use real booleans
(`"isRequired": true`, not `"True"`).

## Other environments

For a CXone cluster that isn't in the Environment list (for example
`uk3.nicecxone-sov2.uk`), choose **Custom** and enter the host in
**Custom environment host**, or add `?env=custom&host=uk3.nicecxone-sov2.uk` to the URL.

## Troubleshooting

- The footer and the first **Event log** line show the page version. After pushing a new
  version, hard-refresh (Ctrl+Shift+R) and check it has changed.
- **Event log (developer)** shows every SDK event, the card inputs found, and exactly what
  was sent.
- **Everything CXone delivered** under each bot message shows the full message the page
  received. **What was sent** under a card reply shows the JSON sent back.
