# CXone native chat widget test harness

A single-page test harness that loads the standard NiCE CXone chat widget (the
Digital First Omnichannel loader) and restyles it to match the GOV.UK-styled
[custom SDK harness](https://webgiticloud.github.io/RaceCarFrontEndHarness/).

Use it to check how the native widget looks and behaves on a GOV.UK-style page,
and to work out the custom CSS before you put it into production.

This is a test tool, not a live service.

## Run it

- **GitHub Pages:** commit `index.html` to the repo root and turn on Pages.
- **Locally:** `npx serve .` (or any static web server), then open the page.

The widget must be served from a website that your CXone chat channel allows.
It will not load from a `file://` URL. If it does not appear within about
25 seconds, the page shows an error status.

## What the page does

On load, the page makes the same calls as the standard embed snippet:

```js
cxone('init', '<brandId>');
cxone('chat', 'init', <brandId>, '<channelId>');
cxone('chat', 'enableChatAlwaysOnline');           // optional
cxone('chat', 'setCustomCss', '<GOV.UK theme CSS>');
```

The page has these sections:

| Section | What it does |
| --- | --- |
| Status panel | Shows whether the widget has loaded and whether the channel is online, and has a **Start chat** button that opens the widget. |
| Connection settings | Set the brand ID, channel ID, environment, page heading, and options. Saving reloads the page, because the widget can only start once per page. |
| Widget styles (CSS) | Edit the CSS sent with `setCustomCss`. **Apply to widget** updates the open widget straight away. **Copy setCustomCss snippet** copies a ready-to-paste call. |
| Widget inspector | Lists every `data-selector` and CSS class prefix currently in the widget, plus the theme colour variables and their values. |
| Event log | Records loading, the window opening and closing, and new widget elements as they appear. |

## Settings

Settings are saved in the browser's local storage. You can also set them in the
URL, which takes priority over saved settings:

| Parameter | Example | Notes |
| --- | --- | --- |
| `brandId` | `?brandId=3756` | CXone brand (tenant) ID |
| `channelId` | `?channelId=chat_xxx` | Chat channel ID |
| `env` | `?env=UK1` | Cluster: UK1, UK2, EU1, EU2, NA1, NA2, AU1, AU2, CA1, JP1, AE1, JO1, KR1, ZA1 |
| `host` | `?host=uk3.nicecxone-sov2.uk` | Custom cluster host, which overrides `env`. The `web-modules-de-` prefix is added if missing, so the SDK harness value works here too |
| `theme` | `?theme=0` | Turn off the GOV.UK theme to see the widget's default look |
| `display` | `?display=floating` | `embedded` (default) puts the chat in the page; `floating` shows the standard corner icon |

The loader URL is built as `https://web-modules-de-<env>.niceincontact.com/loader/1/loader.js`.
For a custom host such as `uk3.nicecxone-sov2.uk` it becomes
`https://web-modules-de-uk3.nicecxone-sov2.uk/loader/1/loader.js`.

Options in Connection settings:

- **How the chat is shown** – embedded in the page (default) or the standard
  floating chat icon. See [Embedded mode](#embedded-mode).
- **Always show chat as online** – sends `cxone('chat', 'enableChatAlwaysOnline')`.
  Without it, a live chat channel that routes to a bot (such as Cognigy) shows
  "We're offline", because CXone reports the channel as offline when no agents
  are available. Use exactly this form: `cxone('chat', 'chat', 'enableChatAlwaysOnline')`
  is silently ignored.
- **Hide the waiting queue pop-up** – adds
  `[class*="WaitingQueueModal_WaitingQueueModal"] { display: none !important; }`.
- **Apply the GOV.UK theme to the widget** – sends the theme CSS.

The defaults are built in at the top of the main script (`DEFAULTS`).

## Embedded mode

The CXone loader has no option to place the chat inside a page element – it always
draws a floating window. In embedded mode the page does three things:

1. Sends `cxone('chat', 'setWindowWidth', '100%')`, `cxone('chat', 'setWindowHeight', '100%')`
   and `cxone('chat', 'openChatWindow')`, so the window opens straight away and fills its frame.
   It does not use `setFullDisplay`: that turns the chat into a full-screen dialog, which makes
   every link on the page unclickable, hides the page from screen readers and locks scrolling.
   If the widget goes full-screen anyway (for example on a small screen), the page undoes this.
2. Hides the window's minimise button with `setCustomCss`, so it stays open.
3. Pins the widget's container (`#be-chat-container`) over the `#chat-slot` panel
   with page CSS and keeps it aligned as the page scrolls or its layout changes.

To change the chat's size, change the height of `.chat-slot` in the page styles
(600px by default, limited to 80% of the screen height).

Because this repositions the widget's own elements, a future CXone release could
change how it behaves. If the chat stops lining up with the panel, switch to the
floating icon with `?display=floating`.

## How the styling works

The widget renders inside its own iframe, so CSS on the host page cannot reach
it. All styling goes through `setCustomCss`, using three kinds of hook:

1. **Theme variables** set most of the colours. The theme overrides them on
   `:root:root` with `!important`:

   | Variable | GOV.UK value |
   | --- | --- |
   | `--chat-chatCustomization-colors-headerBgColor` | `#0b0c0c` (black) |
   | `--chat-chatCustomization-colors-themeColor` | `#1d70b8` (blue) |
   | `--chat-chatCustomization-colors-buttonColor` | `#00703c` (green) |
   | `--chat-chatCustomization-colors-secondaryMessageBgColor` | `#f3f2f1` (light grey) |
   | `--chat-shape-borderRadius` | `0px` |

2. **`data-selector` attributes**, for example `[data-selector="WIDGET"]`,
   `[data-selector="HEADER"]` and `[data-selector="PRIMARY_BUTTON"]`.

3. **CSS module class prefixes**, for example `[class^="Button_Btn"]`. The widget's
   class names end in a hash (such as `Widget_Widget__qwfID`) that changes between
   releases, so match only the part before the double underscore.

Watch out for loose wildcard matches. For example, `[class*="Button_Button"]` also
matches the header's `HeaderArrowButton_Button` – use `[class^="…"]` where you can.

### Styling the conversation view

Message bubbles and the message box only appear once a conversation has started.
To style them more closely:

1. Start a chat in the widget.
2. Open **Widget inspector** and select **Scan widget**.
3. Add rules for the selectors it lists in **Widget styles (CSS)**, then select
   **Apply to widget** to try them.
4. Select **Save** to keep them, or **Copy setCustomCss snippet** to use them elsewhere.

**Reset to GOV.UK theme** restores the built-in CSS.

## Known limits

- The widget's own scripts and stylesheets are cross-origin, so the inspector
  reads the live page structure rather than the widget's source.
- Live CSS edits work because the widget's iframe is same-origin with the page.
  In production, use the copied `setCustomCss` call instead.
- The page font is Arial. GDS Transport is licensed for service.gov.uk domains only.

## Links

- [Custom SDK harness](https://webgiticloud.github.io/RaceCarFrontEndHarness/)
- [CXone chat customisation documentation](https://help.nicecxone.com/content/acd/digital/digitalchat/chatcustomizecss.htm)
