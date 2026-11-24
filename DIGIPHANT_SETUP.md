# DigiPhant setup

## This Mac and project

| Item | Setup |
| --- | --- |
| Computer | Apple Silicon (arm64), macOS 26.5.1 (25F80) |
| Unity | Installed editor 6000.6.10f1, URP 17.6.0, PC quality profile |
| Python | 3.14.6, environment at `DigiPhantStarter/Tracking/.venv` |
| Packages | MediaPipe 0.10.35, opencv-contrib-python 4.14.0.94 (cv2 reports 4.14.0), NumPy 2.5.5 |
| Camera | Built-in MacBook Pro camera reported by macOS. Index 0 is the scene default and awaits an authorized capture. |
| Starter | `https://github.com/kommanderpi/studentstarter.git` at `4e6dab5271c618d1c0bfb2022c41c0d37cd8d69f` |
| Scene | `Assets/DigiPhant/Scenes/DigiPhant.unity`, default three performers, full body, Test sliders |

Imported `DigiPhant` and `Elephant` with their Unity metadata. The separate
DigiPhant scene uses the supplied rig, stage, mappings, locomotion, camera launcher
and recorder. `SampleScene`, its assets, the package configuration and project
settings were preserved. The vendor keyboard controller is disabled on the
DigiPhant elephant, so the project can keep the existing Input System settings.
The imported editor validator now selects Camera mode before injecting seated
tracking packets. The original left it in Test sliders after the UDP check.
Runtime scripts, Python source and shared elephant assets use the starter versions.

## How movement reaches the elephant

1. Unity Play starts `DigiPhantStarter/Tracking/bridge.py` using the local virtual
   environment. OpenCV opens the camera and MediaPipe estimates up to four poses
   using the bundled `pose_landmarker_full.task`.
2. Python assigns IDs left-to-right in the unmirrored image once the selected
   number are visible. Subsequent matching uses hip centers in full-body mode
   and shoulder centers in seated mode. It sends version-1 JSON snapshots with
   left/right hand height, left/right foot lift, lean and arm spread, plus a
   confidence for each signal. These are body-relative image measurements.
3. On localhost, Unity reads tracking on UDP 5055 and sends count, mode and reset
   commands to Python on 5056. The annotated JPEG preview uses UDP 5057, up to
   10 Hz. The camera and tracking data stay on this computer.
4. Unity takes a fresh, confident baseline at the end of the 10-second countdown.
   Controls subtract that baseline, multiply by sensitivity, clamp to [-1, 1]
   and smooth before applying each joint's configured angle. The default joint
   smoothing value is 8, minimum confidence is 0.5 and tracking times out after
   0.5 seconds. Lost joint inputs ease toward rest. Missing locomotion inputs
   stop world movement. The manual animation graph blends in-place gait clips
   before adding the gesture rotations. A parent transform handles travel.

## Live test

1. Open the DigiPhant scene, press **Play**, and select the **Game** tab. Move
   the **Trunk curl** test slider to check the rig. Press **Reset body sliders**.
2. If macOS asks, allow Camera access for the requesting app. If access was
   denied, use **System Settings → Privacy & Security → Camera** and click
   **Retry** in the preview. For independent diagnosis, run the command below
   from this project in Terminal with Unity stopped. Grant Terminal camera
   access if prompted. Use **Q** in the Python window to quit before restarting
   Unity's automatic bridge.
3. Select **1 person**, **Seated / upper body**, and **Camera**. Wait for a P1
   skeleton and visible status. Click **Set neutral pose (10 seconds)**. Keep
   shoulders and hands in frame and hold both hands at mid-height.
4. Raise your right hand and check for trunk curl and right-leg motion. Raise
   your left hand to check travel, then return to neutral. Step out of frame,
   verify visible status clears and movement stops, then return. If the ID is
   wrong, use **Reassign people** and recalibrate. **Return to starting position**
   resets travel and also requires recalibration. Stop Play to close Unity's bridge.
5. For three people choose **3 people** and **Full body**, line up left-to-right
   with shoulders, hips, hands and feet visible, and recalibrate. Hold P1's left
   hand near the waist for travel calibration.

| Performer | Full-body test |
| --- | --- |
| P1 | Lift each foot for front-leg controls. Raise/lower the left hand to travel. Lean to steer. |
| P2 | Lift each foot for rear-leg controls. Lean for tail sway. |
| P3 | Lean for head turn, raise the right hand for trunk curl, spread the hands for ears. |

Seated mode substitutes hand lifts for foot lifts and shoulder tilt for lean.
Solo assigns all controls to P1, so some gestures move several parts together.

```sh
DigiPhantStarter/Tracking/.venv/bin/python DigiPhantStarter/Tracking/bridge.py --people 1 --upper-body-only
```

If index 0 opens the wrong device, try `--camera 1` during the standalone test.
A manually launched bridge remains running until closed and can send preview to
Unity while it runs. Use only one bridge for the default ports.

## Recording and Drive

In Camera mode with a live preview and neutral pose, click **Record performance**.
Keep the Game view visible at a fixed size, preferably 1280×720 or larger. Perform
a short take with both preview and elephant visible. Click **Stop recording and
save** and wait for **Saved**. Unity captures timestamped JPEGs up to 1280×720
at a target of 10 fps. Python exports a silent MP4 using OpenCV and preserves
elapsed time by holding frames across capture gaps.

Play `Recordings/<session>/performance.mp4` and check both panels, the beginning
and end, and each performer's action. Upload the checked MP4 to the team's Drive
folder, verify playback and instructor access, and add the link to the README.
The project has no supplied Drive destination or share link. Git ignores the
recording directory, frame logs and video. To retry export from retained frames:

```sh
DigiPhantStarter/Tracking/.venv/bin/python DigiPhantStarter/Tracking/encode_recording.py --session "Recordings/SESSION_FOLDER"
```

## Verification on 2026-11-24

- Asset provenance passed for 124 fingerprints before the editor-only validation
  adjustment. Python imports and `pip check` passed. All 11 supplied Python tests
  passed, including assignment, confidence/normalization, preview UDP and writing
  and decoding an MP4 with timing gaps.
- Unity imported and compiled the scene and scripts. The supplied batch checks
  passed for rig references, sliders, synthetic camera packets, seated calibration,
  timeout, localhost UDP, group presets, gait playback, additive joints, world
  travel, turns and reset.
- The standalone camera probe detected an OpenCV authorization failure on macOS
  before it could open index 0. It received zero tracking packets and preview
  frames. Live preview, performer assignment, calibration, loss/recovery,
  visual material appearance and a real Game view recording still need the
  interactive checks above. The encoder test used generated frames.

## Repository reconstruction

Initialized the outer repository on `main`. It tracks the original scene and
project configuration, the imported DigiPhant scene and scripts, and their
metadata. The nested starter checkout, vendor assets, pose model, Python
environment, Unity caches and recordings are ignored. The first commit contains
98 tracked files and its largest file is the DigiPhant scene (about 80 KiB).

Verified the README restore in a separate fresh checkout of `dc5e67f` with no
Unity Library. Cloned the starter from GitHub at the pinned commit, restored the
vendor folder with its metadata, created a new Python environment and passed
`pip check`. Unity imported the project and passed both validation suites.
All 15 non-builtin scene GUIDs resolved uniquely to restored assets or installed
packages, including the URP materials and their shader references. The
reconstructed worktree was clean. The working project's DigiPhant scene was
also opened in the graphical editor for the live check.
