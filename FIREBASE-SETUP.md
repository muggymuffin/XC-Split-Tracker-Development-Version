# Turning on live sync (Firebase, free plan)

The app works without any of this (QR codes only). Do these steps once to let helpers' splits reach the organizer automatically.

## 1. Create the project
1. Go to https://console.firebase.google.com and sign in with a Google account.
2. **Create a project** (any name, e.g. `xc-splits`). You can turn Google Analytics off. Stay on the free **Spark** plan; no payment method is needed.

## 2. Create the database
1. In the left menu: **Build → Realtime Database → Create Database**.
2. Pick the location closest to you and choose **Start in locked mode**.
3. Open the **Rules** tab, replace everything with the rules below, and **Publish**.

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "races": {
      "$rid": {
        ".read": "$rid.length <= 12",
        "h": {
          "$hid": {
            ".write": "$rid.length <= 12 && $hid.length <= 16",
            ".validate": "newData.hasChildren()"
          }
        }
      }
    }
  }
}
```

What these rules do: nobody can list or browse the database; you can only read or write a race if you know its random 6-character race code (which only comes from the organizer's invite QR). Helpers can only write inside their own helper slot. There is no sign-in, so anyone who has a race code can see that race's splits, so don't put anything private in runner names.

## 3. Get the config
1. **Project settings (gear icon) → General → Your apps → Web (`</>`)**, register an app (no hosting needed).
2. Copy the `firebaseConfig` values.
3. Open `firebase-config.js` in this folder and replace each `PASTE_...` value. `databaseURL` looks like `https://your-project-default-rtdb.firebaseio.com` (or `...firebasedatabase.app` for non-US regions). If it's missing from the config, copy it from the top of the Realtime Database **Data** tab.

## 4. Publish
Commit and push all files to your GitHub Pages repo as usual (including `firebase-config.js` and the updated `sw.js`). Each phone picks up the update the next time it opens the app online. Open the app once with signal so the service worker saves everything for offline use.

## Using it
- **Organizer**: nothing new to do. Set up, tap **Invite helpers (QR)**, start the race. The line under the title shows `Live — N helpers sending`.
- **Helpers**: scan the invite as before. Taps upload automatically. If signal drops, taps stay on the phone and upload by themselves when a connection returns, even if the app was closed and reopened.
- **Offline mode** (checkbox on the first screen, and on the helper screens): turns all internet use off. Splits are still saved on the phone, and **Send to organizer** / **Import splits** (QR) work exactly as before. Turning offline mode off uploads anything waiting.
- Late helpers can upload after the race is finished; the organizer's phone still receives them while that race is open.

## Housekeeping
- Race data is tiny. To clear old races, delete the `races` node in the console's **Data** tab.
- Free-plan Realtime Database limits (check the Firebase pricing page for current numbers) are far above what a team's meet needs.
