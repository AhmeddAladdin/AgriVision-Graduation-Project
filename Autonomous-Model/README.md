# AgriVision Autonomous Robot Navigation System

## Project Overview

This repository contains the current autonomous navigation stack for the AgriVision robot. The robot uses a camera-only perception pipeline to detect obstacles, estimate relative depth, segment the scene, and generate local navigation commands.

The current system is a prototype navigation pipeline. It is designed to help the AI, embedded, and control teams discuss the architecture, test model integration, and prepare for future field deployment.

## Current Goal

The current goal is not the final agricultural navigation model. The goal is to build a working prototype pipeline for:

- Camera input
- Obstacle detection
- Monocular depth estimation
- Semantic segmentation
- Basic local navigation planning
- Future ESP32 motor control

This prototype proves that the perception and planning components can run together before the team commits to a field-specific agricultural navigation model.

## High-Level Architecture

```text
Camera -> YOLOv8 -> Depth Anything V2 -> SegFormer-B0 -> Navigation Planner -> Speed/Steering Command -> ESP32 -> Motors
```

### Layers

**Perception**

- Reads frames from a single RGB camera.
- Detects possible obstacles using YOLOv8n.
- Estimates relative monocular depth using Depth Anything V2 Small.
- Segments the scene using SegFormer-B0.
- Estimates free-space direction from the lower part of the segmentation map.

**Planning**

- Selects the main obstacle from detected objects.
- Combines obstacle position, obstacle depth, and free-space direction.
- Produces a local navigation command with speed, steering, and reason.

**Communication**

- Planned future layer for sending speed and steering commands to an ESP32 over serial UART.
- `communication/esp32.py` exists as a placeholder, but no serial command protocol is implemented yet.

**Control**

- Planned future ESP32 motor-control layer.
- The ESP32 should eventually convert speed and steering commands into left/right motor PWM values.

## Project Structure

```text
Robot_AI/
|-- main.py
|-- config.py
|-- yolov8n.pt
|-- communication/
|   `-- esp32.py
|-- models/
|   |-- yolo.py
|   |-- depth.py
|   `-- segformer.py
|-- navigation/
|   |-- decision.py
|   `-- planner.py
|-- utils/
|   `-- camera.py
|-- weights/
|   `-- depth_anything_v2_small.pth
`-- venv/
```

### Files and Folders

| Path | Purpose |
| --- | --- |
| `main.py` | Main runtime pipeline. Opens the camera, runs YOLO, depth estimation, SegFormer segmentation, navigation planning, and OpenCV visualization windows. |
| `config.py` | Present but currently empty. Intended for future shared configuration values. |
| `models/yolo.py` | Defines `ObstacleDetector`, a YOLOv8n wrapper that returns detected objects with class name, confidence, bounding box, center point, area, and LEFT/CENTER/RIGHT position. |
| `models/depth.py` | Defines `DepthEstimator`, which loads `depth-anything/Depth-Anything-V2-Small-hf` through Hugging Face Transformers and returns a relative depth map plus a visualization image. |
| `models/segformer.py` | Defines `PathSegmenter`, which loads `nvidia/segformer-b0-finetuned-ade-512-512`, produces a semantic segmentation map, and estimates free-space direction from the bottom ROI. |
| `navigation/decision.py` | Filters YOLO detections into obstacle classes, selects the main obstacle by largest area, and contains an older simple direction decision helper. |
| `navigation/planner.py` | Defines `NavigationPlanner`, the current planner that outputs speed, steering, and reason instead of only LEFT/RIGHT/FORWARD commands. |
| `utils/camera.py` | Defines the OpenCV camera wrapper. Opens `camera_index=0`, sets 640x480 capture size by default, reads frames, and releases the camera. |
| `communication/esp32.py` | Present but empty. Reserved for future ESP32 serial communication. |
| `weights/` | Contains `depth_anything_v2_small.pth`. The current `DepthEstimator` does not load this file directly; it loads the Hugging Face model by name. |
| `yolov8n.pt` | Local YOLOv8n model weight file used by `ObstacleDetector`. |
| `venv/` | Local Python virtual environment. This should not be treated as project source code. |

No `test_*.py` files are currently present in the project folder. If test files such as `test_yolov8.py`, `test_depth.py`, or `test_segformer.py` are added later, they should be documented here and kept runnable from the project root.

## Models Used

### A. YOLOv8n

YOLOv8n is used for object and obstacle detection. The current code loads the local `yolov8n.pt` file through Ultralytics:

```python
detector = ObstacleDetector(
    model_path="yolov8n.pt",
    image_size=320,
    confidence=0.4
)
```

The detector currently uses pretrained COCO-style general object classes. It returns:

- Bounding box: `(x1, y1, x2, y2)`
- Class name
- Confidence score
- Object center point
- Bounding-box width and height
- Bounding-box area
- Horizontal position: `LEFT`, `CENTER`, or `RIGHT`

YOLOv8n was chosen instead of YOLO11n because the AgriVision AI pipeline already uses YOLOv8. Keeping the same model family simplifies integration, debugging, and team knowledge transfer.

