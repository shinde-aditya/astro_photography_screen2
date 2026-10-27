# Sky Wall — Setup Guide

Two files, both free forever on the plans used here:

- **upload.html** — the page visitors reach by scanning the QR code
- **display.html** — the page you open once on the TV

Total cost: ₹0. No staff needed once it's running. Takes about 15–20 minutes to set up.

---

## Step 1 — Create a free Firebase project (the database)

1. Go to **console.firebase.google.com** and sign in with any Google account.
2. Click **Add project** → name it something like `pcsp-sky-wall` → you can skip Google Analytics → **Create project**.
3. In the left sidebar, click **Build → Firestore Database** → **Create database**.
4. Choose a location close to India (e.g. `asia-south1`) → start in **Production mode** → Create.
5. Click the **Rules** tab and replace the contents with this, then **Publish**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /skyPhotos/{photo} {
         allow read: if true;
         allow create: if request.resource.data.image is string
                       && request.resource.data.image.size() < 1000000
                       && request.resource.data.name is string
                       && request.resource.data.name.size() < 50
                       && request.resource.data.location is string
                       && request.resource.data.location.size() < 70;
         allow update, delete: if false;
       }
     }
   }
   ```

   This lets any visitor **add** a photo and lets your TV **read** them, but no one can edit or delete existing entries from the app itself (you can still delete inappropriate ones yourself from the console — see Step 5).

6. Back in the project, click the **gear icon → Project settings**, scroll to "Your apps", click the **`</>`** (web) icon, give it any nickname, **skip** Firebase Hosting here, click **Register app**. You'll see a code block with `apiKey`, `authDomain`, `projectId`, and `appId` — keep this tab open, you need these four values next.

## Step 2 — Paste your config into both files

Open **upload.html** and **display.html** in any text editor (Notepad, VS Code, etc.). Near the top of each, find:

```js
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyC2U5AW64qzJDYliACxlwkHO3t_zSFwQ9M",
  authDomain: "astro-fest-photography.firebaseapp.com",
  projectId: "astro-fest-photography",
  appId: "1:956957907521:web:3eb9787d7f5a99580cc4c0",
};
```

Replace the four values with the ones from Step 1.6. Use the exact same values in both files.

## Step 3 — Put the files online for free

Easiest option — **Netlify Drop**, no account, no coding:

1. Go to **app.netlify.com/drop**.
2. Drag your whole folder (containing `upload.html` and `display.html`) onto the page.
3. It gives you a live URL in seconds, like `https://random-name-123.netlify.app`.
4. Your two pages are now at:
   - `https://random-name-123.netlify.app/upload.html`
   - `https://random-name-123.netlify.app/display.html`

(A free Netlify account lets you rename the site and keep it stable long-term — sign up if you want the link to never change. GitHub Pages is another equally free alternative if you prefer.)

## Step 4 — Connect the QR code to the display

Open **display.html** again and find:

```js
const UPLOAD_URL = "https://PASTE-YOUR-HOSTED-UPLOAD-PAGE-URL-HERE";
```

Paste in your real `upload.html` URL from Step 3, save, and re-drag the folder onto Netlify Drop to update the live site. The TV screen will now show a small QR code in the corner too, in case anyone wants to scan it right off the TV.

## Step 5 — Make the QR code for signage/posters

Use any free QR generator (e.g. **qr-code-generator.com** or **qrcode-monkey.com**) with your `upload.html` link, download the image, and print it on a stand or poster near the sky-viewing spot.

## Step 6 — Run the TV display

On the TV's browser (smart TV browser, or a laptop/mini-PC/Fire Stick browser connected to it):

1. Open your `display.html` URL.
2. Press **F11** (or the browser's fullscreen option) to hide the browser bar.
3. Leave it open. It updates itself automatically the moment someone uploads a photo — no refreshing, no staff needed.

## Moderation (optional but recommended)

Since anyone can upload, it's worth occasionally checking content. In the Firebase Console → **Firestore Database → Data → skyPhotos**, you can view and delete any entry in a few clicks — no coding needed. You don't need to staff this continuously; a daily glance is enough for a park exhibit.

## Free tier limits (plenty for this use case)

Firebase's free "Spark" plan (no credit card required) includes 1 GiB storage and 50,000 reads / 20,000 writes per day — enough for many thousands of visitor photos.

## Troubleshooting

- **Photo won't send:** double-check the four config values are pasted correctly and identical in both files, and that Firestore rules were published (Step 1.5).
- **TV shows "Waiting for photos" forever:** confirm `display.html` has the same `FIREBASE_CONFIG` as `upload.html`.
- **QR corner doesn't show on TV:** you haven't pasted a real `UPLOAD_URL` yet (Step 4) — this is cosmetic only, uploads still work fine.
