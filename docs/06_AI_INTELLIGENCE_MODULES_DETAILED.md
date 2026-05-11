# 06 - AI Intelligence Modules (Detailed Architecture)

## 1. Overview
The AI Intelligence Modules form the computational brain of the system, handling complex analysis of food data, activity tracking, glucose prediction, and personalized recommendation generation. These modules operate asynchronously, often triggered by voice inputs, and feed insights back into the context engine for proactive decision-making.

**Key Principle**: Transform raw user input → Extract meaningful health insights → Guide personalized action

---

## 2. Module 1: Food Recognition & Nutritional Analysis

### 2.1 Multi-Modal Food Input Processing
**Responsibility**: Accept food input through voice or images and extract nutritional information

#### Input Channels:

```python
class FoodInputProcessor:
    """
    Handle food input from voice descriptions or images
    """
    
    # ========== INPUT SOURCE HIERARCHY ==========
    INPUT_PRIORITY = [
        'food_image',           # Highest confidence: direct photo
        'food_voice_description', # High confidence: detailed voice description
        'food_text_input',      # Medium confidence: text query
        'food_database_memory'  # Low confidence: remembered similar meals
    ]
    
    def process_food_input(self, input_data):
        """
        Process food input from any source
        """
        
        # Example 1: Image input
        if input_data['type'] == 'image':
            return self._analyze_food_image(input_data['image'])
        
        # Example 2: Voice input
        elif input_data['type'] == 'voice_text':
            return self._analyze_voice_description(input_data['text'])
        
        # Example 3: Text query
        elif input_data['type'] == 'text':
            return self._analyze_text_query(input_data['query'])
```

#### 2.2 Image-Based Food Recognition

**Technology**: Computer Vision (Object Detection + Nutritional Database Lookup)

