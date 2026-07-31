# CASOR EXCO Portal — Setup Guide

## What you have now
A `/portal` folder with 3 files, meant to sit alongside your existing public site:
- `login.html` — EXCO sign-in page
- `dashboard.html` — protected page: upload/view/delete files, add/edit/delete records
- `firebase-config.js` — where your Firebase project keys go

Your public site (`index.html`) is untouched. The portal is a separate, gated section.

## 1. Create the Firebase project
1. Go to https://console.firebase.google.com → **Add project** → name it (e.g. `casor-uniport`).
2. In the project, click the **Web (</>)** icon to register a web app → copy the `firebaseConfig` object it gives you.
3. Paste those values into `firebase-config.js`, replacing the placeholders.

## 2. Turn on the services you need
In the Firebase Console sidebar:
- **Authentication** → Sign-in method → enable **Email/Password**.
- **Authentication** → Users → **Add user** for each EXCO (their email + a temporary password). They can reset it via "Forgot password?" on the login page.
- **Firestore Database** → Create database → start in **production mode**.
- **Storage** → Get started (default settings are fine).

## 3. Lock down access (important)
By default, production mode blocks everyone — you need to explicitly allow signed-in EXCOs. In **Firestore → Rules**, replace with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

In **Storage → Rules**, replace with:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

This means: only people who are logged in (i.e. EXCOs you created accounts for) can read or write files/records. The public site itself doesn't touch Firebase, so it's unaffected.

## 4. Deploy
Upload the whole site (public pages + `/portal` folder) to your host as normal. Firebase Hosting also works well and is free for this scale, if you want everything under one roof — ask if you'd like that set up instead of your current host.

## 5. Test it
Visit `yoursite.com/portal/login.html`, sign in with an EXCO account, and try uploading a file and adding a record.

---

## What's next (not built yet)

**Admin site editor** — letting an admin edit the public-facing pages (text, events, etc.) without touching code. This requires rebuilding `index.html` to pull its content from Firestore instead of being static HTML, plus an admin-only editing screen. It's a meaningful rework of the public site — let me know when you're ready and I'll scope it properly.

**Automated reminders / daily messages** — sending scheduled messages to fellowship members needs:
- A place to store member contact info (with their consent)
- A scheduled job (Firebase Cloud Functions + Cloud Scheduler) that fires at set times
- A delivery channel — pick one:
  - **Email** (simplest, cheap/free at this scale — e.g. via SendGrid or EmailJS)
  - **WhatsApp/SMS** (most students see it fastest, but costs per message via a provider like Twilio)
  - **Push notifications** (free, but only reaches people who've installed the site as an app)

Tell me which channel fits your members best and I'll build that piece next.
