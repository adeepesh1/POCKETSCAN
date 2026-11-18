# PocketScan: a document scanner PWA for iPhone

## What's in v1
- **Scan or import:** opens the iPhone camera, or picks several photos from the library at once
- **Auto edge detection:** OpenCV.js finds the page. You can drag the 4 corners, and a magnifier loupe helps you place them precisely
- **Perspective correction:** straightens the page (written in plain JS, no library needed)
- **Filters:** Magic (auto-enhance), Original, Grayscale, B&W (adaptive threshold, so shadows don't ruin it)
- **Multi-page documents:** add pages, reorder, delete, rename
- **PDF export:** A4 PDF through the iOS share sheet (WhatsApp, Mail, Files, Drive…)
- **OCR:** English, Hindi, Gujarati, Marathi, Tamil, Bengali. Runs on the phone, so nothing is uploaded
- **Works offline** once each feature has been used one time (libraries get cached)
- **Freemium hooks:** 3 free OCR runs a day, and a small "Scanned with PocketScan" footer on free PDFs

## Put it online (GitHub Pages, free)
1. Create a new GitHub repo, e.g. `pocketscan`, and upload every file in this folder (keep the `icons/` folder).
2. Repo → Settings → Pages → Source: `main` branch, `/root` → Save.
3. Open `https://<your-username>.github.io/pocketscan/` on your iPhone in **Safari**.
4. Tap Share → **Add to Home Screen**. It now opens full-screen like a normal app.

HTTPS is required for the camera and offline mode, and GitHub Pages provides it automatically.

## Where the data is stored
**Now: on the user's phone (IndexedDB).** Scans never leave the device. This is free for you, private for users, and works offline.
Caveat: if a user deletes the app from the home screen, or Safari clears its data, the scans are lost. The app asks iOS for "persistent storage" to reduce this risk.

**Later, as a Pro feature: cloud backup with Firebase.**
- Firebase Auth: sign in with Google, Apple or phone OTP
- Cloud Storage: page JPEGs at `users/{uid}/docs/{docId}/{pageId}.jpg`
- Firestore: document records (name, page order, OCR text) at `users/{uid}/docs/{docId}`
- Security rule: `allow read, write: if request.auth.uid == uid;`
- Cost: the free tier covers roughly the first few hundred users. After that you pay about $0.026 per GB stored per month, which is cheap, and Pro revenue covers it.

## How to charge users
Apple's In-App Purchase is **not available** to PWAs. That works in your favour, because you skip Apple's 15–30% cut. Use a normal payment gateway instead:

| | Razorpay (best for India) | Stripe (international) |
|---|---|---|
| UPI / cards / netbanking | ✅ | Cards mainly |
| Subscriptions | ✅ Razorpay Subscriptions | ✅ Billing |
| Fees | ~2% | ~3–4% intl |

**Suggested model:** a free tier with limits, then Pro at ₹99/month or ₹699/year (unlimited OCR, no PDF footer, cloud backup).

### Step 1: quick start (this week)
Create a Razorpay **Payment Page** or **Subscription link**. Set its success redirect to
`https://<you>.github.io/pocketscan/index.html?pro=1`, and paste the link into `CONFIG.paymentLink` in `index.html`.
⚠️ This is easy to bypass, because anyone who learns the URL gets Pro, and Pro only lasts on that one phone. Use it only to test whether people will pay.

### Step 2: proper version (before you scale)
1. The user signs in with Firebase Auth.
2. A Firebase Cloud Function creates a Razorpay subscription for that `uid`.
3. A Razorpay **webhook** (`subscription.charged`, `subscription.cancelled`) calls another Cloud Function, which verifies the signature and sets `users/{uid}.pro = true/false` in Firestore.
4. The app reads `pro` from Firestore instead of localStorage. Pro then works on all of the user's devices and can't be faked.

Cloud Functions need Firebase's Blaze (pay-as-you-go) plan. At small scale it effectively costs ₹0.

## Customising
At the top of the `<script>` in `index.html`:
```js
const CONFIG = { appName, freeOcrPerDay, freeWatermark, proPriceText, paymentLink };
```
After you change files, bump `VERSION` in `sw.js` so installed copies pick up the update.

## Limits of a PWA on iOS (vs. a native app)
- No live edge overlay while the camera viewfinder is open. Users take the photo first, then the edges are detected.
- Not listed in the App Store, so you share the link yourself.
- If either of these becomes a problem, the same app can later be wrapped as a native app with Capacitor.
