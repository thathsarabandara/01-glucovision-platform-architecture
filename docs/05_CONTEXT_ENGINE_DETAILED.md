# 05 - Context Engine (Detailed Architecture)

## 1. Overview
The Context Engine is the intelligence backbone that transforms the voice-first diabetes management system from command-based interaction into a truly conversational, proactive assistant. It maintains comprehensive awareness of user patterns, temporal context, health states, and behavioral history—enabling the system to make informed decisions and offer personalized guidance without explicit user queries.

**Key Principle**: The system doesn't just respond to commands—it understands "why" and "what next"

---

## 2. Context Types & Architecture

### 2.1 User Profile Context
**Responsibility**: Store and manage static and evolving user characteristics

#### Data Model:
```python
class UserProfile:
    """
    Comprehensive user profile for personalized diabetes management
    """
    
    # ========== STATIC PROFILE ==========
    user_id: str
    age: int
    gender: str  # 'M' / 'F' / Other
    
    # ========== DIABETES INFORMATION ==========
    diabetes_type: str  # 'Type 1', 'Type 2', 'Pre-diabetes'
    diabetes_duration_years: int
    hba1c_target: float  # e.g., 7.0 for most Type 2
    fasting_glucose_range: tuple  # (80, 130)
    
    # ========== ANTHROPOMETRIC DATA ==========
    height_cm: float
    weight_kg: float
    bmi: float  # Calculated
    
    # Convert to BMI category:
    # < 18.5: Underweight
    # 18.5-24.9: Normal
    # 25-29.9: Overweight
    # ≥ 30: Obese
    
    # ========== LIFESTYLE DATA ==========
    occupation: str  # 'sedentary', 'moderate', 'active'
    exercise_frequency: str  # 'never', 'occasionally', 'regularly'
    preferred_exercise: list  # ['walking', 'swimming', 'cycling']
    
    # ========== DIETARY DATA ==========
    dietary_preferences: list  # ['vegetarian', 'non-vegetarian', etc.]
    food_allergies: list
    disliked_foods: list
    preferred_cuisines: list  # ['Sinhala', 'Indian', 'Continental']
    
    # ========== HEALTH HISTORY ==========
    medications: list  # [{'name': 'Metformin', 'dose': '1000mg', 'frequency': 'twice daily'}]
    comorbidities: list  # ['hypertension', 'kidney disease', 'neuropathy']
    hypoglycemia_frequency: str  # 'rare', 'occasional', 'frequent'
    
    # ========== PREFERENCES ==========
    preferred_language: str  # 'english', 'sinhala', 'tamil'
    tone_preference: str  # 'encouraging', 'neutral', 'cautious'
    notification_frequency: str  # 'frequent', 'moderate', 'minimal'
    notification_channels: list  # ['voice', 'text', 'ui']
    
    # ========== SYSTEM PREFERENCES ==========
    offline_mode_enabled: bool
    real_time_prediction_enabled: bool
    image_analysis_enabled: bool
    
    # Timestamp tracking
    created_at: datetime
    updated_at: datetime
    next_profile_review: datetime  # Quarterly
```

#### Profile Storage:
```python
# Database schema (PostgreSQL)
TABLE users {
    id: UUID PRIMARY KEY
    profile_data: JSONB  # Flexible storage for evolving profile
    version: INT  # Track profile changes
    updated_at: TIMESTAMP
    encrypted_fields: ARRAY[TEXT]  # Which fields are encrypted
}
```

---

### 2.2 Temporal Context
**Responsibility**: Track time-based patterns and current temporal state

