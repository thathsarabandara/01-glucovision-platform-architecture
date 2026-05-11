# 12. Sri Lankan Food Recognition Dataset & AR Portion Estimation

## 📊 Executive Summary

This module integrates food recognition with AR-based portion estimation for Sri Lankan cuisine. The dataset focuses on common Sri Lankan dishes and the AR module provides real-time portion size estimation for accurate carbohydrate and calorie counting—critical for diabetes management.

---

## 🎯 Phase 1: Dataset Creation Plan

### 1.1 Sri Lankan Food Categories (70+ Dishes)

#### Staple Carbohydrate Sources (15-20 items)
- **Rice variations**: White rice, brown rice, basmati, coconut rice, pol sambola rice
- **Bread/Roti**: Kottu roti, paratha, dhal puri, hoppers, lamprais
- **Alternatives**: Sweet potato, cassava, plantain, thalippu rice

#### Curries & Main Dishes (20-25 items)
- **Vegetable curries**: Dhal curry, jackfruit curry, chickpea curry, pumpkin curry
- **Meat/Fish**: Chicken curry, fish curry, prawn curry, beef curry, mutton curry
- **Specialty**: Deviled dishes, lamprais, biryani, pilaf

#### Side Dishes & Sambols (15-20 items)
- **Sambols**: Pol sambol, seeni sambol, onion sambol, lime sambol, fish sambol
- **Chutneys**: Coconut chutney, tamarind chutney, mint chutney
- **Pickles**: Mango pickle, lemon pickle, chili pickle

#### Leafy Greens & Vegetables (10-15 items)
- **Local greens**: Mukunuwenna (black nightshade), kathurumurunga (drumstick leaves)
- **Common vegetables**: Cabbage, carrot, beetroot, spinach preparations

#### Accompaniments & Special Items (10 items)
- **Pappadums**: Plain, spicy, curry leaf varieties
- **Special items**: Jaggery, palm jaggery, curd, treacle

### 1.2 Data Collection Strategy

#### Method A: Restaurant & Home Kitchen Collection (500 images per dish × 70 dishes = 35,000 images)
```
✓ Authenticity: Real food preparation
✓ Portion variations: Natural size variations
✓ Lighting conditions: Multiple real-world scenarios
✓ Plating: Traditional presentation styles
```

**Collection Points:**
- Top 20-30 Sri Lankan restaurants in Colombo/Kandy
- 15-20 home-based food preparation sessions
- Food photography studios with controlled settings
- School/institution cafeteria environments

#### Method B: Image Augmentation (Generate 2-3× dataset)
```
- Rotation: 0°, 90°, 180°, 270°
- Scaling: 0.8× to 1.2× size variations
- Brightness/Contrast: Day/Night lighting
- Background variations: White plate, color plates, banana leaves, curry leaves
- Angle variations: Top-down (45°, 60°, 90°)
```

#### Method C: Mobile Phone Documentation (Real-world conditions)
```
- Use smartphone cameras (iPhone, Android)
- Multiple angles: Top, 45°, sides
- Various lighting: Natural, artificial, mixed
- Different plates: Ceramic, stainless steel, banana leaves
```

### 1.3 Annotation Schema

#### Structured Annotation Format
```json
{
  "image_id": "sri_lk_001_rice_white_001.jpg",
  "dish_name": "White Rice",
  "category": "carbohydrate_source",
  "cuisine": "Sri Lankan",
  "nutritional_label": {
    "carbohydrates_per_100g": 28.0,
    "protein_per_100g": 2.6,
    "fat_per_100g": 0.3,
    "fiber_per_100g": 0.4,
    "calories_per_100g": 130,
    "gi_index": 68,
    "gl_per_100g": 19.0
  },
  "portion_sizes": {
    "standard_bowl": { "weight_g": 150, "volume_ml": 180 },
    "cup": { "weight_g": 185, "volume_ml": 240 },
    "plate": { "weight_g": 200, "volume_ml": 240 }
  },
  "appearance_descriptors": {
    "texture": "granular",
    "color_dominant": "white",
    "color_secondary": "off_white",
    "surface_properties": "glossy_from_steam",
    "compaction": "loose"
  },
  "collection_metadata": {
    "location": "Colombo, Sri Lanka",
    "date": "2026-03-15",
    "photographer": "Name",
    "camera_model": "iPhone 14",
    "lighting_condition": "natural_indoor",
    "background": "ceramic_plate",
    "elevation_angle": 45
  },
  "bounding_boxes": [
    { "x": 120, "y": 150, "width": 280, "height": 250, "class": "rice" }
  ],
  "quality_score": 0.95
}
```

### 1.4 Dataset Organization & Versioning

