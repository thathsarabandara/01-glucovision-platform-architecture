# AI-Based Personalized Diet & Activity Recommendation System for Diabetic Individuals

## Slide 1: Title Slide
**AI-Powered Solutions for Diabetes Management in Sri Lanka**
- A Systematic Review of Emerging Technologies
- Research Period: 2014-2025 | 38 Studies Analyzed

---

## Slide 2: The Problem
**Why This Matters?**
- 3.6% of Sri Lankans (18-29 years) have diabetes
- 23% of adults (18+) affected
- 27% in Western Province (age 20+)
- 55% prevalence in ages 60-69

**Key Challenge:** Current systems don't account for culturally diverse eating habits and traditional Sri Lankan cuisine

---

## Slide 3: Research Scope
**Six Key Technology Domains Analyzed**
1. 🍽️ Food Recognition & Classification
2. 📏 Portion Estimation
3. ⌚ Wearable Sensor Monitoring
4. 📊 Glucose Prediction
5. 👤 Digital Twin Technology
6. 🔐 Federated Learning

---

## Slide 4: Food Recognition Systems
**Deep Learning & Computer Vision Advances**
- **State-of-the-Art Architectures:**
  - **ResNet-50 & EfficientNet:** High-precision feature extraction for diverse dish classification.
  - **Vision Transformers (ViT):** Leveraging self-attention to identify multi-component meals (e.g., Rice & Curry).
- **RGB-D Fusion Methods:** Mask R-CNN utilizing depth data for better object segmentation (94.1% detection accuracy).
- **Cross-Modal Learning:** Combining visual data with meta-information (ingredients, recipes) for "nutrient-aware" classification.
- **Critical Limitation:** Lack of large-scale annotated datasets for traditional South Asian and Sri Lankan dishes (e.g., Hopper, Pittu).
- **Project Goal:** Implementation of Domain Adaptation to bridge the gap between international and regional food datasets.

---

## Slide 5: Portion Estimation
**Transitioning from Manual to Automated AR-Based Methods**
- **The Problem:** Self-reported meal logging has an average error rate of >25% in clinical settings.
- **3D Reconstruction Techniques:**
  - **Single-View Depth Estimation:** Predicting volume from a 2D smartphone photo using CNNs.
  - **Multi-View Geometry:** Combining multiple angles to create a precise 3D mesh of the meal.
- **Augmented Reality (AR) Integration:** Virtual "portion guides" and reference object detection (e.g., using a credit card or coin for scale calibration).
- **Performance:** Modern RGB-D methods reduce average volume estimation error to 7–17.6%.
- **Impact:** Essential for accurate glycemic load calculation in Type 1 and Type 2 diabetes management.

---

## Slide 6: Wearable Sensors & Monitoring
**Continuous & Multi-Modal Biometric Tracking**
- **Key Biometrics Tracked:**
  - **PPG (Photoplethysmogram):** Continuous Heart Rate and HRV for stress monitoring.
  - **IMU (Inertial Measurement Units):** 6-axis motion tracking for physical activity intensity.
  - **EDA (Electrodermal Activity):** Measuring sympathetic nervous system activation.
- **Energy Expenditure Estimation:** Real-time metabolic rate calculation using activity and HR fusion.
- **Edge Intelligence:** On-device signal processing to filter noise and detect behavioral patterns (e.g., eating detection).
- **Design Philosophy:** Minimalistic, non-invasive form factors designed for high compliance in resource-limited South Asian settings.

---

## Slide 7: Glucose Prediction & Analytics
**Temporal Deep Learning for Predictive Care**
- **Data Source:** Continuous Glucose Monitoring (CGM) streams synced every 5 minutes.
- **Architecture:** 
  - **LSTM (Long Short-Term Memory):** Capturing long-term dependencies in metabolic data.
  - **Temporal Convolutional Networks (TCNs):** Parallel processing of multi-day glucose trends.
- **Prediction Horizons:** Early detection of hypoglycemic (<70 mg/dL) and hyperglycemic events 30, 60, and 120 minutes in advance.
- **Context-Aware Models:** Integrating exogenous factors (carbohydrate intake, insulin dosage, exercise) to improve prediction accuracy by >20% over baseline linear models.

---

## Slide 8: Digital Twin Technology
**Metabolic Modeling through Personalized Virtual Health Models**
- **Concept:** Creating a high-fidelity virtual representation of an individual's metabolic system.
- **Data Fusion:** Integrating Clinical Lab Results + CGM Data + Wearable Telemetry + Nutritional Logs.
- **"What-If" Simulation Engine:**
  - Safely forecasting blood glucose response to specific meals before they are consumed.
  - Simulating the impact of different insulin-to-carb ratios or physical activity timing.
- **Long-term Forecasting:** Predictive modeling of HbA1c changes and risk of diabetic complications over 6-12 month horizons.
- **Integration:** Directly feeding into the **Personalized Recommendation Engine (15)** for real-time lifestyle adjustments.

---

## Slide 9: Federated Learning
**Collaborative AI with Decentralized Privacy**
- **Mechanism:** Training AI models across multiple distributed nodes (hospitals, user devices) without ever sharing raw patient data.
- **Key Features:**
  - **Local Model Training:** Each user's device trains a model on their own private data.
  - **Secure Parameter Aggregation:** Only encrypted model weights are sent to a central server for global improvement.
- **Security Protocols:** 
  - **Differential Privacy:** Adding noise to parameters to prevent reverse-engineering of patient identities.
  - **SMPC (Secure Multi-Party Computation):** Securely computing global updates.
- **Healthcare Impact:** Enables the system to learn from rare clinical cases across multiple Sri Lankan hospitals while maintaining 100% GDPR and HIPAA compliance.

---

## Slide 10: Key Findings & Research Gaps
**What Works:**
✅ AI achieves human-level accuracy for food recognition
✅ Wearable sensors enable continuous monitoring
✅ Predictive models show promise for glucose control

**Critical Gaps:**
❌ Limited recognition of South Asian/Sri Lankan cuisine
❌ Most systems tested in controlled environments
❌ Cultural eating patterns underrepresented
❌ No integrated end-to-end system yet

---

## Slide 11: Future Recommendations
**For Sri Lanka & Similar Contexts**

1. **Build Cultural Datasets**
   - Sri Lankan food recognition models
   - Regional dietary patterns

2. **Accessible Technology**
   - Portable AR glasses vs. expensive equipment
   - Smartphone-based solutions

3. **Integrated Platform**
   - Combine all technologies in one solution
   - Privacy-first design (Federated Learning)

4. **Community-Centric Design**
   - Consider literacy levels
   - Align with local healthcare systems

---

## Slide 12: Conclusion
**The Future of Diabetes Management**

> *"A complete AI-powered system combining food recognition, portion estimation, wearable monitoring, glucose prediction, digital twins, and federated learning could revolutionize diabetes care."*

**Key Message:** 
- Technology is ready
- Next step: Culturally-adapted, integrated solutions for Sri Lanka
- **Vision:** Help millions manage diabetes confidently with personalized, culturally-smart AI

---

## Slide 13: Q&A
**Key Takeaways**
- AI technologies are advancing rapidly
- Significant gaps in cultural representation
- Privacy-preserving AI (federated learning) is crucial
- Integration & localization are next frontiers

📊 **38 studies analyzed** | 📈 **2014-2025 timeframe** | 🎯 **Sri Lanka focus**