#### Time Components:
```python
class TemporalContext:
    """
    Manage time-based context for food timing, insulin timing, etc.
    """
    
    # ========== CURRENT TIME STATE ==========
    current_datetime: datetime
    day_of_week: str  # 'Monday', 'Tuesday', etc.
    time_of_day: str  # 'morning', 'afternoon', 'evening', 'night'
    
    # ========== MEAL TIMING STATE ==========
    last_meal_time: datetime
    hours_since_last_meal: float
    next_expected_meal: str  # 'breakfast', 'lunch', 'snack', 'dinner'
    expected_meal_time: datetime
    
    # Time-based meal expectations:
    MEAL_SCHEDULE = {
        'breakfast': {
            'typical_time': '07:00-09:00',
            'duration': 30,  # minutes
            'typical_carbs': 45,  # grams
            'insulin_peak': 2  # hours post-meal
        },
        'lunch': {
            'typical_time': '12:00-13:30',
            'duration': 45,
            'typical_carbs': 60,
            'insulin_peak': 2.5
        },
        'dinner': {
            'typical_time': '19:00-20:30',
            'duration': 45,
            'typical_carbs': 50,
            'insulin_peak': 2.5
        },
        'snack': {
            'typical_time': 'flexible',
            'duration': 10,
            'typical_carbs': 15,
            'insulin_peak': 1.5
        }
    }
    
    # ========== ACTIVITY TIMING ==========
    last_activity_time: datetime
    daily_activity_minutes: int
    activity_pattern: str  # 'sedentary', 'light', 'moderate', 'vigorous'
    
    # ========== SLEEP CONTEXT ==========
    last_sleep_time: datetime
    expected_sleep_time: datetime
    average_sleep_hours: float
    sleep_quality: str  # 'poor', 'fair', 'good'
    
    # ========== TEMPORAL PATTERNS ==========
    is_weekend: bool
    is_holiday: bool
    is_sick_day: bool
    is_stress_period: bool


    def get_recommended_action(self):
        """
        Based on temporal context, suggest proactive actions
        """
        
        # Example: 2 hours since lunch, glucose likely declining
        if hours_since_last_meal == 2:
            return {
                'action': 'CHECK_GLUCOSE_PREDICTION',
                'reason': 'Post-meal glucose peak duration exhausted',
                'message': 'Your glucose is likely returning to normal. Let me check...'
            }
        
        # Example: No activity logged, typically sedentary period
        if time_of_day == 'afternoon' and daily_activity_minutes < 2000:
            return {
                'action': 'SUGGEST_ACTIVITY',
                'reason': 'Activity below daily goal',
                'message': 'You\'ve been sitting for a while. A 10-minute walk helps glucose management.'
            }
```

#### Time-Based Smart Notifications:
```python
SMART_NOTIFICATION_TIMING = {
    'pre_meal': {
        'condition': 'predicted_low_glucose AND approaching_meal_time',
        'message': 'You might feel low blood sugar soon. Ready to eat?',
        'action': 'remind_to_check_or_eat'
    },
    'post_meal': {
        'condition': 'meal_logged AND 2_hours_passed',
        'message': 'Your glucose should be back to normal now. How are you feeling?',
        'action': 'check_wellbeing'
    },
    'activity_boost': {
        'condition': 'high_carb_meal AND no_activity_yet',
        'message': 'That was a carb-heavy meal. A short walk would help balance it.',
        'action': 'suggest_activity'
    },
    'bedtime_check': {
        'condition': 'approaching_sleep_time',
        'message': 'Before bed, let me check your predicted overnight glucose.',
        'action': 'predict_glucose_overnight'
    }
}
```

---

### 2.3 Health State Context
**Responsibility**: Track real-time glucose patterns, insulin dynamics, and physiological state