Limitation: the current YOLOv8n weights are general-purpose and not agriculture-specific. The model may detect people, vehicles, animals, bags, bottles, plants, and other COCO-style classes, but it is not trained specifically for crop rows, farm paths, irrigation pipes, field tools, or agricultural obstacles.

### B. Depth Anything V2 Small

Depth Anything V2 Small is used for monocular depth estimation from one RGB camera. The current code loads:

```text
depth-anything/Depth-Anything-V2-Small-hf
```

The model returns relative depth, not accurate metric distance in meters. The values should be interpreted as a relative closeness signal inside the current camera view, not as calibrated physical distance.

In local tests for this project, larger depth values appear to mean closer objects. The planner currently follows that assumption:

```python
obstacle_is_close = depth > self.close_depth_threshold
```

Depth is attached to YOLO detections by taking the median depth value inside each YOLO bounding box:

```python
roi = depth_map[y1:y2, x1:x2]
return float(np.median(roi))
```

Limitation: this is not a replacement for LiDAR, ToF, stereo depth, or other active depth sensors. It needs calibration and real-environment testing with the final camera placement, lighting conditions, lens, robot height, and expected farm obstacles.

### C. SegFormer-B0

SegFormer-B0 is used for semantic segmentation. The current code loads:

```text
nvidia/segformer-b0-finetuned-ade-512-512
```

This model is pretrained/fine-tuned for ADE20K general-scene segmentation. In this project, it is currently used to estimate a coarse free-space direction:

- `LEFT`
- `CENTER`
- `RIGHT`

The current implementation analyzes the bottom region of the segmentation map and counts labels that are treated as walkable/free-space classes:

```python
walkable_labels = {
    3,   # floor
    6,   # road
    13,  # earth/ground
    29,  # field
    46,  # sand
    91,  # dirt track / path
}
```

Limitation: SegFormer-B0 with ADE20K is a temporary placeholder, not the final agricultural navigation model. ADE20K is not a crop-row dataset and is not specialized for agricultural field navigation. The final system should use either a selected agricultural navigation model or a custom model trained/fine-tuned on field images from the real robot environment.

## Data Sources / Pretraining Sources

The current navigation models were not trained by this project team.

- YOLOv8n uses pretrained general object-detection weights with COCO-style classes.
- SegFormer-B0 uses the `nvidia/segformer-b0-finetuned-ade-512-512` checkpoint, fine-tuned on ADE20K-style semantic segmentation.
- Depth Anything V2 Small was pretrained by its authors and is loaded through Hugging Face Transformers.

No custom agricultural navigation dataset has been collected yet for this repository.

Future dataset plan:

- Record field images and videos from the real robot camera.
- Capture different lighting conditions, crop types, soil colors, path shapes, irrigation layouts, and obstacle types.
- Label crop rows, traversable paths, plants, soil, humans, tools, animals, and unsafe regions.
- Use the dataset to evaluate whether the team should fine-tune an existing segmentation/navigation model or train a smaller field-specific model.

## Current Pipeline

1. The camera reads a frame using OpenCV.
2. YOLOv8n detects objects in the frame.
3. Depth Anything V2 Small generates a relative depth map.
4. Each YOLO object gets a median depth value from its bounding-box region.
5. SegFormer-B0 segments the scene.
6. Free space is estimated from the bottom ROI of the segmentation map.
7. The main obstacle is selected from filtered YOLO detections.
8. `NavigationPlanner` generates a command object:

```python
{
    "speed": 0-100,
    "steering": -100 to 100,
    "reason": "string"
}
```

9. The frame, depth view, and segmentation view are displayed with OpenCV.
10. The command is printed to the terminal for debugging.

## Navigation Planner Logic

The current planner uses speed and steering instead of simple LEFT/RIGHT/FORWARD commands.

Command meaning:

- `speed`: percentage-like command from `0` to `100`
- `steering`: steering command from `-100` to `100`
- Negative steering means left
- Positive steering means right
- `reason`: human-readable explanation for debugging

Current threshold:

```python
NavigationPlanner(close_depth_threshold=160)
```

Current behavior:

- If there is no main obstacle, follow the best free-space direction.
- If an obstacle is detected but depth is unknown, stop for safety.
- If an obstacle is close and in the center, avoid toward the best free-space direction.
- If an obstacle is close on the left, steer right.
- If an obstacle is close on the right, steer left.
- If the obstacle is not close, continue following free space.

Current command examples:

```python
{"speed": 75, "steering": 0, "reason": "Path clear ahead"}
{"speed": 65, "steering": -20, "reason": "Following free space on left"}
{"speed": 65, "steering": 20, "reason": "Following free space on right"}
{"speed": 35, "steering": -55, "reason": "Center obstacle close, avoiding left, depth=..."}
{"speed": 35, "steering": 55, "reason": "Center obstacle close, avoiding right, depth=..."}
{"speed": 0, "steering": 0, "reason": "Obstacle detected but depth unknown"}
```

## Current Status

### Completed

