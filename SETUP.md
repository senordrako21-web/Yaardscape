# Yardscape Website + Booking System — GitHub Pages Edition

This is a static website for **Yardscape Landscaping & Beautification Services** with a Google Apps Script booking backend. GitHub Pages hosts the public pages; Apps Script processes requests, writes to your Google Sheet, and sends emails.

## Included files

- `index.html` — mobile-friendly homepage and services
- `booking.html` — booking / quote request form, date calendar input, time selection, price display, availability check
- `admin.html` — private-key-protected calendar, booking details, status changes, recent booking logs
- `style.css` — shared website styling
- `config.js` — one place to paste the Apps Script web-app URL
- `Code.gs` and `apps-script/Code.gs` — Google Apps Script backend (same source)
- `assets/` — logo and landscaping images

## Services and prices

- General Lawncare — JMD 8,000
- Plant Bed Maintenance — Quote Required
- Flowers Planting — Quote Required
- Hedging — JMD 5,000
- Sprinkler Installation — JMD 12,000

Booking hours are Monday–Friday 8:00 AM–6:00 PM and Saturday 9:00 AM–4:00 PM. Sundays are closed. The system accepts **one active booking per date**; a second request for that date is rejected by the backend, even if two customers submit at nearly the same time. Cancelled bookings no longer block the date. Requests start as `Pending Confirmation`.

---

# Part A — Create and configure the Google Sheet

1. Sign in to the Google account that should own/send the Yardscape emails. Ideally, use the account that owns `Yardscapelandscaping@gmail.com`.
2. Create or open the Google Sheet you want to use for bookings. Copy the spreadsheet ID from its URL. For example, in `https://docs.google.com/spreadsheets/d/ABC123xyz/edit`, the ID is `ABC123xyz`.
3. Keep that Sheet accessible to the same Google account that will own the Apps Script deployment.

# Part B — Install the Apps Script backend