#### Health Metrics Data Model:
```python
class HealthStateContext:
    """
    Real-time health state tracking
    """
    
    # ========== GLUCOSE STATE ==========
    current_glucose: float  # mg/dL
    glucose_trend: str  # 'rapidly_rising', 'rising', 'stable', 'falling', 'rapidly_falling'
    glucose_trend_arrow: str  # ↑↑ ↑ → ↓ ↓↓
    
    # Glucose zones
    GLUCOSE_ZONES = {
        'hypoglycemia': (0, 70),
        'low': (70, 100),
        'target': (100, 180),  # Individual variation
        'high': (180, 240),
        'critical_high': (240, 400)
    }
    
    # ========== INSULIN STATE ==========
    insulin_dose_today: float  # units (for insulin users)
    insulin_on_board: float  # IOB - active insulin remaining
    time_since_last_insulin: float  # minutes
    insulin_type: str  # 'rapid', 'long_acting', 'mixed'
    
    # Insulin action profile
    INSULIN_PROFILES = {
        'rapid': {
            'onset': 10,  # minutes
            'peak': 60,
            'duration': 300  # 5 hours
        },
        'long_acting': {
            'onset': 60,
            'peak': 600,  # 10 hours
            'duration': 1440  # 24 hours
        }
    }
    
    # ========== CARB STATE ==========
    carbs_consumed_today: float  # grams
    carbs_in_system: float  # estimated grams, accounting for digestion
    active_carb_duration: int  # minutes remaining
    
    # Carb counting reference
    CARB_IMPACT_BY_TYPE = {
        'simple_carbs': {
            'absorption_time': 30,  # minutes
            'force_multiplier': 1.2  # 20% faster glucose rise
        },
        'complex_carbs': {
            'absorption_time': 90,
            'force_multiplier': 1.0
        },
        'high_fiber': {
            'absorption_time': 120,
            'force_multiplier': 0.8  # Slower glucose impact
        }
    }
    
    # ========== ACTIVITY STATE ==========
    current_activity_level: str  # 'rest', 'light', 'moderate', 'vigorous'
    activity_duration: int  # minutes
    activity_type: str  # 'walking', 'running', 'strength'
    
    # Activity glucose impact
    ACTIVITY_GLUCOSE_IMPACT = {
        'light_activity': {
            'glucose_drop_rate': 2,  # mg/dL per 10 minutes
            'insulin_sensitivity_increase': 1.3  # 30% more insulin sensitive
        },
        'moderate_activity': {
            'glucose_drop_rate': 4,
            'insulin_sensitivity_increase': 1.5
        },
        'vigorous_activity': {
            'glucose_drop_rate': 6,
            'insulin_sensitivity_increase': 1.8
        }
    }
    
    # ========== STRESS & SLEEP STATE ==========
    stress_level: str  # 'low', 'moderate', 'high'
    sleep_hours_last_night: float
    sleep_quality_rating: int  # 1-10
    
    # Stress/sleep impact on glucose
    STRESS_IMPACT = {
        'high_stress': {
            'glucose_increase': '+15-20 mg/dL',
            'mechanism': 'cortisol release'
        },
        'poor_sleep': {
            'glucose_increase': '+10-15 mg/dL',
            'mechanism': 'reduced insulin sensitivity'
        }
    }
    
    # ========== HYDRATION STATE ==========
    water_intake_ml: int  # today
    hydration_level: str  # 'well_hydrated', 'mildly_dehydrated', 'dehydrated'
    
    # ========== ILLNESS STATE ==========
    is_sick: bool
    illness_type: str  # 'cold', 'flu', 'infection'
    fever_celsius: float  # None if no fever
    
    # Illness impacts glucose management
    ILLNESS_IMPACTS = {
        'infection': {
            'glucose_effect': '+20-50 mg/dL (usually)',
            'insulin_requirement': 'May increase'
        },
        'stress_hormone_related': {
            'glucose_effect': 'Unpredictable',
            'recommendation': 'Check glucose more frequently'
        }
    }


class HealthStateUpdater:
    """
    Continuously track and update health state
    """
    
    def update_from_glucose_meter(self, glucose_reading):
        """Update when user measures glucose"""
        self.current_glucose = glucose_reading['value']
        self.glucose_trend = self._calculate_trend(glucose_reading)
        
        # Cross-check with prediction
        predicted_glucose = self.predict_glucose(minutes_ahead=current_time)
        prediction_error = abs(glucose_reading['value'] - predicted_glucose)
        
        if prediction_error > 20:
            log_alert("Large prediction error. Updating model calibration...")
    
    def update_from_activity_log(self, activity):
        """Update when user logs activity"""
        self.current_activity_level = activity['intensity']
        self.activity_duration += activity['duration']
        
        # Adjust insulin impact expectations
        glucose_impact = self._calculate_activity_glucose_impact(activity)
        self.next_predicted_glucose -= glucose_impact
    
    def update_from_meal_log(self, meal):
        """Update when user logs meal"""
        self.carbs_consumed_today += meal['carbs']
        self.carbs_in_system = meal['carbs']
        self.active_carb_duration = self._estimate_digestion_time(meal)
        
        message = f"Logged {meal['carbs']}g carbs. Peak glucose expected in {meal_carbs}:  minutes"
        return message
```

