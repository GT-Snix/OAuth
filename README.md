# OAuth

A small Express app that lets you sign in with your Google account. I built it to learn how OAuth login works with Passport, so it's kept as short as I could make it: one file, three routes.

## What it does

1. You open the home page and click "Login with google".
2. You get sent to Google, pick your account and agree to share your basic profile.
3. Google sends you back to the app, which says hi using your name.

The app only asks for the `profile` scope, so it sees your name and profile picture, not your email or anything else.

## Running it locally

You need Node 20.6 or newer (for the `--env-file` flag) and a Google account.

### 1. Get Google credentials

1. Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a project, or pick one you already have.
2. Open **APIs & Services > OAuth consent screen** and set it up. "External" with yourself added as a test user is enough for local testing.
3. Open **APIs & Services > Credentials**, click **Create credentials > OAuth client ID** and choose **Web application**.
4. Under **Authorized redirect URIs**, add:

   ```
   http://localhost:3003/callback
   ```

5. Save, and copy the Client ID and Client secret it gives you.

### 2. Add them to a .env file

```sh
cp .env.example .env
```

Then open `.env` and paste your values in:

```
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
```

`.env` is in `.gitignore`, so it never gets committed. Please keep it that way. If the secret ever ends up on GitHub, delete it in the Cloud Console and make a new one.

### 3. Install and start

```sh
npm install
npm start
```

Open http://localhost:3003 and click the login link.

If you see "Missing GOOGLE_CLIENT_ID or GOOGLE_CLIENT_SECRET", the `.env` file is missing or empty. If Google shows a `redirect_uri_mismatch` error, the redirect URI in the Cloud Console doesn't exactly match `http://localhost:3003/callback`.

## How it works

Everything is in `index.js`.

| Route | What happens |
|---|---|
| `/` | Shows the login link. |
| `/login` | Passport sends you to Google's sign-in page. |
| `/callback` | Google sends you back here with a code. Passport swaps it for an access token, loads your profile and the app greets you. |

The Google strategy is set up with `passport.use(new GoogleStrategy(...))`. Its callback receives the access token and your profile once Google has confirmed who you are.

Logging tokens is an easy way to leak them, so anything the app prints goes through [keyblob](https://www.npmjs.com/package/keyblob) first. The token shows up in the terminal as `[REDACTED]` while your profile is still readable.

## Before using this for anything real

This is a learning project, and a few things are missing on purpose:

- There are no sessions (`session: false`), so you're "logged in" for one page only. A real app would use `express-session` or a cookie to remember you.
- It runs on plain `http` on localhost. In production you'd need HTTPS and your real domain added as a redirect URI.
- `/failure` doesn't have a page yet, so a cancelled login ends on a "Cannot GET" error.

## Keeping secrets out of the repo

This repo has a keyblob pre-commit hook, so a commit that contains an API key or a `.env` file gets stopped before it happens. If you clone it and want the same check, run:

```sh
npx keyblob hook install
```

## Built with

- [Express](https://expressjs.com/)
- [Passport](https://www.passportjs.org/) with [passport-google-oauth20](https://www.passportjs.org/packages/passport-google-oauth20/)
- [keyblob](https://github.com/GT-Snix/keyblob)
