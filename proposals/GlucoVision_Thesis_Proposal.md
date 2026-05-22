# Faculty of Computing, University of Sri Jayewardenepura
## Undergraduate Final Year Project - Proposal

**Title**: GlucoVision: An AI-Powered Personalized Diabetes Management and Recommendation Platform for Sri Lankan Patients

---

### Candidate Details
**Student Full Name**: Thathsara Bandara
**Registration No.**: [Insert Reg. No.]
**Examination No.**: [Insert Exam No.]

### Supervisor Details
**Supervisor (1) Name**: [Insert Name]
**Department**: [Insert Department]
**Supervisor (2) Name**: [Insert Name]
**Department**: [Insert Department]

---

## 1. Introduction
Diabetes is a growing health concern globally, and effectively managing it requires continuous monitoring of blood glucose levels, diet, and physical activity. Existing mobile health applications often lack localization for specific diets—such as Sri Lankan cuisines—and fail to provide holistic, real-time, and personalized guidance. GlucoVision is a comprehensive 21-microservice platform designed to address these gaps. By integrating advanced Artificial Intelligence (AI), Internet of Things (IoT) wearables, and Continuous Glucose Monitoring (CGM) systems, GlucoVision offers a Personalized Diet and Activity Recommendation (PDAR) system. The platform employs cutting-edge technologies, including federated learning for privacy-preserving model training and neural ODE-based digital twins for predictive health simulations.

## 2. Problem Statement
The management of diabetes heavily relies on accurate dietary logging, continuous glucose monitoring, and tailored lifestyle interventions. However, current systems present several limitations:
1. **Lack of Localized Models**: Existing food recognition systems perform poorly on complex Sri Lankan cuisines.
2. **Inaccurate Logging**: Manual portion size estimation is error-prone, leading to incorrect macronutrient calculation.
3. **Reactive Interventions**: Alerts for hypoglycemia and hyperglycemia are often reactive rather than proactively predicted.
4. **Generalized Recommendations**: Dietary advice often does not account for individual metabolic responses, cultural preferences, and real-time contextual factors.
5. **Data Privacy**: Centralized aggregation of sensitive medical and continuous glucose data raises significant privacy concerns.
Consequently, patients struggle with adherence to health guidelines, and clinicians lack the integrated, real-time data required for effective intervention.

## 3. Aim and Objectives

**Aim**: 
To design and develop an AI-powered personalized diabetes management platform featuring localized food recognition, continuous glucose prediction, and privacy-preserving digital health simulations.

**Objectives**:
1. To develop a localized food recognition and portion estimation module capable of classifying Sri Lankan cuisines (ResNet-50/ViT) and estimating geometric volume using RGB-D depth data.
2. To design a medically-safe glucose prediction and risk alert engine that forecasts blood glucose levels (Bi-LSTM/Transformer) and triggers real-time alerts based on Continuous Glucose Monitor (CGM) data.
3. To implement a context-aware recommendation engine that synthesizes nutritional data, energy expenditure, and continuous glucose trends to generate personalized meal and activity plans.
4. To engineer a Neural ODE-based digital twin simulation model for predicting a patient's physiological state over 24-72 hours under different lifestyle scenarios.
5. To integrate a federated learning framework ensuring the privacy-preserving training of machine learning models across decentralized nodes without centralizing sensitive patient data.