---

### 2.4 Behavioral Context
**Responsibility**: Learn and track user patterns, preferences, and behaviors

#### Behavioral Tracking:
```python
class BehavioralContext:
    """
    Learn and adapt to individual user behaviors
    """
    
    # ========== FOOD PATTERNS ==========
    favorite_foods: list
    favorite_meals_by_time: dict  # {time: [meals]}
    food_portion_preferences: dict  # {food: average_portion}
    frequent_combinations: list  # Foods often eaten together
    
    FOOD_PATTERN_EXAMPLE = {
        'user_123': {
            'monday_breakfast': ['bread', 'eggs', 'tea'],  # 47% consistency
            'tuesday_lunch': ['rice', 'curry'],  # 65% consistency
            'weekend_eating': ['restaurant_food'],  # 78% consistency
        }
    }
    
    # ========== ACTIVITY PATTERNS ==========
    preferred_exercise_time: str  # 'morning', 'afternoon', 'evening'
    exercise_consistency: float  # 0-1, how often they exercise when planned
    typical_weekly_activity_minutes: int
    
    # ========== GLUCOSE RESPONSE PATTERNS ==========
    food_glucose_sensitivity: dict
    # Example:
    # {
    #     'white_rice': 1.2,  # 20% stronger glucose response than average
    #     'whole_wheat': 0.9,  # 10% weaker response
    #     'indian_curry': 1.1
    # }
    
    # ========== MEDICATION ADHERENCE ==========
    medication_adherence_rate: float  # 0-1
    typical_medication_times: dict
    missed_doses_monthly: int
    
    # ========== STRESS & LIFE EVENTS ==========
    major_life_events: list
    # [
    #     {'event': 'job_change', 'date': '2024-01-15', 'glucose_impact': 'high'},
    #     {'event': 'vacation', 'date': '2024-03-10', 'glucose_impact': 'variable'}
    # ]
    
    # ========== COMMUNICATION PREFERENCES ==========
    response_to_encouragement: str  # 'positive', 'neutral', 'demotivating'
    prefers_detailed_explanations: bool
    preferred_communication_tone: str  # 'formal', 'casual', 'motivational'
    
    # ========== LEARNING PROFILE ==========
    how_user_learns: str  # 'examples', 'explanations', 'data_visualization'
    attention_span: str  # 'short', 'medium', 'long'
    preferred_advice_length: str  # 'brief', 'detailed', 'comprehensive'


def learn_user_patterns():
    """
    Machine learning to discover behavioral patterns
    """
    
    # Algorithm: Time-series clustering + frequent itemset mining
    
    # Example discovery:
    # Pattern 1: "Every Tuesday at 1 PM, user eats rice with curry"
    # Pattern 2: "After exercise, user needs snack within 1 hour"
    # Pattern 3: "On Fridays, glucose Control is worse (eating out)"
    
    patterns = {
        'temporal_patterns': {
            'day_specific': identify_day_patterns(),
            'time_specific': identify_time_patterns(),
            'cyclical': identify_weekly_cycles()
        },
        'food_patterns': {
            'favorites': identify_favorite_foods(),
            'combinations': identify_food_pairs(),
            'timing': identify_meal_times()
        },
        'glucose_patterns': {
            'post_meal_response': learn_individual_carb_sensitivity(),
            'activity_response': learn_activity_glucose_impact(),
            'stress_response': learn_stress_glucose_correlation()
        },
        'lifestyle_patterns': {
            'exercise_routine': identify_exercise_schedule(),
            'sleep_schedule': identify_sleep_patterns(),
            'stress_periods': identify_stressful_times()
        }
    }
    
    return patterns
```

