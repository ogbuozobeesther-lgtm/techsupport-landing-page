# TechSupport Africa enquiry page

A single-page website for TechSupport Africa. Visitors send an enquiry through a form with exactly three fields, and each submission is posted to an n8n webhook.

## Files

| File | Purpose |
| --- | --- |
| `index.htm` | The whole website (HTML, CSS and JavaScript in one file) |
| `README.md` | This guide |

## The form

| Field | Type | Rules |
| --- | --- | --- |
| `name` | text | At least 2 characters |
| `email` | email | Must look like `name@example.com` |
| `enquiry` | textarea | At least 10 characters, up to 3000 |

Nothing else is collected or sent.

### What is sent

On submit the page makes a `POST` request with `Content-Type: application/json`:

```json
{
  "name": "Amina Okafor",
  "email": "amina@example.com",
  "enquiry": "My laptop shows a black screen after the Windows logo."
}
```

The webhook address is set in one place, near the bottom of `index.htm`:

```js
var WEBHOOK_URL = "https://essywhite.app.n8n.cloud/webhook-test/customer%20enquiry";
```

The address you gave contains a space (`customer enquiry`). It is written as `%20` in the code, which is the same address. If your n8n Webhook node path has no space (for example `customer-enquiry`), copy the exact URL from the node and paste it in here.

## Run it

Open `index.htm` in a browser. To put it online, upload it to any static host (Netlify Drop, GitHub Pages, Cloudflare Pages, or your own web hosting). Rename it to `index.html` if your host requires that name.

The page loads three fonts from Google Fonts. If they cannot load, it falls back to system fonts and still works.

## Set up the n8n Webhook node

These settings must match, or the browser will block the request.

1. **HTTP Method**: set to `POST`. The node defaults to `GET`.
2. **Path**: `customer enquiry` (or whatever you changed `WEBHOOK_URL` to).
3. **Options > Allowed Origins (CORS)**: add the address the page is served from, for example `https://yoursite.com`. Use `*` while testing. Without this, the browser reports a network error even though n8n is working. If you open `index.htm` straight from your computer, the origin is `null`, so use `*` for that.
4. **Respond**: "Immediately" or "When Last Node Finishes" both work. The page treats any `2xx` response as success.

### Test URL versus production URL

- `.../webhook-test/...` works only while the n8n editor is waiting. Open the workflow, click **Listen for test event** (or Execute workflow), then submit the form. It accepts one request per click.
- `.../webhook/...` works all the time once the workflow is **Active**.

When you are ready to go live, activate the workflow and change `webhook-test` to `webhook` in `WEBHOOK_URL`.

### Reading the data in n8n

The JSON arrives in the node's body, so later nodes can use:

- `{{ $json.body.name }}`
- `{{ $json.body.email }}`
- `{{ $json.body.enquiry }}`

Common next steps are an email node that notifies your team, an auto-reply to the sender, or a row in Google Sheets or Airtable.

## Customising

- **Business name and copy**: edit the text in `index.htm`. The page has no placeholder text, but you may want to adjust the services and the three-step section to match how you work.
- **Colours and fonts**: all colours and fonts are defined as variables at the top of the `<style>` block (`--teal`, `--sun`, `--ink`, `--paper`, and the `--font-*` ones). A dark theme is included and follows the visitor's device setting.
- **Error and success messages**: found in the `ok` and `fail` blocks of the HTML, and in the `catch` handler of the script.

## Troubleshooting

| Symptom | Likely cause |
| --- | --- |
| "We could not send your enquiry" and nothing reaches n8n | CORS. Set **Allowed Origins** on the Webhook node. |
| Message says the service is not accepting enquiries (404) | The workflow is not listening (test URL) or not active (production URL). |
| n8n receives nothing and the browser shows a 404 or 405 | The Webhook node method is `GET`. Change it to `POST`. |
| Data arrives but fields look empty | Read them from `$json.body`, not `$json`. |
| Works from the hosted site but not from a local file | Add `*` to Allowed Origins, or test from the hosted address. |

## Notes

- The form has no spam protection beyond browser validation. If you start receiving junk, add a CAPTCHA or filter in n8n (for example, reject enquiries from the same email more than a few times an hour).
- The webhook address is visible to anyone who views the page source. Treat it as public, and keep any private logic inside n8n.
