# Sharing this calendar with your team via Firebase

This app now stores its data in a shared Firebase (Firestore) database
instead of the browser's local storage, so everyone who signs in sees the
same boards, events, attendance, and budget data. Access is restricted to
a fixed list of Google accounts.

You need to do the one-time setup below in the Firebase console, then
paste a few values into `index.html`.

## 1. Create a Firebase project

1. Go to https://console.firebase.google.com and click **Add project**.
2. Name it something like `dpi-work-platform`. Google Analytics is not
   needed — you can decline it.
3. Once created, click the **Web** icon (`</>`) to register a web app.
   Give it any nickname. You do **not** need Firebase Hosting for this step.
4. Firebase will show a `firebaseConfig` object with six values
   (`apiKey`, `authDomain`, `projectId`, `storageBucket`,
   `messagingSenderId`, `appId`). Copy these.

## 2. Enable Google sign-in

1. In the console, go to **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Google**.
3. Under **Authentication → Settings → Authorized domains**, add the
   domain your site is hosted on (e.g. `yourname.github.io`). Without
   this, sign-in popups will fail on your live site.

## 3. Create the Firestore database

1. Go to **Build → Firestore Database → Create database**.
2. Choose a region close to your team and start in **production mode**
   (we'll set real rules in step 5, so it's safe to skip test mode).

## 4. Update `index.html`

Near the top of the `<script>` block, find `FIREBASE_CONFIG` and
`ALLOWED_EMAILS` and fill in your real values:

```js
var FIREBASE_CONFIG = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};

var ALLOWED_EMAILS = [
  "person1@example.com",
  "person2@example.com",
  "person3@example.com",
  "person4@example.com"
];
```

`ALLOWED_EMAILS` only controls what the app's UI shows (a friendlier
"you're not authorized" message). The actual access control is enforced
server-side by the Firestore rules below — **use the same four addresses
in both places.**

## 5. Set Firestore security rules

In **Firestore Database → Rules**, replace the contents with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /dpiCalendar/sharedState {
      allow read, write: if request.auth != null &&
        request.auth.token.email in [
          "person1@example.com",
          "person2@example.com",
          "person3@example.com",
          "person4@example.com"
        ];
    }
  }
}
```

Replace the four emails with your real team list (must match
`ALLOWED_EMAILS` in `index.html`), then click **Publish**.

## 6. Commit and push

Fill in the config, commit, and push to your GitHub Pages branch as
usual. Visit the live site — you should see a "Sign in with Google"
screen before the calendar loads.

## Notes

- Adding or removing someone later just means editing the email list in
  both `index.html` and the Firestore rules, then republishing the rules.
- The free ("Spark") Firebase plan comfortably covers a small team's
  usage — no billing setup required.
- If you ever want to inspect or manually fix the raw data, it lives in
  Firestore under `dpiCalendar/sharedState` — viewable in the Firebase
  console under **Firestore Database → Data**.
