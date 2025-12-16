#  Vision Assistant for the Visually Impaired

**Authors:** Victor Micha, Nawel Zait  
**Date:** December 2025  
**Course:** Computer Vision Project

---

##  Overview

This project implements a real-time assistance system for visually impaired people using computer vision and text-to-speech technologies. The system analyzes video streams and provides intelligent voice alerts about obstacles, text signs, and environment descriptions.

###  Key Features

- **Real-time object detection** - Identifies obstacles (people, vehicles, animals)
- **Text reading (OCR)** - Reads street signs, building numbers, license plates
- **Scene description** - Describes the environment context
- **Object classification** - Identifies main objects in the scene
- **Smart voice alerts** - Natural language audio feedback with priority system
- **Alert filtering** - Prevents repetitive/spam alerts

---

##  System Architecture

The system combines **4 AI modules** working together:

### Module 1: Object Detection (YOLO v8)
- Detects obstacles in real-time
- Estimates relative distance (close/far)
- Identifies position (left/ahead/right)
- Prioritizes critical obstacles (cars, people, bicycles)

### Module 2: Optical Character Recognition (EasyOCR)
- Reads text from signs and labels
- Supports French + English
- Filters duplicate/similar texts
- Identifies street names and important signs

### Module 3: Scene Description (BLIP)
- Generates natural language scene descriptions
- Provides environmental context
- Helps users understand their surroundings

### Module 4: Object Classification (ResNet-18)
- Classifies main objects using ImageNet
- Provides additional context about the scene
- 1000+ object categories

---

##  Installation

### Prerequisites
- Python 3.8+
- CUDA-capable GPU (optional, but recommended)
- Internet connection (for gTTS voice synthesis)

### Step 1: Clone the repository
```bash
git clone https://github.com/your-username/vision-assistant.git
cd vision-assistant
```

### Step 2: Install dependencies
```bash
pip install ultralytics opencv-python-headless pillow numpy matplotlib
pip install easyocr torch torchvision transformers
pip install gtts
```

### Step 3: Install Tesseract (for OCR)
```bash
# Ubuntu/Debian
sudo apt-get install -y tesseract-ocr tesseract-ocr-fra

# macOS
brew install tesseract tesseract-lang

# Windows
# Download from: https://github.com/UB-Mannheim/tesseract/wiki
```

---

## 💻 Usage

### Basic Usage - Process Video

```python
from vision_assistant import process_video_for_blind_assistance

# Process first 10 seconds of video with voice alerts
stats, assistant = process_video_for_blind_assistance(
    video_path='street_video.mp4',
    max_frames=300,  # ~10 seconds at 30fps
    language='en'
)
```

### Create Annotated Video with Audio

```python
from vision_assistant import create_final_demo_with_audio

# Creates video with annotations + audio alerts
final_video, stats = create_final_demo_with_audio(
    video_path='input.mp4',
    output_path='output_with_audio.mp4'
)
```

### Process Single Image

```python
from vision_assistant import ModelLoader, YOLOAnalyzer, CONFIG
import cv2

# Load models
models = ModelLoader()
analyzer = YOLOAnalyzer(models.yolo, CONFIG)

# Analyze image
frame = cv2.imread('image.jpg')
detections, alerts, annotated_image = analyzer.analyze_frame(frame)

# Print alerts
for alert in alerts:
    print(f"⚠️  {alert}")
```

---

##  Configuration

Edit `ProjectConfig` class to customize behavior:

```python
class ProjectConfig:
    # Model selection
    YOLO_MODEL = 'yolov8n.pt'  # Options: yolov8n, yolov8s, yolov8m, yolov8l
    
    # Detection thresholds
    YOLO_CONF_THRESHOLD = 0.5  # Object detection confidence
    OCR_CONF_THRESHOLD = 0.3   # Text reading confidence
    
    # Processing intervals (analyze every N frames)
    YOLO_INTERVAL = 5   # Every 5 frames
    OCR_INTERVAL = 30   # Every 30 frames (~1 sec at 30fps)
    BLIP_INTERVAL = 60  # Every 60 frames (~2 sec)
    
    # Voice alerts
    TTS_LANGUAGE = 'en'  # 'en' or 'fr'
    MIN_ALERT_INTERVAL = 5.0  # Seconds between duplicate alerts
```

