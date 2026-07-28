# Everything Budgeted — Setup &amp; Deployment Guide

This is a single static site — `index.html` plus two small support files
(`manifest.json`, `robots.txt`). It works fully offline out of the box
(everything saves to the browser's local storage). This guide covers:

1. Deploying it to **everythingbudgeted.com**
2. The *optional* Firebase cloud sync + accounts upgrade
3. The *optional* Stripe $5/month plan with a 7-day free trial

Parts 2 and 3 are independent of Part 1 — you can deploy the site today and
add cloud sync or billing later without redeploying anything but this one
file.

---

## Part 1 — Deploy to everythingbudgeted.com

Because it's a single static file, any static host works. Pick whichever
you're already comfortable with:

### Option A — Netlify (drag and drop, easiest)
1. Go to [app.netlify.com](https://app.netlify.com) → **Add new site → Deploy manually**.
2. Drag the folder containing `index.html`, `manifest.json`, and `robots.txt` onto the upload area.
3. Once it's live, go to **Domain management → Add a domain** → enter `everythingbudgeted.com`.
4. Netlify will show you DNS records to add at your domain registrar (usually an `A` record pointing at Netlify's load balancer IP, or a `CNAME` if using a subdomain). Add those records where you bought the domain, then wait for DNS to propagate (a few minutes to a few hours).

### Option B — Vercel
1. Go to [vercel.com](https://vercel.com) → **Add New → Project → Upload** (or connect a GitHub repo containing these files).
2. After deploy, go to **Settings → Domains → Add** → `everythingbudgeted.com`, and follow the DNS instructions shown.

### Option C — Firebase Hosting (handy if you're already using Firebase for Part 2)
1. `npm install -g firebase-tools`, then `firebase login`.
2. In the folder with `index.html`: `firebase init hosting` → pick your Firebase project → set the public directory to `.` → say **No** to "configure as a single-page app" → don't overwrite `index.html`.
3. `firebase deploy --only hosting`.
4. In the Firebase console → **Hosting → Add custom domain** → `everythingbudgeted.com` → follow the DNS verification + `A`/`TXT` record steps shown.

### Option D — GitHub Pages
1. Push these files to a GitHub repo.
2. **Settings → Pages** → deploy from the root of your main branch.
3. **Settings → Pages → Custom domain** → `everythingbudgeted.com`. GitHub will show you the DNS records to add; also add a `CNAME` file to the repo containing `everythingbudgeted.com` (GitHub does this for you automatically once you save the custom domain in settings).

Whichever host you pick, once DNS is pointed correctly you'll want HTTPS
enabled — all four of the above provision a free SSL certificate
automatically within a few minutes of the domain verifying.

---

## Part 2 — Firebase (accounts + cloud sync)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project**.
2. Inside the project, click the **`</>`** (web) icon to register a web app. Any nickname is fine — you don't need Firebase Hosting for this part unless you chose Option C above.
3. Firebase shows you a config object like:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```

4. Open `index.html`, find `var firebaseConfig = { ... }` near the top of
   the `<script>` tag, and paste your real values over the `YOUR_...`
   placeholders.
5. In the Firebase console:
   - **Authentication → Sign-in method** → enable **Email/Password**.
   - **Firestore Database → Create database** → start in production mode.
6. Add this security rule under **Firestore → Rules** so people can only
   ever read/write their own data:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{uid} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```

7. Redeploy (or just reopen `index.html` if testing locally). You'll now
   see a sign-in screen. Create an account — cloud sync is live, with local
   storage still used as an offline cache.

Leave the placeholders untouched and the app just runs locally forever —
nothing breaks.

---

## Part 3 — Stripe ($5 USD/month, 7-day free trial)

Since this is a static file with no server, it can't securely verify
payments on its own — a client-side-only check can always be edited around
in devtools. The setup below gives you a real working checkout and trial
with the least effort (a no-code Payment Link), plus an optional step to
make the "subscribed" flag tamper-proof with a small Cloud Function.

### Step 1 — Create the product
1. Sign in to [dashboard.stripe.com](https://dashboard.stripe.com).
2. **Product catalog → Add product** → name it `Everything Budgeted Pro`, pricing **Recurring**, `$5.00 USD`, **Monthly**.

### Step 2 — Create a Payment Link with the trial
1. **Payment links → Create payment link** → select that price.
2. Find **Free trial** in the subscription options and set it to **7 days**.
   (Trial settings only appear for recurring prices — make sure the price
   itself is monthly recurring, not one-time.)
3. Create the link, then copy the URL (`https://buy.stripe.com/xxxxxxxx`).

### Step 3 — Wire it into the app
Open `index.html`, find:

```js
var STRIPE_PAYMENT_LINK = "https://buy.stripe.com/YOUR_PAYMENT_LINK";
```

Replace it with your real link and save.

Every signed-in user now automatically gets a 7-day trial (tracked from
account creation), a countdown banner, and an **Upgrade — $5/mo** button
that opens Checkout with their account already attached
(`client_reference_id` + email are appended automatically).

### Step 4 (optional, recommended) — make it tamper-proof with a webhook
Out of the box, nothing sets `subscribed: true` automatically — you'd flip
it manually in the Firestore console after each payment, or add this
webhook to automate and secure it:

1. `npm install -g firebase-tools` (if not already installed).
2. In a new folder: `firebase login`, then `firebase init functions` (pick your project, JavaScript). This needs Firebase's **Blaze** (pay-as-you-go) plan — typical usage here costs a few cents a month.
3. In `functions/`: `npm install stripe`.
4. Replace `functions/index.js` with:

   ```js
   const functions = require("firebase-functions");
   const admin = require("firebase-admin");
   const stripe = require("stripe")(functions.config().stripe.secret);
   admin.initializeApp();

   exports.stripeWebhook = functions.https.onRequest(async (req, res) => {
     let event;
     try {
       event = stripe.webhooks.constructEvent(
         req.rawBody,
         req.headers["stripe-signature"],
         functions.config().stripe.webhook_secret
       );
     } catch (err) {
       return res.status(400).send(`Webhook Error: ${err.message}`);
     }

     const db = admin.firestore();

     if (event.type === "checkout.session.completed") {
       const session = event.data.object;
       const uid = session.client_reference_id;
       if (uid) {
         await db.collection("users").doc(uid).set(
           { subscribed: true, stripeCustomerId: session.customer },
           { merge: true }
         );
       }
     }

     if (event.type === "customer.subscription.deleted") {
       const sub = event.data.object;
       const snap = await db.collection("users")
         .where("stripeCustomerId", "==", sub.customer)
         .limit(1)
         .get();
       if (!snap.empty) {
         await snap.docs[0].ref.set({ subscribed: false }, { merge: true });
       }
     }

     res.json({ received: true });
   });
   ```

5. `firebase functions:config:set stripe.secret="sk_live_..." stripe.webhook_secret="whsec_..."` (you'll get the webhook secret in the next step, then re-run this and redeploy).
6. `firebase deploy --only functions` — note the URL Firebase gives you.
7. In Stripe: **Developers → Webhooks → Add endpoint** → paste that URL → select `checkout.session.completed` and `customer.subscription.deleted` → save. Copy the signing secret, plug it into the config command above, redeploy.

From then on, Stripe notifies your function the moment someone checks out,
the function flips `subscribed: true` in Firestore, and the app unlocks
automatically the next time they hit "Refresh status" (or reload).

### Letting people manage/cancel
Turn on Stripe's **Customer Portal** (**Settings → Billing → Customer
portal**) and link to it from wherever you like (e.g. next to the Sign out
button) so people can update their card or cancel without emailing you.

---

## What changed in this update

- **Rebrand** — the app, page title, and metadata now read "Everything
  Budgeted" throughout. Nothing about your saved data changed: local
  storage and Firestore both still use the same internal keys as before, so
  existing accounts and browsers keep all their history.
- **Checklist ↔ Analytics link** — checking off a ledger item now logs it
  as a real spend for that day (tagged "ledger" in the Analytics list), so
  your daily checklist and monthly breakdown finally agree with each other.
  Unchecking or deleting that entry keeps both sides in sync; editing the
  price while checked updates the logged amount too.
- **Currency display fix** — the app was correctly switching currencies
  already, but several decorative `$` prompt glyphs (meant to look like a
  terminal `$`) were easy to mistake for the currency symbol. Those are now
  a neutral `›` so the only real currency symbols you'll see are the ones
  tied to your actual amounts. (The Stripe Pro plan is always billed in USD
  regardless of your display currency — that's now labeled explicitly.)
- **Highlight color picker** — Settings → Highlight color lets you pick the
  accent used for progress bars, charts, and highlighted headings.
- **Smaller UX polish** — a boot screen instead of a blank flash on load,
  thousands separators in larger amounts, and a few earlier touches
  (undo-on-delete, quick-start tags, always-visible delete buttons on
  touch) are all still in place.

## Files in this folder

| File | Purpose |
|---|---|
| `index.html` | The entire app — deploy this as your site's root file |
| `manifest.json` | Lets phones "Add to Home Screen" with a proper name/icon |
| `robots.txt` | Standard crawler permissions for search engines |

## Quick reference — what to edit in `index.html`

| What | Where |
|---|---|
| Firebase config | `var firebaseConfig = { ... }` near the top of the `<script>` tag |
| Stripe Payment Link | `var STRIPE_PAYMENT_LINK = "..."` right below it |
| Trial length | `var TRIAL_DAYS = 7;` |
