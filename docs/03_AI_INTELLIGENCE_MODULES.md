# 03. AI Intelligence Modules - Complete Architecture & Implementation

## 📋 Overview
The AI Intelligence Modules are the core AI/ML systems that provide the actual diabetes management intelligence: food recognition, activity analysis, glucose prediction, and personalized recommendations.

---

## 🏗️ Module Architecture

### 3.1 Food Recognition & Analysis Module

#### Purpose
Identify food items, estimate portions, calculate nutritional values, and predict glycemic impact.

#### 3.1.1 Vision-Based Food Recognition

**Technology: YOLOv8 + OpenAI Vision API**

```python
from ultralytics import YOLO
import cv2
import numpy as np
from typing import List, Dict
import openai

class FoodRecognition:
    """
    Multi-stage food recognition system
    Stage 1: YOLOv8 for real-time detection
    Stage 2: OpenAI Vision for detailed classification
    """
    
    def __init__(self):
        # Load YOLO model
        self.detection_model = YOLO('yolov8x.pt')
        
        # Food-specific model (can be fine-tuned)
        self.food_model = YOLO('yolov8-food.pt')
        
        openai.api_key = "YOUR_API_KEY"
        
        # Nutrition database
        self.nutrition_db = self._load_nutrition_db()
    
    def recognize_food_from_image(self, image_path):
        """
        End-to-end food recognition pipeline
        """
        # Load image
        image = cv2.imread(image_path)
        
        # Stage 1: YOLO Detection
        results = self.food_model(image)
        detected_foods = self._extract_yolo_detections(results)
        
        # Stage 2: OpenAI Vision for verification
        verified_foods = self._verify_with_vision_api(image_path, detected_foods)
        
        # Stage 3: Portion estimation
        portions = self._estimate_portions(image, verified_foods)
        
        # Stage 4: Nutritional analysis
        nutrition = self._calculate_nutrition(verified_foods, portions)
        
        return {
            "foods": verified_foods,
            "portions": portions,
            "nutrition": nutrition,
            "confidence": self._calculate_confidence(detected_foods, verified_foods)
        }
    
    def _extract_yolo_detections(self, results):
        """Extract detected food items from YOLO results"""
        detected_items = []
        
        for result in results:
            for box in result.boxes:
                confidence = box.conf[0].item()
                class_id = int(box.cls[0].item())
                class_name = result.names[class_id]
                
                # Confidence threshold
                if confidence > 0.5:
                    detected_items.append({
                        "name": class_name,
                        "confidence": confidence,
                        "bounding_box": box.xyxy[0].tolist()
                    })
        
        return detected_items
    
    def _verify_with_vision_api(self, image_path, yolo_detections):
        """
        Use OpenAI Vision API for verification
        Better accuracy for food classification
        """
        with open(image_path, "rb") as image_file:
            image_data = base64.b64encode(image_file.read()).decode("utf-8")
        
        # Create prompt for vision API
        detected_names = ", ".join([f["name"] for f in yolo_detections])
        prompt = f"""
        Analyze this food image and verify the detected items: {detected_names}
        
        For each food item, provide:
        1. Food name (be specific)
        2. Estimated portion size
        3. Whether it's in the detected list
        4. Any additional items not detected
        
        Format as JSON with keys: foods (list of items with name, portion)
        """
        
        response = openai.ChatCompletion.create(
            model="gpt-4-vision-preview",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "image_url",
                            "image_url": {
                                "url": f"data:image/jpeg;base64,{image_data}"
                            }
                        },
                        {
                            "type": "text",
                            "text": prompt
                        }
                    ]
                }
            ]
        )
        
        return json.loads(response.choices[0].message.content)["foods"]
    
    def _estimate_portions(self, image, foods):
        """
        Estimate portion sizes using reference objects
        Uses depth estimation or known food sizes
        """
        import json
        
        # Get image dimensions for reference
        height, width, _ = image.shape
        
        # Standard portion sizes (in grams/ml)
        standard_portions = {
            "rice": 150,  # 1 cup cooked
            "bread": 30,   # 1 slice
            "chicken": 100,  # typical serving
            "curry": 200,  # 1 bowl
            "apple": 180,  # medium
            "water": 250   # 1 glass
        }
        
        portions = {}
        for food in foods:
            food_name = food["name"].lower()
            
            # Simple estimation based on image fraction
            # In production, use more sophisticated methods
            portion_multiplier = 1.0
            
            standard = standard_portions.get(food_name, 100)
            portions[food_name] = standard * portion_multiplier
        
        return portions
    
    def _calculate_nutrition(self, foods, portions):
        """
        Calculate complete nutritional information
        """
        total_nutrition = {
            "calories": 0,
            "protein": 0,
            "carbs": 0,
            "fat": 0,
            "fiber": 0,
            "glycemic_index": 0,
            "glycemic_load": 0,
            "items_breakdown": []
        }
        
        for food in foods:
            food_name = food["name"].lower()
            portion = portions.get(food_name, 100)
            
            # Look up nutrition data
            nutrition_data = self.nutrition_db.get(food_name, {})
            
            if nutrition_data:
                # Scale nutrition by portion
                scaled_nutrition = self._scale_nutrition(nutrition_data, portion)
                
                total_nutrition["calories"] += scaled_nutrition["calories"]
                total_nutrition["protein"] += scaled_nutrition["protein"]
                total_nutrition["carbs"] += scaled_nutrition["carbs"]
                total_nutrition["fat"] += scaled_nutrition["fat"]
                total_nutrition["fiber"] += scaled_nutrition.get("fiber", 0)
                
                total_nutrition["items_breakdown"].append({
                    "food": food_name,
                    "portion": f"{portion}g",
                    "nutrition": scaled_nutrition
                })
        
        # Calculate weighted GI/GL
        if total_nutrition["items_breakdown"]:
            total_nutrition["glycemic_load"] = self._calc_glycemic_load(
                total_nutrition["items_breakdown"]
            )
        
        return total_nutrition
    
    def _scale_nutrition(self, nutrition_data, portion_in_grams):
        """Scale nutrition data by portion size"""
        # Assuming nutrition_data is per 100g
        scalar = portion_in_grams / 100
        
        return {
            "calories": nutrition_data.get("calories", 0) * scalar,
            "protein": nutrition_data.get("protein", 0) * scalar,
            "carbs": nutrition_data.get("carbs", 0) * scalar,
            "fat": nutrition_data.get("fat", 0) * scalar,
            "fiber": nutrition_data.get("fiber", 0) * scalar
        }
    
    def _calc_glycemic_load(self, items_breakdown):
        """Calculate total glycemic load"""
        total_gl = 0
        
        for item in items_breakdown:
            gi = item["nutrition"].get("glycemic_index", 50)
            carbs = item["nutrition"].get("carbs", 0)
            
            # GL = (GI/100) * carb amount
            gl = (gi / 100) * carbs
            total_gl += gl
        
        return total_gl
    
    def _calculate_confidence(self, yolo_results, vision_results):
        """Calculate overall confidence score"""
        if not yolo_results or not vision_results:
            return 0.0
        
        yolo_confidence = sum(r["confidence"] for r in yolo_results) / len(yolo_results)
        
        # Cross-verification: how many items match between YOLO and Vision
        match_count = 0
        for vision_item in vision_results:
            for yolo_item in yolo_results:
                if self._similarity(vision_item["name"], yolo_item["name"]) > 0.7:
                    match_count += 1
        
        match_rate = match_count / max(len(vision_results), len(yolo_results))
        
        overall_confidence = (yolo_confidence * 0.4) + (match_rate * 0.6)
        
        return overall_confidence
    
    def _similarity(self, str1, str2):
        """Calculate string similarity"""
        from difflib import SequenceMatcher
        return SequenceMatcher(None, str1.lower(), str2.lower()).ratio()
    
    def _load_nutrition_db(self):
        """
        Load nutrition database
        In production, use USDA FoodData Central or similar
        """
        # Simplified nutrition database (per 100g)
        return {
            "rice": {"calories": 130, "protein": 2.7, "carbs": 28, "fat": 0.3, "gi": 68},
            "chicken": {"calories": 165, "protein": 31, "carbs": 0, "fat": 3.6, "gi": 0},
            "apple": {"calories": 52, "protein": 0.3, "carbs": 14, "fat": 0.2, "gi": 36},
            "bread": {"calories": 265, "protein": 9, "carbs": 49, "fat": 3.3, "gi": 71},
            "curry": {"calories": 150, "protein": 8, "carbs": 5, "fat": 10, "gi": 40},
            "water": {"calories": 0, "protein": 0, "carbs": 0, "fat": 0, "gi": 0}
        }
```

