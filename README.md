# Genius Sell Bios

Tutor photo + sell bio, ready to send on WhatsApp. Two tabs — **Send Sell Bio**
(search, send one or several) and **View All Sell Bios** (A–Z) — plus an
**Admin** tab that only appears for people allowed to upload.

Built on the same pattern as the Genius Schedule Maker: one static HTML file,
Firebase for auth and data, no build step and no server.

---

## Before you start

You need a Google account for Firebase (use `zak.k@geniuspremium.com`), a GitHub
account, and Git installed. Nothing else — no Node, no npm, no build tools.

**Work in this order.** Firebase first, then GitHub. Doing GitHub first means
loading a site that cannot connect to anything.

This app cannot be meaningfully tested on your own machine: sharing to WhatsApp
and copying the caption both need HTTPS, which `localhost` fakes but a real
phone does not. The first genuine test is on the live GitHub Pages URL, on a
phone. Everything up to that point is setup.

---

## Part A — Create the Firebase project

**A1.** Go to <https://console.firebase.google.com> and click **Create a
project**.

**A2.** Name it `genius-sell-bios`. Google will append a few random characters
to make the project ID unique — that is normal and fine.

**A3.** Google Analytics: **turn it off**. Nothing here uses it.

**A4. Enable Google sign-in.** In the left sidebar: **Build → Authentication →
Get started → Google → Enable**. Set the support email to your own address, then
**Save**.

> Skipping this is the single most common way to end up with a site where the
> login button does nothing.

**A5. Create the database.** **Build → Firestore Database → Create database**.

- Choose **Production mode**. (Test mode leaves your data open to the world for
  30 days, then silently locks everything out. You are pasting real rules in
  step A7 anyway.)
- **Location: pick carefully — it is permanent.** `africa-south1`
  (Johannesburg) if offered, otherwise `europe-west1`. You cannot change this
  later without deleting the project.

**A6. Copy your web config.** Click the **gear icon → Project settings**. Under
**Your apps**, click the web icon **`</>`**. Give it the nickname `sell-bios`.
Do **not** tick "Also set up Firebase Hosting" — GitHub Pages is doing that job.

Register the app, and you will be shown a `firebaseConfig` object like:

```js
const firebaseConfig = {
  apiKey: "AIzaSy…",
  authDomain: "genius-sell-bios-a1b2c.firebaseapp.com",
  projectId: "genius-sell-bios-a1b2c",
  storageBucket: "genius-sell-bios-a1b2c.firebasestorage.app",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abc123def456"
};
```

**Keep this tab open** — you need these six values in Part B.

> These values are not secrets. A Firebase web config is a public identifier by
> design; your data is protected by the rules in step A7, not by hiding this.
> That is why it is safe to commit to a public GitHub repo.

**A7. Publish the security rules.** **Build → Firestore Database → Rules** tab.
Select everything in the editor, delete it, then paste the entire contents of
[`firestore.rules`](firestore.rules) from this folder. Click **Publish**.

These rules mean: only `@geniuspremium.com` accounts can read anything, only
people on the admin list can upload, and only you can change that list.

> If a tutor ever needs access with a personal Gmail, that is the
> `isStaff()` function — there is a comment above it explaining the change.

---

## Part B — Point the app at your project

**B1.** Open `index.html` in a text editor. At the very top is a block marked
`SETTINGS — this is the ONLY block you need to edit`. Replace the six
`PASTE_…_HERE` values with the ones from step A6:

```js
window.GENIUS = {
    firebase: {
        apiKey:            "AIzaSy…",
        authDomain:        "genius-sell-bios-a1b2c.firebaseapp.com",
        projectId:         "genius-sell-bios-a1b2c",
        storageBucket:     "genius-sell-bios-a1b2c.firebasestorage.app",
        messagingSenderId: "123456789012",
        appId:             "1:123456789012:web:abc123def456"
    },

    OWNER_EMAIL: 'zak.k@geniuspremium.com',
};
```

