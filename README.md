# FleetView — Shared Fleet ERP

Single-file app (`index.html`) with a free real-time backend (Firebase) so 2–3 people
can view and edit the same data at once, hosted free on GitHub Pages.

## 1. Create the free Firebase backend (~10 min)

1. Go to https://console.firebase.google.com → **Add project** → name it (e.g. `fleetview-mycompany`) → keep default settings → Create.
2. In the left menu: **Build → Firestore Database → Create database**.
   - Choose a location close to you (e.g. `asia-south1` for India), start in **production mode**.
3. Go to **Firestore Database → Rules** tab, replace the contents with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /fleetview/{doc} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
   Click **Publish**.
4. Go to **Build → Authentication → Get started → Sign-in method** → enable **Email/Password**.
5. Go to **Authentication → Users → Add user** — create one login per teammate
   (e.g. `you@company.com` + a password you choose, repeat for each of the 2–3 people).
6. Go to **Project settings** (gear icon, top left) → scroll to **Your apps** → click the **</>** (Web) icon →
   register an app (any nickname) → you'll get a config object like:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "fleetview-mycompany.firebaseapp.com",
     projectId: "fleetview-mycompany",
     storageBucket: "fleetview-mycompany.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
7. Open `index.html` in this repo, find `const FIREBASE_CONFIG = {...}` near the top of the
   `<script>` block, and paste your real values in place of the `PASTE_YOUR_...` placeholders.

These values are meant to be public in client-side code — they're not secret. Your data is
protected by the Firestore **rules** (step 3) plus the login requirement, not by hiding this config.

## 2. Host it free on GitHub Pages (~5 min)

1. Create a new GitHub repo (public or private both work with GitHub Pages on a paid plan;
   public repos get Pages free — private repos need GitHub Pro/Team/Enterprise for Pages).
2. Upload `index.html` (with your Firebase config filled in) to the repo root.
3. Repo → **Settings → Pages** → under "Build and deployment", set **Source: Deploy from a branch**,
   branch **main**, folder **/ (root)** → Save.
4. Wait ~1 minute, then your app is live at `https://<your-username>.github.io/<repo-name>/`.
5. Share that URL with your 2–3 teammates, plus the email/password you created for each of them in step 1.5.

## 3. How the sync works

- All the app's data is stored as one document in Firestore (`fleetview/shared-data`).
- Every edit saves to `localStorage` instantly (so the UI never feels slow) and pushes to
  Firestore ~0.6s later (batches rapid edits into one write).
- A real-time listener (`onSnapshot`) means when a teammate saves something, everyone else's
  screen updates automatically within a second or two — no refresh needed.
- The small dot + label next to the theme toggle in the top bar shows sync status
  (Connecting / Synced live / Saving / Offline).
- If the internet drops, the app keeps working off `localStorage` and re-syncs once back online.

## 4. Free tier limits (Firebase Spark plan)

For 2-3 people using this day-to-day, you'll be nowhere near these:
- Firestore: 50,000 reads/day, 20,000 writes/day, 1 GiB storage — all free.
- Authentication: unlimited email/password users, free.
- One caveat: this app stores the *entire* dataset as a single Firestore document, which has
  a 1 MiB size limit. That's roughly tens of thousands of trip records — comfortable for years
  of use for a small fleet, but if you ever hit that ceiling, the data model would need to
  split into per-collection documents (a bigger rework — ping me if you get there).

## 5. Local development / testing

You can just open `index.html` directly in a browser to test — no build step, no npm install,
it's plain HTML/CSS/JS. Just make sure your Firebase config is filled in first.
