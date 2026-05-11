# 02. Context Engine - Complete Architecture & Implementation

## 📋 Overview
The Context Engine is the intelligent memory system that maintains detailed user context, learns from interactions, and enables proactive recommendations. It transforms the assistant from command-based to genuinely context-aware.

---

## 🏗️ Core Architecture

### 2.1 User Context Model

#### User Profile Structure
```python
from dataclasses import dataclass
from datetime import datetime
from typing import List, Dict

@dataclass
class UserProfile:
    """Complete user context model"""
    
    # Basic Demographics
    user_id: str
    age: int
    gender: str
    weight: float  # kg
    height: float  # cm
    diabetes_type: str  # Type 1, Type 2, Gestational
    
    # Health Metrics
    baseline_glucose: float  # mmol/L or mg/dL
    target_glucose_range: tuple  # (min, max)
    hba1c: float  # % (last known)
    insulin_regime: str  # if applicable
    medications: List[str]
    
    # Lifestyle
    job_type: str  # sedentary, moderate, active
    exercise_frequency: int  # hours/week
    sleep_pattern: str  # early_bird, night_owl
    stress_level: str  # low, medium, high
    
    # Preferences
    preferred_language: str
    dietary_restrictions: List[str]
    allergies: List[str]
    favorite_foods: List[str]
    favorite_activities: List[str]
    
    # Temporal Data
    created_at: datetime
    last_updated: datetime
    interaction_count: int = 0

class UserContext:
    """Dynamic user context that evolves"""
    
    def __init__(self, user_profile: UserProfile):
        self.profile = user_profile
        self.daily_context = {}
        self.weekly_patterns = {}
        self.monthly_trends = {}
        self.interaction_history = []
    
    def update_context(self, interaction):
        """Update context based on new interaction"""
        timestamp = datetime.now()
        
        self.interaction_history.append({
            "timestamp": timestamp,
            "type": interaction.type,
            "data": interaction.data,
            "response": interaction.response
        })
        
        # Update daily context
        date_key = timestamp.date()
        if date_key not in self.daily_context:
            self.daily_context[date_key] = self._initialize_daily_context()
        
        self._update_daily_context(interaction)
        self._update_patterns()
```

### 2.2 Context Tracking Components

#### 2.2.1 Food Logging Context