Keep the quotes and the commas. If you leave a placeholder in, the page tells
you so instead of failing with a cryptic Firebase error.

**B2.** Leave `OWNER_EMAIL` as is unless you are handing ownership to someone
else. If you do change it, change `isOwner()` in `firestore.rules` to match and
re-publish, or the Admin tab will appear and every write will be refused.

---

## Part C — Put it on GitHub

**C1.** In this folder, run:

```bash
git init && git add -A && git commit -m "Genius Sell Bios"
```

**C2.** Create the repo on GitHub: <https://github.com/new>. Name it
`genius-sell-bios`. **Do not** add a README, .gitignore or licence — this folder
already has what it needs.

Public is fine and is what free GitHub Pages requires. Nothing secret is in
here (see the note in A6). Pages on a private repo needs GitHub Pro.

**C3.** Push, using the two commands GitHub shows you on the new-repo page:

```bash
git remote add origin https://github.com/YOUR-USERNAME/genius-sell-bios.git
git branch -M main && git push -u origin main
```

**C4. Turn on Pages.** In the repo: **Settings → Pages**. Under **Build and
deployment**, set **Source** to `Deploy from a branch`, **Branch** to `main` and
folder to `/ (root)`. **Save**.

Wait a minute or two, then reload the Settings → Pages page. Your URL appears at
the top:

```
https://YOUR-USERNAME.github.io/genius-sell-bios/
```

---

## Part D — Let Firebase trust that URL

**This step is not optional, and it is the one everybody forgets.** Google login
will fail on the live site until you do it, even though everything else looks
fine.

**D1.** Firebase console → **Build → Authentication → Settings** tab →
**Authorised domains** → **Add domain**.

**D2.** Add exactly:

```
YOUR-USERNAME.github.io
```

Just the domain — no `https://`, no `/genius-sell-bios/` path.

---

## Part E — First run

Open your Pages URL and click **Login with Google**. Sign in with
`zak.k@geniuspremium.com`. You should see `Signed in as …` and an **Admin** tab.

### E1. Import the tutor photos

**Admin → 1 · Import tutor photos → Choose photos…**, then select **every file**
in your Staff Photos folder at once (Ctrl+A in the file picker).

From your current folder that is **488 files → 220 tutors → 255 photos**, and it
will take a few minutes. The log tells you exactly what happened:

- Files are grouped per person, ignoring the naming inconsistencies —
  `- Full`, `_Resized`, `Original`, `Resize`, trailing dots, `'s`, underscores.
- `Resized` is preferred over `Full`. Two people have no `Resized` version and
  are logged as using `Full`.
- **GI / EL / GPL are treated as photo options, not different people.** 35
  tutors have two options; you choose which to send at send time.
- One option (`Munoshamisa` "Main") is a HEIC file, which Chrome and Firefox
  cannot open. It is skipped with a note and that tutor keeps their GI photo.
- Photos already under 1200px and 400KB are stored **as-is**, not re-compressed,
  so nothing is degraded. Only genuinely oversized photos get resized.

Every tutor starts with an obvious **placeholder** bio, badged in red
everywhere, and the app warns you before sending one.

Use a **laptop on wi-fi** for this, not a phone.

### E2. Import the real bios

**Admin → 2 · Import sell bios from Excel → Choose spreadsheet…**

Column A = tutor name, column B = bio. A header row is detected and skipped.
Names are matched against existing tutors; a single close match is accepted and
logged, an ambiguous one is reported rather than guessed. Nothing is written
until you confirm, and unmatched rows are listed rather than silently dropped.

This overwrites bios and clears the placeholder badge. It never touches photos.

> 19 tutors are first-name-only in the photo filenames (Arefa, Bruno, Buhle,
> Damian, Giselle, Himara, Lukhanyiso, Maelona, Munoshamisa, Neuza, Ntando,
> Shannon, Sria, Thato, Tinotenda, Zak…). If column A has their full name it
> will not match, because matching is on name. Either use the same first name
> in the spreadsheet, or ask for a rename feature in the Admin tab.

### E3. Give someone else upload rights

