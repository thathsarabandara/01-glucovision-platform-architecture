# Data Collection Strategy: Sri Lankan Cuisines for GlucoVision

## 1. Introduction and Objectives
The accurate recognition and portion estimation of Sri Lankan food are critical components of the GlucoVision platform. Sri Lankan cuisine is characterized by its high complexity, mixed textures, and non-standardized plating, making it challenging for conventional 2D-image-based models. 

The objective of this strategy is to establish a comprehensive, multi-modal data collection pipeline that gathers high-fidelity data to train AI models (like ResNet-50/ViT for classification and Mask R-CNN for segmentation) for precise nutritional and glycemic impact analysis.

## 2. Multi-Modal Data Acquisition

### 2.1 RGB-D Depth Data Collection
To estimate the geometric volume of food accurately, standard 2D color images are insufficient. We will utilize RGB-D (Red, Green, Blue + Depth) sensors to capture spatial information.
*   **Hardware Setup**: Utilization of devices such as Intel RealSense depth cameras and smartphone LiDAR (e.g., iPhone Pro models).
*   **Data Pairs**: Every capture will include a high-resolution 2D RGB image and a corresponding 3D depth map.
*   **Angles & Perspectives**: Data will be captured from multiple angles (top-down, 45-degree, and side profiles) to build a robust 3D profile of the food items.

### 2.2 Virtual Reality (VR) Capabilities
To overcome the logistical challenges of collecting massive real-world datasets, we will leverage VR environments to simulate dining scenarios.
*   **Immersive Simulations**: Creating a VR environment where users interact with virtual representations of Sri Lankan meals.
*   **Behavioral Data**: Capturing telemetry on how individuals plate food, their serving sizes, and interaction habits.
*   **Synthetic Data Generation**: Using VR engines (like Unity/Unreal Engine) to generate photo-realistic synthetic datasets of food under various lighting conditions, plate types, and arrangements to augment the real-world dataset.

## 3. Targeted Sri Lankan Cuisine Challenges

Sri Lankan food presents unique challenges that the data collection protocol must specifically address:
*   **Mixed Plating (Rice and Curry)**: Sri Lankan meals typically consist of a base (rice) mixed with several curries. The data collection must capture individual components before mixing, as well as the final mixed plate.
*   **Amorphous Textures**: Curries and gravies do not have fixed geometric shapes. Depth data is crucial here to capture the curvature of the liquid/semi-solid surfaces within the serving vessel.
*   **Variable Portion Sizes**: Capturing the same meal type in small, medium, and large portions to train the model on volume-to-calorie mapping.

## 4. Data Collection Protocol

### Phase 1: Controlled Environment (Lab-Setting)
*   **Standardization**: Food captured under uniform lighting (5000K-6500K daylight equivalent), on standardized plates/bowls, and at fixed distances from the camera.
*   **Ground Truth Measurement**: Every food item is weighed using a highly precise digital scale before and after plating to establish ground truth mass and volume.

### Phase 2: "In-The-Wild" Collection (User-Generated)
*   **Crowdsourcing via GlucoVision Data Collect App**: Deploying the mobile app to a beta group of users to capture real-world meals in diverse lighting and backgrounds.
*   **Fiducial Markers**: Users will be asked to place a standardized object (like a coin or credit card) next to the plate to provide a reference scale for the AI models when depth sensors are unavailable.

## 5. Annotation and Labeling Pipeline
The collected raw data will undergo a rigorous annotation process:
1.  **Bounding Boxes & Segmentation**: Drawing precise polygon masks around individual food items on a plate (e.g., separating the Dhal curry from the Rice).
2.  **Depth Map Alignment**: Ensuring the RGB image perfectly overlays with the Depth map.
3.  **Nutritional Tagging**: Expert dietitians will tag each image with metadata including ingredient breakdown, macronutrients (Carbs, Proteins, Fats), and Glycemic Index (GI).

## 6. Data Privacy and Ethics
*   **Anonymization**: All user-generated images will be scrubbed of metadata that could identify the location or the individual.
*   **Consent**: Explicit opt-in consent will be acquired from all participants contributing to the "in-the-wild" dataset.
*   **Federated Learning Integration**: In future phases, data collected on user devices will not be centrally uploaded; instead, the AI models will be trained locally on the device to preserve privacy.

## 7. Storage and Management
*   **Architecture**: Utilizing scalable cloud object storage (e.g., AWS S3 or GCP Cloud Storage) for raw image and depth files.
*   **Metadata DB**: A NoSQL database (e.g., MongoDB) to store the complex JSON structures associated with the image metadata, annotations, and nutritional values.