---

## 3. Context Integration Engine

### 3.1 Real-Time Context Synthesis:
```python
class ContextSynthesizer:
    """
    Combine all context types into holistic understanding
    """
    
    def synthesize_current_context(self):
        """
        Create unified picture of user's current situation
        """
        
        context = {
            'user_profile': self.user_profile,
            'temporal': self.temporal_context,
            'health': self.health_state,
            'behavioral': self.behavioral_context
        }
        
        # Example synthesis:
        synthesis = {
            'current_situation': {
                'user': "Amal, 32-year-old with Type 2 diabetes",
                'time': "Tuesday, 1:00 PM (lunch time)",
                'health': "Glucose 145 mg/dL (slightly high), stable trend",
                'recent_activity': "No exercise yet this week",
                'context': "Work day, moderate stress"
            },
            'patterns': {
                'typical_tuesday': "Usually eats rice with curry at this time",
                'glucose_sensitivity': "Curries cause 25% stronger glucose response",
                'activity': "Usually walks after lunch on Tuesdays",
                'stress': "Work is typically stressful on Tuesdays"
            },
            'risk_factors': {
                'high_carb_day': "Already consumed 80g carbs",
                'no_activity': "Hasn't exercised in 5 days",
                'stress': "Work pressure is high"
            },
            'opportunities': {
                'timing': "Perfect time for 15-min walk post-meal",
                'education': "Can discuss portion control options",
                'motivation': "Recent 2-day walking streak - keep momentum"
            }
        }
        
        return synthesis
    
    def calculate_context_confidence(self):
        """
        Measure certainty of context understanding
        """
        confidence_factors = {
            'user_profile_completeness': 0.95,  # 95% complete profile
            'health_data_freshness': 0.90,  # Glucose measured 10 min ago
            'behavioral_data_sample_size': 0.85,  # 2 weeks of data
            'temporal_accuracy': 0.99,  # System time is accurate
        }
        
        overall_confidence = geometric_mean(confidence_factors.values())
        return overall_confidence
```

---

### 3.2 Proactive Action Engine:
```python
class ProactiveActionEngine:
    """
    Based on context, determine what to do without user asking
    """
    
    def determine_proactive_actions(self):
        """
        What should AI initiate conversation about?
        """
        
        actions = []
        
        # ========== MEDICAL ALERTS ==========
        if self.health_state.current_glucose < 70:
            actions.append({
                'priority': 'CRITICAL',
                'type': 'HYPOGLYCEMIA_ALERT',
                'message': 'Your glucose is low! Eat something with quick carbs now.',
                'trigger_immediately': True
            })
        
        # ========== PATTERN ALERTS ==========
        if self._is_pattern_deviation():
            actions.append({
                'priority': 'HIGH',
                'type': 'PATTERN_DEVIATION',
                'message': 'You\'re usually eating lunch by now. Everything okay?',
                'trigger_timing': 'next_interaction'
            })
        
        # ========== PREDICTIVE ALERTS ==========
        if self._predict_high_glucose():
            actions.append({
                'priority': 'MEDIUM',
                'type': 'PREDICTIVE_ALERT',
                'message': 'Glucose may spike soon. Ready for that walk?',
                'trigger_timing': 'next_interaction'
            })
        
        # ========== MOTIVATIONAL ==========
        if self._is_motivation_opportunity():
            actions.append({
                'priority': 'LOW',
                'type': 'ENCOURAGEMENT',
                'message': 'You\'ve been managing glucose well this week!',
                'trigger_timing': 'next_interaction'
            })
        
        # ========== EDUCATIONAL ==========
        if self._should_educate_on_topic():
            actions.append({
                'priority': 'LOW',
                'type': 'EDUCATION',
                'message': 'Did you know that whole wheat has a slower glucose impact?',
                'trigger_timing': 'next_interaction'
            })
        
        # Sort by priority
        actions.sort(key=lambda x: ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW'].index(x['priority']))
        
        return actions

    def _is_pattern_deviation(self):
        """Detect if user is deviating from typical patterns"""
        expected_meal = self.behavioral_context.get_expected_meal()
        current_time = datetime.now()
        
        # If typically eating by now but hasn't logged meal
        if expected_meal and (current_time - expected_meal['time']).minutes > 15:
            if not self.temporal_context.meal_logged_today:
                return True
        
        return False
    
    def _predict_high_glucose(self):
        """Use predictive model to anticipate high glucose"""
        carbs_in_system = self.health_state.carbs_in_system
        insulin_on_board = self.health_state.insulin_on_board
        current_activity = self.health_state.current_activity_level
        
        # Simplified prediction:
        glucose_projection = (
            self.health_state.current_glucose +
            (carbs_in_system * self.behavioral_context.carb_sensitivity[user_id]) -
            (current_activity * activity_factor) -
            (insulin_on_board * 2)  # Insulin lowers glucose by ~2 points per unit
        )
        
        return glucose_projection > 180
```

