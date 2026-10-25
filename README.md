# Snap Vanish (Android)

Join a live meeting from your phone, snap your fingers, dissolve into dust, and leave the other participants
looking at the reconstructed empty room. Snap again and the *same* particles fly back and rebuild you.

> **Status: source complete, NOT yet compiled or run.** See `VERIFICATION.md` for exactly what is and is not verified.
> There is no APK in this archive; the included CI workflow builds one (below).

## Get the APK (no PC needed after the first upload)
You need a free GitHub account. From a phone browser (desktop-site mode helps) you can do all of this:
1. Create a new empty GitHub repository and upload the contents of this folder (the `.github` folder must come along).
   Easiest: GitHub web → *Add file → Upload files*, or use the repo's "Import" tools.
2. The workflow `.github/workflows/build-apk.yml` runs on push (or *Actions → Build Snap Vanish APK → Run workflow*).
   It downloads the two on-device models, runs the unit tests, builds the APK and publishes it under **Releases**.
3. Open the Release on your phone, download `app-debug.apk`, allow "install unknown apps" for your browser, install.
4. If the build fails, the Actions log lists the compile errors. Paste them back and they get fixed — the SDK calls most
   likely to need adjusting are listed in `VERIFICATION.md`.

(With a PC: `bash scripts/fetch_models.sh && gradle assembleDebug`, JDK 17 + Android SDK 34.)

## One-time meeting setup (unavoidable)
Meetings use **LiveKit** (open-source SDK; LiveKit Cloud has a hosted tier — check its current pricing/terms yourself,
I could not verify them from my sandbox). You need three values from a LiveKit project: `wss://` URL, API key, API secret.
Pick one:
- **Quick test:** in the LiveKit dashboard create a token for your room, paste URL + token in *Service setup*. Tokens expire.
- **Proper:** deploy `token-server/` (Node ≥ 18, or the Dockerfile) on any host with env vars `LIVEKIT_URL`,
  `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET` and optionally `TOKEN_SERVER_KEY`. Put its https:// address (and key) in
  *Service setup*. The app then mints tokens itself and can create **invite links**: guests open a browser link
  (meet.livekit.io) and do not install anything. Most PaaS dashboards can deploy a Dockerfile from a phone browser.

## Using it
1. Grant camera/microphone/notification permissions.
2. **Calibrate** (Home → Calibrate): prop the phone where it will sit, step fully out of frame ~3 s. The empty-room image is saved
   privately on the phone; *Delete* removes it.
3. **Practice** lets you test the effect, sound and snap detection without a meeting. Tune *Sound ↔ visual sync offset* in Settings.
4. **Join meeting**. Snap (thumb + middle finger, with a real snap sound) or tap **Snap Now**. Snap again / **Reappear** to return.
   The mic is muted mid-dissolve only if it was on, and restored only if it was on before, unless you toggle it yourself meanwhile.

## What runs where
- On the phone only: hand landmarks, person segmentation, snap-sound detection, background reconstruction, particles, sound synthesis.
- Sent to the meeting service: your processed outgoing video and microphone audio (that is what a call is). Nothing else is uploaded;
  there is no analytics or ads SDK. Settings (incl. token) are in app-private storage; backups are disabled.

## Known limitations / not implemented
- Works inside Snap Vanish only; it cannot alter Google Meet/Zoom/Teams.
- Front camera, portrait only. Output 270p/360p/540p by performance mode (CPU compositing; not a GPU shader pipeline).
- Reconstruction is as good as the calibration: unobserved areas are filled by smooth diffusion, **not** an ML inpainting model.
- Snap recognition is a heuristic over hand landmarks fused with an audio transient; it needs real-world tuning and will miss/false-trigger sometimes.
- During the dissolve the body is a snapshot taken at the snap (the live moving figure is not tracked).
- Sending the snap sound into the meeting is **not** implemented (it plays locally only).
- No ML segmentation → no effect: if the model files are absent the app says so and the vanish has nothing to dissolve.