```python
from enum import Enum
from typing import Optional
import json

class MealCategory(Enum):
    BREAKFAST = "breakfast"
    LUNCH = "lunch"
    DINNER = "dinner"
    SNACK = "snack"
    BEVERAGE = "beverage"

@dataclass
class FoodEntry:
    """Detailed food logging entry"""
    timestamp: datetime
    meal_category: MealCategory
    items: List[Dict]  # [{"name": "rice", "qty": "1 cup", "calories": 200}]
    total_calories: float
    
    # Macronutrients (in grams)
    carbs: float
    protein: float
    fat: float
    fiber: float
    
    # Glycemic Index
    average_gi: float  # 0-100
    glycemic_load: float
    
    # Logging method
    logged_by: str  # "voice", "image", "manual"
    confidence: float  # Model confidence 0-1
    notes: Optional[str] = None
    image_uri: Optional[str] = None

class FoodContextEngine:
    """Track food patterns and provide insights"""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.food_entries: List[FoodEntry] = []
        self.meal_patterns = {}
        self.favorite_foods = {}
        self.nutritional_summary = {}
    
    def log_food(self, meal: FoodEntry):
        """Log food with automatic pattern recognition"""
        self.food_entries.append(meal)
        
        # Update meal timing patterns
        hour = meal.timestamp.hour
        meal_key = f"{meal.meal_category.value}_{hour}"
        
        if meal_key not in self.meal_patterns:
            self.meal_patterns[meal_key] = {
                "frequency": 0,
                "avg_calories": 0,
                "avg_carbs": 0,
                "typical_items": []
            }
        
        self.meal_patterns[meal_key]["frequency"] += 1
        self.meal_patterns[meal_key]["typical_items"].append(meal.items)
        
        # Update favorite foods
        for item in meal.items:
            food_name = item["name"]
            if food_name not in self.favorite_foods:
                self.favorite_foods[food_name] = {"count": 0, "avg_gi": 0}
            self.favorite_foods[food_name]["count"] += 1
    
    def predict_next_meal_time(self, current_time):
        """
        Predict when user typically has next meal
        Returns: expected_meal_time, confidence
        """
        current_hour = current_time.hour
        
        # Find most common meal times
        meal_times = {}
        for pattern_key, data in self.meal_patterns.items():
            meal_type, hour = pattern_key.split("_")
            if meal_type not in meal_times:
                meal_times[meal_type] = []
            meal_times[meal_type].append({
                "hour": int(hour),
                "frequency": data["frequency"]
            })
        
        # Sort by frequency and predict
        predictions = {}
        for meal_type, times in meal_times.items():
            if times:
                avg_hour = sum(t["hour"] * t["frequency"] for t in times) // sum(t["frequency"] for t in times)
                predictions[meal_type] = avg_hour
        
        return predictions
    
    def get_nutritional_summary(self, days=7):
        """
        Get last N days nutrition summary
        """
        recent_entries = [e for e in self.food_entries 
                         if (datetime.now() - e.timestamp).days <= days]
        
        summary = {
            "total_days": len(set(e.timestamp.date() for e in recent_entries)),
            "total_calories": sum(e.total_calories for e in recent_entries),
            "avg_daily_calories": sum(e.total_calories for e in recent_entries) / max(len(set(e.timestamp.date() for e in recent_entries)), 1),
            "total_carbs": sum(e.carbs for e in recent_entries),
            "avg_daily_carbs": sum(e.carbs for e in recent_entries) / max(len(set(e.timestamp.date() for e in recent_entries)), 1),
            "carb_variance": self._calculate_variance([e.carbs for e in recent_entries]),
            "avg_glycemic_load": sum(e.glycemic_load for e in recent_entries) / max(len(recent_entries), 1)
        }
        
        return summary
    
    def provide_food_insight(self):
        """
        Generate intelligent food insights
        """
        if len(self.food_entries) < 7:
            return "Not enough data for insights yet"
        
        # Most frequent foods
        most_frequent = sorted(
            self.favorite_foods.items(),
            key=lambda x: x[1]["count"],
            reverse=True
        )[:5]
        
        insight = f"""
        Food Insights (Last 7 days):
        - Most eaten: {', '.join([f[0] for f in most_frequent])}
        - Average daily carbs: {self.nutritional_summary['avg_daily_carbs']:.1f}g
        - Meal regularity: {'Very regular' if self._check_regularity() else 'Variable'}
        """
        
        return insight
    
    def _check_regularity(self):
        """Check if meals are at regular times"""
        if not self.meal_patterns:
            return False
        
        frequencies = [p["frequency"] for p in self.meal_patterns.values()]
        return max(frequencies) > len(frequencies) * 0.5
```

#### 2.2.2 Activity Tracking Context