```python
class FoodImageAnalyzer:
    """
    Analyze food images to extract nutritional information
    """
    
    # ========== MODELS ==========
    MODELS = {
        'segmentation': 'YOLOv8-seg',  # Identify individual food items
        'classification': 'ResNet-152 fine-tuned on food dataset',
        'volume_estimation': 'Custom 3D reconstruction model',
        'database': 'Indian Food Database + USDA FoodData Central'
    }
    
    def analyze_food_image(self, image_bytes, user_id):
        """
        Complete food image analysis pipeline
        """
        
        # Step 1: Food Detection & Segmentation
        detected_items = self._detect_food_items(image_bytes)
        # Output: [
        #     {'item': 'rice', 'bbox': [x1,y1,x2,y2], 'confidence': 0.94},
        #     {'item': 'curry', 'bbox': [x1,y1,x2,y2], 'confidence': 0.91}
        # ]
        
        # Step 2: Volume & Portion Estimation
        portions = self._estimate_portions(image_bytes, detected_items)
        # Output: [
        #     {'item': 'rice', 'amount': '1 cup', 'ml': 240},
        #     {'item': 'curry', 'amount': '1.5 cups', 'ml': 360}
        # ]
        
        # Step 3: Nutritional Lookup
        nutrition_data = self._lookup_nutrition(detected_items, portions)
        # Output: {
        #     'total_calories': 450,
        #     'carbs_g': 62,
        #     'protein_g': 12,
        #     'fat_g': 8,
        #     'fiber_g': 3
        # }
        
        # Step 4: User Adjustment UI (for accuracy)
        # Show image with detected items, allow corrections
        # "Detected: Rice (1 cup), Curry (1.5 cups) - Adjust if needed"
        
        # Step 5: Confidence Assessment
        confidence = self._calculate_confidence(
            detection_confidence=detected_items[0]['confidence'],
            model_accuracy=0.91,
            user_adjustment=user_made_changes
        )
        
        analysis = {
            'detected_items': detected_items,
            'portions': portions,
            'nutrition': nutrition_data,
            'confidence': confidence,
            'user_input_required': confidence < 0.75
        }
        
        return analysis
    
    def _detect_food_items(self, image_bytes):
        """
        Use YOLOv8 to detect and segment food items
        """
        
        model = YOLO('yolov8-seg.pt')  # Segmentation model
        results = model(image_bytes)
        
        # Post-process results
        detections = []
        for result in results:
            for box in result.boxes:
                detections.append({
                    'class': FOOD_CLASSES[int(box.cls)],
                    'confidence': float(box.conf),
                    'bbox': box.xyxy.tolist()
                })
        
        return sorted(detections, key=lambda x: x['confidence'], reverse=True)
    
    def _estimate_portions(self, image_bytes, detected_items):
        """
        Estimate portion sizes using 3D reconstruction
        
        Method: 
        1. Identify reference object (plate diameter = ~25cm)
        2. Use reference to scale detected items
        3. 3D depth estimation from 2D image
        4. Volume calculation for each item
        """
        
        # Reference object detection
        plate_diameter_px = self._detect_plate_size(image_bytes)  # pixels
        plate_diameter_cm = 25  # standard plate
        
        px_to_cm = plate_diameter_cm / plate_diameter_px
        
        portions = []
        for item in detected_items:
            # Area in image space
            bbox = item['bbox']
            area_px = (bbox[2] - bbox[0]) * (bbox[3] - bbox[1])
            
            # Area in real space (cm²)
            area_cm2 = area_px * (px_to_cm ** 2)
            
            # 3D volume estimation using depth prediction
            depth_map = self._estimate_depth(image_bytes, item['bbox'])
            volume_cm3 = area_cm2 * average_depth(depth_map)
            
            # Convert to user-friendly units
            volume_ml = volume_cm3
            volume_cups = volume_ml / 240
            volume_grams = volume_ml * density_map[item['class']]
            
            portions.append({
                'item': item['class'],
                'volume_ml': volume_ml,
                'volume_cups': round(volume_cups, 1),
                'volume_grams': round(volume_grams, 0),
                'estimated': 'cups' if volume_cups > 0.5 else 'grams'
            })
        
        return portions
    
    def _lookup_nutrition(self, detected_items, portions):
        """
        Look up nutritional values from database
        """
        
        NUTRITION_DB = {
            'white_rice': {
                'per_cup': {
                    'calories': 206,
                    'carbs': 45,
                    'protein': 4,
                    'fat': 0.2,
                    'fiber': 0.6
                },
                'per_100g': {
                    'calories': 130,
                    'carbs': 28,
                    'protein': 2.7,
                    'fat': 0.3,
                    'fiber': 0.4
                }
            },
            'indian_curry': {
                'per_cup': {
                    'calories': 180,
                    'carbs': 8,
                    'protein': 15,
                    'fat': 9,
                    'fiber': 1.5
                }
            },
            # ... extensive food database
        }
        
        total_nutrition = {
            'calories': 0,
            'carbs': 0,
            'protein': 0,
            'fat': 0,
            'fiber': 0,
            'items_analyzed': []
        }
        
        for item, portion in zip(detected_items, portions):
            food_name = item['class'].lower()
            
            if food_name in NUTRITION_DB:
                nutrition = NUTRITION_DB[food_name]
                
                # Determine portion type
                if portion['estimated'] == 'cups':
                    unit_nutrition = nutrition['per_cup']
                    amount = portion['volume_cups']
                else:
                    unit_nutrition = nutrition.get('per_100g', nutrition['per_cup'])
                    amount = portion['volume_grams'] / 100
                
                # Calculate for this portion
                item_nutrition = {
                    food_name: {
                        'portion': portion,
                        'calories': int(unit_nutrition['calories'] * amount),
                        'carbs': round(unit_nutrition['carbs'] * amount, 1),
                        'protein': round(unit_nutrition['protein'] * amount, 1),
                        'fat': round(unit_nutrition['fat'] * amount, 1),
                        'fiber': round(unit_nutrition['fiber'] * amount, 1)
                    }
                }
                
                total_nutrition['items_analyzed'].append(item_nutrition)
                
                # Add to totals
                for key in ['calories', 'carbs', 'protein', 'fat', 'fiber']:
                    total_nutrition[key] += item_nutrition[food_name][key]
        
        return total_nutrition
```

#### 2.3 Voice-Based Food Description Processing

**Technology**: Voice → Text → Entity Extraction → Nutrition Lookup