#### 3.1.2 Voice-Based Food Logging

```python
class VoiceFoodLogger:
    """
    Log food via natural language description
    """
    
    def __init__(self):
        self.food_recognition = FoodRecognition()
        self.nutrition_db = self.food_recognition.nutrition_db
    
    def log_meal_from_voice(self, voice_transcript, optional_image=None):
        """
        Log meal from voice description, optionally with image
        Example: "I ate one cup of rice with chicken curry"
        """
        # Extract food items from transcript
        meal_items = self._parse_meal_description(voice_transcript)
        
        # If image provided, use vision for verification
        if optional_image:
            vision_results = self.food_recognition.recognize_food_from_image(optional_image)
            meal_items = self._merge_results(meal_items, vision_results)
        
        # Calculate nutrition
        nutrition = self._calculate_meal_nutrition(meal_items)
        
        return {
            "items": meal_items,
            "nutrition": nutrition,
            "confidence": self._confidence_score(meal_items, optional_image is not None)
        }
    
    def _parse_meal_description(self, text):
        """
        Parse food items and portions from natural language
        """
        import re
        
        # Regex patterns for common phrases
        patterns = {
            r"(\d+\.?\d*)\s*cup\s+of\s+(\w+)": "portion_name",
            r"(\d+\.?\d*)\s*spoon\s+of\s+(\w+)": "portion_name",
            r"one\s+(\w+)": "single_item",
            r"(\w+)\s+curry": "dish"
        }
        
        meal_items = []
        
        for pattern, pattern_type in patterns.items():
            matches = re.finditer(pattern, text, re.IGNORECASE)
            
            for match in matches:
                if pattern_type == "portion_name":
                    quantity = float(match.group(1))
                    food = match.group(2)
                    meal_items.append({
                        "name": food,
                        "quantity": quantity,
                        "unit": match.group(0).split()[-2]
                    })
                
                elif pattern_type == "single_item":
                    meal_items.append({
                        "name": match.group(1),
                        "quantity": 1,
                        "unit": "whole"
                    })
                
                elif pattern_type == "dish":
                    meal_items.append({
                        "name": match.group(0),
                        "quantity": 1,
                        "unit": "bowl"
                    })
        
        return meal_items
    
    def _calculate_meal_nutrition(self, meal_items):
        """Calculate total nutrition for meal"""
        total = {
            "calories": 0,
            "protein": 0,
            "carbs": 0,
            "fat": 0,
            "glycemic_load": 0
        }
        
        for item in meal_items:
            # Normalize item name
            normalized_name = item["name"].lower()
            
            # Get nutrition data
            nutrition = self.nutrition_db.get(normalized_name, {})
            
            if nutrition:
                # Scale by portion
                portion = self._normalize_portion(item["quantity"], item["unit"])
                scale = portion / 100
                
                total["calories"] += nutrition.get("calories", 0) * scale
                total["protein"] += nutrition.get("protein", 0) * scale
                total["carbs"] += nutrition.get("carbs", 0) * scale
                total["fat"] += nutrition.get("fat", 0) * scale
                
                # Calculate glycemic load
                gi = nutrition.get("gi", 50)
                carbs = nutrition.get("carbs", 0) * scale
                gl = (gi / 100) * carbs
                total["glycemic_load"] += gl
        
        return total
    
    def _normalize_portion(self, quantity, unit):
        """Convert portion to grams"""
        conversions = {
            "cup": 240,
            "spoon": 15,
            "bowl": 250,
            "whole": 100,
            "slice": 30,
            "glass": 250
        }
        
        base_grams = conversions.get(unit.lower(), 100)
        return quantity * base_grams
    
    def _merge_results(self, voice_items, vision_items):
        """Merge voice and vision recognition results"""
        # If both agree, increase confidence
        merged = voice_items.copy()
        
        # Add any items from vision not in voice
        voice_names = [item["name"].lower() for item in voice_items]
        
        for vision_item in vision_items["foods"]:
            if vision_item["name"].lower() not in voice_names:
                merged.append({
                    "name": vision_item["name"],
                    "quantity": 1,
                    "unit": "unit",
                    "source": "vision"
                })
        
        return merged
    
    def _confidence_score(self, meal_items, has_image):
        """Calculate confidence in meal logging"""
        base_confidence = 0.7  # Voice alone
        
        if has_image:
            base_confidence = 0.95  # Image improves confidence
        
        # Reduce if items have unknown nutrition data
        known_items = sum(1 for item in meal_items if item["name"].lower() in self.nutrition_db)
        coverage = known_items / len(meal_items) if meal_items else 1.0
        
        return base_confidence * coverage
```

