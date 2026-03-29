# 🔥 Firebase + GitHub + Netlify Setup Guide
## Jaclyn Donnellan Portfolio

---

## What Was Fixed in This File

### Bug 1 — Text Not Editable in Admin Mode
**Problem:** CSS `pointer-events: none` was overriding the admin edit mode.  
**Fix:** Added `pointer-events: auto !important` to the admin-mode CSS rule so all `contenteditable` elements respond to clicks when logged in as Admin.

### Bug 2 — PDFs Disappear After Save / Reload
**Problem:** PDFs were stored as `blob://` URLs — temporary memory pointers that die when the page reloads.  
**Fix:** All PDF uploads now use `FileReader.readAsDataURL()` to convert PDFs to base64 strings. These persist in localStorage AND Firebase Storage. Affected areas: Event card covers, Dashboard documents, Resume PDF.

---

## Step 1 — Create a Firebase Project

1. Go to **https://firebase.google.com** and click **Get started**
2. Sign in with your Google account
3. Click **+ Add project**
4. Name it: `donnellan-portfolio` → click **Continue**
5. Disable Google Analytics (optional) → click **Create project**
6. Wait for it to finish → click **Continue**

---

## Step 2 — Enable Firestore Database

1. In the Firebase Console left sidebar, click **Build → Firestore Database**
2. Click **Create database**
3. Select **Start in test mode** → click **Next**
4. Choose region: **us-east1** → click **Enable**
5. Wait for Firestore to finish provisioning

---

## Step 3 — Enable Firebase Storage

1. In the sidebar, click **Build → Storage**
2. Click **Get started**
3. Select **Start in test mode** → click **Next**
4. Keep the default storage bucket → click **Done**

---

## Step 4 — Get Your Firebase Config

1. Click the **gear icon (⚙)** at the top of the sidebar → **Project settings**
2. Scroll down to **Your apps** section
3. Click the **`</>`** (web) icon to register a web app
4. Enter nickname: `portfolio-web` → click **Register app**
5. You'll see a config block like this:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyAbc123...",
  authDomain: "donnellan-portfolio.firebaseapp.com",
  projectId: "donnellan-portfolio",
  storageBucket: "donnellan-portfolio.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

6. **Copy this entire block** (just the object values)

---

## Step 5 — Add Your Config to the HTML File

1. Open `Donnellan_Portfolio-FIXED.html` in a text editor  
   *(Recommended: VS Code — free at https://code.visualstudio.com)*
2. Press **Ctrl+F** (or Cmd+F on Mac) and search for:
   ```
   YOUR_API_KEY
   ```
3. You'll find the `FIREBASE_CONFIG` block near the bottom of the file:
   ```javascript
   const FIREBASE_CONFIG = {
     apiKey:            "YOUR_API_KEY",
     authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
     ...
   };
   ```
4. Replace each `YOUR_*` value with the real values from Step 4
5. **Save the file**

---

## Step 6 — Set Up GitHub

1. Go to **https://github.com** and create a free account (if you don't have one)
2. Click the **+** button (top right) → **New repository**
3. Repository name: `donnellan-portfolio`
4. Set to **Public**
5. Click **Create repository**
6. On the next page, click **uploading an existing file**
7. Drag and drop your `Donnellan_Portfolio-FIXED.html` into the upload box
8. Scroll down and click **Commit changes**

> **Tip:** Every time you update the HTML file, come back here and upload the new version. GitHub keeps the history automatically.

---

## Step 7 — Deploy to Netlify

1. Go to **https://netlify.com** and sign up (use your GitHub account — easiest)
2. Click **Add new site** → **Import an existing project**
3. Click **GitHub** → Authorize Netlify if prompted
4. Select your `donnellan-portfolio` repository
5. Leave all build settings blank (no build command needed — it's just HTML)
6. Click **Deploy site**
7. Netlify will give you a URL like: `https://random-words-123.netlify.app`

### Set a Custom Subdomain (Free)
1. In your Netlify site dashboard, go to **Site configuration → Domain management**
2. Click **Options → Edit site name**
3. Change it to something like: `donnellan-portfolio` → Save
4. Your site will now be: `https://donnellan-portfolio.netlify.app`

### Connect a Custom Domain (like donnellan-portfolio.com)
1. Buy the domain at **Namecheap** or **Google Domains**
2. In Netlify → **Domain management → Add custom domain**
3. Follow the instructions to point your domain's DNS to Netlify

---

## Step 8 — Lock Down Firebase Security Rules

Right now Firestore is in "test mode" — **anyone** could write to it.  
Once your site is live, update the rules:

1. In Firebase Console → **Firestore Database → Rules tab**
2. Replace the rules with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /portfolio/{docId} {
      allow read: if true;    // ✓ Anyone can VIEW the portfolio
      allow write: if false;  // ✗ Writes only allowed via the Admin panel (client-side password gate)
    }
  }
}
```

3. For Storage, go to **Storage → Rules** and set:
```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /portfolio/{allPaths=**} {
      allow read: if true;
      allow write: if false; // Only the Admin panel uploads
    }
  }
}
```

4. Click **Publish** on both

> **Note:** The admin password is enforced in the browser (JavaScript). For a fully secured system you'd add Firebase Authentication, but the current password gate is appropriate for a portfolio site.

---

## Updating the Site

Whenever you make edits through the Admin panel:

1. Log in as Admin
2. Make your changes (text, photos, PDFs)
3. Click **☁ Save to Cloud** — changes save to Firebase and appear on ANY device/browser immediately

To change the HTML structure itself (rare):
1. Edit the HTML file locally
2. Re-upload to GitHub
3. Netlify auto-deploys within ~30 seconds

---

## Credentials

| Role | Username | Password |
|------|----------|----------|
| Admin | `admin` | `Leadership2026` |
| Guest | `guest` | `Guest` |

> To change the admin password: open the HTML file, search for `Leadership2026`, and replace it.

---

## Troubleshooting

**"Firebase save failed"** — Check that your `FIREBASE_CONFIG` values are correct and that Firestore/Storage are enabled in the Firebase Console.

**PDFs still not showing** — Make sure you clicked "Save to Cloud" AFTER uploading the PDF. The PDF is stored in Firebase Storage and the URL is saved to Firestore.

**Edits not visible to visitors** — You must click **☁ Save to Cloud** to publish changes. Visitors always load the last cloud-saved version.

**Site not updating on Netlify** — Check the Netlify dashboard → Deploys tab. If a deploy failed, there will be an error message.