```python
class VoiceFoodDescriptionAnalyzer:
    """
    Process natural voice descriptions of food
    """
    
    def analyze_voice_description(self, transcribed_text, user_id):
        """
        Example:
        Input: "I had rice and chicken curry for lunch"
        
        Process:
        1. Extract food items: [rice, chicken, curry]
        2. Extract quantity hints: [not explicit]
        3. Link to behavioral patterns: "User typically eats 1 cup rice + 1.5 cups curry"
        4. Ask for clarification if needed
        """
        
        # Step 1: Extract food entities
        entities = self._extract_food_entities(transcribed_text)
        # Output: [
        #     {'food': 'rice', 'modifier': None, 'type': 'grain'},
        #     {'food': 'chicken', 'type': 'protein'},
        #     {'food': 'curry', 'type': 'sauce'}
        # ]
        
        # Step 2: Extract quantity hints from text
        quantities = self._extract_quantities(transcribed_text)
        # Output: [
        #     {'food': 'rice', 'quantity': None, 'certainty': 0.0},
        #     {'food': 'curry', 'quantity': None, 'certainty': 0.0}
        # ]
        
        # Step 3: Check user history for defaults
        user_defaults = self._get_user_food_portions(user_id)
        # Output: {
        #     'rice': {'average_portion': 1.0, 'unit': 'cups'},
        #     'curry': {'average_portion': 1.5, 'unit': 'cups'}
        # }
        
        # Step 4: Build hypothesis with uncertainties
        if quantities[0]['certainty'] < 0.5:
            # Ask user for clarification
            return {
                'status': 'needs_clarification',
                'detected_items': entities,
                'message': "How much rice did you have? (small bowl, medium, or large?)",
                'options': [
                    {'volume': '0.5 cups', 'description': 'small bowl'},
                    {'volume': '1.0 cups', 'description': 'medium portion'},
                    {'volume': '1.5 cups', 'description': 'large portion'}
                ]
            }
        else:
            # Proceed with estimation
            nutrition = self._lookup_nutrition_from_entities(
                entities,
                quantities,
                user_defaults
            )
            return {
                'status': 'analyzed',
                'nutrition': nutrition,
                'confidence': 0.82,
                'message': f"Logged {nutrition['carbs']}g carbs. Does that sound right?"
            }
    
    def _extract_food_entities(self, text):
        """
        Use NER to extract food items and modifiers
        """
        
        # Using spaCy or BERT-based food NER
        doc = food_nlp(text)
        
        extracted = []
        for ent in doc.ents:
            if ent.label_ == 'FOOD':
                extracted.append({
                    'food': ent.text,
                    'type': self._classify_food_type(ent.text),
                    'modifiers': self._extract_modifiers(text, ent)
                })
        
        return extracted
    
    def _extract_quantities(self, text):
        """
        Extract quantity references from natural language
        
        Patterns:
        - "a cup of rice" → 1 cup
        - "two servings" → 2 standard servings
        - "a bowl of curry" → 1.5-2 cups (context dependent)
        - "a lot of food" → high quantity, low certainty
        """
        
        import re
        
        quantity_patterns = {
            r'(\d+\.?\d*)\s+(cup|cups|serving|servings|bowl)': 1.0,
            r'(a|one)\s+(cup|serving|bowl)': 0.8,
            r'(little|small|medium|large)': 0.5,
            r'(a lot of)': 0.3
        }
        
        quantities = []
        for pattern, certainty in quantity_patterns.items():
            matches = re.findall(pattern, text, re.IGNORECASE)
            if matches:
                quantities.append({
                    'quantity': matches[0],
                    'certainty': certainty,
                    'pattern': pattern
                })
        
        return quantities
```

#### 2.4 Hybrid Food Analysis (Image + Voice)

```python
class HybridFoodAnalyzer:
    """
    Combine image and voice for maximum accuracy
    """
    
    def analyze_food_with_image_and_description(self, image, voice_text):
        """
        Cross-validate image and voice analysis
        
        Scenario:
        Image: Detects "rice, curry, vegetables"
        Voice: User says "I had rice and curry"
        
        → Combining gives best results:
          - Image provides quantity precision
          - Voice provides item identification
          - Conflict resolution: "You said curry but image shows dal curry?"
        """
        
        # Analyze both independently
        image_analysis = self._analyze_food_image(image)
        voice_analysis = self._analyze_voice_description(voice_text)
        
        # Cross-validate
        image_items = set([item['item'] for item in image_analysis['detected_items']])
        voice_items = set([entity['food'] for entity in voice_analysis['entities']])
        
        # Conflict detection
        missing_in_voice = image_items - voice_items
        extra_in_voice = voice_items - image_items
        
        if missing_in_voice:
            return {
                'status': 'conflict_detected',
                'message': f"Image shows {missing_in_voice} but you didn't mention it. Include in your meal?",
                'image_analysis': image_analysis,
                'voice_analysis': voice_analysis
            }
        
        # Resolution: Combine image portions with voice item certainty
        final_analysis = {
            'items': [],
            'total_nutrition': {},
            'confidence': min(image_analysis['confidence'], 
                            voice_analysis['confidence']),
            'method': 'hybrid_optimized'
        }
        
        return final_analysis
```

---

## 3. Module 2: Activity Tracking & Energy Expenditure

### 3.1 Activity Input Processing

