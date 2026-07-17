# 🌙 DayFlow – Sleep, Gym & Class Planner

A **Progressive Web App (PWA)** for iPhone that builds your bedtime and wake-up time around your class timetable, gym routine and daily tasks — so you actually hit a minimum amount of sleep instead of just guessing.

**Live app:** https://aastha381.github.io/DayFlow-Sleep-Planner/

## Why this exists

Normal alarm apps only let you set a time. DayFlow instead looks at:
- Your weekly class timetable
- Whether you go to the gym today (and how long/intense the session is)
- Any other one-off tasks on your plate tonight (study group, errands, family time…)
- Heavy meals, which need a longer wind-down gap before bed

…and calculates the **earliest safe bedtime** that still guarantees your minimum sleep goal, so a lighter day naturally gives you *more* sleep instead of always exactly 8 hours.

## Features

- **Dynamic sleep plan** — bedtime/wake time recalculated from tonight's actual schedule, always respecting a minimum (default 8h) sleep goal, never capping the maximum.
- **Weekly class timetable** — fully editable, day-by-day.
- **Optional gym tracking** — log arrival time + intensity (light/normal/heavy), or turn it off entirely in Settings if gym isn't part of your routine.
- **Custom day-to-day tasks** — add as many one-off tasks as you want (with start/end times); they show up in the timeline and factor into the sleep calculation.
- **Manual override** — don't like the suggestion? Tap **Edit** on the sleep plan card and type your own bedtime/wake time directly.
- **Smart alarms** — an alarm can auto-adjust its ring time based on today's class schedule (e.g. ring 30 min before an early class, or fall back to a set breakfast-reminder time on light days) instead of a fixed clock time.
- **Photo-to-dismiss alarms** — when an alarm rings, it can require you to photograph a random, easy-to-find household item (cup, charger, book…) before it stops, verified on-device with TensorFlow.js/MobileNet.
- **Multi-user accounts** — sign in with your mobile number + password; each account's data lives in its own private Firestore document, fully isolated from other users. Works fully offline/single-device too if you skip sign-in.
- **Installable PWA** — add to your iPhone Home Screen via Safari for a full-screen, app-like experience with offline support.

## Known iOS limitations (by design of the platform, not this app)

- Safari fully stops running JavaScript when the app/tab is closed, so alarms only ring reliably while the app is open (or the phone is on and unlocked) — true background alarms need a native app with real local notifications.
- Alarm sound is silenced by the iPhone's physical **Ring/Silent switch** — this is an OS-level restriction no website can override.

## Tech stack

Plain HTML/CSS/JavaScript — no build step, no framework, no bundler.

- **Firebase Authentication** (email/password, using a phone-number-as-username trick to avoid SMS costs)
- **Cloud Firestore** for per-user data storage, with Firestore security rules enforcing that each user can only read/write their own document
- **Service worker** for offline caching (network-first, so updates always reach your device)
- **TensorFlow.js + MobileNet** (loaded from CDN) for the on-device photo-verification alarm challenge

## Project structure

```
├── index.html      # The entire app - markup, styles and logic in one file
├── manifest.json   # PWA manifest (name, icons, theme colour, display mode)
├── sw.js           # Service worker for offline caching
└── icons/
    └── icon.svg     # App icon
```

## Run locally

No build step needed.

```bash
git clone https://github.com/AASTHA381/DayFlow-Sleep-Planner.git
cd DayFlow-Sleep-Planner
python3 -m http.server 4173
# Open http://localhost:4173
```

## Install on iPhone

1. Open the [live app](https://aastha381.github.io/DayFlow-Sleep-Planner/) in **Safari**.
2. Tap the **Share** button (square with an arrow).
3. Tap **"Add to Home Screen"**.
4. Launch it from your Home Screen — it opens full-screen like a native app.

## Setting up your own backend (optional)

The app works fully in **single-device, no-account mode** out of the box (data stored only in `localStorage`). To enable multi-user accounts:

1. Create a free project at [console.firebase.google.com](https://console.firebase.google.com) (Spark plan — no billing required).
2. **Authentication** → Sign-in method → enable **Email/Password**.
3. **Firestore Database** → Create database.
4. In Firestore → **Rules**, publish:
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
5. Project settings → General → "Your apps" → add a Web app → copy the `firebaseConfig` object.
6. Paste those values into the `firebaseConfig` object near the bottom of `index.html`.

Until configured, the app automatically falls back to single-device mode with no errors.

## Data & privacy

- **No account:** everything stays in your browser's `localStorage`. Nothing is ever sent anywhere.
- **With an account:** your schedule, gym log, tasks and alarms are stored in your own private Firestore document, isolated from every other user by Firestore security rules. Only synced data is your DayFlow app state — no analytics, tracking, or third-party data sharing.