```python
@dataclass
class ActivityEntry:
    """Physical activity logging"""
    timestamp: datetime
    activity_type: str  # "walking", "yoga", "cardio", etc.
    duration: int  # minutes
    intensity: str  # "light", "moderate", "vigorous"
    calories_burned: float
    distance: Optional[float] = None  # km
    heart_rate_avg: Optional[int] = None
    notes: Optional[str] = None

class ActivityContextEngine:
    """Track activity patterns and provide recommendations"""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.activity_entries: List[ActivityEntry] = []
        self.activity_patterns = {}
        self.total_weekly_activity = 0
    
    def log_activity(self, activity: ActivityEntry):
        """Log and learn from activity patterns"""
        self.activity_entries.append(activity)
        
        # Categorize activity
        activity_key = f"{activity.activity_type}_{activity.intensity}"
        
        if activity_key not in self.activity_patterns:
            self.activity_patterns[activity_key] = {
                "count": 0,
                "total_duration": 0,
                "total_calories": 0,
                "avg_duration": 0
            }
        
        self.activity_patterns[activity_key]["count"] += 1
        self.activity_patterns[activity_key]["total_duration"] += activity.duration
        self.activity_patterns[activity_key]["total_calories"] += activity.calories_burned
    
    def recommend_activity(self, current_blood_glucose, time_of_day):
        """
        Intelligent activity recommendation based on context
        """
        recommendations = []
        
        # High glucose -> recommend activity
        if current_blood_glucose > 10:
            recommendations.append({
                "activity": "Walking",
                "duration": 20,
                "intensity": "moderate",
                "reason": "High glucose detected. Light exercise helps reduce blood sugar.",
                "timing": "Now"
            })
        
        # Check if user has been sedentary too long
        hours_since_activity = self._hours_since_last_activity()
        if hours_since_activity > 4:
            recommendations.append({
                "activity": "Stretching",
                "duration": 5,
                "intensity": "light",
                "reason": "You've been inactive for a while. Light stretching helps circulation.",
                "timing": "Now"
            })
        
        # Time-based recommendations
        if time_of_day == "morning":
            recommendations.append({
                "activity": "Morning walk",
                "duration": 30,
                "intensity": "moderate",
                "reason": "Morning activity improves insulin sensitivity for the day.",
                "timing": "Everyday"
            })
        
        return recommendations
    
    def predict_activity_time(self):
        """Predict when user typically exercises"""
        activity_times = {}
        
        for entry in self.activity_entries:
            hour = entry.timestamp.hour
            activity_times[hour] = activity_times.get(hour, 0) + 1
        
        if not activity_times:
            return None
        
        most_common_hour = max(activity_times, key=activity_times.get)
        return most_common_hour
    
    def get_weekly_activity_summary(self):
        """Summary of weekly activity"""
        week_ago = datetime.now() - timedelta(days=7)
        weekly_entries = [e for e in self.activity_entries if e.timestamp > week_ago]
        
        return {
            "total_hours": sum(e.duration for e in weekly_entries) / 60,
            "total_calories": sum(e.calories_burned for e in weekly_entries),
            "activity_types": list(set(e.activity_type for e in weekly_entries)),
            "frequency": len(weekly_entries),
            "meets_recommendation": sum(e.duration for e in weekly_entries) / 60 >= 150
        }
    
    def _hours_since_last_activity(self):
        """Get hours since last physical activity"""
        if not self.activity_entries:
            return float('inf')
        
        last_activity = max(self.activity_entries, key=lambda x: x.timestamp)
        return (datetime.now() - last_activity.timestamp).total_seconds() / 3600
```

#### 2.2.3 Health Metrics Context

