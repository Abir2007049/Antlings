# 🚁 Aerial/Drone Computer Vision Pipeline: Detection, Tracking & Counting

![YOLOv8](https://img.shields.io/badge/YOLOv8-Medium-blue) ![Tracking](https://img.shields.io/badge/Tracker-ByteTrack-success) ![Evaluation](https://img.shields.io/badge/mAP50-0.57-orange)

A complete computer vision system built to find, track, and count people and vehicles in drone videos. This repository holds everything I did for the Antlings AI/ML Technical Assessment, including data preparation, model training, tracking code, and my final results.

---

## 📋 Task-01: Dataset Understanding & Preprocessing

**Dataset Used:** [VisDrone2019-DET](https://www.kaggle.com/datasets/banuprasadb/visdrone-dataset?resource=download)

**Dataset Structure & Prepping the Data:**
The VisDrone dataset has high-quality, crowded street photos taken from the sky. The data was already formatted with YOLO's required number system, but I had to fix the folder structures (changing the annotation folders to `labels` so the YOLO training script could find them properly).

**The Challenge & Upgrading to a 4-Class Model:**
The assignment asked to detect "Humans" and "Cars". But during my initial checks, I noticed a problem: from a drone's top-down view, big cars, trucks, and buses look almost exactly the same. If I only trained the model on 2 classes, it would get confused and label buses as "cars."

To fix this, I built a **4-Class system**. I wrote a Python script to clean the raw data and organize it into four specific groups:
* `Class 0`: Human (Combined from pedestrians and people)
* `Class 1`: Car
* `Class 2`: Truck
* `Class 3`: Bus

Doing this helped the model learn the actual size differences between vehicles, which made the bounding boxes much more accurate. I also programmed the script to drop any images that were completely empty so the model wouldn't get confused by blank backgrounds.

---

## 🧠 Task-02: Model Training Architecture

**Framework:** YOLOv8 (Ultralytics)
**Base Model:** `yolov8m.pt` (Medium)

**Training Approach & Why I Chose It:**
Drone pictures have very tiny objects. The smaller YOLO models (like Nano or Small) struggle to see those tiny details clearly. I chose the **YOLOv8 Medium** model because it is deep enough to find small targets but still lightweight enough to run super fast in real-time.

**Hyperparameters:**
* **Epochs:** 25 (Fine-tuning phase)
* **Image Size:** 640x640 (To keep the small objects from getting too blurry)
* **Batch Size:** 16
* **Hardware:** NVIDIA Tesla T4 GPU

---

## 🎯 Task-03 & Task-04 (Bonus): Detection, Tracking & Counting

**Implemented Tracker:** `ByteTrack`

Instead of just counting boxes in every single frame (which creates a huge error by counting the same car over and over again), I used **ByteTrack** to do true object tracking and hit the bonus criteria.

**How the logic works:**
1. **Finding Objects:** YOLOv8 draws boxes around the people and vehicles it sees in the current video frame.
2. **Tracking:** ByteTrack remembers these boxes as they move across the screen and gives each one a permanent, unique ID number (like `Car #14`).
3. **Counting:** The system just counts the total number of unique IDs it generates. This gives a highly accurate, cumulative count of how many actual cars and humans passed through the video.

---

## 📊 Task-05: Evaluation, Visualization & Analysis

### 📊 Final Evaluation Metrics (Validation Split)

The customized YOLOv8 Medium model was formally evaluated on the processed validation dataset. Drone imagery is notoriously difficult due to extreme altitudes and small target footprints, making these baseline results highly competitive:

| Metric | System Output | Engineering Interpretation |
| :--- | :---: | :--- |
| **mAP50 (All Classes)** | **0.5735** | Strong overall baseline accuracy for high-altitude, multi-class object detection. |
| **mAP50-95** | **0.3632** | Reflects strict bounding-box localization accuracy across variable IoU thresholds. |
| **Precision (P)** | **0.7032** | High precision: when the model detects an object, it is correct over 70% of the time. |
| **Recall (R)** | **0.5271** | The model successfully localized over half of all valid, heavily-obscured targets. |
| **Inference Speed** | **16.5 ms** | Operating at **~60.6 FPS**, the pipeline exceeds real-time constraints (double standard 30 FPS feeds). |

**Class-Specific Highlights:** The architecture performed exceptionally well on rigid vehicle geometries, achieving an **80.1% mAP50 specifically for the `Car` class**.

### Engineering Analysis
* **Strengths:** * **Vehicle Detection:** The model is fantastic at finding cars, scoring an 80.1% mAP50. 
  * **Real-Time Speed:** Running at over 60 FPS on a basic T4 GPU proves this code is fast enough to be used on live drone cameras in the real world.
* **Limitations:** * **Finding Small Humans:** While the model is accurate when it finds a person (70% precision), it misses a good amount of them (46.5% recall). From high up, single pedestrians are just tiny blur marks and get easily hidden by shadows or trees.
* **Challenges Faced:** * While testing on extremely crowded images (like scenes with 150+ cars at once), the code experienced a slight slowdown. This happens during a step called Non-Maximum Suppression (NMS), where the computer has to delete overlapping duplicate boxes. For a real-world product, I would fix this by splitting the image into smaller tiles or using a tool like TensorRT to optimize the math.

---

## 📁 Repository Structure & Deliverables

Here is how the project files are organized to make it easy to review my code, models, and final results:

```text
Antlings_Drone_Detection/
│
├── README.md                  # This document explaining the project workflow
├── requirements.txt           # List of Python libraries used (Ultralytics, OpenCV, etc.)
│
├── src/                       
│   └── drone_pipeline.ipynb   # The main Google Colab notebook containing all my Python code for cleaning data, training, and tracking
│
├── models/                    
│   └── best.pt                # The final trained YOLOv8 Medium weights (The "brain" of the model)
│
└── outputs/                   
    ├── metrics/               # Evaluation charts (Precision-Recall curve, Confusion Matrix)
    ├── sample_predictions/    # Example images showing the model working flawlessly on highly crowded test photos
    └── tracking_video/        # My final 3-minute demo video showing real-time tracking on a Chittagong flyover

---