---

## 4. Long-Term Learning & Adaptation

### 4.1 User Model Evolution:
```python
class UserModelLearning:
    """
    Continuously improve understanding of individual user
    """
    
    def weekly_pattern_analysis(self):
        """
        Each week, discover new patterns
        """
        
        # Statistical analysis of past week
        weekly_stats = {
            'avg_glucose': 135,
            'glucose_variability': 45,  # mg/dL std dev
            'carb_intake_patterns': identify_carb_patterns(),
            'activity_consistency': 0.65,
            'adherence_rate': 0.92
        }
        
        # Generate insights
        insights = {
            'glucose_control': "Good control this week (avg 135)",
            'best_day': "Wednesday had most stable glucose",
            'worst_time': "Evenings show higher glucose",
            'adherence': "Excellent medication adherence",
            'opportunity': "Activity on weekends would help"
        }
        
        return insights
    
    def personalized_model_updates(self):
        """
        Update prediction models based on individual responses
        """
        
        # Re-calibrate carb sensitivity
        # Original: 1 carb raises glucose by 3 mg/dL
        # User-specific: 1 carb raises glucose by 3.5 mg/dL (higher sensitivity)
        
        # Re-calibrate activity impact
        # Original: 10 min walk lowers glucose by 2 mg/dL
        # User-specific: 10 min walk lowers glucose by 3 mg/dL (more sensitive)
        
        # Re-calibrate stress impact
        # User shows 25 mg/dL glucose rise during work stress
        
        personalized_params = {
            'carb_sensitivity': 1.15,  # 15% higher than population average
            'activity_factor': 1.5,    # 50% more sensitive to exercise
            'stress_factor': 1.8,      # 80% more glucose rise from stress
            'insulin_sensitivity': 1.1 # 10% more insulin-sensitive
        }
        
        return personalized_params
    
    def recommendation_personalization(self):
        """
        Tailor recommendations based on what works for this specific user
        """
        
        recommendations = {
            'for_amal': {
                'portion_strategy': "Amal does better with smaller portions, more frequent meals",
                'activity_strategy': "Amal responds well to afternoon walks post-lunch",
                'food_strategy': "Amal handles whole wheat better than white rice",
                'mood_strategy': "Amal needs positive reinforcement, not warnings",
                'notification_strategy': "Amal prefers 2-3 notifications/day, not constant checking"
            }
        }
        
        return recommendations
```

---

## 5. Context Storage & Retrieval

