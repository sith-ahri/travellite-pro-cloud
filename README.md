# TravelLite Pro Cloud — agency-isolated preview

**This is a development preview, not production-ready financial software.** It extends the exact TravelLite 4 HTML supplied by the user, preserving quotation, pricing, suppliers, itinerary, bookings, dashboard, marketing, and backup tools. It adds agency registration, password-based staff login, server-side sessions, PostgreSQL workspace storage, and a Billplz sandbox payment adapter. Payments are in MOCK mode by default. Do not use real customer information yet.

## Easiest way to get it online (Windows beginner)

1. Create a free account at https://github.com and a Render account at https://dashboard.render.com. Hosting and database plans may cost money.
2. Extract the ZIP. Open the `TravelLite_Pro_Cloud` folder. Upload **the contents** of that folder into a new **private** GitHub repository (not the parent ZIP). Never upload `.env`, API keys, or customer data.
3. In Render, choose **New → Blueprint** and connect the GitHub repository. Render reads `render.yaml` and proposes a Node web service and PostgreSQL database. **Review the prices before confirming**. The Blueprint deliberately uses paid plans to avoid relying on free database availability.
4. During setup, Billplz variables can remain blank while `PAYMENT_MODE=mock`. If Render requires a value for a `sync: false` field, supply a placeholder (not a real secret) and replace it later.
5. Deploy, then visit the generated HTTPS address. Register your agency with an email and password of **at least 12 characters**. Store that password safely.
6. Create a quotation, convert it to a booking, open **Billplz Payments**, create a deposit checkout, click **Simulate payment**, return, and click **Refresh payments**. This does not move money.
7. Test the same agency login on Samsung Chrome. Create another agency account with a different email to check data separation. Each agency gets a different workspace.

## Local Windows test (advanced)

Node.js 22+, a running PostgreSQL database, and a database connection URL are required. Unlike the previous edition, Node.js alone is insufficient.

In PowerShell, inside the extracted folder:

```powershell
npm install
$env:DATABASE_URL='postgresql://postgres:YOUR_PASSWORD@localhost:5432/travellite'
$env:PAYMENT_MODE='mock'
node server.js
```

Open http://localhost:3000. Database tables are created automatically on startup.

## Billplz SANDBOX (only after you create a sandbox merchant account)

1. Create a Billplz **sandbox** account and collection at https://www.billplz-sandbox.com/ . Get sandbox Secret Key, Collection ID, and X Signature key.
2. In Render service → Environment, set `PAYMENT_MODE=sandbox`, `BILLPLZ_API_KEY`, `BILLPLZ_COLLECTION_ID`, `BILLPLZ_X_SIGNATURE_KEY`, and `PUBLIC_BASE_URL` (the public HTTPS URL of your Render service, no trailing slash). Save and redeploy.
3. Create a **test** booking with RM currency and a valid test email, create a payment, and follow Billplz's sandbox hosted page. Return to TravelLite and refresh. Callbacks are authenticated with X Signature and confirmed by querying Billplz's Get Bill endpoint before marking paid.
4. Test successful, failed, duplicate, and delayed callbacks. Confirm the balance never increases twice.
5. **Never** paste API keys into the HTML or send them in chat. Production Billplz credentials are different. This package intentionally does not support production-mode payment processing.

## Known limits and important security tasks before selling

- Workspace sync uses **optimistic revision checking**, not per-record transactions. Two staff editing at once may receive a conflict; the losing browser must reload. This is a prototype, not a finished concurrent CRM.
- Agency accounts are isolated by `agency_id` on server queries. Email uniqueness is global. Owner can create staff accounts; self-service password reset, MFA, email verification, rate limiting, audit trails, and staff deactivation are **not yet implemented**.
- Passwords are bcrypt-hashed; sessions are opaque random cookies, HTTP-only, SameSite=Lax, HTTPS-only in production. Add formal CSRF protection and an independent security review before storing customer PII.
- Old browser-local data is **not automatically migrated**. Export a backup from the old version; import into the correct new agency only after checking its contents. Backups can contain customer data; handle securely.
- Old manual payments may still be present in imported bookings; gateway payments are separately verified. Do not treat imported/manual amounts as gateway-confirmed.
- The cloud frontend currently retains the legacy localStorage initialization; authenticated cloud workspace data overrides it. Clear old browser data when switching accounts. Future versions should remove the legacy localStorage logic entirely.
- Gateway callbacks only update gateway payments; dashboard booking balances update after a signed-in staff member clicks **Refresh payments**.
- **Do not use public mock mode to imply payment collection.** Mock checkout is deliberately a simulation; use only fake bookings and test accounts.
- Add backups, retention policy, deletion/export controls, encryption and data protection compliance, monitoring, tests against real PostgreSQL, penetration testing, and disaster recovery before commercial launch.

## Files

- `public/index.html` — TravelLite 4 with cloud login and payments UI
- `server.js` — Node/Express API and Billplz sandbox adapter
- `render.yaml` — Render Blueprint with PostgreSQL
- `package.json` — dependencies
- `.env.example` — local configuration example; no credentials
- `tests/` — basic smoke tests (not an end-to-end certification)

## Official documentation

- https://render.com/docs/blueprint-spec
- https://support.billplz.com/guide/api-quick-start
- https://www.billplz.com/api/
