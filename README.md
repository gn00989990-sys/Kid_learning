# Kid Learning (Kidlearning)

A kids' English/Chinese vocabulary and speaking-practice app, packaged as a native iOS app using [Capacitor](https://capacitorjs.com/).

- **App ID:** `com.fafa.kidlearning`
- **Platform:** iOS (native app via Capacitor), plus works as a plain web page in any modern browser
- **Distribution:** TestFlight, via automated builds on [Codemagic](https://codemagic.io/)

## Features

- **生字 (Vocabulary) flashcards** — grouped by topic, each word shown with an emoji, English name, and Chinese name
- **Text-to-speech playback** for every word:
  - 🇺🇸 English
  - 🇨🇳 Mandarin (普通話)
  - 🇭🇰 Cantonese (廣東話)
  - 🎵 "三語連讀" — plays all three in sequence, one second apart
- **生字小測驗 (Vocabulary quiz)** — multiple-choice quiz per topic, with the question read aloud in Cantonese and a "🔊 再聽一次" replay button
- **Mixed quiz mode** — questions drawn across all topics
- **對話練習 (Speaking / dialogue practice)** — roleplay scenarios with three distinct character voices:
  - **Alice** — child-like girl voice
  - **Bob** — child-like boy voice
  - **Server** — adult male voice

## Tech stack

- Plain HTML/CSS/JavaScript (no framework, no build step for the web layer)
- [Capacitor 8](https://capacitorjs.com/) — wraps the web app as a native iOS app
- [`@capacitor-community/text-to-speech`](https://github.com/capacitor-community/text-to-speech) — native on-device text-to-speech, with a fallback to the browser's Web Speech API when the app is opened in a normal browser
- Xcode / CocoaPods for the native iOS build
- [Codemagic](https://codemagic.io/) for CI — builds the app and publishes to TestFlight automatically on every push to `main`

## Project structure

```
.
├── web/                        # Source of truth for the app's web content
│   ├── index.html              #   — edit this file for any app changes
│   ├── BG/                     #   — background images
│   └── dialogue/                #   — dialogue/speaking practice assets
├── ios/
│   └── App/                    # Native iOS Xcode project
│       ├── App.xcodeproj
│       ├── App.xcworkspace
│       ├── Podfile
│       └── App/
│           ├── AppDelegate.swift
│           ├── Info.plist
│           └── public/         # Generated copy of web/ — DO NOT edit directly,
│                                #   it gets overwritten by `npx cap sync ios`
├── package.json
├── capacitor.config.json       # webDir points at "web"
└── codemagic.yaml              # CI build + TestFlight publish pipeline
```

> ⚠️ **Only edit files inside `web/`.** The `ios/App/App/public/` folder is a build
> artifact, regenerated from `web/` every time `npx cap sync ios` runs (including
> automatically on every Codemagic build). Changes made directly inside
> `ios/App/App/public/` will be silently overwritten.

## Local development

Since this is a plain HTML/JS app, you can just open `web/index.html` directly in a
browser to work on layout, content, or quiz logic — text-to-speech will use the
browser's built-in Web Speech API in that case.

To test the native TTS plugin and other native behavior, you need to build and run
the iOS app itself (see below).

## Building the iOS app

```bash
npm install
npx cap sync ios
cd ios/App
pod install
```

Then open `ios/App/App.xcworkspace` in Xcode (not the `.xcodeproj`) and build/run as
usual.

### CI (Codemagic)

Every push to `main` triggers an automated build defined in `codemagic.yaml`, which:

1. Installs Capacitor + the text-to-speech plugin
2. Runs `npx cap sync ios` (copies `web/` into `ios/App/App/public/`, generates
   `capacitor.js`, and updates the `Podfile` with plugin dependencies)
3. Installs CocoaPods dependencies
4. Sets up code signing
5. Bumps the build number to one above the latest build already on TestFlight
   (fetched live from App Store Connect, so it never collides with a previous upload)
6. Builds and archives the `.ipa`
7. Uploads to TestFlight and emails a notification

Before this pipeline will work, make sure `codemagic.yaml` has:

- A valid **App Store Connect integration** name matching one configured in your
  Codemagic account (`integrations.app_store_connect`)
- Your app's numeric **Apple ID** filled in under `APP_STORE_APPLE_ID` (found in App
  Store Connect → your app → App Information → Apple ID)

## Text-to-speech notes

- On-device Chinese voices (Mandarin/Cantonese) depend on what voice packs are
  actually downloaded on the user's phone. If a language isn't installed, the app
  will prompt the user and open the device's voice-download settings automatically.
  Users can also install voices manually via **Settings → Accessibility → Spoken
  Content → Voices → Chinese**.
- Character voices (Alice/Bob/Server) are differentiated by pitch/rate as well as by
  selecting different underlying system voices where available, so they still sound
  distinct even on devices with a limited set of installed voices.

## Troubleshooting

- **`cap sync` fails with "ios platform has not been added yet"** — the CLI expects
  the native project at `ios/App/App.xcodeproj`. Don't move or rename the `ios/`
  folder structure.
- **`cap sync` fails trying to copy a folder onto itself** — `webDir` in
  `capacitor.config.json` must point at `web` (the source folder), never at
  `ios/App/App/public` (the destination folder) — otherwise Capacitor tries to copy
  that folder into itself and fails immediately.
- **TestFlight build doesn't trigger an email / doesn't appear** — almost always a
  build-number problem. Apple silently ignores any upload whose build number isn't
  strictly higher than one already on TestFlight. The current pipeline fetches the
  real latest build number from App Store Connect automatically to avoid this.