- Virtual environment fixed on Python 3.10.
- YOLOv8n integration is present.
- Depth Anything V2 Small integration is present.
- SegFormer-B0 integration is present.
- Full camera-to-planner pipeline is integrated in `main.py`.
- Planner outputs speed and steering commands.
- OpenCV visualization windows are available for RGB frame, depth view, and SegFormer view.

### Pending

- FPS optimization.
- Model scheduling.
- ROI optimization.
- Stronger safety fallback logic.
- ESP32 serial communication.
- Motor control code.
- Raspberry Pi deployment.
- Real field testing.
- Agricultural-specific segmentation/navigation model.
- Custom field dataset collection.
- Automated tests for detector, depth estimator, segmenter, and planner.

## Installation

Use Python 3.10 on Windows.

```powershell
python -m venv venv
venv\Scripts\activate
python -m pip install --upgrade pip
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install ultralytics opencv-python transformers accelerate safetensors pillow matplotlib pyserial
```

Notes:

- The commands above install the CPU PyTorch build.
- GPU installation depends on the CUDA version installed on the machine and should be handled separately if needed.
- The first run may download Hugging Face model files if they are not already cached.

## Running

Run the full prototype pipeline from the project root:

```powershell
python main.py
```

Press `q` in the OpenCV display window to quit.

Expected windows:

- `AgriVision Navigation - Full Pipeline`
- `Depth View`
- `SegFormer View`

If test files are added later, they should be runnable like this:

```powershell
python test_yolov8.py
python test_depth.py
python test_segformer.py
```

At the time this README was written, these test files are not present in the project folder.

## Calibration Notes

Depth Anything V2 provides relative depth, not exact meters. The current threshold must be calibrated with the real robot camera and real field environment.

Recommended calibration steps:

1. Mount the camera in the expected robot position.
2. Test known close and far objects in front of the robot.
3. Log median depth values from YOLO bounding boxes.
4. Compare values under different lighting conditions and camera angles.
5. Update `close_depth_threshold` in `main.py`:

```python
planner = NavigationPlanner(close_depth_threshold=160)
```

The correct threshold may change with camera height, lens, exposure, lighting, crop density, soil color, and model preprocessing.

## Performance Notes

Running YOLOv8n, Depth Anything V2, and SegFormer-B0 every frame may be too heavy for Raspberry Pi deployment.

Future optimization plan:

- Run YOLO every frame.
- Run depth estimation every 3 frames.
- Run SegFormer every 5 frames.
- Reuse the last valid depth and segmentation outputs between model runs.
- Reduce input image size.
- Crop the processing area to the navigation ROI.
- Profile CPU, RAM, and temperature on the target Raspberry Pi.
- Consider ONNX, TensorRT, OpenVINO, or TFLite later depending on deployment hardware.

The project should not claim FPS numbers until they are measured on the actual target hardware.

## Safety Notes

This is a prototype and should not be treated as production-safe robot navigation.

Safety rules for future implementation:

- If the camera fails, stop.
- If depth is `None`, stop.
- If planner confidence is low, stop.
- Never send movement commands without valid perception.
- Add timeout handling so stale perception does not keep the robot moving.
- Add command limits for maximum speed and maximum steering change.
- Add a manual emergency stop before field testing.

Camera-only navigation is risky in real farms compared to systems that include ToF, LiDAR, stereo, ultrasonic sensors, bump sensors, or wheel odometry. This project intentionally uses camera-only perception for the current prototype, so field testing must be conservative.

## Future Improvements

- Implement ESP32 UART communication in `communication/esp32.py`.
- Define a serial command protocol for speed, steering, heartbeat, and emergency stop.
- Convert speed/steering to left/right motor PWM on the ESP32.
- Deploy and benchmark on Raspberry Pi.
- Add model scheduling to improve FPS.
- Add agricultural crop-row segmentation or path-following model.
- Collect a custom field dataset from the robot camera.
- Fine-tune segmentation and obstacle models on agricultural data.
- Export suitable models to ONNX.
- Evaluate TFLite, OpenVINO, or TensorRT depending on hardware.
- Add ROS2 integration if the robot architecture grows beyond a simple prototype.
- Add sensor fusion in the future if camera-only constraints are relaxed.
- Integrate existing AgriVision disease, insect, and growth models only when the robot stops in front of a plant. Navigation should stay lightweight while the robot is moving.

## References

- [Ultralytics YOLOv8 documentation](https://docs.ultralytics.com/models/yolov8/)
- [Depth Anything V2 Small - Hugging Face](https://huggingface.co/depth-anything/Depth-Anything-V2-Small-hf)
- [Depth Anything V2 - GitHub](https://github.com/DepthAnything/Depth-Anything-V2)
- [NVIDIA SegFormer-B0 ADE checkpoint - Hugging Face](https://huggingface.co/nvidia/segformer-b0-finetuned-ade-512-512)
- [ADE20K dataset](https://ade20k.csail.mit.edu/)
- [COCO dataset](https://cocodataset.org/)
- [PyTorch documentation](https://docs.pytorch.org/docs/stable/index.html)
- [OpenCV documentation](https://docs.opencv.org/4.x/)
- [Hugging Face Transformers documentation](https://huggingface.co/docs/transformers/en/index)
