# Verification report (honest status)

Environment used to write this: a sandbox with a JRE but **no Android SDK, no Gradle, no Kotlin compiler and no network**.

| Item | Status |
|---|---|
| Kotlin source (29 main + 7 test files), Gradle/Manifest/resources, CI workflow, token server | **Written** |
| Compiles / builds an APK | **NOT verified** (never compiled) |
| Unit tests (7 files in `app/src/test`) | **Written; NOT run.** Expected values were hand-traced against the code, which is not the same as running them. CI runs them. |
| Static check done here | brace/paren balance per file only |
| Installation, camera, meeting, segmentation, hand tracking, sound sync, mic handling on a device | **Unverified** |

## Acceptance tests A–M
All **unverified**. A (real meeting), B, C, D, E, F, G, H, I, K, L, M need a device and a LiveKit project.
J (manual controls) and the state-machine parts of D/G/K are covered by unit tests that have not yet been executed.

## API calls most likely to need a fix at first compile (written from memory, not checked against the pinned versions)
1. `MeetingManager.join`: `localParticipant.createVideoTrack(name = ..., videoProcessor = ...)` — LiveKit 2.5.0 parameter names/availability.
2. `MeetingManager`: `RoomEvent.*` class names/fields (`error`), `room.remoteParticipants`, `participant.identity?.value`, `TrackPublication.muted`.
3. `MlWorker`: MediaPipe `confidenceMasks()` Optional handling and `ByteBufferExtractor` float mask layout; whether the multiclass model's class 0 is background (assumed).
4. `SnapVideoProcessor`: `org.webrtc.VideoProcessor` method set in the WebRTC build LiveKit bundles.
5. Model URLs in `scripts/fetch_models.sh` and SDK versions (`livekit-android:2.5.0`, `tasks-vision:0.10.14`) were not checked online.

## Design points worth testing first on a real phone
- Does the processed track keep publishing at ≥15 fps in Balanced mode? (CPU compositing + I420 conversion per frame.)
- Snap recognition thresholds (`SnapGestureDetector`, `AudioTransientDetector`) and the sync-offset slider.
- Does a second capture (AudioRecord) coexist with the SDK's microphone on your device? If not, the app falls back to camera-only detection.
- Calibrated background vs. LiveKit's capture crop (both should be 16:9 → centre-cropped to 9:16).

## Not implemented
Sending the snap sound to meeting participants; ML inpainting; GPU shaders; landscape; rear camera.
