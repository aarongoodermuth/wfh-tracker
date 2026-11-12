# WFH Tracker

Rolling Top 8 of 12 — past 12 completed weeks, top 8 count toward
the total. Next 52 weeks show the rolling total and whether each
week is counted / dropped in the windows it falls in.

## Hosting — GitHub Pages

GitHub Pages serves this static site. It cannot store data, so the
data lives in Firebase Firestore (free) and syncs across devices.

1. Create a **public** repo, e.g. `wfh-tracker`, and put
   `index.html` at the repo root.
2. Repo Settings → Pages → Source: **Deploy from a branch**,
   Branch: `main`, Folder: `/ (root)`.
3. Site will be at `https://aarongoodermuth.github.io/wfh-tracker/`

## Data — Firebase Firestore (free Spark plan)

One-time setup, ~10 minutes:

1. https://console.firebase.google.com → Add project
   (name it e.g. `wfh-tracker`, turn Google Analytics off).
2. Build → Firestore Database → Create database →
   Start in production mode → location `us-central` (or nearest).
3. Build → Authentication → Get started → Sign-in method →
   enable **Google**.
4. Project settings (gear icon) → Your apps → Web app `</>` →
   register the app, copy the `firebaseConfig` object.
5. In `index.html`, paste that object into `FIREBASE_CONFIG`,
   replacing the `PASTE_YOUR_*` placeholders.
6. Firestore → Rules, replace the rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /appData/{doc} {
      allow read, write: if request.auth != null
        && request.auth.token.email == 'aaron.goodermuth@gmail.com';
    }
  }
}
```

   Only that Google account can read or write the data, even
   though the Pages site itself is public.

7. Push, open the site, click **Sign in with Google** once per
   device. Edits sync in about a second.

## How sync works

- One Firestore document: `appData / weekValues`, field `values`
  is a map of `MM/DD/YYYY → number`.
- Typing saves to localStorage immediately (offline works) and
  debounces a cloud write by 600 ms.
- Other devices get updates live via `onSnapshot`.
- First sign-in: if the cloud is empty and the device has local
  values, those are uploaded once — so your existing local data
  is not lost.
- Export / Import / Clear All still work, and also update the
  cloud when signed in.

## Without Firebase

If `FIREBASE_CONFIG` is left as placeholders, the app runs
exactly like the original: localStorage only, per-device,
with a "Local only" badge.