```python
class ActivityTracker:
    """
    Track and analyze physical activity
    """
    
    INPUT_SOURCES = {
        'voice_report': "I walked for 30 minutes",
        'wearable_device': "Steps: 5000, Heart Rate: 120",
        'gps_data': "Jogging route detected",
        'manual_input': "Select from predefined activities"
    }
    
    def log_activity_from_voice(self, voice_text):
        """
        Example: "I walked for 30 minutes after lunch"
        
        Extract:
        - Activity: walking
        - Duration: 30 minutes
        - Intensity: moderate (inferred)
        - Timing: after lunch
        """
        
        # NLU extracts
        activity_type = extract_activity_type(voice_text)  # 'walking'
        duration_minutes = extract_duration(voice_text)     # 30
        intensity = infer_intensity(activity_type, duration_minutes)  # 'moderate'
        timing = extract_timing(voice_text)  # 'after lunch'
        
        activity_entry = {
            'type': activity_type,
            'duration_minutes': duration_minutes,
            'intensity': intensity,
            'timing': timing,
            'timestamp': datetime.now(),
            'calculated_calories': self._calculate_calorie_burn(
                activity_type,
                duration_minutes,
                intensity,
                user_weight_kg,
                user_age
            )
        }
        
        return activity_entry
    
    def _calculate_calorie_burn(self, activity, duration, intensity, weight, age):
        """
        Calculate calories burned using MET (Metabolic Equivalent Task) values
        
        Formula: Calories = MET × Body Weight (kg) × Time (hours)
        
        Activity MET values:
        - Walking (slow): 2.8
        - Walking (moderate): 3.5
        - Walking (brisk): 4.5
        - Jogging: 7.0
        - Running: 10-15+
        """
        
        MET_VALUES = {
            'walking': {'light': 2.8, 'moderate': 3.5, 'brisk': 4.5},
            'jogging': {'easy': 7.0, 'moderate': 9.0, 'fast': 11.0},
            'running': {'moderate': 10.0, 'fast': 12.5, 'sprint': 15.0},
            'cycling': {'light': 4.0, 'moderate': 8.0, 'vigorous': 12.0},
            'swimming': {'light': 4.0, 'moderate': 8.0, 'vigorous': 11.0},
            'yoga': {'light': 2.5, 'moderate': 3.0},
            'strength_training': {'light': 3.5, 'moderate': 6.0, 'vigorous': 8.0}
        }
        
        try:
            met = MET_VALUES[activity][intensity]
            calories = met * weight * (duration / 60)  # Convert minutes to hours
            
            # Age adjustment (slight decrease in metabolism with age)
            age_factor = max(0.8, 1 - (age - 20) * 0.005)
            calories *= age_factor
            
            return round(calories, 0)
        
        except KeyError:
            return estimate_by_intensity(duration, intensity)
    
    def integrate_wearable_data(self, wearable_data):
        """
        Integrate data from fitness trackers, smartwatches, etc.
        
        Supported devices:
        - Fitbit
        - Apple Watch
        - Garmin
        - Google Fit
        """
        
        normalized_data = {
            'steps': wearable_data.get('steps', 0),
            'heart_rate_avg': wearable_data.get('heart_rate_avg', 0),
            'heart_rate_max': wearable_data.get('heart_rate_max', 0),
            'distance_km': wearable_data.get('distance_km', 0),
            'calories_burned': wearable_data.get('calories', 0),
            'floors_climbed': wearable_data.get('floors', 0),
            'duration_minutes': wearable_data.get('duration_minutes', 0),
            'source_device': wearable_data.get('source')
        }
        
        # Infer activity type from metrics
        inferred_activity = self._infer_activity_from_metrics(normalized_data)
        
        return {
            'activity': inferred_activity,
            'metrics': normalized_data,
            'data_source': 'wearable',
            'reliability': 'high'
        }
    
    def _infer_activity_from_metrics(self, metrics):
        """
        Infer activity type from metrics
        """
        
        # Heart rate patterns indicate activity type
        hr_intensity = (metrics['heart_rate_avg'] - 60) / (220 - User_Age - 60)
        
        if hr_intensity < 0.5:
            if metrics['distance_km'] > metrics['steps'] / 1000:
                return {'type': 'cycling', 'intensity': 'moderate'}
            else:
                return {'type': 'walking', 'intensity': 'light'}
        
        elif hr_intensity < 0.7:
            return {'type': 'walking', 'intensity': 'brisk'}
        
        else:
            if metrics['distance_km'] > 1:
                return {'type': 'jogging', 'intensity': 'moderate'}
            else:
                return {'type': 'running', 'intensity': 'vigorous'}
```

### 3.2 Activity Impact on Glucose

