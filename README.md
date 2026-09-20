# HaemoCare

*Your transfusion companion · เพื่อนร่วมทางการรับเลือด*

A mobile companion for patients who depend on regular blood transfusions (thalassemia, sickle cell, aplastic anemia) and for the clinicians who care for them. Patients carry a transfusion passport, log symptoms after each transfusion, keep appointments and medication reminders in one place, and share exactly as much as they choose. Clinicians get a triage-ordered view of their cohort.

Built for Thailand: bilingual English / ไทย on every screen, designed around the Personal Data Protection Act (PDPA).

**Status:** v0.1.0 public pilot on Android (sideloaded APK). iOS builds from the same codebase; distribution is pending. A web build exists for clinicians who prefer a desktop view.

<p align="center">
  <img src="docs/screenshots/01-login.png" alt="HaemoCare login screen" width="260" />
  <img src="docs/screenshots/01-login-th.png" alt="HaemoCare login screen in Thai" width="260" />
</p>

## What it does

**For patients**

- **Transfusion passport.** Blood type and Rh, antibody profile, known reactions, current medications, and an anonymised patient ID. Shareable as a QR code for staff to scan, or exported to PDF. The full name is hidden by default and shared only if the patient opts in.
- **Post-transfusion symptom monitor.** A 72-hour window opens after each transfusion. Symptoms are logged with a severity slider and evaluated against clinical thresholds into Normal, Monitor closely, or Seek medical attention. If the patient is past their planned transfusion interval, the tiers are bumped upward and the app says why.
- **Appointments.** Added by hand, imported from an `.ics` file, or fetched from a hospital FHIR endpoint that follows the TH Core implementation guide.
- **Pre-appointment brief.** Hemoglobin trend, recurring symptom patterns over 30 days, medication adherence, and counts, ready to copy, export, or share with the clinician.
- **Transfusion history.** Manual entry, or scan a transfusion slip: an AI assist extracts the fields and asks the patient to verify each one before saving.
- **Medication reminders.** Local notifications, taken and skipped tracking, an adherence score, and a daily streak.
- **Emergency contacts** with a one-tap SOS sheet, and **in-app messaging** with the care team.

**For clinicians**

- Cohort dashboard with Overdue / Monitor / Stable badges, filters, and search.
- Triage scoring that surfaces patients whose latest outcome is urgent, who are most overdue, or who have reactions on file.
- Patient detail pane and message inbox. Clinician access is admin-provisioned.

**Privacy**

Consent screen before any health data is captured. QR codes and PDFs use the patient ID rather than the name unless the patient flips the switch. One-tap view, export, and delete of everything stored. Details in [docs/HaemoCare-Overview.md](docs/HaemoCare-Overview.md).

## Repository layout

```
HaemoCare/            Expo / React Native app (the code)
  src/
    screens/          tab screens, auth, clinician, chat, admin, detail views
    components/       passport, symptoms, transfusions, medications, clinician, chat, ...
    services/         Supabase access, FHIR appointments, AI extraction, notifications
    analytics/        triage, adherence, Hb decay, symptom timelines (pure functions, tested)
    hooks/ contexts/  app state
    i18n/             en.ts and th.ts
    mock/             bundled demo data behind the demo accounts
  supabase/           schema.sql, migrations/, seed.sql, edge functions, FHIR fixtures
  scripts/            seeding, end-to-end checks, screenshot and QA helpers
docs/                 overview, install guide (EN/TH), demo walkthrough, update strategy
scripts/              QR regeneration
update-manifest.json  native-version manifest the app polls for "new APK available"
```

## Running it locally

Requires Node 20+ and the Expo Go app on a phone (or an emulator).

```bash
cd HaemoCare
npm install
cp .env.example .env      # set EXPO_PUBLIC_SUPABASE_URL and EXPO_PUBLIC_SUPABASE_ANON_KEY
npm start                 # Expo dev server; scan the QR with Expo Go
```

Other targets:

```bash
npm run web               # browser build (clinician desktop layout)
npm run android           # Android emulator
npm run ios               # iOS simulator
npm test                  # jest-expo unit tests
```

The Supabase URL and anon key must be set or the client fails to initialise. You do not need a seeded database to explore the app: sign in with the demo patient or clinician accounts listed in [HaemoCare/INSTALL.md](HaemoCare/INSTALL.md) and [docs/DEMO_GUIDE.md](docs/DEMO_GUIDE.md) and it runs against bundled mock data. On the web build at `localhost` the app signs in as the mock clinician automatically; add `?as=patient`, `?as=admin`, or `?as=none` to the URL to change that.

## Backend

Supabase (Postgres, auth, storage, realtime). Everything needed to stand it up is in `HaemoCare/supabase/`:

- `schema.sql` and dated `migrations/` for tables, row-level security, and RPC functions.
- `seed.sql` plus `scripts/seed-*.mjs` for demo patients, clinicians, and admins.
- Edge functions: `extract-transfusion` (slip scanning; the Gemini key lives in Supabase secrets, never in the app bundle) and `notify-new-message`.
- `fhir-fixtures/` and `scripts/seed-fhir.sh` to run a local HAPI FHIR server seeded with TH Core resources for appointment import.

## Tests

`npm test` runs the jest-expo suite. The clinical logic lives in pure functions so it can be tested without a device: triage outcomes and overdue bumps, adherence scoring, hemoglobin decay, symptom timelines, clinical thresholds, the `.ics` parser, cohort alerts, and notification scheduling.

## Shipping

Two update channels, described in [docs/update-strategy.md](docs/update-strategy.md):

- **JS-only changes** go out through EAS Update and install silently on the next launch.
- **Native changes** need a new APK. The app polls `update-manifest.json` and shows an in-app banner when one is available.

The step-by-step runbook is [HaemoCare/BUILD_INSTRUCTIONS.md](HaemoCare/BUILD_INSTRUCTIONS.md). Testers can also open a hosted preview in Expo Go: [HaemoCare/EXPO_GO_TESTER_GUIDE.md](HaemoCare/EXPO_GO_TESTER_GUIDE.md).

## Roadmap

- Two-way appointment sync with Thai hospital systems (FHIR, Health Link). Reading works today; writing back needs hospital partnership access.
- Wearable and passive vitals (Apple Health, Google Fit, Fitbit, Garmin) alongside symptom logs.
- iOS distribution and push notifications for overdue thresholds.

## Stack

Expo SDK 54 · React Native 0.81 · TypeScript · NativeWind · React Navigation · Supabase · expo-notifications · expo-updates · jest-expo

## Author

Hatayasit Aroonvanichporn · [hatayasit.com](https://www.hatayasit.com) · [@ggmrcodes](https://github.com/ggmrcodes)
