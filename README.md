# VisionEye — Object Monitoring with YOLO26s

A computer vision project built with **Ultralytics `VisionEye`** and **YOLO26s** (`yolo26s.pt`) using **`uv`** for dependency management. It detects and tracks three object classes from a fixed camera viewpoint (`vision_point` pinned to the top-left ceiling camera) and exports an H.264 `.mp4` video ready for **WhatsApp** and **LinkedIn**.

---

## Output Demo

[▶ Watch the VisionEye Output Video](outputs/input_video_visioneye.mp4)

> **Tip for GitHub:** When editing on GitHub, you can also drag and drop `outputs/input_video_visioneye.mp4` directly into the `README.md` editor if you want GitHub to host an inline playable video player.

---

## Clone & Pull from GitHub

```bash
# Clone the repository
git clone [https://github.com/mahmoudrabea01/eye_vision_task.git](https://github.com/mahmoudrabea01/eye_vision_task.git)
cd eye_vision_task

# Pull the latest updates later
git pull origin main
```

---

## What We Built & Project Setup

### 1. Folder Structure

```text
eye_vision_task/
├── inputs/                 # Place input videos here
├── models/                 # Stores yolo26s.pt weights
├── outputs/                # Final H.264 .mp4 outputs
├── .env                    # Local environment variables (not committed)
├── .env.example            # Template environment file
├── main.py                 # VisionEye processing & H.264 export script
└── pyproject.toml          # uv project & Python 3.12 dependencies
```

### 2. Environment Setup with `uv`

The project pins **Python 3.12** in `pyproject.toml` with `opencv-python`, `numpy`, `torch`, `torchvision`, `mediapipe`, `pillow`, `scipy`, `ultralytics`, `python-dotenv`, and `imageio-ffmpeg`.

```bash
# Sync virtual environment and lockfile
uv sync

# Create your local .env from the template
cp .env.example .env        # Windows PowerShell: Copy-Item .env.example .env
```

*(On Windows PowerShell, if `.venv\Scripts\activate` is blocked by execution policy, you can either run `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` or skip activation altogether by using `uv run`.)*

### 3. Configuration (`.env`)

```dotenv
MODEL_WEIGHTS=yolo26s.pt
CONFIDENCE_THRESHOLD=0.25
IOU_THRESHOLD=0.45
SOURCE=inputs/input_video.mp4
```

### 4. Key Implementation Details

- **Model storage:** `yolo26s.pt` is automatically saved to and loaded from `models/yolo26s.pt`.
- **3-Class Detection:** Configured with `classes=[0, 1, 2]` (COCO: `person`, `bicycle`, `car`).
- **Camera-Aligned Vision Point:** `vision_point` is set dynamically to `(int(w * 0.079), int(h * 0.143))` so the VisionEye origin sits directly on the top-left hanging dome camera regardless of video resolution (you can decrease `x` slightly, e.g., `eye_x - 20`, to shift it further left).
- **WhatsApp & LinkedIn-Ready MP4:** OpenCV writes a temporary `.mp4`, and `imageio-ffmpeg` transcodes it to **H.264 (`libx264`)** with `yuv420p` pixel format and `+faststart` into `outputs/input_video_visioneye.mp4`.

### 5. Run the Script

```bash
uv run python main.py
```

---

## Frequently Asked Questions (VisionEye & Video Detection)

### 1. What is VisionEye?
**VisionEye** is an Ultralytics computer vision solution that simulates a fixed "eye" or camera viewpoint inside a video frame. For every detected object, it draws a sightline connecting the configured `vision_point` (such as a ceiling CCTV camera) to the object's bounding box, making it immediately clear where each object is relative to that vantage point.

### 2. Why is VisionEye useful in visual monitoring?
In surveillance, retail analytics, and traffic monitoring, raw bounding boxes can feel cluttered and disconnected. VisionEye anchors all detections to the physical camera's position, helping operators quickly judge **direction, distance, and spatial distribution** relative to the sensor—for example, seeing which shoppers or vehicles are moving toward or away from a monitored entrance.

### 3. How does it differ from normal object detection?
Normal object detection only answers **"what is in the frame and where is its box?"** VisionEye runs object detection under the hood and adds a **spatial perspective layer** by linking every tracked/detected object back to a single reference coordinate (`vision_point`) with directional sightlines and optional class filtering.

### 4. Why does video quality affect detection?
YOLO models rely on visual features like edges, textures, contrast, and shape to classify and localize objects. Low resolution, heavy compression artifacts (blocking/blur), motion blur, poor lighting, or lens glare smear those features—causing missed detections, lower confidence scores, or misclassified objects, especially for small or distant targets.

### 5. Why are frame size and FPS required?
When writing an output video with `cv2.VideoWriter`, OpenCV needs to know the exact canvas dimensions (`width × height`) to allocate and encode every frame consistently, and the **FPS (frames per second)** so the output video plays back at the same real-world speed as the input. Without matching `w`, `h`, and `fps`, frames fail to write or play back too fast/slow.

### 6. Why might some videos not produce good detections?
Several real-world factors can degrade detection quality:
- **Small or occluded objects:** People or items partially hidden behind counters, shelves, or other people have fewer visible features.
- **Domain mismatch:** A model pretrained on COCO (general scenes) may miss specialized objects or unusual top-down CCTV angles without fine-tuning.
- **Extreme lighting or motion blur:** Dark areas, harsh backlighting, or fast-moving subjects reduce contrast and edge sharpness.
- **Class or threshold settings:** If `classes` excludes the objects in the scene or `conf` is set too high, valid detections get filtered out.