### 5.1 Database Schema:
```sql
-- User Profile
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY,
    profile_data JSONB,
    version INT,
    updated_at TIMESTAMP
);

-- Temporal Context (Real-time cache)
CREATE TABLE temporal_context (
    user_id UUID PRIMARY KEY,
    last_meal_time TIMESTAMP,
    last_activity_time TIMESTAMP,
    current_time_state VARCHAR,
    FOREIGN KEY (user_id) REFERENCES user_profiles(user_id)
);

-- Health State (Time-series)
CREATE TABLE health_state_history (
    id UUID PRIMARY KEY,
    user_id UUID,
    timestamp TIMESTAMP,
    glucose_value FLOAT,
    glucose_trend VARCHAR,
    insulin_on_board FLOAT,
    carbs_in_system FLOAT,
    activity_level VARCHAR,
    FOREIGN KEY (user_id) REFERENCES user_profiles(user_id),
    INDEX idx_user_timestamp (user_id, timestamp DESC)
);

-- Behavioral Patterns
CREATE TABLE behavioral_patterns (
    user_id UUID PRIMARY KEY,
    patterns_data JSONB,
    last_updated TIMESTAMP,
    confidence_score FLOAT,
    FOREIGN KEY (user_id) REFERENCES user_profiles(user_id)
);

-- Proactive Actions Queue
CREATE TABLE proactive_actions (
    id UUID PRIMARY KEY,
    user_id UUID,
    action_type VARCHAR,
    priority VARCHAR,
    message TEXT,
    trigger_time TIMESTAMP,
    executed BOOLEAN,
    FOREIGN KEY (user_id) REFERENCES user_profiles(user_id),
    INDEX idx_pending (user_id, executed)
);
```

### 5.2 Caching Strategy:
```python
# Redis caching for fast context access
cache_strategy = {
    'user_profile': {
        'ttl': 86400,  # 24 hours
        'serialization': 'json',
        'key': 'user:{user_id}:profile'
    },
    'current_health_state': {
        'ttl': 300,  # 5 minutes (frequently updates)
        'serialization': 'json',
        'key': 'user:{user_id}:health:current'
    },
    'temporal_context': {
        'ttl': 60,  # 1 minute (real-time)
        'serialization': 'json',
        'key': 'user:{user_id}:temporal'
    },
    'behavioral_patterns': {
        'ttl': 604800,  # 7 days
        'serialization': 'json',
        'key': 'user:{user_id}:patterns'
    }
}
```

---

## 6. Context-Aware Conversation Examples

### Example 1: Proactive Meal Suggestion
```
Time: 12:45 PM (Tuesday - user's typical lunch time)
Context State:
  - Glucose: 110 mg/dL (slightly low for pre-meal)
  - Activity today: None
  - Expected meal: Lunch
  - Behavioral pattern: Rice + curry on Tuesdays

AI decides to initiate (proactively):
  "I notice it's your usual lunch time and your glucose is on the lower side. 
   Ready for your typical rice and curry? Remember to include a 15-minute walk 
   after eating - it worked great for you last week!"

User: "Yes, actually I was just about to eat"
AI: "Perfect! Enjoy your meal, and I'll check in after you eat."
```

### Example 2: Pattern Deviation Alert
```
Time: 7:30 PM (Friday - user typically relaxed)
Context State:
  - Glucose: 180 mg/dL (trending high)
  - Activity: None
  - Behavioral pattern: "Usually eats out on Fridays"
  - But: No meal logged yet

AI initiates:
  "Hi Amal! I notice it's Friday evening (when you usually go out) but your glucose 
   is running high and I haven't seen a meal logged. Did your plans change? Or are 
   you waiting to log your food?"

User: "Working late, skipped lunch. Eating something now."
AI: "Got it. What are you having? I can help estimate the carbs."
```

### Example 3: Predictive Action
```
Time: 1:15 PM
Context State:
  - Recent meal: Rice + curry (60g carbs, 20 min ago)
  - Predicted glucose peak: 180 mg/dL in 20 minutes
  - Activity: None planned
  - Behavioral pattern: "Responds well to afternoon walks"

AI proactively suggests:
  "Your meal should peak in about 20 minutes. How about we do a 20-minute walk 
   together right now? You handled it great last week, and you still have 5 more 
   minutes on that activity streak!"

User: "Not right now, maybe later"
AI: "No problem! Let's set a reminder for 2:30 if your glucose gets high."
```

