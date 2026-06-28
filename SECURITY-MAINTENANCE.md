# clalaw.me — Security Layer Maintenance

## Overview

Access to the tools at clalaw.me is controlled by three components that must be kept in sync:

1. **Firebase Auth** — holds login credentials (passwords for email/password users)
2. **Firestore config document** — the JS allowlist (controls what the pages check at runtime)
3. **Firestore security rules** — the enforcement layer (controls what the database will actually serve)

All three are managed in the [Firebase console](https://console.firebase.google.com/) under the `clalaw-media-watchlist` project.

---

## Adding a New User

### Step 1 — Create their login credentials (Firebase Auth)

> Skip this step if they will use Google SSO with a Gmail account. Google users don't need a password entry — they just need to be on the allowlist.

1. Firebase console → **Authentication** → **Users** tab
2. Click **Add user**
3. Enter their email and set a password
4. Click **Add user** to save

### Step 2 — Add them to the Firestore allowlist

1. Firebase console → **Firestore Database** → **Data** tab
2. Navigate: `config` → `allowed-emails`
3. Click the `emails` field to edit it
4. Click the **+** button to add a new array item
5. Type: `string` / enter their email address
6. Click **Update**

### Step 3 — Add them to the Firestore security rules

1. Firebase console → **Firestore Database** → **Rules** tab
2. Find the two places the email list appears (the `config` write rule and the main `/{document=**}` rule)
3. Add their email to both lists in the same format: `'their@email.com'`
4. Click **Publish**

---

## Removing a User

### Step 1 — Delete their Auth account (if email/password user)

1. Firebase console → **Authentication** → **Users** tab
2. Find their email, click the three-dot menu → **Delete account**

### Step 2 — Remove them from the Firestore allowlist

1. Firebase console → **Firestore Database** → **Data** tab
2. Navigate: `config` → `allowed-emails`
3. Click the `emails` field to edit it
4. Click the **−** button next to their entry
5. Click **Update**

### Step 3 — Remove them from the Firestore security rules

1. Firebase console → **Firestore Database** → **Rules** tab
2. Delete their email from both places in the rules
3. Click **Publish**

> **Note:** Removing someone from the rules takes effect immediately on new requests. Any active browser session will fail on the next data read (Firestore will return a permission error). They won't be silently left in — they'll just get bounced on the next page load or data sync.

---

## Changing a Password

1. Firebase console → **Authentication** → **Users** tab
2. Find their email, click the three-dot menu → **Reset password** (sends a reset email to them) — or click into the user and set a new password directly

---

## Current Authorized Users

| Email | Auth method | Notes |
|---|---|---|
| anderson.cl@gmail.com | Google SSO | Primary |
| vbrotski@gmail.com | Google SSO | Valérie |
| anderson.cl@pm.me | Email/password | ProtonMail account |
| chris@clalaw.me | Email/password | clalaw.me account |

---

## Where Things Live

| What | Where in Firebase console |
|---|---|
| Login credentials / passwords | Authentication → Users |
| JS allowlist (what pages check) | Firestore → Data → config → allowed-emails → emails (array) |
| Database enforcement | Firestore → Rules |

---

## Important: Keep All Three in Sync

The Firestore config document and the security rules must always match. If someone is in the config document but not the rules, the page will show them in but Firestore will block their data reads — they'll see an empty or broken page. If they're in the rules but not the config document, the page will bounce them even though the database would have let them in.

When in doubt, the rules are the real enforcement. The config document is just what the JavaScript reads to decide whether to show the page.
