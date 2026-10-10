# HR Outreach Desk — JavaScript edition

Complete standalone HTML, CSS and JavaScript source for the outreach app. No React, npm dependencies, build step, ChatGPT subscription, AI API key or backend is required. Gmail authorization is required for sending.

## Files

- public/index.html — campaign editor, preview, approval and setup dialogs
- public/style.css — responsive design
- public/app.js — import, validation, attachment encoding, Google OAuth and sending queue
- public/_headers — browser headers for Cloudflare Pages
- sample-recipients.csv — import format example (replace example addresses)

## 1. Upload to GitHub

1. Extract this ZIP on your computer.
2. Create a GitHub repository, for example `hr-outreach-desk`. A private repository is fine.
3. Use **Add file → Upload files** (or the initial upload link on an empty repository).
4. Upload the contents INSIDE `HR_Outreach_JavaScript`, including the complete `public` folder. Do not upload just the ZIP.
5. Commit the upload to `main`.
6. At repository root you should see `README.md`, `sample-recipients.csv`, and `public/`. The entry file must be `public/index.html`.

## 2. Deploy on Cloudflare Pages

Open https://dash.cloudflare.com/ and go to **Workers & Pages → Create application → Pages → Import an existing Git repository**. Choose GitHub, authorize access to your repository, and select it.

Use these settings:

| Setting | Value |
| --- | --- |
| Production branch | main |
| Framework preset | None |
| Build command | exit 0 |
| Build output directory | public |
| Root directory | Leave blank (repository root) |
| Environment variables | None |

Choose **Save and Deploy**. Use the resulting HTTPS `https://YOUR-PROJECT.pages.dev` address. Select Pages, not a Worker requiring a Worker entry file. Pushing changes to the connected production branch updates the deployment.

## 3. Update Google OAuth for the new address

You can keep the same Google Cloud project and Web application OAuth Client ID you created for the original app.

1. Open https://console.cloud.google.com/ and select that project.
2. Ensure **Gmail API** is enabled.
3. Go to **Google Auth Platform → Clients** and open your Web application OAuth client.
4. Under **Authorized JavaScript origins**, ADD the exact production origin, for example `https://YOUR-PROJECT.pages.dev`. Do not add a path or trailing slash. Keep your old origin if you still use the original site.
5. Save. Google configuration changes may take time to apply.
6. Under **Audience → Test users**, add the Gmail account that will SEND emails if the app is in Testing mode. HR recipients do not need to be test users.
7. Under **Data Access**, include `https://www.googleapis.com/auth/gmail.send`.
8. Open the new site → **Connect Gmail** → paste the Web application Client ID → **Save & authorize Gmail**. Select the added test-user account and grant sending permission.

No client secret, API key, Gmail password or redirect URL is needed for this browser token flow. Never put a password or client secret in this source or GitHub. The client ID is a public identifier; this app stores it in your browser after entry.

## 4. First send

Start with ONE address that you control to verify that Gmail receives the message and the attached resume opens correctly. Then replace it with your HR list, review the message and resume, and explicitly approve the campaign.

- Up to 100 unique recipients per campaign.
- Paste addresses separated by commas, spaces, semicolons or new lines.
- CSV/TXT imports: use one `email` column only. This is not a multi-column contact importer.
- PDF or DOCX resume, maximum 5 MB.
- Each recipient receives an individual email with the attachment, without CC/BCC.
- Already sent or uncertain addresses in this browser's history are blocked from another send. Remove them from the input to continue with remaining recipients.
- Keep the tab open, internet connected, and computer awake. Background tabs can be throttled; use the foreground tab for reliable progress. This is not a server-side/background queue.
- Stop pauses after the current request. Already accepted emails cannot be recalled.
- Gmail limits and account restrictions still apply. The 100-person app limit is not a promise that Gmail will accept all 100.

## Data and migration

The new Cloudflare site is public by default; it does NOT inherit the original ChatGPT site's owner-only access. OAuth authorizes the Gmail account chosen by the visitor. Keep personal use in Testing mode with only your sending account added.

Recipient input and resume are held in browser memory, then sent directly to Google's Gmail API after approval. Access tokens are held only in memory. History and the OAuth client ID are stored in localStorage in the browser. Google Identity Services and Google Fonts load from Google.

History does not transfer between origins or browsers. Export your history from the original site before moving, and manually exclude already-contacted addresses on the new site. This version has CSV history export, not history import. Clearing browser data also clears duplicate protection. A `sent` status means Gmail accepted the message, not that it was delivered or read.

`unknown` or a leftover `sending` status means delivery may have succeeded: check Gmail Sent before trying that recipient elsewhere. The app never automatically retries ambiguous sends.

## Local use on Ubuntu

From this extracted project folder run:

```bash
python3 -m http.server 8000 --directory public
```

Open http://localhost:8000 . For local Gmail authorization add `http://localhost:8000` as an Authorized JavaScript origin too. Do not open `index.html` directly with a file:// URL for Gmail authorization.

## Troubleshooting

- **403 access_denied:** confirm the exact Gmail account is in Audience → Test users in the same Google Cloud project as the Client ID. Work/school accounts may also have administrator restrictions.
- **origin_mismatch:** add the exact current origin displayed in the app's Gmail setup dialog to your Web OAuth client's Authorized JavaScript origins.
- **404 on Cloudflare:** confirm `public/index.html` exists and the output directory is `public`.
- **Google sign-in did not load:** check your connection, browser extensions and script blocking; refresh.
- **Missing attachment after reload:** choose the resume again; the file is intentionally not persisted.

## Validation

JavaScript syntax, recipient parsing, duplicate removal and MIME attachment checks were run. Live Gmail sending requires your account authorization and has not been verified by the developer. Perform the one-address test above. Browser visual QA was unavailable in the build environment.

## Official references

- https://developers.cloudflare.com/pages/framework-guides/deploy-anything/
- https://developers.cloudflare.com/pages/get-started/git-integration/
- https://developers.google.com/identity/oauth2/web/guides/use-token-model
- https://developers.google.com/workspace/gmail/api/guides/sending
- https://support.google.com/cloud/answer/15549945