---

## 📊 Example Output

```
============================================================
BLIND ASSISTANCE SYSTEM - Starting Analysis
============================================================
 Device: cuda

 TRAITEMENT VIDÉO: street_video.mp4
 FPS: 30 | Résolution: 1920x1080
 Frames totales: 900
  Durée: 30.0s

⏱ 0.0s |  YOLO: 5 objets
            car devant, Proche
            person à gauche, TRÈS PROCHE
⏱ 1.0s |  OCR: 2 textes
         "Avenue Bernard Palissy" (0.92)
⏱ 2.0s |   BLIP: a city street with cars and buildings
⏱ 2.5s |   Classe: street_sign (0.87)

 TRAITEMENT TERMINÉ
  Temps: 8.5s
 Vitesse: 105.9 fps

 Critical Alerts:
    WARNING! person very close on your left

 Warning Alerts:
    car ahead of you
    bicycle on your right

  Information:
    Street sign Avenue Bernard Palissy

 Environment:
    You are in a city street with cars and buildings

 Total alerts spoken: 5
```

---

## 🎬 Demo Video

![Demo GIF](demo.gif)

*Video demonstrating real-time obstacle detection and voice alerts*

---

## 📈 Performance

| Hardware | Processing Speed | Real-time Capable |
|----------|------------------|-------------------|
| NVIDIA RTX 3080 | ~100-120 FPS | ✅ Yes (4x real-time) |
| NVIDIA GTX 1660 | ~40-50 FPS | ✅ Yes (1.5x real-time) |
| CPU only (Intel i7) | ~5-8 FPS | ⚠️ Borderline |

**Note:** Real-time = processing faster than video playback (30 FPS)

---

## 🔬 Technical Details

### Distance Estimation
- Uses bounding box size as proxy for distance
- **Limitation:** Not metric (meters), only relative (close/far)
- Accuracy depends on object type and camera calibration
- Future improvement: Depth estimation with stereo camera

### Alert Priority System
1. **Critical** (red) - Very close obstacles requiring immediate attention
2. **Warning** (yellow) - Nearby obstacles to be aware of
3. **Information** (blue) - Text signs, street names
4. **Context** (green) - General scene description

### Smart Alert Filtering
- Deduplicates similar texts (85% similarity threshold)
- Prevents alert spam (5-second cooldown)
- Prioritizes important information (streets > general signs)
- Limits alerts per category (max 3 warnings)

---

##  Known Limitations

1. **Distance Accuracy**
   - Relative estimates only, not absolute distances in meters
   - Varies with object size and camera angle

2. **Performance**
   - 4 heavy models = high computational cost
   - Requires GPU for real-time processing
   - ~5-10 FPS on GPU, ~1-2 FPS on CPU

3. **OCR Reliability**
   - Best with clear, large, well-lit text
   - Degrades with motion blur or small text
   - Mixed language text can confuse model

4. **Internet Dependency**
   - gTTS requires internet connection
   - Offline alternative: pyttsx3 (lower voice quality)

5. **Environmental Constraints**
   - Optimized for outdoor/street scenes
   - Indoor performance may vary
   - Poor lighting affects all modules

---

##  Future Improvements

### Short-term
- [ ] Add offline TTS with pyttsx3
- [ ] Implement camera calibration for metric distances
- [ ] Add haptic feedback (vibration patterns)
- [ ] Support multiple languages (Spanish, German, etc.)

### Medium-term
- [ ] Optimize with TensorRT for faster inference
- [ ] Add depth estimation module
- [ ] Implement path planning and navigation
- [ ] Mobile app (iOS/Android)

### Long-term
- [ ] Integration with GPS for turn-by-turn navigation
- [ ] Obstacle avoidance suggestions
- [ ] Indoor navigation with AR
- [ ] Community-driven location database

---

##  Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Setup
```bash
# Clone your fork
git clone https://github.com/your-username/vision-assistant.git
cd vision-assistant

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dev dependencies
pip install -r requirements-dev.txt

# Run tests
pytest tests/
```