```python
@dataclass
class GlucoseReading:
    """Blood glucose measurement"""
    timestamp: datetime
    value: float  # mmol/L
    measurement_type: str  # "fasting", "before_meal", "after_meal", "random"
    reliability: float  # 0-1 confidence
    notes: Optional[str] = None

class HealthMetricsContext:
    """Track health metrics and detect patterns"""
    
    def __init__(self, user_id: str, glucose_unit="mmol"):
        self.user_id = user_id
        self.glucose_unit = glucose_unit
        self.glucose_readings: List[GlucoseReading] = []
        self.blood_pressure_readings = []
        self.weight_entries = []
        
        # Statistics
        self.daily_averages = {}
        self.time_patterns = {}
    
    def add_glucose_reading(self, reading: GlucoseReading):
        """Add glucose reading and update patterns"""
        self.glucose_readings.append(reading)
        
        # Update daily average
        date_key = reading.timestamp.date()
        if date_key not in self.daily_averages:
            self.daily_averages[date_key] = []
        
        self.daily_averages[date_key].append(reading.value)
        
        # Update time-of-day patterns
        hour_key = f"hour_{reading.timestamp.hour}"
        if hour_key not in self.time_patterns:
            self.time_patterns[hour_key] = []
        
        self.time_patterns[hour_key].append(reading.value)
    
    def detect_hypoglycemia_risk(self):
        """
        Detect if user is at risk of low blood sugar
        Based on recent trends
        """
        if len(self.glucose_readings) < 5:
            return {"risk": "unknown", "confidence": 0}
        
        recent = self.glucose_readings[-5:]
        values = [r.value for r in recent]
        
        # Check if trending down
        if all(values[i] >= values[i+1] for i in range(len(values)-1)):
            slope = (values[-1] - values[0]) / 5
            
            if slope < -1:  # Dropping more than 1 per reading
                return {
                    "risk": "high",
                    "confidence": 0.85,
                    "estimated_time_to_hypoglycemia": self._estimate_time_to_threshold(values, 3.9)
                }
        
        return {"risk": "low", "confidence": 0.9}
    
    def detect_hyperglycemia_pattern(self):
        """Detect if readings are consistently high"""
        if len(self.glucose_readings) < 7:
            return {"pattern": "unknown"}
        
        last_week = self.glucose_readings[-7*4:]  # Last 7 days of readings
        values = [r.value for r in last_week]
        avg = sum(values) / len(values)
        
        if avg > 10:
            return {
                "pattern": "high",
                "average": avg,
                "severity": "mild" if avg < 13 else "moderate" if avg < 15 else "severe"
            }
        
        return {"pattern": "normal"}
    
    def _estimate_time_to_threshold(self, recent_values, threshold):
        """Estimate when glucose will reach threshold"""
        if len(recent_values) < 2:
            return "unknown"
        
        rate_of_change = (recent_values[-1] - recent_values[-2])
        if rate_of_change >= 0:
            return "not_trending_down"
        
        minutes_to_threshold = abs(threshold - recent_values[-1]) / abs(rate_of_change) * 5
        return max(minutes_to_threshold, 0)
    
    def get_glucose_statistics(self, days=7):
        """Get detailed glucose statistics"""
        week_ago = datetime.now() - timedelta(days=days)
        recent = [r for r in self.glucose_readings if r.timestamp > week_ago]
        
        if not recent:
            return {}
        
        values = [r.value for r in recent]
        
        return {
            "count": len(recent),
            "average": sum(values) / len(values),
            "min": min(values),
            "max": max(values),
            "std_dev": self._calculate_std_dev(values),
            "range": max(values) - min(values),
            "time_in_range": self._calc_time_in_range(recent),
            "variability": self._calculate_variability(values)
        }
    
    def _calc_time_in_range(self, readings, target_min=4, target_max=7):
        """Calculate % time in target range"""
        in_range = sum(1 for r in readings if target_min <= r.value <= target_max)
        return (in_range / len(readings) * 100) if readings else 0
    
    def _calculate_std_dev(self, values):
        """Calculate standard deviation"""
        if len(values) < 2:
            return 0
        mean = sum(values) / len(values)
        variance = sum((x - mean) ** 2 for x in values) / len(values)
        return variance ** 0.5
    
    def _calculate_variability(self, values):
        """Calculate glucose variability score"""
        std_dev = self._calculate_std_dev(values)
        mean = sum(values) / len(values)
        return (std_dev / mean) * 100 if mean > 0 else 0
```

### 2.3 Temporal Context

```python
class TemporalContextEngine:
    """
    Understand time-based patterns in user behavior
    Enables proactive interactions
    """
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.daily_patterns = {}
        self.weekly_patterns = {}
        self.seasonal_patterns = {}
    
    def analyze_time_patterns(self, interaction_history):
        """
        Analyze when user typically:
        - Logs meals
        - Exercises
        - Has low/high glucose
        """
        from collections import defaultdict
        
        interactions_by_hour = defaultdict(list)
        interactions_by_dow = defaultdict(list)
        
        for interaction in interaction_history:
            hour = interaction['timestamp'].hour
            dow = interaction['timestamp'].strftime("%A")
            
            interactions_by_hour[hour].append(interaction)
            interactions_by_dow[dow].append(interaction)
        
        return {
            "hourly_pattern": dict(interactions_by_hour),
            "daily_pattern": dict(interactions_by_dow),
            "peak_hours": sorted(
                interactions_by_hour.items(),
                key=lambda x: len(x[1]),
                reverse=True
            )[:3]
        }
    
    def predict_user_availability(self, current_time):
        """
        Predict if user is likely available for interaction
        """
        hour = current_time.hour
        dow = current_time.strftime("%A")
        
        if hour >= 22 or hour < 6:
            return {"available": False, "reason": "Likely sleeping"}
        
        if dow in ["Saturday", "Sunday"]:
            if 10 <= hour <= 22:
                return {"available": True, "reason": "Weekend active hours"}
        
        return {"available": True, "reason": "Typical activity hours"}
    
    def optimal_interaction_time(self):
        """
        Determine best time to send notifications/recommendations
        """
        # Analyze when user is most responsive
        responsive_hours = self._analyze_response_time()
        
        return max(responsive_hours, key=responsive_hours.get) if responsive_hours else 9
```

---

## 🧠 Context-Aware Decision Making

