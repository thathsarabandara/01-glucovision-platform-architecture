# Detailed Technical Data Collection Strategy for GlucoVision

## 1. Hardware & Camera Specifications

### 1.1 Analysis of Current Setup: Samsung Galaxy M32
*   **Hardware Capabilities**: The Samsung Galaxy M32 features a 64MP primary sensor and a basic 2MP depth sensor.
*   **Limitations for Portion Estimation**: 
    *   **Lack of True 3D Depth**: While the M32 has a "depth" sensor, it is primarily used for portrait mode blur (bokeh) and is not designed to output highly accurate, metric 3D depth maps. Furthermore, web browsers (running your React/Vite app) cannot access this raw depth data via standard WebRTC/`getUserMedia` APIs.
    *   **Monocular Ambiguity**: A single 2D image from this phone cannot accurately calculate the volume of a mound of rice or a bowl of curry without a known reference scale.
    *   **Lighting Variability**: Smartphone cameras apply aggressive post-processing algorithms (HDR, sharpening) that can alter the natural color of food, tricking classification models.

### 1.2 Recommended Alternative Methods (For High Accuracy)

If you require **highly accurate** portion estimation and food recognition, you must transition to one of the following methods:

#### Method A: True RGB-D Sensors (The Gold Standard)
*   **Camera**: Intel RealSense D435i or D415.
*   **Why**: These output synchronized 2D RGB images and highly precise infrared stereo depth maps.
*   **Parameters**: Capture at 1080p (RGB) and 720p (Depth) at 30 FPS. Ensure alignment between the RGB and depth frames.

#### Method B: LiDAR-Equipped Smartphones (The Practical Upgrade)
*   **Camera**: iPhone 12 Pro / 13 Pro / 14 Pro / 15 Pro, or iPad Pro.
*   **Why**: Apple's LiDAR sensor projects a grid of infrared dots to generate a highly accurate 3D mesh of the environment. A custom iOS app (using ARKit) can export the raw depth map alongside the image.

#### Method C: Enhancing the Samsung M32 (If budget constraints exist)
If you *must* stick with the M32, you have to alter your collection protocol to compensate for the hardware:
1.  **Fiducial Markers (Mandatory)**: Every single photo MUST include a standard reference object next to the plate (e.g., a standard Sri Lankan 10 Rupee coin, or a credit card). The AI will use this known size to calculate pixels-to-centimeters.
2.  **Multi-Angle Shots**: Require the user to take *two* photos per meal: one top-down (90 degrees) and one at a 45-degree angle. This allows you to estimate height.

#### Method D: The AR + CNN Hybrid Approach (Highly Feasible & Scalable)
*   **Concept**: Use Augmented Reality (e.g., ARCore on Android, including your M32) to map the 3D space and establish scale, while simultaneously using a CNN (like Mask R-CNN or YOLOv8-Seg) on the 2D image feed to recognize and segment the food.
*   **How it Works**: The AR engine builds a physical mesh/plane of the table and plate. The CNN draws a 2D boundary (mask) around the food item (e.g., "Dhal Curry"). This 2D mask is then mathematically projected onto the AR 3D mesh, instantly calculating the exact real-world volume and height of that specific area without needing a physical fiducial marker or a dedicated LiDAR sensor.
*   **Feasibility**: **Extremely High and highly recommended.** This is often a much better and more scalable approach than relying on raw monocular depth estimation or forcing users to carry fiducial markers. It works on standard smartphones because ARCore uses the camera feed combined with the phone's internal gyroscope and accelerometer to build the 3D map via slight movement (parallax).
*   **Limitation/Requirement**: To implement this, you will likely need to transition your `02-glucovision-data-collect-app` from a pure web app (React/Vite) to a native or hybrid mobile app (e.g., React Native with AR integration or Flutter). Standard mobile web browsers (WebXR) currently lack the robust plane-tracking and API access required for high-fidelity AR volume calculation.

## 2. Tools for Analysis and Processing

To process this data for your AI models, you must use the following stack:

1.  **Data Annotation & Labeling**:
    *   **CVAT (Computer Vision Annotation Tool)**: Free, open-source, and allows for drawing precise polygon masks (crucial for amorphous food like curries).
    *   **Roboflow**: Excellent for managing datasets, applying augmentations (rotation, brightness changes), and exporting directly to PyTorch formats.
2.  **AI Models (Recognition & Segmentation)**:
    *   **YOLOv8 (Ultralytics)**: Use for initial bounding box detection of plates and food items.
    *   **Segment Anything Model (SAM) by Meta**: Use this to rapidly generate masks around food items during the annotation phase, saving hundreds of hours of manual clicking.
    *   **Mask R-CNN**: The standard architecture for instance segmentation (identifying *what* the food is and *where* its exact boundaries are).
3.  **Depth Estimation (If using Samsung M32)**:
    *   **Depth Anything v2 / MiDaS**: If you only have 2D images, you must run them through a Monocular Depth Estimation model to generate a "fake" depth map, which you then scale using your fiducial marker.

## 3. Analysis of Current `02-glucovision-data-collect-app`

Based on standard web-app data collection methods, your current Vite/React app likely has the following critical limitations:
*   **Loss of EXIF Data**: Web uploads often strip crucial metadata (focal length, ISO, exposure time) which can be useful for advanced computer vision.
*   **Browser API Limits**: The `getUserMedia` API often defaults to lower resolutions to save bandwidth. 
*   **Lack of Sensor Access**: The web app cannot access the phone's gyroscope/accelerometer to record the exact angle at which the photo was taken (e.g., confirming it's exactly 45 degrees).
*   **Solution**: You need to ensure the React app forces `ideal: 4096` resolution constraints on the video track and prompts the user with an on-screen guide (like a bounding box overlay) to align their phone correctly.

## 4. Data Quantity Requirements

Deep learning models are data-hungry. Because Sri Lankan food is highly complex (often mixed together on one plate), you need a substantial dataset.

*   **Classes**: Assume you are tracking the 30 most common Sri Lankan food items (Samba Rice, Keeri Samba, Dhal Curry, Chicken Curry, Pol Sambol, Gotukola Mallum, etc.).
*   **Images per Class**: To achieve >85% accuracy using Transfer Learning (starting with a pre-trained ResNet-50 or ViT model), you need a minimum of **300 to 500 diverse images per class**.
*   **Total Dataset Size**: 30 classes × 400 images = **12,000 Annotated Images**.
*   **Segmentation Masks**: Of those 12,000 images, at least **3,000 to 5,000** must have high-quality pixel-level polygon masks drawn around the food for the portion estimation model to learn volume.
*   **Variance**: Ensure the 400 images per class are not just taken in the same kitchen. You need different lighting (daylight, fluorescent, warm yellow light), different plates (white, patterned, banana leaves), and different cooking styles (dry curries vs. heavy gravy).

## 5. The Ultimate Protocol for Success

To achieve your thesis objectives for accurate recognition and portion estimation, you should adopt the **AR + CNN Hybrid Approach** as your primary strategy:

1.  **Migrate the App (Crucial Step)**: Transition `02-glucovision-data-collect-app` from a React/Vite web application to a native mobile framework (like React Native with AR libraries or Flutter). This unlocks the device's native AR capabilities (ARCore) and gyroscope sensors.
2.  **The New Capture Workflow**:
    *   The user opens the app and slowly moves the camera across the table to allow ARCore to detect the 3D plane and establish real-world scale.
    *   The user takes the photo of their meal.
    *   The app sends both the 2D high-resolution image and the AR spatial mesh data to your backend.
3.  **Backend Processing Synergy**: 
    *   Run the 2D image through your CNN (YOLOv8/Mask R-CNN) to perfectly segment the boundaries of the food items (e.g., "Dhal Curry").
    *   Project that 2D segmentation mask onto the AR spatial mesh to accurately calculate the physical volume (cm³) without needing any fiducial markers or coins.
4.  **Controlled Baseline Collection (Ground Truth)**: Before releasing the app to the public, conduct a controlled lab collection. Cook 30 standard meals and weigh every component on a digital scale (in grams) before taking the AR-assisted photo. This establishes the critical AI mapping model: `AR Volume (cm³) -> Mass (grams)` for each specific food type.
