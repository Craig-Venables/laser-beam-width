# Laser Beam Width

Live PyQt5 tool to measure **laser spot size** from an overhead camera (typically back-illuminated sample). Reports **FWHM** and **1/e²** widths on X/Y axes via Gaussian profile fits, with optional µm/px calibration and Thorlabs Kinesis Z focus jog.

Private lab repository (Craig Venables / University of Nottingham). Intended for group sharing later — keep instrument serials in local config, not in commits when possible.

---

## Hardware

| Device | Role | Notes |
|--------|------|--------|
| **Thorlabs C1284R13C (uc480)** | Primary camera | Needs **ThorCam** SDK installed |
| OpenCV USB webcam | Fallback | e.g. Logitech C920 |
| Thorlabs TLCamera (Kiralux/Zelux) | Optional | Not for C1284R13C |
| Thorlabs Kinesis Z stage | Optional focus jog | Via `pylablib` `KinesisMotor` |

**OS:** Windows recommended (DirectShow / ThorCam). Close ThorCam before connecting from this app (exclusive device access).

This tool does **not** control the laser or a power meter — camera (+ optional Z) only.

---

## Requirements

- Python **3.10+** (3.10 recommended on lab PCs)
- ThorCam if using uc480

```bash
pip install -r requirements.txt
```

| Package | Use |
|---------|-----|
| PyQt5 | GUI |
| opencv-python | Capture / image ops |
| numpy, scipy | Arrays + Gaussian `curve_fit` |
| pylablib | uc480, TLCamera, Kinesis |
| pygrabber | Windows camera names |
| matplotlib, pandas | Offline focus-scan plots (`plot_focus_scan.py`) |

---

## Quick start

```bash
python main.py
```

Optional:

```bash
python smoke_test_uc480.py
python plot_focus_scan.py <focus_scan_folder> [-o out] [-t title]
python -m unittest tests.test_gaussian_fit
```

---

## Camera connect (uc480)

1. Close **ThorCam** and any other app using the camera.
2. Start this tool → backend **Thorlabs UC480 (ThorCam)**.
3. **Scan cameras** → select your device → **Connect**.
4. If the camera shows `[in use]`, close other programs and retry.

OpenCV only sees UVC webcams. With both a Thorlabs C1284R13C and a Logitech connected, OpenCV will typically list only the Logitech — use the **uc480** backend for the Thorlabs camera.

---

## Small-spot workflow

For spots only a few pixels wide:

1. **Display zoom** — `8×`–`32×` under Zoom & spot.
2. **Click image to set spot center** — magenta crosshair.
3. **ROI half-size** — typically `10–30` px.
4. **Sensor ROI (uc480)** — optional hardware crop (e.g. 256×256), then **Apply sensor ROI**.

Use Navitar (or other) lens zoom on the hardware first, then fine-tune in software.

---

## Calibration and measurement

1. Connect camera.
2. Optional: **Capture background** with laser OFF; enable **Subtract background**.
3. **Calibrate** (optional): enter a known distance (µm) at the sample plane, click two points — or leave pixels-only mode.
4. Read FWHM / 1/e² in the results panel.
5. **Freeze frame** → **Save snapshot + results** (PNG + JSON + CSV).

Config is stored in `beam_width_config.json` (camera serial, exposure, calibration, stage limits). Ship with empty serials; fill in locally.

---

## Z focus (Kinesis)

Sidebar **Z focus** panel:

1. Connect camera first.
2. **Connect Z** (enter serial in config under `"stage"`).
3. Jog with **Z − / Z +** while watching the live image.

You can run a separate motor-control GUI at the same time if it does not open the same camera. Prefer the direct `KinesisMotor` import path in other apps (avoid loading the full TLCamera SDK when only Kinesis is needed).

---

## Exports and companion tools

Saved `beam_width_*.json` files include fit metrics and `calibration.um_per_pixel`. Companion repos / tools can load them for focus scans and power-density estimates (e.g. **laser-power-lab** focus-scan spot size).

Offline plot of a folder of focus-scan JSONs:

```bash
python plot_focus_scan.py path/to/folder -o focus_scan.png
```

---

## Laser safety

Laser radiation can cause **permanent eye damage**. Use minimum power for alignment, keep beam paths enclosed, and wear appropriate eyewear for the wavelength.

---

## Repository layout

```
main.py                 # GUI entrypoint
plot_focus_scan.py      # Offline focus-scan plots
smoke_test_uc480.py     # Hardware smoke test
beam_width_config.json  # Local settings (sanitize before sharing)
analysis/               # Calibration, Gaussian fit, preprocessing
camera/                 # OpenCV / uc480 / TLCamera sources
gui/                    # Main window, video, results, overlays
stage/                  # Kinesis Z motor helper
tests/                  # Unit tests (no hardware required)
```

---

## License

MIT — see [LICENSE](LICENSE).