```python
class ContextAwareness:
    """
    Use context to make intelligent decisions and recommendations
    """
    
    def __init__(self, user_context: UserContext):
        self.context = user_context
    
    def generate_proactive_message(self, current_state):
        """
        Generate intelligent, context-aware messages
        """
        messages = []
        
        # Check various conditions
        time_since_meal = self._time_since_last_meal()
        blood_glucose = current_state.get("glucose")
        activity_level = current_state.get("activity")
        
        # Rule 1: Proactive meal reminders
        if time_since_meal > 4 and blood_glucose < 8:
            messages.append({
                "type": "meal_reminder",
                "message": "You haven't eaten in 4 hours. Should we log your next meal?",
                "priority": "medium",
                "action": "log_meal"
            })
        
        # Rule 2: Glucose spike warning
        if blood_glucose > 12 and activity_level == "sedentary":
            messages.append({
                "type": "health_alert",
                "message": "Your glucose is elevated. A 15-minute walk could help.",
                "priority": "high",
                "action": "recommend_activity"
            })
        
        # Rule 3: Low glucose risk
        if blood_glucose < 4:
            messages.append({
                "type": "emergency_alert",
                "message": "Low glucose detected! Have a quick sugar source.",
                "priority": "critical",
                "action": "emergency_response"
            })
        
        # Rule 4: Medication reminder
        if hasattr(self.context, 'medication_schedule'):
            if self._is_medication_time():
                messages.append({
                    "type": "medication_reminder",
                    "message": "It's time for your medication.",
                    "priority": "high",
                    "action": "confirm_medication"
                })
        
        return messages
    
    def personalize_recommendation(self, base_recommendation):
        """
        Personalize recommendations based on user context
        """
        personalized = base_recommendation.copy()
        
        # Adjust based on user preferences
        if "activity" in base_recommendation:
            preferred_activities = self.context.profile.favorite_activities
            if base_recommendation["activity"] not in preferred_activities:
                # Suggest preferred activity instead
                personalized["activity"] = preferred_activities[0] if preferred_activities else base_recommendation["activity"]
        
        # Adjust timing based on user patterns
        preferred_time = self._get_preferred_time_for_activity(base_recommendation)
        personalized["suggested_time"] = preferred_time
        
        # Adjust messaging based on user language/style
        personalized["message_lang"] = self.context.profile.preferred_language
        personalized["formality"] = self.context.profile.voice_preferences.get("formality_level", "casual")
        
        return personalized
    
    def _time_since_last_meal(self):
        """Calculate hours since last meal"""
        if not hasattr(self.context, 'food_entries') or not self.context.food_entries:
            return float('inf')
        
        last_meal = max(self.context.food_entries, key=lambda x: x.timestamp)
        return (datetime.now() - last_meal.timestamp).total_seconds() / 3600
    
    def _is_medication_time(self):
        """Check if it's time for medication"""
        # This would depend on medication schedule
        pass
    
    def _get_preferred_time_for_activity(self, recommendation):
        """Get best time for activity based on patterns"""
        pass
```

---

## 💾 Context Storage & Persistence

```python
import json
from typing import Dict
import sqlite3

class ContextStorage:
    """
    Persist context data for retrieval
    """
    
    def __init__(self, db_path="context_db.sqlite"):
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """Initialize SQLite database for context"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        # Create tables for each context type
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS food_entries (
                id INTEGER PRIMARY KEY,
                user_id TEXT,
                timestamp DATETIME,
                meal_category TEXT,
                items JSON,
                total_calories REAL,
                carbs REAL,
                protein REAL,
                fat REAL,
                glycemic_load REAL
            )
        ''')
        
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS activity_entries (
                id INTEGER PRIMARY KEY,
                user_id TEXT,
                timestamp DATETIME,
                activity_type TEXT,
                duration INTEGER,
                intensity TEXT,
                calories_burned REAL
            )
        ''')
        
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS glucose_readings (
                id INTEGER PRIMARY KEY,
                user_id TEXT,
                timestamp DATETIME,
                value REAL,
                measurement_type TEXT,
                reliability REAL
            )
        ''')
        
        conn.commit()
        conn.close()
    
    def save_food_entry(self, user_id, entry: FoodEntry):
        """Save food entry to database"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('''
            INSERT INTO food_entries 
            (user_id, timestamp, meal_category, items, total_calories, carbs, protein, fat, glycemic_load)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
        ''', (
            user_id,
            entry.timestamp,
            entry.meal_category.value,
            json.dumps(entry.items),
            entry.total_calories,
            entry.carbs,
            entry.protein,
            entry.fat,
            entry.glycemic_load
        ))
        
        conn.commit()
        conn.close()
    
    def load_recent_entries(self, user_id, entry_type="food", days=7):
        """Load recent entries from database"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        table_name = f"{entry_type}_entries"
        
        cursor.execute(f'''
            SELECT * FROM {table_name}
            WHERE user_id = ? AND timestamp > datetime('now', '-{days} days')
            ORDER BY timestamp DESC
        ''', (user_id,))
        
        results = cursor.fetchall()
        conn.close()
        
        return results
```