---

### 3.2 Activity Recognition & Tracking Module

```python
class ActivityRecognition:
    """
    Recognize and track physical activities
    Can use wearable data (accelerometer, heart rate) or voice input
    """
    
    def __init__(self):
        self.activity_classifier = self._load_activity_model()
        self.calorie_burn_model = self._load_calorie_model()
    
    def recognize_activity_from_wearable(self, sensor_data):
        """
        Recognize activity from accelerometer/gyroscope data
        """
        # Extract features from sensor data
        features = self._extract_activity_features(sensor_data)
        
        # Classify activity
        activity_prediction = self.activity_classifier.predict([features])
        confidence = self.activity_classifier.predict_proba([features]).max()
        
        return {
            "activity": activity_prediction[0],
            "confidence": confidence,
            "duration": self._estimate_duration(sensor_data),
            "intensity": self._estimate_intensity(sensor_data)
        }
    
    def recognize_activity_from_voice(self, voice_transcript):
        """
        Recognize activity from voice description
        Examples: "I did yoga for 30 minutes", "I walked 5 km"
        """
        import re
        
        # Extract activity information
        activity_info = self._parse_activity_description(voice_transcript)
        
        return activity_info
    
    def _parse_activity_description(self, text):
        """Parse activity details from natural language"""
        
        activities = {
            "walking": r"(walk|walked|walking)",
            "running": r"(run|running|jogging)",
            "cycling": r"(cycle|cycling|bike)",
            "yoga": r"(yoga)",
            "swimming": r"(swim|swimming)",
            "strength": r"(weight|strength|gym|exercise)",
            "sports": r"(football|badminton|cricket|tennis)"
        }
        
        duration_pattern = r"(\d+\.?\d*)\s*(minute|hour|min|hr)"
        distance_pattern = r"(\d+\.?\d*)\s*(km|mile)"
        intensity_pattern = r"(light|moderate|vigorous|intense)"
        
        # Find activity type
        activity_type = "general"
        for activity, pattern in activities.items():
            if re.search(pattern, text, re.IGNORECASE):
                activity_type = activity
                break
        
        # Extract duration
        duration_match = re.search(duration_pattern, text)
        duration = 30  # default
        if duration_match:
            value = float(duration_match.group(1))
            unit = duration_match.group(2).lower()
            duration = value if "min" in unit else value * 60
        
        # Extract distance
        distance = None
        distance_match = re.search(distance_pattern, text)
        if distance_match:
            distance = float(distance_match.group(1))
        
        # Extract intensity
        intensity = "moderate"  # default
        intensity_match = re.search(intensity_pattern, text)
        if intensity_match:
            intensity = intensity_match.group(1).lower()
        
        return {
            "activity_type": activity_type,
            "duration_minutes": int(duration),
            "distance_km": distance,
            "intensity": intensity,
            "calories_burned": self._estimate_calories(activity_type, duration, intensity)
        }
    
    def _estimate_calories(self, activity_type, duration_mins, intensity):
        """
        Estimate calories burned based on activity
        Formula: (MET * weight in kg * duration in hours) = calories
        """
        
        # Metabolic Equivalent (MET) values
        met_values = {
            "walking": {"light": 2.0, "moderate": 3.5, "vigorous": 5.0},
            "running": {"light": 6.0, "moderate": 9.8, "vigorous": 13.0},
            "cycling": {"light": 4.0, "moderate": 8.0, "vigorous": 12.0},
            "yoga": {"light": 2.5, "moderate": 3.0, "vigorous": 4.0},
            "swimming": {"light": 4.0, "moderate": 8.0, "vigorous": 11.0},
            "strength": {"light": 3.5, "moderate": 6.0, "vigorous": 8.0},
            "general": {"light": 2.5, "moderate": 5.0, "vigorous": 8.0}
        }
        
        # Get MET value
        met = met_values.get(activity_type, met_values["general"]).get(intensity, 5.0)
        
        # Assume average weight of 70 kg (will be personalized)
        weight_kg = 70
        duration_hours = duration_mins / 60
        
        calories = met * weight_kg * duration_hours
        
        return calories
    
    def _extract_activity_features(self, sensor_data):
        """Extract features from accelerometer/gyroscope data"""
        import numpy as np
        
        # Common features for activity recognition
        features = {
            "mean_accel": np.mean(np.linalg.norm(sensor_data["acceleration"], axis=1)),
            "std_accel": np.std(np.linalg.norm(sensor_data["acceleration"], axis=1)),
            "mean_gyro": np.mean(np.linalg.norm(sensor_data["gyroscope"], axis=1)),
            "std_gyro": np.std(np.linalg.norm(sensor_data["gyroscope"], axis=1)),
            "peak_frequency": self._get_peak_frequency(sensor_data["acceleration"]),
            "signal_magnitude_area": self._calc_signal_magnitude_area(sensor_data["acceleration"])
        }
        
        return list(features.values())
    
    def _load_activity_model(self):
        """Load pre-trained activity classification model"""
        # In production, load trained scikit-learn or TensorFlow model
        from sklearn.ensemble import RandomForestClassifier
        return RandomForestClassifier()
    
    def _load_calorie_model(self):
        """Load model for calorie estimation"""
        return None  # Use MET formula for now
    
    def _estimate_duration(self, sensor_data):
        """Estimate activity duration from sensor data"""
        if "timestamp" in sensor_data:
            return (sensor_data["timestamp"][-1] - sensor_data["timestamp"][0]) / 1000  # milliseconds to seconds
        return None
    
    def _estimate_intensity(self, sensor_data):
        """Estimate activity intensity from sensor data"""
        import numpy as np
        
        acceleration = np.linalg.norm(sensor_data["acceleration"], axis=1)
        std_accel = np.std(acceleration)
        
        if std_accel < 2:
            return "light"
        elif std_accel < 5:
            return "moderate"
        else:
            return "vigorous"
    
    def _get_peak_frequency(self, acceleration):
        """Get dominant frequency from acceleration data"""
        from scipy.fft import fft
        import numpy as np
        
        # FFT on acceleration magnitude
        accel_mag = np.linalg.norm(acceleration, axis=1)
        fft_result = np.abs(fft(accel_mag))
        peak_freq = np.argmax(fft_result[:len(fft_result)//2])
        
        return peak_freq
    
    def _calc_signal_magnitude_area(self, acceleration):
        """Calculate signal magnitude area feature"""
        import numpy as np
        
        sma = np.sum(np.abs(acceleration)) / len(acceleration)
        return sma
```