```
dataset/
├── raw/
│   ├── carbohydrate_sources/        # 20 items × 500 images
│   ├── curries_main/                # 25 items × 500 images
│   ├── sides_sambols/               # 20 items × 400 images
│   ├── vegetables_greens/           # 15 items × 300 images
│   └── accompaniments/              # 10 items × 200 images
│
├── processed/
│   ├── train/ (70% - 24,500 images)
│   ├── val/   (15% - 5,250 images)
│   └── test/  (15% - 5,250 images)
│
├── augmented/
│   └── (2-3× the original)
│
├── annotations/
│   ├── sri_lankan_food_metadata.json
│   ├── nutritional_data.csv
│   └── portion_standards.json
│
└── docs/
    ├── COLLECTION_GUIDELINES.md
    ├── ANNOTATION_GUIDELINES.md
    └── QUALITY_ASSURANCE.md
```

### 1.5 Data Collection Timeline & Resources

| Phase | Duration | Tasks | Resources |
|-------|----------|-------|-----------|
| **Planning** | 2 weeks | Define locations, recruit photographers, prepare guidelines | 1 coordinator |
| **Collection Phase 1** | 4 weeks | Restaurant & home collection (25,000 images) | 3-4 photographers |
| **Collection Phase 2** | 2 weeks | Controlled studio setup (10,000 images) | 1-2 studio staff |
| **Quality Check** | 2 weeks | Remove blurry, poorly exposed images | 2 QA personnel |
| **Annotation** | 4 weeks | Label all images with metadata | 3-4 annotators |
| **Augmentation** | 2 weeks | Generate 2-3× dataset via transforms | 1 engineer |
| **Validation** | 2 weeks | Cross-check annotations, manual review (10%) | 2 annotators |

**Total Duration**: ~16 weeks (~4 months)
**Estimated Cost**: $8,000-12,000 USD (Sri Lankan context)

### 1.6 Quality Assurance Checklist

✅ Image clarity: Minimum 300dpi, <5% motion blur
✅ Exposure: No overblown highlights, no crushed shadows
✅ Focus: Entire dish in focus, center of dish sharp
✅ Annotation accuracy: Cross-checked by 2 annotators
✅ Nutrition data: Verified against government food composition tables
✅ Portion consistency: Weighed and measured for accuracy
✅ Batch balance: Each class has similar distribution (~500 images)

---

## 🎯 Phase 2: AR-Based Portion Estimation Architecture

### 2.1 System Components

```
┌──────────────────────────────────────────────────────────┐
│                    USER (Mobile Phone)                    │
│                   (Android/iOS + ARCore)                  │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│         AR PORTION ESTIMATION MODULE (2D Vision)         │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Frame Capture & Preprocessing                      │ │
│  │  • 30-60 FPS frame capture                          │ │
│  │  • Lens distortion correction                       │ │
│  │  • Illumination normalization                       │ │
│  └─────────────────────────────────────────────────────┘ │
│                           ↓                               │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Dish Detection & Segmentation                      │ │
│  │  • Real-time object detection (YOLOv8)             │ │
│  │  • Food segmentation model                          │ │
│  │  • Plate/background removal                         │ │
│  └─────────────────────────────────────────────────────┘ │
│                           ↓                               │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Reference Calibration System                       │ │
│  │  • Detect smartphone device (pixel ratio)           │ │
│  │  • Reference object recognition (coin, card, hand)  │ │
│  │  • Calculate real-world scale (pixels → cm)         │ │
│  └─────────────────────────────────────────────────────┘ │
│                           ↓                               │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Volume & Weight Estimation                         │ │
│  │  • 2D→3D reconstruction (depth estimation)          │ │
│  │  • Shape recognition (rice, curry, bread, etc.)    │ │
│  │  • Volume calculation (convex hull/voxels)          │ │
│  │  • Weight estimation (texture + density + volume)   │ │
│  └─────────────────────────────────────────────────────┘ │
│                           ↓                               │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Nutritional Calculation                            │ │
│  │  • Look up: carbs, protein, fat, fiber, calories   │ │
│  │  • Apply portion multiplier                         │ │
│  │  • Calculate GI/GL impact                           │ │
│  │  • Suggest insulin dosage (if diabetic user)        │ │
│  └─────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────┐
│            INTEGRATION WITH DIGITAL TWIN                 │
│  • Send meal info to prediction engine                   │
│  • Simulate glucose response                             │
│  • Show organ risks updated with this meal               │
│  • Display 3D body model with new risk levels            │
└──────────────────────────────────────────────────────────┘
```

### 2.2 Technical Implementation Details

#### A. Reference Calibration System

```
OPTION 1: Physical Reference Object
  ↓ User places credit card next to dish
  ↓ Card dimensions: Known (85.6mm × 53.98mm)
  ↓ Detect card in image → pixel-to-cm conversion

OPTION 2: Hand-Based Calibration
  ↓ Detect hand in frame
  ↓ Average hand palm width: 8-9cm
  ↓ Estimate real-world scale

OPTION 3: Device Gyroscope + Trigonometry
  ↓ Use phone tilt angle + distance estimation
  ↓ MediaPipe hand detection + known dimensions
```

#### B. Volume Estimation Pipeline