```python
class ActivityGlucoseImpactCalculator:
    """
    Predict how activity will affect glucose levels
    """
    
    ACTIVITY_GLUCOSE_IMPACT = {
        'walking': {
            'light': {'glucose_drop_rate': 1.5, 'duration_minutes': 30},
            'moderate': {'glucose_drop_rate': 3.0, 'duration_minutes': 30},
            'brisk': {'glucose_drop_rate': 4.5, 'duration_minutes': 30}
        },
        'jogging': {
            'moderate': {'glucose_drop_rate': 5.0, 'duration_minutes': 30},
            'fast': {'glucose_drop_rate': 7.0, 'duration_minutes': 30}
        },
        'cycling': {
            'moderate': {'glucose_drop_rate': 4.0, 'duration_minutes': 30},
            'vigorous': {'glucose_drop_rate': 6.0, 'duration_minutes': 30}
        }
    }
    
    def predict_glucose_impact(self, activity_type, duration, intensity, 
                              current_glucose, user_profile):
        """
        Predict glucose change from activity
        
        Factors affecting impact:
        1. Activity type and intensity
        2. Duration
        3. Current glucose level
        4. Insulin on board
        5. Carbs in system
        6. User's fitness level
        7. Meal timing relative to activity
        """
        
        # Base glucose drop from activity
        base_drop = (
            self.ACTIVITY_GLUCOSE_IMPACT[activity_type][intensity]['glucose_drop_rate'] *
            (duration / 30)  # Normalize to 30-minute unit
        )
        
        # Insulin sensitivity adjustment
        if user_profile.insulin_on_board > 0:
            # More insulin + activity = higher glucose drop
            base_drop *= (1 + user_profile.insulin_on_board / 10)
        
        # Carbs in system adjustment
        if user_profile.carbs_in_system > 20:
            # Burning carbs available in system reduces overall drop
            base_drop *= 0.7
        
        # Fitness level adjustment
        fitness_factor = user_profile.fitness_level_factor  # 0.8-1.2
        base_drop *= fitness_factor
        
        predicted_glucose = current_glucose - base_drop
        
        # Safety check
        if predicted_glucose < 70:
            return {
                'predicted_glucose': max(predicted_glucose, 40),
                'risk_level': 'HIGH_HYPOGLYCEMIA_RISK',
                'recommendation': 'Eat 15g fast carbs before activity or reduce duration',
                'glucose_drop': base_drop,
                'certainty': 0.75
            }
        else:
            return {
                'predicted_glucose': predicted_glucose,
                'risk_level': 'LOW',
                'recommendation': f'Good activity! You\'ll be at ~{int(predicted_glucose)} mg/dL',
                'glucose_drop': base_drop,
                'certainty': 0.82
            }
```

---

## 4. Module 3: Glucose Prediction & Forecasting

### 4.1 Prediction Model Architecture