---

### 3.3 Glucose Prediction Module

```python
class GlucosePredictionModel:
    """
    Predict future blood glucose levels based on meals, activity, and patterns
    """
    
    def __init__(self):
        self.prediction_model = self._load_glucose_model()
        self.physiological_model = self._init_physiological_model()
    
    def predict_glucose_level(self, context, hours_ahead=2):
        """
        Predict glucose level for specified time in future
        
        Args:
            context: Current user context (food, activity, baseline glucose, etc.)
            hours_ahead: How many hours in future to predict
        
        Returns:
            predicted_glucose, confidence, risk_level
        """
        
        # Extract features from context
        features = self._extract_prediction_features(context)
        
        # Generate multiple predictions using different methods
        ml_prediction = self._ml_predict(features)
        physio_prediction = self._physiological_predict(context, hours_ahead)
        
        # Ensemble average
        final_prediction = (ml_prediction * 0.5 + physio_prediction * 0.5)
        confidence = self._calculate_confidence(ml_prediction, physio_prediction)
        
        # Determine risk level
        risk_level = self._assess_risk(final_prediction)
        
        return {
            "predicted_glucose": final_prediction,
            "confidence": confidence,
            "risk_level": risk_level,
            "time_ahead_hours": hours_ahead,
            "ml_estimate": ml_prediction,
            "physio_estimate": physio_prediction
        }
    
    def _extract_prediction_features(self, context):
        """
        Extract ML features from context
        """
        features = {
            "current_glucose": context.get("current_glucose", 7.0),
            "recent_avg_glucose": self._calc_recent_avg(context),
            "last_meal_time_hours": context.get("hours_since_meal", 2),
            "last_meal_carbs": context.get("last_meal_carbs", 0),
            "last_meal_gi": context.get("last_meal_gi", 50),
            "activity_last_hour": context.get("activity_intensity", 0),
            "insulin_dose": context.get("insulin_dose", 0),
            "time_of_day": context.get("hour_of_day", 12),
            "day_of_week": context.get("day_of_week", 0),
            "stress_level": context.get("stress_level", 0),
            "sleep_quality": context.get("sleep_quality", 5),
            "menstrual_cycle": context.get("menstrual_cycle_day", -1)
        }
        
        return features
    
    def _ml_predict(self, features):
        """
        Use machine learning model for prediction
        Model trained on historical glucose data
        """
        import numpy as np
        
        # Convert features to array
        feature_array = np.array(list(features.values())).reshape(1, -1)
        
        # Predict
        prediction = self.prediction_model.predict(feature_array)[0]
        
        return prediction
    
    def _physiological_predict(self, context, hours_ahead):
        """
        Use physiological model for prediction
        Based on glucose dynamics, insulin action, and carbohydrate absorption
        """
        
        # Current state
        G_t = context.get("current_glucose", 7.0)  # mmol/L
        
        # Glucose production
        carbs = context.get("last_meal_carbs", 0)
        time_since_meal = context.get("hours_since_meal", 2)
        
        # Carbohydrate absorption
        # Typical absorption rate: 30-60% in first hour
        absorption_rate = self._calculate_absorption_rate(carbs, time_since_meal)
        glucose_from_carbs = carbs * 0.017 * absorption_rate.get(hours_ahead, 0)
        
        # Insulin action
        insulin_action = context.get("insulin_effect", 0)
        
        # Activity effect (glucose utilization)
        activity_effect = context.get("activity_glucose_utilization", 0)
        
        # Simplified glucose dynamics equation
        # dG/dt = glucose_production - insulin_action - activity_effect
        
        # Predict trajectory
        predicted_glucose = G_t + (glucose_from_carbs - insulin_action - activity_effect) * hours_ahead
        
        # Constraints
        predicted_glucose = max(predicted_glucose, 2.0)  # Can't go below 2.0 mmol/L
        
        return predicted_glucose
    
    def _calculate_absorption_rate(self, carbs, time_since_meal):
        """
        Calculate how much carbohydrate has been absorbed
        Uses standard meal absorption curves
        """
        
        # Simplified absorption model
        # Peak absorption at 30-60 minutes, trails off
        
        absorption_by_hour = {
            0: 0.3,    # First hour: 30%
            1: 0.5,    # Second hour: 50% (20% additional)
            2: 0.7,    # Third hour: 70% (20% additional)
            3: 0.85,   # Fourth hour: 85% (15% additional)
            4: 0.95,   # Fifth hour: 95% (10% additional)
            5: 1.0     # Complete absorption
        }
        
        return absorption_by_hour
    
    def _assess_risk(self, predicted_glucose):
        """
        Assess hypoglycemia/hyperglycemia risk
        """
        if predicted_glucose < 3.9:
            return {
                "level": "critical",
                "description": "Severe hypoglycemia risk",
                "action": "Consume fast-acting carbs immediately"
            }
        elif predicted_glucose < 5.5:
            return {
                "level": "warning",
                "description": "Hypoglycemia risk",
                "action": "Monitor closely, have fast carbs available"
            }
        elif predicted_glucose > 13:
            return {
                "level": "warning",
                "description": "Hyperglycemia risk",
                "action": "Increase activity, review meal composition"
            }
        elif predicted_glucose > 15:
            return {
                "level": "critical",
                "description": "Severe hyperglycemia risk",
                "action": "Consult healthcare provider"
            }
        else:
            return {
                "level": "normal",
                "description": "Glucose in target range",
                "action": "Continue current management"
            }
    
    def _calculate_confidence(self, ml_pred, physio_pred):
        """Calculate prediction confidence based on model agreement"""
        difference = abs(ml_pred - physio_pred)
        
        if difference < 0.5:  # Predictions agree
            return 0.95
        elif difference < 1.0:
            return 0.85
        elif difference < 2.0:
            return 0.70
        else:
            return 0.50
    
    def _load_glucose_model(self):
        """Load pre-trained glucose prediction model"""
        # In production, load trained neural network
        return None
    
    def _init_physiological_model(self):
        """Initialize physiological parameters"""
        return {
            "insulin_sensitivity": 1.8,  # mmol/L per unit insulin
            "carb_absorption_rate": 0.5,  # per hour
            "glucose_turnover": 1.2  # mg/dL/min
        }
    
    def _calc_recent_avg(self, context):
        """Calculate recent average glucose from context"""
        return context.get("glucose_avg_7day", 7.0)
```

