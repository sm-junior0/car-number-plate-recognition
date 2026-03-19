# 🚗 Rwandan Automatic Number Plate Recognition (ANPR)

> A real-time computer vision system that detects, aligns, reads, and logs Rwandan vehicle license plates using a live webcam feed — built with **OpenCV** and **Tesseract OCR**.

[![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python&logoColor=white)](https://python.org)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.8%2B-5C3EE8?logo=opencv&logoColor=white)](https://opencv.org)
[![Tesseract](https://img.shields.io/badge/Tesseract-OCR-4285F4?logo=google&logoColor=white)](https://github.com/tesseract-ocr/tesseract)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📋 Table of Contents

- [About the Project](#about-the-project)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Usage](#usage)
- [How Each Stage Works](#how-each-stage-works)
- [Output Format](#output-format)
- [Troubleshooting](#troubleshooting)
- [Future Improvements](#future-improvements)
- [References](#references)
- [Author](#author)

---

## About the Project

This project implements a full **Automatic Number Plate Recognition (ANPR)** pipeline designed specifically for **Rwandan vehicle license plates** (format: `AAA999A`, e.g. `RAA123B`, `RCA456Z`). It was developed as part of the **Intelligent Robotics** course at [Rwanda Coding Academy (RCA)](https://rca.ac.rw), Year 3.

The system captures live video from a webcam and processes each frame through a five-stage pipeline — from raw detection all the way to validated, timestamped CSV logging — without requiring any deep learning models or GPU hardware.

### Why This Matters

Manual vehicle identification is slow and error-prone. An automated plate reading system can be applied to:

- 🚧 **Toll collection & highway checkpoints**
- 🅿️ **Parking management systems**
- 🔐 **Access control for gated communities or institutions**
- 🚔 **Law enforcement & traffic monitoring**

---

## Key Features

| Feature | Description |
|---|---|
| 🎥 **Real-time processing** | Processes live webcam feed frame-by-frame |
| 🔍 **Contour-based detection** | No ML model needed — uses edge detection + geometric filtering |
| 📐 **Perspective correction** | Warps skewed plates into a clean 450 × 140 px image |
| 🔤 **OCR with Tesseract** | Extracts text using LSTM engine with character whitelist |
| ✅ **Regex validation** | Filters OCR noise using Rwandan plate pattern `[A-Z]{3}[0-9]{3}[A-Z]` |
| 🗳️ **Temporal majority vote** | Confirms plates via a rolling 5-frame buffer to eliminate false reads |
| 📝 **CSV logging** | Saves confirmed plates with timestamps; 10-second cooldown prevents duplicates |
| 🧩 **Modular architecture** | Each stage can be tested independently |

---

## System Architecture

The pipeline processes each video frame through the following stages:

```
┌──────────┐    ┌───────────┐    ┌───────────┐    ┌─────┐    ┌────────────┐    ┌──────────────┐    ┌──────────┐
│  Webcam  │───▶│ Detection │───▶│ Alignment │───▶│ OCR │───▶│ Validation │───▶│ Confirmation │───▶│ CSV Log  │
│ (camera) │    │ (contour) │    │ (warp)    │    │     │    │  (regex)   │    │  (majority)  │    │ (output) │
└──────────┘    └───────────┘    └───────────┘    └─────┘    └────────────┘    └──────────────┘    └──────────┘
```

| Stage | Module | Responsibility |
|-------|--------|----------------|
| **0 — Camera** | `src/camera.py` | Verify webcam connectivity and frame capture |
| **1 — Detection** | `src/detect.py` | Locate plate-shaped rectangular contours in each frame |
| **2 — Alignment** | `src/align.py` | Perspective-warp detected plate to a canonical 450 × 140 px image |
| **3 — OCR** | `src/ocr.py` | Preprocess the aligned plate and extract text via Tesseract |
| **4 — Validation** | `src/validate.py` | Match OCR output against the Rwandan plate regex pattern |
| **5 — Full Pipeline** | `src/main.py` | Orchestrate all stages + temporal confirmation + CSV logging |

---

## Project Structure

```
car-plate-recognition/
│
├── README.md                ← You are here
├── requirements.txt         ← Python dependencies (opencv, numpy, pytesseract)
├── .gitignore               ← Excludes venv, __pycache__
│
├── src/
│   ├── camera.py            ← Stage 0: webcam test
│   ├── detect.py            ← Stage 1: plate detection via contour analysis
│   ├── align.py             ← Stage 2: perspective rectification
│   ├── ocr.py               ← Stage 3: Tesseract OCR with preprocessing
│   ├── validate.py          ← Stage 4: regex-based plate validation
│   └── main.py              ← Stage 5: full integrated pipeline
│
├── data/
│   └── plates.csv           ← Auto-generated log of confirmed plates
│
└── screenshots/
    └── (place test screenshots here after running with real vehicles)
```

---

## Prerequisites

| Requirement | Version | Purpose |
|-------------|---------|---------|
| **Python** | ≥ 3.8 | Runtime environment |
| **OpenCV** | ≥ 4.8.0 | Image processing and video capture |
| **NumPy** | ≥ 1.24.0 | Numerical operations (array manipulation) |
| **pytesseract** | ≥ 0.3.10 | Python wrapper for Tesseract OCR |
| **Tesseract OCR** | ≥ 4.0 | The OCR engine itself (installed externally) |
| **Webcam** | — | USB or built-in camera for live capture |

---

## Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/sm-junior0/car-number-plate-recognition.git
cd car-number-plate-recognition
```

### 2. Create a Virtual Environment *(recommended)*

```bash
python -m venv .venv

# Activate on Windows:
.venv\Scripts\activate

# Activate on macOS / Linux:
source .venv/bin/activate
```

### 3. Install Python Dependencies

```bash
pip install -r requirements.txt
```

### 4. Install Tesseract OCR Engine

> ⚠️ **Important:** `pytesseract` is only a Python wrapper. The Tesseract binary must be installed separately on your system.

| Platform | Installation |
|----------|-------------|
| **Windows** | Download the installer from [UB Mannheim Tesseract builds](https://github.com/UB-Mannheim/tesseract/wiki), install, and add the install directory to your system `PATH` |
| **macOS** | `brew install tesseract` |
| **Ubuntu / Debian** | `sudo apt update && sudo apt install tesseract-ocr` |

**Verify installation:**

```bash
tesseract --version
# Expected: tesseract 5.x.x (or 4.x.x)
```

---

## Usage

You can run each stage independently to test and debug step by step, or launch the full pipeline directly.

### Run Individual Stages

```bash
# Stage 0 — Confirm your webcam is working
python src/camera.py

# Stage 1 — Plate detection only (bounding boxes drawn on frame)
python src/detect.py

# Stage 2 — Detection + perspective alignment (shows warped plate)
python src/align.py

# Stage 3 — Detection + alignment + OCR (shows extracted text)
python src/ocr.py

# Stage 4 — Detection + alignment + OCR + regex validation
python src/validate.py
```

### Run the Full Pipeline

```bash
# Stage 5 — Complete pipeline with temporal confirmation + CSV logging
python src/main.py
```

> **Press `q`** in any OpenCV window to quit gracefully.

---

## How Each Stage Works

### Stage 1 — Detection (`detect.py`)

Locates license plate candidates using classical computer vision techniques — no deep learning required.

**Algorithm:**
1. Convert the BGR frame to **grayscale**
2. Apply **Gaussian blur** (5×5 kernel) to suppress noise
3. Run **Canny edge detection** (thresholds: 100, 200)
4. Extract **external contours** from the edge map
5. Filter contours by two geometric constraints:
   - **Minimum area** ≥ 600 px² — eliminates small noise
   - **Aspect ratio** between 2.0 and 8.0 — plates are rectangular (wider than tall)

**Output:** A list of `cv2.minAreaRect` tuples representing plate candidates.

---

### Stage 2 — Alignment (`align.py`)

Corrects rotation, slant, and perspective distortion so OCR receives a clean, upright plate image.

**Algorithm:**
1. Extract the 4 corner points of the rotated bounding box using `cv2.boxPoints()`
2. **Order corners** consistently: top-left → top-right → bottom-right → bottom-left
   - Uses sum (`x + y`) and difference (`x − y`) of coordinates
3. Compute a **perspective transform matrix** (`cv2.getPerspectiveTransform`)
4. **Warp** the plate region to a fixed **450 × 140 px** output

---

### Stage 3 — OCR (`ocr.py`)

Extracts alphanumeric characters from the aligned plate image.

**Preprocessing pipeline:**
1. Convert to **grayscale**
2. Apply **Gaussian blur** (5×5) for noise reduction
3. Apply **Otsu's binarisation** — automatic thresholding that separates text from background

**Tesseract configuration:**

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `--psm` | 8 | Treat image as a single word |
| `--oem` | 3 | Use default LSTM neural net engine |
| `whitelist` | `A-Z`, `0-9` | Restrict output to uppercase letters and digits only |

---

### Stage 4 — Validation (`validate.py`)

Filters out OCR noise and misreads using a regex tailored to the Rwandan plate format.

**Pattern:**
```
[A-Z]{3}[0-9]{3}[A-Z]
```
- 3 uppercase letters + 3 digits + 1 uppercase letter
- Examples: `RAA123A`, `RAB456C`, `RCA789Z`

**Why validation is needed:**
- OCR may confuse visually similar characters: `B↔8`, `O↔0`, `S↔5`, `I↔1`
- National emblems or decorative plate borders may produce false characters
- The regex acts as a hard filter to only accept structurally valid plates

---

### Stage 5 — Full Pipeline (`main.py`)

Orchestrates all stages and adds two reliability mechanisms:

#### Temporal Confirmation (Majority Vote)
- Valid OCR readings are collected into a **rolling buffer of 5 frames**
- A **majority vote** (most frequent plate in the buffer) determines the confirmed reading
- This eliminates single-frame OCR errors and jitter

#### CSV Logging with Cooldown
- Confirmed plates are written to `data/plates.csv` with a timestamp
- A **10-second cooldown** per plate prevents duplicate entries when a car remains in frame
- The CSV file is automatically created with headers if it doesn't exist

---

## Output Format

Confirmed plates are logged in `data/plates.csv`:

```csv
Plate Number,Timestamp
RAA123B,2026-03-19 14:05:32
RCA456A,2026-03-19 14:07:18
RAB789C,2026-03-19 14:12:45
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `Camera not opened` error | Ensure webcam is connected; try changing camera index from `0` to `1` in the source |
| `TesseractNotFoundError` | Tesseract binary is not in your `PATH`. Re-install and verify with `tesseract --version` |
| OCR returns empty/garbage text | Ensure adequate lighting; plates should be in focus and at a readable distance |
| No plate detected | Adjust `MIN_AREA`, `AR_MIN`, or `AR_MAX` constants in `detect.py` for your camera setup |
| `ModuleNotFoundError` | Activate your virtual environment and run `pip install -r requirements.txt` |

---

## Future Improvements

- [ ] Add support for additional plate formats (e.g. international/diplomatic plates)
- [ ] Integrate a deep learning detector (YOLO or SSD) for more robust detection in varied conditions
- [ ] Add a GUI dashboard for real-time monitoring and plate history browsing
- [ ] Support video file input alongside live webcam feed
- [ ] Implement night/low-light plate detection using infrared preprocessing
- [ ] Deploy as a REST API for integration with parking or access control systems

---

## References

1. OpenCV Documentation — [https://docs.opencv.org](https://docs.opencv.org)
2. Tesseract OCR — [https://github.com/tesseract-ocr/tesseract](https://github.com/tesseract-ocr/tesseract)
3. Gabriel Baziramwabo, *Car Number Plate Extraction in Three Steps — Detection, Alignment, and OCR*, Benax Technologies Ltd / Rwanda Coding Academy
4. Smith, R. (2007). *An Overview of the Tesseract OCR Engine*. Ninth International Conference on Document Analysis and Recognition (ICDAR)
5. Bradski, G. (2000). *The OpenCV Library*. Dr. Dobb's Journal of Software Tools

---

## Author

**SM Junior** — [@sm-junior0](https://github.com/sm-junior0)

📚 Year 3, **Intelligent Robotics** — [Rwanda Coding Academy](https://rca.ac.rw)

---

<p align="center">
  <em>Built with ❤️ in Rwanda 🇷🇼</em>
</p>