---

## 📊 Context Analytics

```python
class ContextAnalytics:
    """
    Analyze context data for insights and trend detection
    """
    
    def __init__(self, context: UserContext):
        self.context = context
    
    def detect_anomalies(self):
        """
        Detect unusual patterns in user behavior
        """
        anomalies = []
        
        # Food anomalies
        if hasattr(self.context, 'daily_food_intake'):
            recent_avg = self._calc_avg_daily_calories(days=7)
            today_intake = self._calc_today_calories()
            
            if today_intake > recent_avg * 1.5:
                anomalies.append({
                    "type": "high_food_intake",
                    "severity": "medium",
                    "message": f"You've eaten {(today_intake/recent_avg - 1) * 100:.0f}% more than usual"
                })
        
        # Activity anomalies
        if hasattr(self.context, 'activity_entries'):
            if len(self.context.activity_entries) == 0:
                anomalies.append({
                    "type": "no_activity",
                    "severity": "medium",
                    "message": "You haven't logged any activity recently"
                })
        
        return anomalies
    
    def generate_insights(self):
        """
        Generate actionable insights from context data
        """
        insights = []
        
        # Pattern-based insights
        if hasattr(self.context, 'meal_patterns'):
            most_common_meals = sorted(
                self.context.meal_patterns.items(),
                key=lambda x: x[1]["frequency"],
                reverse=True
            )[:3]
            
            insights.append({
                "insight_type": "meal_pattern",
                "finding": f"Most common meals: {', '.join([m[0] for m in most_common_meals])}",
                "actionable": True,
                "recommendation": "Try to reduce high-carb meals"
            })
        
        return insights
```

---

## 🔄 Context Update Cycle

```
┌─────────────────────────────────────┐
│      User Interaction (Voice)       │
└──────────────┬──────────────────────┘
               │
               ▼
    ┌──────────────────────┐
    │  Parse Intent/Data   │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │  Update Context Engines:     │
    │  - Food Context              │
    │  - Activity Context          │
    │  - Health Metrics Context    │
    │  - Temporal Context          │
    └──────────┬───────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │  Analyze Patterns             │
    │  Generate Insights            │
    │  Detect Anomalies             │
    └──────────┬───────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │  Context-Aware Response       │
    │  Generation                   │
    └──────────┬───────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │  Persist to Database          │
    └──────────────────────────────┘
```

---

## 📈 Implementation Roadmap

### Phase 1: Basic Context (Week 1-2)
- [ ] Implement UserProfile and UserContext
- [ ] FoodContextEngine with basic logging
- [ ] Simple pattern detection

### Phase 2: Advanced Tracking (Week 3-4)
- [ ] ActivityContextEngine
- [ ] HealthMetricsContext with glucose analysis
- [ ] Temporal pattern detection

### Phase 3: Intelligence (Week 5)
- [ ] Anomaly detection
- [ ] Proactive message generation
- [ ] Recommendation personalization

---

## 📚 Dependencies

```yaml
context_engine_dependencies:
  Core:
    - sqlalchemy>=2.0.0
    - sqlite3  # built-in
    - numpy>=1.24.0
    - pandas>=2.0.0
  
  Analytics:
    - scipy>=1.10.0
    - scikit-learn>=1.2.0
  
  Time Handling:
    - pytz>=2023.3
    - python-dateutil>=2.8.2
```

---

## 🔐 Privacy Considerations

- All context stored locally
- Encryption for sensitive health data
- User controls over data usage
- No external sharing without consent