---

## 7. Context Engine Monitoring

### 7.1 Performance Metrics:
```python
CONTEXT_METRICS = {
    'context_completeness': {
        'metric': '% fields populated in context',
        'target': '>90%',
        'example': 'User should have 90+ essential context fields'
    },
    'pattern_detection_latency': {
        'metric': 'Time to detect new pattern (hours)',
        'target': '<24h',
        'example': 'Discover repeat pattern within 1 day'
    },
    'prediction_accuracy': {
        'metric': 'Match between predicted and actual glucose',
        'target': '>85%',
        'example': 'Prediction within ±15 mg/dL'
    },
    'proactive_action_relevance': {
        'metric': 'User finds suggestions helpful',
        'target': '>80% positive feedback',
        'example': 'User ratings on suggestions'
    },
    'context_update_freshness': {
        'metric': 'How recent is most important data',
        'target': '<30 minutes',
        'example': 'Health state data not more than 30 min old'
    }
}
```

---

## 8. Privacy & Security in Context Storage

```python
SENSITIVE_CONTEXT_FIELDS = {
    'health_state': ['glucose_value', 'insulin_dose', 'medication'],
    'behavioral': ['eating_patterns', 'sleep_patterns'],
    'profile': ['weight', 'age', 'health_conditions']
}

# Encryption at rest
context_encryption = {
    'algorithm': 'AES-256',
    'key_management': 'AWS KMS',
    'sensitive_fields': SENSITIVE_CONTEXT_FIELDS,
    'data_retention': '90 days then delete (user configurable)'
}
```

---

## 9. Implementation Checklist

- [ ] **User Profile Context**
  - [ ] Database schema design
  - [ ] Profile initialization on signup
  - [ ] Profile update mechanisms
  - [ ] Privacy encryption

- [ ] **Temporal Context**
  - [ ] Clock system integration
  - [ ] Meal timing logic
  - [ ] Time-based pattern detection
  - [ ] Notification scheduling

- [ ] **Health State Context**
  - [ ] Real-time glucose integration
  - [ ] Insulin tracking
  - [ ] Carb tracking
  - [ ] Activity logging
  - [ ] Prediction model integration

- [ ] **Behavioral Context**
  - [ ] Pattern mining algorithms
  - [ ] Food frequency analysis
  - [ ] Activity pattern detection
  - [ ] Glucose response learning

- [ ] **Context Synthesis**
  - [ ] Integration engine
  - [ ] Confidence scoring
  - [ ] Conflict resolution

- [ ] **Proactive Actions**
  - [ ] Action determination logic
  - [ ] Priority system
  - [ ] Notification delivery

- [ ] **Testing & Validation**
  - [ ] Unit tests for each context type
  - [ ] Integration tests
  - [ ] Pattern detection accuracy tests

---

## 10. Technology Stack Summary

```
┌─────────────────────────────────────────────────────────────┐
│ CONTEXT ENGINE                                              │
├─────────────────────────────────────────────────────────────┤
│ Profile Storage    │ PostgreSQL + encrypted JSONB            │
│ Real-time Cache    │ Redis (temporal, health state)          │
│ Time-series DB     │ InfluxDB / TimescaleDB                  │
│ Pattern Mining     │ scikit-learn + custom algorithms        │
│ Prediction Models  │ XGBoost / Neural Networks               │
│ Orchestration      │ Python async + Celery for background    │
│ Monitoring         │ Prometheus + Grafana                    │
└─────────────────────────────────────────────────────────────┘
```

---

**Last Updated**: March 22, 2026
**Status**: Complete
**Next Module**: 06_AI_INTELLIGENCE_MODULES_DETAILED.md