## 4. Methods
The project adopts a robust microservices architecture comprising 21 independent repositories, categorized into domains to ensure separation of concerns, especially isolating medically-critical components:
- **Food AI Layer**: Utilizes ResNet-50 / ViT-B16 for 2D food classification (fine-tuned on a custom Sri Lankan dataset) and Mask R-CNN with RGB-D depth inference for 3D portion volume estimation.
- **Health AI Layer**: Implements Bi-LSTM and Time-Series Transformer models for continuous glucose forecasting. A real-time risk alert engine evaluates glucose data streams via Kafka/WebSocket from Dexcom/Libre CGM devices.
- **Intelligence Layer**: Employs a reinforcement learning-based personalization engine and a LLaMA-3-based LLM for conversational dietary advisory, factoring in macronutrients, Glycemic Index (GI), and Base Metabolic Rate (BMR).
- **IoT & Edge**: Connects ESP32-CAM (via WebRTC) for real-time cooking state analysis using 3D-CNNs and wearable smart bands (BLE GATT) for activity tracking.
- **Research Models**: Implements Neural Ordinary Differential Equations (ODE) to model the physiological digital twin of the patient, and the Flower framework for federated learning with differential privacy (ε-DP noise).
- **Infrastructure**: Deployed using Docker, orchestrated via Kubernetes, and managed through Terraform for infrastructure-as-code, ensuring high availability and secure isolation.
- **Data Collection Strategy**: Employs a multi-modal approach for capturing Sri Lankan dietary data. RGB-D cameras (e.g., Intel RealSense, smartphone LiDAR) are utilized to gather paired 2D color images and 3D depth maps, establishing a robust dataset for precise volume and portion estimation. Furthermore, Virtual Reality (VR) capabilities will be leveraged to simulate immersive dining and cooking scenarios. This VR-based approach enables the collection of rich, annotated behavioral and interaction data—such as plating habits and portion selection—for training context-aware AI models in a controlled, scalable, and safe environment.

## 5. Research Contribution
GlucoVision contributes to the field of health informatics by providing:
- A novel Sri Lankan food recognition and portion estimation pipeline combining 2D classification and 3D depth geometry.
- A secure, decentralized approach to training diabetes prediction models utilizing federated learning to preserve patient data privacy.
- The introduction of a physiological digital twin for diabetes management, allowing for predictive "what-if" scenario simulations before real-world interventions.
- A highly scalable 21-service microservice architecture that logically separates medically critical components (like the glucose prediction engine) from general features for independent safety lifecycle management.

## 6. Project Timeline

| No. | Task | Duration (Weeks) | Start Date | End Date |
|---|---|---|---|---|
| 1. | Phase 1: Foundation (Architecture & Core Services) | 8 Weeks | 2026-05-01 | 2026-06-30 |
| 2. | Phase 2: Food AI (Recognition & Portion Estimation) | 8 Weeks | 2026-07-01 | 2026-08-31 |
| 3. | Phase 3: Health Monitoring (CGM & Prediction Engine) | 8 Weeks | 2026-09-01 | 2026-10-31 |
| 4. | Phase 4: Intelligence (Recommendation & Assistant) | 8 Weeks | 2026-11-01 | 2026-12-31 |
| 5. | Phase 5: Research Grade (Digital Twin & Federated Learning) | 16 Weeks | 2027-01-01 | 2027-04-30 |
| 6. | Testing, Refinement & Final Thesis Documentation | 4 Weeks | 2027-05-01 | 2027-05-31 |

## 7. References
[1] T. Bandara et al., "Systematic Review of AI-Based Personalized Diet and Activity Recommendation System for Diabetic Individuals," *Journal of Health Informatics*, vol. 12, no. 3, pp. 45-60, 2025.
[2] K. He, X. Zhang, S. Ren, and J. Sun, "Deep Residual Learning for Image Recognition," in *Proceedings of the IEEE conference on computer vision and pattern recognition*, 2016, pp. 770-778.
[3] R. T. Q. Chen, Y. Rubanova, J. Bettencourt, and D. K. Duvenaud, "Neural ordinary differential equations," *Advances in neural information processing systems*, vol. 31, 2018.
[4] B. McMahan, E. Moore, D. Ramage, S. Hampson, and B. A. y Arcas, "Communication-efficient learning of deep networks from decentralized data," in *Artificial intelligence and statistics*, PMLR, 2017, pp. 1273-1282.

---
### Certification Section
I hereby certify to the best of my knowledge that the details mentioned above are true and accurate.

___________________________  
**Signature of Candidate**  
**Date**: 

I/We hereby certify to the best of my/our knowledge that the details mentioned above are true and accurate.

___________________________  
**Signature of Supervisor(s)**  
**Date**:
