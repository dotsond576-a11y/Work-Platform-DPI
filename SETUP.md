# Sharing this calendar with your team via Firebase

This app stores its data in a shared Firebase (Firestore) database instead
of the browser's local storage, so everyone who signs in sees the same
boards, events, attendance, and budget data. Access is restricted to
accounts you create yourself (email + password — no Google account
required).

The Firebase project (`dpi-ttprojects`) is already set up and its config
is wired into `index.html`. What's left is creating the sign-in accounts
and publishing the Firestore rules below.

## Team access list

```
jriel2@uic.edu
markh3@illinois.edu
tmcfar1@uillinois.edu
jdanish@illinois.edu
ddotson2@uillinois.edu
```

This list lives in two places and **must match exactly**:
- `ALLOWED_EMAILS` in `index.html` (controls what the app's UI shows)
- The Firestore security rules below (the actual server-side enforcement)

## 1. Create the sign-in accounts

1. Go to https://console.firebase.google.com, open the `dpi-ttprojects`
   project.
2. Go to **Build → Authentication → Sign-in method**, confirm
   **Email/Password** is enabled (toggle it on if not, then Save).
3. Go to the **Users** tab → **Add user**.
4. Enter one person's email (from the list above) and a password you
   choose, then **Add user**. Repeat for everyone on the list.
5. Share each person's password with them directly (not over a public
   channel). They can't self-reset yet — see "Password resets" below.

## 2. Create the Firestore database (if not already done)

1. **Build → Firestore Database → Create database**.
2. Pick a region, choose **Start in production mode**, then **Create**.

## 3. Set the Firestore security rules

In **Firestore Database → Rules**, paste this and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /dpiCalendar/sharedState {
      allow read, write: if request.auth != null &&
        request.auth.token.email in [
          "jriel2@uic.edu",
          "markh3@illinois.edu",
          "tmcfar1@uillinois.edu",
          "jdanish@illinois.edu",
          "ddotson2@uillinois.edu"
        ];
    }
  }
}
```

## 4. Push and test

`index.html` is already configured — just deploy it (e.g. push to GitHub
Pages). Visit the live site: you should see an email/password sign-in
screen. Sign in with one of the allowlisted accounts to load (or
bootstrap) the shared calendar.

## Adding or removing someone later

1. Firebase console → Authentication → Users: add/remove their account.
2. Update `ALLOWED_EMAILS` in `index.html` and redeploy.
3. Update the email list in the Firestore rules and republish.

All three steps are needed — missing one either leaves a removed person
with access (rules) or blocks a new person from seeing the app's UI
correctly (client list), so keep them in sync.

## Password resets

There's no self-service "forgot password" flow wired up in the app. If
someone forgets their password, go to Authentication → Users, find their
account, and use the console's reset options (or delete and recreate the
account with a new password).

## Notes

- The free ("Spark") Firebase plan comfortably covers a small team's
  usage — no billing setup required.
- The `apiKey` in `index.html` is meant to be public for Firebase web
  apps — it does not grant access by itself. Actual access control is
  the Firestore rules above.
- To inspect or manually fix the raw data, it lives in Firestore under
  `dpiCalendar/sharedState` — viewable in the Firebase console under
  **Firestore Database → Data**.