```python
class GlucosePredictionEngine:
    """
    Multi-model glucose prediction pipeline
    """
    
    PREDICTION_MODELS = {
        'short_term': {
            'model': 'LSTM Neural Network',
            'input': 'Recent glucose readings + context',
            'horizon': '15-30 minutes ahead',
            'accuracy': '±15 mg/dL'
        },
        'medium_term': {
            'model': 'XGBoost with glucose history + meals + activity',
            'input': 'Last 2-3 hours of data',
            'horizon': '2-4 hours ahead',
            'accuracy': '±20-25 mg/dL'
        },
        'long_term': {
            'model': 'Random Forest + circadian patterns',
            'input': 'Day-level features',
            'horizon': '24 hours ahead',
            'accuracy': '±30-40 mg/dL'
        }
    }
    
    def __init__(self):
        self.short_term_model = load_lstm_model('glucose_lstm_v2.h5')
        self.medium_term_model = load_xgboost_model('glucose_xgb_v1.pkl')
        self.long_term_model = load_rf_model('glucose_rf_v1.pkl')
    
    def predict_glucose(self, user_id, minutes_ahead):
        """
        Main prediction function
        """
        
        # Retrieve recent context
        context = get_user_context(user_id)
        user_model = get_personalized_model(user_id)
        
        if minutes_ahead <= 30:
            return self._predict_short_term(context, minutes_ahead)
        elif minutes_ahead <= 240:
            return self._predict_medium_term(context, minutes_ahead)
        else:
            return self._predict_long_term(context, minutes_ahead)
    
    def _predict_short_term(self, context, minutes_ahead):
        """
        LSTM-based short-term prediction
        Uses recent glucose trend
        """
        
        # Prepare input
        glucose_history = context['recent_glucose_readings']  # Last 1 hour
        carbs_timeline = context['carbs_in_system']
        insulin_timeline = context['insulin_on_board']
        
        # Feature engineering
        features = np.array([
            glucose_history,
            carbs_timeline,
            insulin_timeline,
            context['glucose_trend'],
            context['current_activity']
        ]).reshape(1, -1)
        
        # LSTM prediction
        prediction = self.short_term_model.predict(features)
        
        return {
            'predicted_glucose': float(prediction[0]),
            'model': 'LSTM',
            'horizon': f'{minutes_ahead} minutes',
            'confidence': 0.88,
            'confidence_interval': (prediction[0] - 15, prediction[0] + 15)
        }
    
    def _predict_medium_term(self, context, minutes_ahead):
        """
        XGBoost-based medium-term prediction
        Combines multiple factors
        """
        
        # Feature extraction
        features = {
            # Recent glucose readings
            'glucose_avg_30min': context['glucose_avg_30min'],
            'glucose_trend': context['glucose_trend_value'],
            
            # Carbs and digestion
            'carbs_in_system': context['carbs_in_system'],
            'carb_absorption_rate': context['carb_absorption_rate'],
            
            # Insulin
            'insulin_on_board': context['insulin_on_board'],
            'insulin_activity_remaining': context['insulin_activity_remaining'],
            
            # Activity
            'current_activity_intensity': context['current_activity_intensity'],
            'activity_duration_minutes': context['activity_duration_minutes'],
            
            # Time and stress
            'time_of_day': context['time_of_day_encoded'],
            'stress_level': context['stress_level'],
            'sleep_quality': context['sleep_quality'],
            
            # Personalization
            'user_carb_sensitivity': context['user_carb_sensitivity'],
            'user_insulin_sensitivity': context['user_insulin_sensitivity'],
            'user_activity_factor': context['user_activity_factor']
        }
        
        # Convert to array
        feature_vector = np.array([features[key] for key in features.keys()])
        feature_vector = feature_vector.reshape(1, -1)
        
        # XGBoost prediction
        prediction = self.medium_term_model.predict(feature_vector)[0]
        
        return {
            'predicted_glucose': float(prediction),
            'model': 'XGBoost',
            'horizon': f'{minutes_ahead} minutes',
            'confidence': 0.82,
            'confidence_interval': (prediction - 20, prediction + 20),
            'key_drivers': self._identify_key_drivers(features)
        }
    
    def _predict_long_term(self, context, minutes_ahead):
        """
        Random Forest-based long-term prediction
        Uses daily patterns and circadian rhythm
        """
        
        # Day-level features
        features = {
            'time_of_day': context['time_of_day'],
            'day_of_week': context['day_of_week'],
            'carbs_so_far_today': context['carbs_so_far'],
            'activity_minutes_today': context['activity_minutes'],
            'stress_level_today': context['stress_level'],
            'sleep_hours_last_night': context['sleep_hours'],
            'typical_glucose_at_this_time': context['typical_time_glucose'],
            'is_weekend': context['is_weekend'],
            'is_holiday': context['is_holiday']
        }
        
        # Random Forest prediction
        prediction = self.long_term_model.predict(np.array(list(features.values())).reshape(1, -1))[0]
        
        return {
            'predicted_glucose': float(prediction),
            'model': 'Random Forest',
            'horizon': f'{minutes_ahead} minutes',
            'confidence': 0.75,
            'confidence_interval': (prediction - 40, prediction + 40),
            'pattern_explanation': f"At this time, you typically have ~{prediction} mg/dL"
        }
    
    def _identify_key_drivers(self, features):
        """
        Explain what factors most affect the prediction
        """
        
        drivers = []
        
        if features['carbs_in_system'] > 30:
            drivers.append(f"High carbs in system (+{features['carbs_in_system']}g)")
        
        if features['insulin_on_board'] > 2:
            drivers.append(f"Active insulin will lower glucose")
        
        if features['current_activity_intensity'] > 0.5:
            drivers.append(f"Activity increasing glucose burn")
        
        if features['stress_level'] > 0.7:
            drivers.append(f"High stress may raise glucose")
        
        return drivers
```

### 4.2 Prediction Accuracy & Calibration

```python
class PredictionCalibration:
    """
    Continuously improve predictions for individual users
    """
    
    def compare_prediction_vs_actual(self, prediction, actual_glucose, minutes_ahead):
        """
        After time passes, compare prediction to reality
        """
        
        error = abs(prediction['predicted_glucose'] - actual_glucose)
        
        # Calculate prediction accuracy
        accuracy_rating = {
            'error_mg_dl': error,
            'percent_error': (error / actual_glucose) * 100,
            'accuracy_level': self._error_to_accuracy(error),
            'model_used': prediction['model']
        }
        
        # Store for model retraining
        self._log_prediction_error(
            user_id=prediction['user_id'],
            prediction=prediction,
            actual=actual_glucose,
            error=error
        )
        
        # Auto-calibration
        if error > 25:  # Large error
            self._trigger_model_recalibration(prediction['user_id'])
        
        return accuracy_rating
    
    def _error_to_accuracy(self, error_mg_dl):
        """Convert error to accuracy description"""
        
        if error_mg_dl <= 10:
            return 'excellent'
        elif error_mg_dl <= 15:
            return 'very_good'
        elif error_mg_dl <= 20:
            return 'good'
        elif error_mg_dl <= 30:
            return 'acceptable'
        else:
            return 'poor'
```

---

## 5. Module 4: Personalized Recommendation Engine

### 5.1 Recommendation Framework

