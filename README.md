# SPORTIQ AI

Android app (Kotlin, Jetpack Compose, MVVM, Hilt, Retrofit, Room) plus a FastAPI backend.
The app never talks to sports-data providers directly: provider API keys live only on the backend.

## Status checklist

| Stage | Scope | State |
|---|---|---|
| 1 | Gradle project, manifest, theme, navigation, dashboard shell, backend /health, API-status + offline cache, CI workflows | **Written. Backend tested; Android code not yet compiled** |
| 2 | Sports discovery, fixtures, live scores, match details | Not started |
| 3 | Real provider integrations (API-Football, football-data.org, The Odds API, Sportmonks) | Not started |
| 4 | Statistical engines (Dixon-Coles/Poisson, Elo, Monte Carlo) | Not started |
| 5 | AI reports, probabilities, value/EV analysis | Not started |
| 6 | Prediction ledger, Brier/log loss, bankroll tools | Not started |
| 7 | R8 rules, instrumentation tests, hardening | Not started |

## Build the debug APK in the cloud
1. Create a GitHub repo and push this folder to the `main` branch.
2. (Optional now, required for live status) Repo → Settings → Secrets and variables → Actions → **Variables** →
   add `BACKEND_URL` = `https://your-backend.example.com/` . This is a variable, not a secret (it is visible in the APK).
3. Actions tab → "Build debug APK" (`.github/workflows/build.yml`) runs on every push to `main` (or run it manually).
4. Open the finished run → **Artifacts** → `sportiq-ai-debug-apk` → unzip → install `app-debug.apk`
   (allow "install unknown apps" on your phone).

Without `BACKEND_URL` the app still installs and shows "Not configured" honestly.

## Build locally (Android Studio)
Open the folder in Android Studio (JDK 17). The Gradle 8.9 wrapper (`gradlew`, `gradle/wrapper/`) is included.
For a backend on your computer use the emulator address: `-PBACKEND_URL=http://10.0.2.2:8000/` (debug builds only allow cleartext to 10.0.2.2/localhost).

## Run the backend
```
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # add provider keys here; never commit .env
uvicorn app.main:app --reload
pytest
```
Deploy with the included Dockerfile (Render, Fly.io, Railway, etc.). Serve it over HTTPS and set provider keys as environment variables on the host.

## Signed release APK (secure)
1. Create a keystore once, on your machine, and back it up somewhere safe (losing it means you can never update the app under the same identity):
   `keytool -genkeypair -v -keystore sportiq-release.jks -alias sportiq -keyalg RSA -keysize 2048 -validity 10000`
2. Base64-encode it: `base64 -w0 sportiq-release.jks` (macOS: `base64 -i sportiq-release.jks`).
3. Repo → Settings → Secrets and variables → Actions → **Secrets**, add:
   `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`.
4. Push a tag (`git tag v0.1.0 && git push origin v0.1.0`) or run "Build signed release APK" manually.
   The keystore is decoded to a temp file, used for the build, then deleted. Download `sportiq-ai-release-apk` from the run's Artifacts.
Never commit `.jks` files or passwords (the `.gitignore` already excludes `*.jks`).
The release workflow has not been run yet.

## Known limits of Stage 1
- Dashboard shows only backend/provider status. There is no match data, and none is simulated.
- Provider status says "Key set, unverified" until Stage 3 adds real connectivity checks.
- Release builds are not minified yet (R8 rules come in Stage 7).