**Admin → 3 · Who can upload.** Type their `@geniuspremium.com` address and
**Add**. They get the Admin tab next time they load the page. Only you see this
section.

---

## Part F — Sending

**Send Sell Bio** tab. Type part of a name — matches hit anywhere in the name,
so `ty` finds **Ty**ler, Chrys**ty**ler and S**ty**les, and word-start matches
rank first. Accents are ignored (`lisamare` finds Lisamaré) and typos still land
(`jsmth` finds Josh Smith). No matches offers the closest names.

**Send** on any row goes straight to one tutor. **Add** collects several, then
**Send selected** steps through them one at a time.

When a tutor has more than one photo, **GI / EL** chips appear above the preview
— tap to switch which photo gets sent.

### How the caption works, and why

WhatsApp keeps the image and **discards accompanying text** when a share carries
a file. `wa.me` links can prefill text but cannot attach an image. There is no
way to guarantee both in one action — this is a WhatsApp limitation, not
something the app can fix.

So tapping **Send** does two things in the same tap: opens the share sheet with
the photo, **and** copies the bio to your clipboard. Choose WhatsApp, choose the
chat, then long-press the caption box and paste. One extra paste, every time,
on every phone.

If the text does come through by itself on your phone, take the win — it is
passed along too. Do not rely on it.

---

## Troubleshooting

| What you see | What it is |
|---|---|
| Login button does nothing / popup closes instantly | Google sign-in not enabled — step **A4** |
| `auth/unauthorized-domain` | Pages domain not authorised — step **D2**. Domain only, no `https://` or path |
| "Setup not finished" red box | Firebase config placeholders still in `index.html` — step **B1** |
| "your account may not have read access yet" | Not a `@geniuspremium.com` account, or rules not published — step **A7** |
| Signed in but no Admin tab | You are not the owner and not on the admin list. Check `OWNER_EMAIL` matches your login exactly |
| Admin list will not save | `isOwner()` in the rules does not match `OWNER_EMAIL` in the HTML — step **B2** |
| Import says "HEIC, which this browser cannot open" | Expected, for one option. Convert to JPEG and re-import that file if you want it |
| Photos import but previews are blank | Rules published? A read is failing. Check the browser console |
| 404 on the Pages URL | Pages still building, or branch/folder wrong — step **C4** |

### Free tier

Firestore's free plan covers this comfortably: 1 GiB stored, 50k reads and 20k
writes a day. The full import is about **475 writes and 41 MB**. Day to day, the
whole tutor list is one cheap read and is cached locally, and full photos load
only when you open a tutor. You do not need the paid plan, and unlike Firebase
Storage this needs no billing account.

---

## Updating the site

Edit `index.html`, then:

```bash
git add -A && git commit -m "what changed" && git push
```

Pages redeploys in a minute or so. If you do not see the change, hard-reload
(Ctrl+Shift+R) — the old file is cached.

---

## Files

| File | What it is |
|---|---|
| `index.html` | The whole app. Settings block at the top is the only part to edit |
| `firestore.rules` | Security rules. Paste into the Firebase console — not read from here |
| `README.md` | This file |
| `.gitignore` | Keeps local junk out of the repo |
| `.claude/launch.json` | Local preview server config, harmless |

### Where the data lives

```
artifacts/genius-sellbios/public/data/
├── tutors/{tutorKey}                    name, bio, 96px thumbnail, photo options
├── tutorPhotos/{tutorKey}::{variantId}  one full photo per option
└── config/access                        { bioAdmins: [ ...emails ] }
```

`tutorKey` is the name lowercased and stripped of accents and punctuation, so
`Zoë da Silva` becomes `zoe da silva`. `variantId` is the brand tag — `gi`,
`el`, `gpl`, or `main` when the filename had none.

Near-duplicate people (`Zoe Nel` / `Zoe Nell`, `Arianna` / `Arianna S`,
`Sria` / `Sria Govender`) are deliberately kept as **separate tutors**. Merging
them on a guess would be worse than showing both.