1. Visit [Google Apps Script](https://script.google.com/) and create a **New project**.
2. Open the default `Code.gs`, delete its sample code, and paste all the code from this package's `Code.gs` file. (The same file is also in `apps-script/Code.gs`.)
3. In Apps Script, open **Project Settings** (gear icon) → **Script properties** → **Add script property**. Add these two properties:
   - `YARDSCAPE_SHEET_ID` = the spreadsheet ID from Part A
   - `YARDSCAPE_ADMIN_KEY` = a long, private password/key that you create
4. Do not put the Sheet ID or admin key in website files or commit them to GitHub. Save the project.
5. In the function dropdown above the editor, select **`initializeYardscape`**, then click **Run**. Approve the requested Google permissions. If prompted, choose the correct Google account.
6. Return to your spreadsheet. You should see two tabs: `Bookings` and `Booking Logs`. If setup errors, fix the reported Script Property or access problem before continuing.
7. In **Project Settings → General settings**, set the project time zone to **(GMT-05:00) Jamaica** if that setting is available. The code itself uses `America/Jamaica` for date handling.

## What the two tabs store

`Bookings` contains timestamp, service, price/quote, date, time, customer name, email, phone, property location, notes, status and booking reference.

`Booking Logs` records booking creation, rejected duplicate-date requests, status changes, setup events, email failures and backend errors. The backend logs the failure reason when email delivery fails.

# Part C — Deploy the Apps Script web app

1. In Apps Script click **Deploy → New deployment**.
2. Click the gear beside “Select type” and choose **Web app**.
3. Set **Execute as** to **Me** (the account that owns the script) and **Who has access** to **Anyone**. This is required so public visitors can submit requests.
4. Click **Deploy** and complete any authorization prompts.
5. Copy the **Web app URL** ending in `/exec`. Do not use the `/dev` test URL.
6. Open `config.js` in this package and replace the placeholder value with the URL, keeping the quotation marks. Example:

   `window.YARDSCAPE_API_URL = "https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec";`

7. Save `config.js`. If you later change the deployment URL, update this one file and republish the site.
8. Test the endpoint by opening `YOUR_WEB_APP_URL?action=health` in a browser. It should return JSON with `"ok":true` and your spreadsheet name. If it reports that the sheet is not configured, recheck Script Properties and run `initializeYardscape` again.

**Important:** When you change Apps Script code later, use **Deploy → Manage deployments → Edit → New version → Deploy**. A saved code edit alone does not update the deployed web app.

# Part D — Publish on GitHub Pages

1. Open your GitHub repository. If you are using your existing Yardscape repository, keep a backup/download of its current files first.
2. Upload the contents of this package to the repository's **root** folder (not the ZIP file itself). The root should directly contain `index.html`, `booking.html`, `admin.html`, `style.css`, `config.js`, and `assets/`.
3. Commit the changes to your main branch.
4. In the repository, open **Settings → Pages**. Choose **Deploy from a branch**, select your main branch and `/ (root)`, then save. If Pages is already configured that way, no change is needed.
5. Wait for the GitHub Pages deployment to finish. Open the published site URL and test the homepage, booking page and contact links.
6. If the homepage returns 404, check that the filename is exactly `index.html` (lowercase) and that it is in the published root, then check the latest deployment under the repository's **Actions** tab.

GitHub Pages hosts the front end only. It does not run the booking code itself; Apps Script does that work.

# Part E — End-to-end test before sharing publicly

1. Open `booking.html` on the published website.
2. Submit a test booking using an email address you can access and a date that is free. Use your own details for testing.
3. Confirm the new row appears in `Bookings`, and `BOOKING_CREATED` appears in `Booking Logs`.
4. Confirm the business inbox receives a new booking email and the test customer inbox receives a booking-request receipt. Check Spam/Junk too.
5. Try to submit a second booking on the same date. The page should show that the date is unavailable; the backend independently rejects duplicate active bookings.
6. Open `admin.html`, enter the private key matching `YARDSCAPE_ADMIN_KEY`, and click **Load bookings**. Select the booking date, change its status and save. Confirm the sheet and log are updated.
7. Cancel the test booking from the admin page after testing. A cancelled booking will not block that date.

## Email behaviour

Emails are sent using Apps Script `MailApp` under the Google account that owns and authorizes the script. To have them sent from your business account, create/deploy the script while signed into the Google account for `Yardscapelandscaping@gmail.com` and authorize it there. The customer email's reply-to is the business email, and the owner notification's reply-to is the customer's address. If you deploy under a different Google account, the actual sender may be that account instead.

The website reports a booking as received only after it is saved to the Sheet. If an email fails after the booking is saved, the request remains in the Sheet and the log records the email failure. Google account quotas and permissions can affect email delivery.

## Security and privacy notes

- The public website repository must contain only the public `config.js` API URL. Never add `YARDSCAPE_ADMIN_KEY`, passwords or private credentials to the repository.
- The admin page is publicly reachable, but admin actions require the private key. Use a long unique key and do not share it. The current static admin interface passes the key to Apps Script in the request URL; avoid using the admin page on shared/public computers. For stronger security, use Google-account-authenticated admin hosting/backend.
- The web app must be publicly callable for customer bookings. Do not store sensitive customer information beyond what is needed to manage appointments.
- The site and booking backend are prepared in this package, but cannot be considered live or verified until you configure the Google account, Sheet, Apps Script deployment and test actual email delivery.

## Troubleshooting

- **`initializeYardscape` does not appear:** Make sure the entire `Code.gs` was pasted into the Apps Script editor, click Save, then reload the editor and check the function dropdown. Search the editor for `function initializeYardscape()`.
- **`Missing Script Property YARDSCAPE_SHEET_ID`:** Add that exact property in Project Settings → Script properties. The value must be the spreadsheet ID only, not the whole URL.
- **`Missing Script Property YARDSCAPE_ADMIN_KEY`:** Add that exact property and use the same key in `admin.html`'s form when logging in.
- **Could not connect from website:** Confirm `config.js` contains the current deployed URL ending in `/exec`; deployment access must be **Anyone**. Redeploy a new version after backend code changes.
- **Bookings are not in the Sheet:** Check that the script's `YARDSCAPE_SHEET_ID` points to the intended spreadsheet and that the deployment executes as the account with edit access. Check Apps Script **Executions** and the `Booking Logs` tab.
- **Email did not arrive:** Check Spam/Junk, Apps Script authorization, the account that owns the deployment, Gmail Sent, and `Booking Logs` for `OWNER_EMAIL_FAILED` or `CUSTOMER_EMAIL_FAILED`.
- **Admin says Unauthorized:** Enter the exact same value saved as `YARDSCAPE_ADMIN_KEY`; it is case-sensitive.

## Contact details shown on the site

Yardscape Landscaping & Beautification Services  
Email: Yardscapelandscaping@gmail.com  
Phone: +1 876 342-7405  
Location: St. Ann, Jamaica — serving islandwide  
Hours: Monday–Friday 8:00 AM–6:00 PM; Saturday 9:00 AM–4:00 PM