```
2D Image
    ↓
┌─────────────────────────────────┐
│ Depth Estimation (MiDaS model)  │
│ • Single image → depth map      │
│ • Relative depth: background=0, │
│   food=1, plate=-1              │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│ 3D Point Cloud Generation       │
│ • Convert 2D + depth to 3D      │
│ • Remove background points      │
│ • Food only: X,Y,Z coordinates  │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│ Volume Calculation              │
│ • Convex hull: ~80% accuracy    │
│ • Voxel counting: ~85% accuracy │
│ • Surface reconstruction: ~90%  │
│ • Use shape-specific formula    │
│ • Apply density correction      │
└─────────────────────────────────┘
    ↓
Weight (grams) = Volume (cm³) × Density (g/cm³)
```

#### C. Density Database for Sri Lankan Foods

| Food Item | Density (g/cm³) | Compaction Factor | Notes |
|-----------|-----------------|-------------------|-------|
| White Rice (cooked) | 1.51 | 1.0 | Loose granules |
| Rice (compacted) | 1.78 | 1.2 | Pressed down |
| Dhal Curry | 1.02 | 0.8-1.0 | Liquid + solids |
| Chicken Curry | 1.08 | 0.9-1.1 | Mixed liquid/meat |
| Pol Sambol | 0.95 | 1.0 | Coconut, oil-based |
| Kottu Roti | 1.35 | 0.9-1.1 | Shredded, airy |
| Hoppers | 0.85 | 0.8 | Hollow, airy |

### 2.3 Integration with Digital Twin System

```python
# AR Portion Estimation → Digital Twin Flow

meal_data = {
    "dish_name": "White Rice + Chicken Curry",
    "estimated_weight": 240,  # grams
    "carbohydrates": 45,      # grams
    "protein": 18,            # grams
    "fat": 8,                 # grams
    "calories": 350,          # kcal
    "gi_index": 68,           # Glycemic Index
    "gl": 30.6,               # Glycemic Load
    "confidence": 0.87,       # 87% confidence
    "timestamp": "2026-03-27T14:30:00Z"
}

# Pass to Digital Twin
prediction = digital_twin.predict_with_meal(meal_data)

# Display result in 3D body model with updated risks
visualization.update_body_systems(prediction)
```

---

## 📋 Phase 3: Implementation Roadmap

### Sprint 1: Dataset Preparation (Weeks 1-4)
- [ ] Finalize list of 70 Sri Lankan dishes
- [ ] Create annotation guidelines document
- [ ] Set up directory structure
- [ ] Recruit photographers & annotators
- [ ] Develop collection app (mobile photo logging)

### Sprint 2: Data Collection (Weeks 5-12)
- [ ] Collect 25,000 images from restaurants/homes
- [ ] Studio photography sessions
- [ ] Quality review and filtering (remove <5% poor quality)
- [ ] Initial annotation pass

### Sprint 3: Annotation & Augmentation (Weeks 13-16)
- [ ] Complete full annotation of 35,000 images
- [ ] Cross-validation (2 annotators per 10% sample)
- [ ] Generate augmented dataset (2-3×)
- [ ] Create final train/val/test splits

### Sprint 4: AR Module Development (Weeks 17-24)
- [ ] Design and implement reference calibration system
- [ ] Integrate depth estimation model
- [ ] Build volume/weight calculation pipeline
- [ ] Create food recognition pipeline (YOLOv8)
- [ ] Implement density database

### Sprint 5: Integration & Testing (Weeks 25-30)
- [ ] Integrate with Digital Twin system
- [ ] Performance optimization for mobile
- [ ] Extensive testing with real users
- [ ] Create UI for AR visualization
- [ ] Document API and usage

### Sprint 6: Validation & Deployment (Weeks 31-32)
- [ ] Accuracy benchmarking (±10% weight estimation)
- [ ] User acceptance testing
- [ ] Nutritional analysis validation
- [ ] Diabetes prediction accuracy checks

---

## 🎯 Success Metrics

| Metric | Target | Validation Method |
|--------|--------|-------------------|
| **Dataset Completeness** | 70 dishes × 500 images | Count images per category |
| **Annotation Quality** | 95% inter-rater agreement | Cohen's kappa score |
| **Food Recognition Accuracy** | >90% top-1, >95% top-3 | Test set evaluation |
| **Weight Estimation** | ±15% accuracy | Compare with actual weights |
| **Volume Estimation** | ±20% accuracy | Water displacement validation |
| **Real-time Performance** | 30 FPS on mobile | Benchmarks on Android/iOS |
| **User Satisfaction** | >4.5/5 rating | In-field testing feedback |
| **Diabetes Prediction Accuracy** | >85% glucose prediction | Retrospective analysis |

---

## 📁 Deliverables

1. **Sri Lankan Food Dataset (v1.0)**
   - 35,000+ annotated images
   - Complete nutritional metadata
   - Augmented version: 70,000+ images
   - Documentation & guidelines

2. **AR Portion Estimation Module**
   - Production-ready Python/mobile code
   - Integration endpoints
   - API documentation
   - Performance benchmarks

3. **Documentation**
   - Dataset collection guidelines
   - Annotation schema
   - System architecture
   - Integration guide with Digital Twin
   - User manual for mobile app

4. **Quality Assurance Reports**
   - Accuracy benchmarks
   - User acceptance test results
   - Nutritional validation report
   - Performance optimization report