---

### 3.4 Recommendation Engine

```python
class RecommendationEngine:
    """
    Generate personalized recommendations for meals, activities, and health actions
    """
    
    def __init__(self, user_context):
        self.context = user_context
        self.food_recommender = FoodRecommender(user_context)
        self.activity_recommender = ActivityRecommender(user_context)
        self.health_recommender = HealthRecommender(user_context)
    
    def generate_recommendations(self, current_state):
        """
        Generate comprehensive recommendations
        """
        recommendations = []
        
        # Food recommendations
        food_recs = self.food_recommender.recommend_meal(current_state)
        recommendations.extend(food_recs)
        
        # Activity recommendations
        activity_recs = self.activity_recommender.recommend_activity(current_state)
        recommendations.extend(activity_recs)
        
        # Health recommendations
        health_recs = self.health_recommender.recommend_health_action(current_state)
        recommendations.extend(health_recs)
        
        # Sort by priority and relevance
        recommendations = self._rank_recommendations(recommendations, current_state)
        
        return recommendations[:5]  # Top 5 recommendations
    
    def _rank_recommendations(self, recommendations, current_state):
        """Rank recommendations by priority and relevance"""
        
        for rec in recommendations:
            # Priority score (0-100)
            rec["priority_score"] = 50
            
            # Urgency
            if rec.get("urgency") == "critical":
                rec["priority_score"] += 40
            elif rec.get("urgency") == "high":
                rec["priority_score"] += 20
            
            # Personalization match
            if rec.get("personalized"):
                rec["priority_score"] += 10
            
            # Likelihood of user following
            if rec.get("alignment_with_user"):
                rec["priority_score"] += 10
        
        return sorted(recommendations, key=lambda x: x.get("priority_score", 0), reverse=True)

class FoodRecommender:
    """Recommend foods based on glucose, patterns, and preferences"""
    
    def __init__(self, context):
        self.context = context
    
    def recommend_meal(self, current_state):
        """Recommend next meal"""
        glucose = current_state["glucose"]
        
        recommendations = []
        
        # High glucose: recommend low-GI meal
        if glucose > 10:
            recommendations.append({
                "type": "food",
                "recommendation": "Eat a low-GI meal (high in fiber)",
                "examples": ["Quinoa bowl with vegetables", "Brown rice with beans"],
                "reason": "Your glucose is elevated. Low-GI foods help prevent spikes.",
                "urgency": "medium",
                "priority_score": 70
            })
        
        # Low glucose: recommend quick carbs (if trending down)
        elif glucose < 5 and current_state.get("glucose_trend") == "declining":
            recommendations.append({
                "type": "food",
                "recommendation": "Eat a quick-acting carb source",
                "examples": ["Orange juice", "Glucose tablets", "Banana"],
                "reason": "Glucose is low and declining. Eat fast-acting carbs.",
                "urgency": "critical",
                "priority_score": 95
            })
        
        # Normal glucose: personalized recommendation
        else:
            personalized = self._get_personalized_meal_suggestion()
            recommendations.append(personalized)
        
        return recommendations
    
    def _get_personalized_meal_suggestion(self):
        """Determine best meal based on user preferences and patterns"""
        
        # What time of day is it?
        current_hour = datetime.now().hour
        
        if 6 <= current_hour < 10:
            meal_type = "breakfast"
        elif 12 <= current_hour < 14:
            meal_type = "lunch"
        elif 18 <= current_hour < 20:
            meal_type = "dinner"
        else:
            meal_type = "snack"
        
        # Use user's favorite foods
        user_preferences = self.context.profile.favorite_foods
        
        return {
            "type": "food",
            "recommendation": f"Time for {meal_type}",
            "examples": user_preferences[:3],
            "reason": "Based on your eating patterns and preferences",
            "urgency": "low",
            "personalized": True
        }
```