```python
class RecommendationEngine:
    """
    Generate personalized, context-aware recommendations
    """
    
    RECOMMENDATION_TYPES = {
        'IMMEDIATE': {
            'delay': 0,
            'examples': ['Low glucose - eat now', 'High predicted glucose - walk now'],
            'tone': 'urgent'
        },
        'SHORT_TERM': {
            'delay': '5-30 minutes',
            'examples': ['Activity suggestion for after meal', 'Snack recommendation'],
            'tone': 'actionable'
        },
        'MEDIUM_TERM': {
            'delay': '1-6 hours',
            'examples': ['Meal timing suggestion', 'Medication reminder'],
            'tone': 'helpful'
        },
        'LONG_TERM': {
            'delay': 'Next few days',
            'examples': ['Dietary pattern insight', 'Weekly activity goal'],
            'tone': 'educational'
        }
    }
    
    def generate_recommendation(self, user_context, recommendation_type='SHORT_TERM'):
        """
        Generate a personalized recommendation
        """
        
        # Assess current state
        assessment = self._assess_state(user_context)
        
        # Identify opportunities
        opportunities = self._identify_opportunities(assessment, user_context)
        
        # Generate options
        options = self._generate_options(opportunities, user_context)
        
        # Score and rank options
        scored_options = self._score_options(options, user_context)
        
        # Select top recommendation
        best_option = scored_options[0]
        
        # Personalize message
        message = self._personalize_message(
            best_option,
            user_context,
            recommendation_type
        )
        
        return {
            'recommendation': best_option['action'],
            'message': message,
            'confidence': best_option['score'],
            'alternatives': scored_options[1:3],  # Provide alternatives
            'reason': best_option['reasoning'],
            'expected_outcome': best_option['expected_outcome']
        }
    
    def _assess_state(self, user_context):
        """Comprehensive state assessment"""
        
        return {
            'glucose_status': self._assess_glucose(user_context),
            'carb_status': self._assess_carbs(user_context),
            'activity_status': self._assess_activity(user_context),
            'time_status': self._assess_time(user_context),
            'health_status': self._assess_health(user_context)
        }
    
    def _assess_glucose(self, context):
        """Analyze glucose situation"""
        
        glucose = context['current_glucose']
        glucose_trend = context['glucose_trend']
        predicted_glucose = context['predicted_glucose_30min']
        
        if glucose < 70:
            return {'status': 'hypoglycemia', 'urgency': 'critical'}
        elif glucose < 100 or predicted_glucose < 70:
            return {'status': 'low_risk', 'urgency': 'high'}
        elif glucose <= 180 and glucose_trend != 'rapidly_rising':
            return {'status': 'optimal', 'urgency': 'low'}
        elif glucose > 240:
            return {'status': 'hyperglycemia', 'urgency': 'medium'}
        else:
            return {'status': 'suboptimal', 'urgency': 'medium'}
    
    def _generate_options(self, opportunities, user_context):
        """Generate multiple recommendation options"""
        
        options = []
        
        # Option 1: Food-related
        if opportunities['food']:
            options.append({
                'type': 'food',
                'action': self._generate_food_recommendation(user_context),
                'category': 'nutrition'
            })
        
        # Option 2: Activity-related
        if opportunities['activity']:
            options.append({
                'type': 'activity',
                'action': self._generate_activity_recommendation(user_context),
                'category': 'lifestyle'
            })
        
        # Option 3: Monitoring-related
        if opportunities['monitoring']:
            options.append({
                'type': 'monitoring',
                'action': self._generate_monitoring_recommendation(user_context),
                'category': 'health'
            })
        
        # Option 4: Educational insights
        if opportunities['learning']:
            options.append({
                'type': 'education',
                'action': self._generate_educational_recommendation(user_context),
                'category': 'insight'
            })
        
        return options
    
    def _generate_food_recommendation(self, user_context):
        """Generate food recommendation"""
        
        assessment = self._assess_glucose(user_context)
        user_prefs = user_context['food_preferences']
        
        if assessment['status'] == 'hypoglycemia':
            return {
                'recommendation': 'Eat fast-acting carbs now',
                'options': ['15g glucose tabs', '1 glass juice', '5 gummy candies'],
                'timing': 'immediate'
            }
        
        elif assessment['urgency'] == 'high':
            return {
                'recommendation': 'Light snack to stabilize glucose',
                'options': [
                    'Apple + 1 tbsp peanut butter',
                    'Yogurt (plain)',
                    'Handful of almonds'
                ],
                'timing': 'next 30 minutes',
                'reasoning': 'Will prevent glucose from dropping further'
            }
        
        else:
            # Suggest meal based on time and history
            meal_suggestion = self._suggest_meal(user_context)
            return {
                'recommendation': f'Time for {meal_suggestion["meal"]}',
                'options': meal_suggestion['user_preferred_foods'],
                'timing': 'next hour',
                'reasoning': 'You typically eat around this time'
            }
    
    def _generate_activity_recommendation(self, user_context):
        """Generate activity recommendation"""
        
        glucose = user_context['current_glucose']
        predicted_glucose = user_context['predicted_glucose_1hr']
        carbs_in_system = user_context['carbs_in_system']
        
        # High glucose after high-carb meal = perfect time for activity
        if predicted_glucose > 180 and carbs_in_system > 40:
            return {
                'recommendation': 'Perfect time for a walk!',
                'suggestion': 'A 20-30 minute walk now will help balance your meal',
                'explanation': 'You just ate carbs, activity will help prevent glucose spike',
                'options': [
                    '15-min walk (light)',
                    '30-min walk (moderate)',
                    '10-min stairs (vigorous)'
                ]
            }
        
        # Low activity day = general encouragement
        elif user_context['activity_minutes_today'] < 3000:
            return {
                'recommendation': 'Add some movement',
                'suggestion': 'Even a 10-minute walk helps glucose management',
                'explanation': 'Low activity days often show higher glucose variance',
                'options': ['10-min walk', '5-min stretching', 'Yoga session']
            }
        
        else:
            return {
                'recommendation': 'Great activity today!',
                'suggestion': 'Keep up the good work',
                'explanation': 'Your activity is helping glucose control'
            }
    
    def _score_options(self, options, user_context):
        """Score options based on relevance and effectiveness"""
        
        user_profile = user_context['user_profile']
        
        scored = []
        for option in options:
            score = 0.5  # Base score
            
            # Scoring factors
            if option['type'] in user_context['preference_order']:
                score += 0.2
            
            if self._aligns_with_goals(option, user_context):
                score += 0.2
            
            if self._matches_user_capacity(option, user_context):
                score += 0.1
            
            option['score'] = min(score, 1.0)
            scored.append(option)
        
        # Sort by score
        return sorted(scored, key=lambda x: x['score'], reverse=True)
    
    def _personalize_message(self, option, user_context, rec_type):
        """Personalize message to user's preferences"""
        
        user_tone = user_context['tone_preference']
        user_name = user_context['first_name']
        
        # Template selection based on tone
        if user_tone == 'encouraging':
            template = f"Hey {user_name}! {option['action']['recommendation']}! {option['action']['suggestion']}"
        
        elif user_tone == 'neutral':
            template = f"{option['action']['recommendation']}. {option['action']['suggestion']}"
        
        elif user_tone == 'cautious':
            template = f"Recommendation: {option['action']['recommendation']}. "
        
        else:
            template = f"I suggest: {option['action']['recommendation']}."
        
        return template
```