---

## 📊 Integration & Data Flow

```
User Input (Voice/Image/Sensors)
    ↓
├─ Food Recognition
│   ├─ Vision Analysis (YOLO + OpenAI Vision)
│   ├─ Voice Parsing (NLU)
│   └─ Nutrition Calculation
│
├─ Activity Recognition
│   ├─ Wearable Data (Accelerometer/Heart Rate)
│   ├─ Voice Description
│   └─ Duration/Intensity Estimation
│
└─ Health Status
    ├─ Glucose Reading (if available)
    ├─ Symptom Detection
    └─ Biometric Analysis
        ↓
    [Context Engine Update]
        ↓
    ├─ Glucose Prediction
    ├─ Risk Assessment
    └─ Recommendation Generation
        ↓
    [Voice Response Generation]
```

---

## 📈 Implementation Roadmap

### Phase 1: Core Intelligence (Week 1-2)
- [ ] Implement Voice Food Logger
- [ ] Activity Recognition from voice
- [ ] Basic glucose prediction
- [ ] Simple recommendation engine

### Phase 2: Vision & Advanced Models (Week 3-4)
- [ ] YOLO food detection integration
- [ ] OpenAI Vision API integration
- [ ] Wearable sensor support
- [ ] ML-based glucose prediction

### Phase 3: Personalization & Optimization (Week 5)
- [ ] User preference learning
- [ ] Model fine-tuning
- [ ] Recommendation ranking
- [ ] Performance optimization

---

## 📚 Dependencies

```yaml
ai_intelligence_dependencies:
  Computer Vision:
    - ultralytics>=8.0.0
    - opencv-python>=4.8.0
    - pillow>=10.0.0
    - openai>=1.0.0
  
  ML/Prediction:
    - scikit-learn>=1.3.0
    - tensorflow>=2.13.0
    - numpy>=1.24.0
    - scipy>=1.11.0
  
  Wearable/Sensors:
    - pandas>=2.0.0

```