---

## 6. Implementation Checklist

- [ ] **Food Recognition Module**
  - [ ] YOLOv8 model training on food images
  - [ ] Portion estimation algorithm
  - [ ] Nutritional database setup
  - [ ] Voice description NER
  - [ ] Hybrid image+voice processor

- [ ] **Activity Tracking**
  - [ ] Activity type classification
  - [ ] Calorie burn calculation
  - [ ] Wearable device integration
  - [ ] Glucose impact prediction
  - [ ] Activity logging UI

- [ ] **Glucose Prediction**
  - [ ] LSTM model training
  - [ ] XGBoost model training
  - [ ] RF model training
  - [ ] Feature engineering pipeline
  - [ ] Model calibration system

- [ ] **Recommendation Engine**
  - [ ] Recommendation generation logic
  - [ ] Personalization engine
  - [ ] Option scoring system
  - [ ] Message templating

- [ ] **Testing & Validation**
  - [ ] Food image accuracy tests
  - [ ] Activity classification tests
  - [ ] Glucose prediction accuracy
  - [ ] Recommendation relevance feedback

---

## 7. Technology Stack Summary

```
┌─────────────────────────────────────────────────────────────┐
│ AI INTELLIGENCE MODULES                                     │
├─────────────────────────────────────────────────────────────┤
│ Food Recognition   │ YOLOv8 + TensorFlow + ResNet-152      │
│ NER Processing     │ spaCy + Custom food NER models        │
│ Activity Tracking  │ ML classification + MET calculations   │
│ Glucose Prediction │ LSTM + XGBoost + Random Forest        │
│ ML Framework       │ TensorFlow / PyTorch                  │
│ Data Processing    │ Pandas + NumPy + Scikit-learn        │
│ Orchestration      │ Apache Airflow (model training)       │
│ Serving            │ TensorFlow Serving / FastAPI          │
└─────────────────────────────────────────────────────────────┘
```

---

**Last Updated**: March 22, 2026
**Status**: Complete
**Next Module**: 07_UI_FALLBACK_LAYER_DETAILED.md
