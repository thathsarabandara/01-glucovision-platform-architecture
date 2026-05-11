# 10 - Digital Twins Architecture
## Virtual Patient Models for Diabetes Management Simulation & Personalization

**Project**: Voice-First Multilingual AI Assistant for Diabetes Management  
**Component**: Digital Twins Infrastructure  
**Version**: 1.0  
**Date**: March 23, 2026  
**Status**: Architecture & Implementation Design  

---

## 1. Executive Summary

Digital Twins are virtual replicas of individual patients created from real-time health and behavioral data. They enable:

1. **Predictive Simulation** - Test interventions before real implementation
2. **Personalized Precision Medicine** - Optimize treatment for individual patient
3. **Early Warning Systems** - Detect potential health crises before they occur
4. **Drug & Food Interaction Modeling** - Predict glucose response to new medications or foods
5. **Lifestyle Intervention Testing** - Simulate impact of exercise, diet changes
6. **Research Without Risk** - Experiment safely on virtual patients

### Key Innovation Points:
1. **Real-Time Digital Duplicate** - Continuously updated with patient data
2. **Individual Glucose Dynamics** - Personalized insulin sensitivity, meal timing
3. **Behavioral Integration** - Models stress, sleep, emotions, social factors
4. **What-If Simulation** - Explore scenarios before real-world implementation
5. **Clinical Decision Support** - Evidence-based recommendations grounded in simulation

---

## 2. Digital Twin Architecture Overview

### 2.1 System Architecture Diagram

```
╔═══════════════════════════════════════════════════════════════════╗
║                    REAL PATIENT (Layer 1)                         ║
║  Data Input: Voice interactions, glucose readings, activity, food ║
╚═══════════════════════════════════════════════════════════════════╝
                                    ↓
                    [Data Collection & Processing]
                                    ↓
╔═══════════════════════════════════════════════════════════════════╗
║            DIGITAL TWIN (Layer 2) - Virtual Patient               ║
║  ┌─────────────────────────────────────────────────────────────┐ ║
║  │ Individual Glucose Dynamics Model                           │ ║
║  │ (DELC: Dynamic Extended Least-Squares Cellular)             │ ║
║  └─────────────────────────────────────────────────────────────┘ ║
║  ┌─────────────────────────────────────────────────────────────┐ ║
║  │ Insulin-Carb-Glucose Relationships (Personalized)           │ ║
║  │ ├─ Insulin-to-Carbs Ratio (ICR)                             │ ║
║  │ ├─ Insulin Sensitivity Factor (ISF)                         │ ║
║  │ ├─ Absorption rates                                          │ ║
║  │ └─ Insulin activity curves                                  │ ║
║  └─────────────────────────────────────────────────────────────┘ ║
║  ┌─────────────────────────────────────────────────────────────┐ ║
║  │ Behavioral & Lifestyle Model                               │ ║
║  │ ├─ Sleep patterns & impact on glucose                      │ ║
║  │ ├─ Stress response curves                                  │ ║
║  │ ├─ Activity energy expenditure                              │ ║
║  │ └─ Social & emotional factors                              │ ║
║  └─────────────────────────────────────────────────────────────┘ ║
║  ┌─────────────────────────────────────────────────────────────┐ ║
║  │ Medication & Food Interaction Database                      │ ║
║  │ ├─ Drug-glucose interaction effects                        │ ║
║  │ ├─ Food glycemic index personalization                    │ ║
║  │ └─ Herb & supplement effects                               │ ║
║  └─────────────────────────────────────────────────────────────┘ ║
╚═══════════════════════════════════════════════════════════════════╝
                                    ↓
                    [Simulation & Prediction]
                                    ↓
╔═══════════════════════════════════════════════════════════════════╗
║              SIMULATION ENGINE (Layer 3)                           ║
║  ├─ What-if scenario testing                                      ║
║  ├─ Intervention outcome prediction                               ║
║  ├─ Risk stratification & alerts                                  ║
║  └─ Clinical decision support                                     ║
╚═══════════════════════════════════════════════════════════════════╝
                                    ↓
                    [Validation & Feedback]
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  RECOMMENDATION & INTERVENTION                                   │
│  ├─ Personalized meal recommendations                           │
│  ├─ Activity suggestions based on glucose impact                │
│  ├─ Medication timing optimization                              │
│  └─ Early warning alerts                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Core Digital Twin Components

### 3.1 Individual Glucose Dynamics Model

```python
import numpy as np
from datetime import datetime, timedelta
from typing import Dict, List, Tuple
from scipy.integrate import odeint
import torch
import torch.nn as nn

class PersonalizedGlucoseDynamicsModel:
    """
    Simulate individual patient glucose dynamics
    Based on Compartmental Models adapted to personal physiology
    
    Uses differential equations to model:
    1. Glucose absorption from food
    2. Insulin secretion/injection
    3. Insulin action on glucose clearance
    """
    
    def __init__(self, patient_profile: Dict):
        """
        Initialize personalized glucose model
        
        Args:
            patient_profile: Individual patient parameters
                - weight_kg, age, diabetes_type
                - insulin_sensitivity_factor (ISF)
                - insulin_carb_ratio (ICR)
                - basal_insulin_rate
                - absorption_delay_time
        """
        
        self.patient_profile = patient_profile
        self.parameter_history = []  # Track parameter evolution
        
        # Initialize personalized parameters
        self.parameters = self._initialize_parameters(patient_profile)
    
    def _initialize_parameters(self, patient_profile: Dict) -> Dict:
        """
        Initialize model parameters from patient profile
        
        These parameters are calibrated to individual physiology
        """
        
        return {
            # Glucose kinetics
            'glucose_distribution_volume': patient_profile.get('weight_kg') / 5.0,  # L
            'glucose_elimination_rate': 0.03,  # 1/min (basal clearance)
            
            # Insulin parameters (HIGHLY PERSONALIZED)
            'insulin_sensitivity_factor': \
                patient_profile.get('insulin_sensitivity_factor', 1800.0 / patient_profile.get('total_daily_insulin', 50)),
            'insulin_carb_ratio': \
                patient_profile.get('insulin_carb_ratio', 500.0 / patient_profile.get('total_daily_insulin', 50)),
            
            # Absorption parameters
            'glucose_absorption_rate': 0.04,  # fraction/min
            'disturbance_delay': 40,  # minutes
            'carb_absorption_time_constant': 45,  # minutes
            
            # Insulin action
            'rapid_insulin_peak_time': 80,  # minutes (for rapid-acting insulin)
            'long_insulin_duration': 300,  # minutes (for basal insulin)
            'insulin_elimination_rate': 0.05,  # 1/min
            
            # Individual correction factors
            'stress_multiplier': 1.0,  # Stress increases glucose
            'activity_multiplier': 1.0,  # Activity decreases glucose
            'sleep_quality_factor': 1.0  # Poor sleep raises glucose
        }
    
    def simulate_glucose_trajectory(self,
                                   current_glucose: float,  # mg/dL
                                   meal_carbs: float,  # grams
                                   meal_fat: float,  # grams
                                   meal_protein: float,  # grams
                                   insulin_bolus: float = 0,  # units
                                   activity_level: str = 'sedentary',  # sedentary, light, moderate, vigorous
                                   hours_ahead: int = 4,  # Predict 4 hours
                                   context: Dict = None) -> Dict:
        """
        Simulate glucose response to meal + insulin + activity
        
        Returns:
            Dictionary with:
            - time_points: Array of time values (minutes)
            - glucose_trajectory: Predicted glucose values
            - peak_glucose: Maximum glucose during period
            - min_glucose: Minimum glucose during period
            - time_to_peak: When maximum occurs
            - hypoglycemia_risk: Probability of low glucose
            - recommendations: Suggested actions
        """
        
        if context is None:
            context = {}
        
        # Initialize state variables
        state = {
            'glucose': current_glucose,  # mg/dL
            'carbs_in_system': 0,  # grams
            'insulin_in_system': 0,  # units
            'insulin_on_board': 0  # units still acting
        }
        
        # Adjust parameters based on context
        adjusted_params = self._adjust_parameters(context)
        
        # Time points for simulation
        dt = 5  # 5-minute intervals
        total_minutes = hours_ahead * 60
        time_points = np.arange(0, total_minutes, dt)
        
        # Storage for trajectory
        glucose_trajectory = []
        carbs_trajectory = []
        insulin_trajectory = []
        
        # Run simulation
        for t in time_points:
            # Calculate derivatives
            dG_dt, dC_dt, dI_dt = self._calculate_derivatives(
                state, 
                meal_carbs, meal_fat, meal_protein,
                insulin_bolus,
                t,
                activity_level,
                adjusted_params
            )
            
            # Update state (Euler method)
            state['glucose'] += dG_dt * dt / 60  # Convert to mg/dL
            state['carbs_in_system'] += dC_dt * dt / 60
            state['insulin_in_system'] += dI_dt * dt / 60
            
            # Bounds checking
            state['glucose'] = max(20, min(400, state['glucose']))  # Clip to realistic range
            state['carbs_in_system'] = max(0, state['carbs_in_system'])
            state['insulin_in_system'] = max(0, state['insulin_in_system'])
            
            # Store trajectory
            glucose_trajectory.append(state['glucose'])
            carbs_trajectory.append(state['carbs_in_system'])
            insulin_trajectory.append(state['insulin_in_system'])
        
        # Analyze trajectory for insights
        glucose_array = np.array(glucose_trajectory)
        peak_idx = np.argmax(glucose_array)
        min_idx = np.argmin(glucose_array)
        
        # Risk assessment
        hypoglycemia_risk = np.sum(glucose_array < 70) / len(glucose_array)  # % time < 70 mg/dL
        hyperglycemia_risk = np.sum(glucose_array > 180) / len(glucose_array)  # % time > 180 mg/dL
        
        return {
            'time_points': time_points,
            'glucose_trajectory': glucose_trajectory,
            'carbs_trajectory': carbs_trajectory,
            'insulin_trajectory': insulin_trajectory,
            'peak_glucose': float(glucose_array[peak_idx]),
            'min_glucose': float(glucose_array[min_idx]),
            'time_to_peak_minutes': int(time_points[peak_idx]),
            'average_glucose': float(np.mean(glucose_array)),
            'glucose_variability': float(np.std(glucose_array)),
            'hypoglycemia_risk_percentage': float(hypoglycemia_risk * 100),
            'hyperglycemia_risk_percentage': float(hyperglycemia_risk * 100),
            'recommendations': self._generate_recommendations(glucose_array, state)
        }
    
    def _calculate_derivatives(self,
                              state: Dict,
                              meal_carbs: float,
                              meal_fat: float,
                              meal_protein: float,
                              insulin_bolus: float,
                              time_since_meal: float,
                              activity_level: str,
                              params: Dict) -> Tuple[float, float, float]:
        """
        Calculate rate of change for glucose compartment model
        
        Uses system of differential equations:
        dG/dt = -k_g * G + (absorbed carbs) + (basal glucose production)
                - (insulin effect on glucose clearance)
        
        dCarbs/dt = -k_carbs * Carbs + (meal carbs input)
        
        dInsulin/dt = -k_insulin * Insulin + (insulin input)
        """
        
        # Glucose elimination (basal clearance)
        glucose_clearance = params['glucose_elimination_rate'] * state['glucose']
        
        # Carbohydrate absorption (from meal)
        carb_absorption = 0
        if time_since_meal < 300:  # Within 5 hours of meal
            # Gaussian-like absorption curve
            absorption_rate = self._meal_absorption_rate(
                time_since_meal,
                params['carb_absorption_time_constant']
            )
            carb_absorption = absorption_rate * meal_carbs
        
        # Insulin effect on glucose (glucose lowering)
        insulin_effect = params['insulin_sensitivity_factor'] * state['insulin_in_system']
        
        # Activity effect (increases glucose clearance)
        activity_factor = self._get_activity_factor(activity_level)
        
        # Stress & sleep effects (modify glucose production)
        stress_effect = self.parameters['stress_multiplier']
        sleep_effect = self.parameters['sleep_quality_factor']
        
        # Calculate glucose rate of change
        dG_dt = (-glucose_clearance - insulin_effect + carb_absorption) * stress_effect * sleep_effect * activity_factor
        
        # Carbohydrate depletion
        dC_dt = -0.04 * state['carbs_in_system']  # Depletion rate
        
        # Insulin action
        if insulin_bolus > 0 and time_since_meal < 10:
            # Insulin injection/infusion (fast distribution)
            dI_dt = insulin_bolus - 0.05 * state['insulin_in_system']
        else:
            # Insulin decay
            dI_dt = -0.05 * state['insulin_in_system']
        
        return dG_dt, dC_dt, dI_dt
    
    def _meal_absorption_rate(self, time_since_meal: float, time_constant: float) -> float:
        """
        Model carb absorption as exponential decay from maximum
        
        Peaks around time_constant, then decays
        """
        
        if time_since_meal < 0:
            return 0
        
        # Gaussian-like curve for absorption
        return np.exp(-(((time_since_meal - time_constant) ** 2) / (2 * time_constant ** 2)))
    
    def _get_activity_factor(self, activity_level: str) -> float:
        """
        Activity increases glucose clearance (muscle utilization)
        """
        
        factors = {
            'sedentary': 1.0,
            'light': 1.2,
            'moderate': 1.5,
            'vigorous': 2.0
        }
        
        return factors.get(activity_level, 1.0)
    
    def _adjust_parameters(self, context: Dict) -> Dict:
        """
        Adjust parameters based on contextual factors
        """
        
        adjusted = self.parameters.copy()
        
        if 'sleep_quality' in context:
            # Poor sleep increases glucose production
            quality_map = {'poor': 1.3, 'fair': 1.15, 'good': 1.0, 'excellent': 0.95}
            adjusted['stress_multiplier'] *= quality_map.get(context['sleep_quality'], 1.0)
        
        if 'stress_level' in context:
            stress_map = {'low': 0.95, 'moderate': 1.0, 'high': 1.3, 'very_high': 1.6}
            adjusted['stress_multiplier'] *= stress_map.get(context['stress_level'], 1.0)
        
        if 'menstrual_cycle_phase' in context:
            # Hormonal changes affect insulin sensitivity
            phase_map = {'follicular': 1.0, 'ovulation': 0.85, 'luteal': 1.2}
            adjusted['insulin_sensitivity_factor'] *= phase_map.get(context['menstrual_cycle_phase'], 1.0)
        
        if 'recent_illness' in context:
            # Illness increases insulin resistance
            adjusted['insulin_sensitivity_factor'] *= 0.7
        
        return adjusted
    
    def _generate_recommendations(self, glucose_array: np.ndarray, state: Dict) -> List[str]:
        """
        Generate action recommendations based on predicted trajectory
        """
        
        recommendations = []
        
        if np.max(glucose_array) > 180:
            recommendations.append("⚠️ ALERT: Peak glucose predicted above 180 mg/dL. Consider additional insulin.")
        
        if np.min(glucose_array) < 70:
            recommendations.append("⚠️ ALERT: Hypoglycemia risk detected. Have fast-acting carbs available.")
        
        if np.std(glucose_array) > 80:
            recommendations.append("📊 High glucose variability predicted. Check meal composition and timing.")
        
        time_in_range = np.sum((glucose_array >= 70) & (glucose_array <= 180)) / len(glucose_array)
        if time_in_range > 0.8:
            recommendations.append("✓ GOOD: Predicted glucose control within target range >80% of time.")
        
        return recommendations
```

### 3.2 Personalized Insulin-Carb Dynamics

```python
class InsulinCarbDynamicsModel:
    """
    Model insulin-carbohydrate-glucose interactions specific to individual
    """
    
    def __init__(self, patient_id: str, calibration_data: List[Dict]):
        """
        Initialize model from patient's historical data
        
        Args:
            patient_id: Unique patient identifier
            calibration_data: List of {time, glucose, carbs, insulin} records
        """
        
        self.patient_id = patient_id
        self.calibration_data = calibration_data
        
        # Calibrate personalized parameters from data
        self.icr = self._calibrate_insulin_carb_ratio()  # Units insulin per grams carbs
        self.isf = self._calibrate_insulin_sensitivity_factor()  # mg/dL per unit insulin
        self.absorption_time = self._calibrate_absorption_time()  # Minutes
    
    def _calibrate_insulin_carb_ratio(self) -> float:
        """
        Determine how many grams of carbs 1 unit of insulin covers
        
        Method: Analyze meal logs where insulin and carbs are known
        """
        
        # Filter data for meals with insulin and known carbs
        meal_events = [d for d in self.calibration_data if d.get('insulin', 0) > 0 and d.get('carbs', 0) > 0]
        
        if not meal_events:
            return 10.0  # Default: 1 unit covers 10g carbs
        
        # Calculate ICR from multiple meals
        ratios = [d['carbs'] / d['insulin'] for d in meal_events]
        icr = np.median(ratios)
        
        return float(icr)
    
    def _calibrate_insulin_sensitivity_factor(self) -> float:
        """
        Determine how much insulin lowers glucose
        
        Method: Use Rule of 1800
        ISF = 1800 / Total Daily Insulin
        Then refine with actual data
        """
        
        # Filter glucose change observations
        observations = []
        
        for i in range(1, len(self.calibration_data)):
            prev = self.calibration_data[i-1]
            curr = self.calibration_data[i]
            
            glucose_change = curr.get('glucose', 0) - prev.get('glucose', 0)
            insulin_given = prev.get('insulin', 0)
            
            if insulin_given > 0 and glucose_change < 0:  # Insulin lowered glucose
                isf = abs(glucose_change) / insulin_given
                observations.append(isf)
        
        if not observations:
            return 50.0  # Default: 1 unit lowers glucose 50 mg/dL
        
        return float(np.median(observations))
    
    def _calibrate_absorption_time(self) -> float:
        """
        Determine how quickly patient absorbs carbohydrates
        
        Method: Analyze time to peak glucose after meals
        """
        
        # Find peak times after meals
        peak_times = []
        
        for i in range(len(self.calibration_data)):
            if self.calibration_data[i].get('meal_type'):
                # Look for peak in next 4 hours
                future_glucose = [self.calibration_data[j].get('glucose', 0) 
                                for j in range(i, min(i+48, len(self.calibration_data)))]
                if future_glucose:
                    peak_idx = np.argmax(future_glucose)
                    peak_times.append(peak_idx * 5)  # 5-minute intervals
        
        if not peak_times:
            return 45.0  # Default: 45 minutes
        
        return float(np.median(peak_times))
```

---

## 4. Behavioral & Lifestyle Model

### 4.1 Sleep Impact on Glucose

```python
class SleepGlucoseModel:
    """
    Model how sleep patterns affect glucose dynamics the next day
    """
    
    def __init__(self):
        self.sleep_history = []
        self.glucose_history = []
    
    def analyze_sleep_impact(self, sleep_data: Dict) -> Dict:
        """
        Analyze how sleep duration/quality predicts next-day glucose
        
        Sleep impacts:
        1. Fasting glucose (poor sleep → higher baseline)
        2. Insulin sensitivity (reduced sleep → insulin resistance)
        3. Glucose variability (poor sleep → more volatile)
        """
        
        sleep_quality_score = sleep_data.get('quality_rating', 5) / 10  # 0-1
        sleep_duration = sleep_data.get('duration_hours', 7)
        sleep_consistency = sleep_data.get('bedtime_consistency', 0.8)  # 0-1
        
        # Model coefficients (from literature + personal data)
        baseline_glucose_impact = 1 + (7 - sleep_duration) * 5  # +5 mg/dL per hour of lost sleep
        insulin_sensitivity_reduction = 1 - (sleep_quality_score * 0.3)  # Up to 30% reduction
        variability_increase = 1 + (1 - sleep_quality_score) * 0.5  # More variability with poor sleep
        
        return {
            'predicted_fasting_glucose_adjustment': baseline_glucose_impact,
            'insulin_sensitivity_multiplier': insulin_sensitivity_reduction,
            'glucose_variability_multiplier': variability_increase,
            'recommendation': self._generate_sleep_recommendation(sleep_quality_score, sleep_duration)
        }
    
    def _generate_sleep_recommendation(self, quality: float, duration: float) -> str:
        recommendations = {
            (quality < 0.5, duration < 6): "🔴 URGENT: Very poor sleep. Expect higher glucose variability. Monitor closely.",
            (quality < 0.7, duration < 7): "⚠️ Suboptimal sleep. Plan easier glucose management day.",
            (quality > 0.8, duration >= 7): "✓ Good sleep. Expect normal glucose response.",
        }
        
        for (q_bad, d_short), msg in recommendations.items():
            if quality < 0.5 or duration < 6:
                return "🔴 URGENT: Very poor sleep. Monitor glucose closely."
            elif quality < 0.7 or duration < 7:
                return "⚠️ Suboptimal sleep. Plan easier glucose management day."
        
        return "✓ Good sleep. Normal glucose response expected."
```

### 4.2 Stress Impact Model

```python
class StressGlucoseModel:
    """
    Model stress hormone effects on glucose
    """
    
    def __init__(self):
        self.stress_events = []
    
    def calculate_stress_response(self, stress_context: Dict) -> Dict:
        """
        Predict glucose impact of stress
        
        Stress increases glucose through:
        1. Cortisol release → glucose production
        2. Adrenaline → insulin resistance
        3. Reduced physical activity
        """
        
        stress_level = stress_context.get('stress_level', 'moderate')  # low, moderate, high, very_high
        stress_duration = stress_context.get('duration_hours', 1)
        stress_type = stress_context.get('type', 'work')  # work, family, health, financial, etc.
        
        # Stress response curves (mg/dL increase in glucose)
        stress_response_map = {
            'low': 5,
            'moderate': 15,
            'high': 35,
            'very_high': 60
        }
        
        glucose_increase = stress_response_map.get(stress_level, 15) * min(stress_duration / 2, 1.0)
        
        return {
            'predicted_glucose_increase': glucose_increase,
            'duration_hours': stress_duration,
            'cortisol_peak_time': 30,  # minutes
            'recommendation': f"⚠️ Stress detected. Expect +{glucose_increase:.0f} mg/dL glucose increase. Practice stress-relief techniques."
        }
```

---

## 5. Digital Twin Simulation Examples

### 5.1 What-If Scenario Analysis

```python
class WhatIfSimulation:
    """
    Explore intervention outcomes using patient digital twin
    """
    
    def __init__(self, digital_twin: PersonalizedGlucoseDynamicsModel):
        self.digital_twin = digital_twin
    
    def scenario_meal_modification(self, 
                                  planned_meal: Dict) -> Dict:
        """
        Question: "What if I eat whole wheat rice instead of white rice?"
        
        Simulates different food choices for same meal
        """
        
        # Current plan: 1 cup white rice (60g carbs, GI=73)
        white_rice_glycemic_load = 60 * 0.73
        
        # Alternative: Whole wheat rice (60g carbs, GI=55)
        brown_rice_glycemic_load = 60 * 0.55
        
        # Simulate both scenarios
        current_scenario = self.digital_twin.simulate_glucose_trajectory(
            current_glucose=120,
            meal_carbs=60,
            meal_fat=5,
            meal_protein=12,
            insulin_bolus=4,
            hours_ahead=4,
            context={'meal_type': 'lunch', 'gi_factor': 0.73}
        )
        
        alternative_scenario = self.digital_twin.simulate_glucose_trajectory(
            current_glucose=120,
            meal_carbs=60,
            meal_fat=5,
            meal_protein=12,
            insulin_bolus=4,
            hours_ahead=4,
            context={'meal_type': 'lunch', 'gi_factor': 0.55}
        )
        
        return {
            'scenario_1_white_rice': {
                'peak_glucose': current_scenario['peak_glucose'],
                'time_in_range': sum(1 for g in current_scenario['glucose_trajectory'] 
                                    if 70 <= g <= 180) / len(current_scenario['glucose_trajectory'])
            },
            'scenario_2_brown_rice': {
                'peak_glucose': alternative_scenario['peak_glucose'],
                'time_in_range': sum(1 for g in alternative_scenario['glucose_trajectory'] 
                                    if 70 <= g <= 180) / len(alternative_scenario['glucose_trajectory'])
            },
            'recommendation': 'Brown rice provides smoother glucose control with 20+ mg/dL lower peak'
        }
    
    def scenario_activity_modification(self,
                                      planned_activity: Dict) -> Dict:
        """
        Question: "Should I go for a 30-min walk or 60-min walk after lunch?"
        """
        
        # Scenario 1: No activity
        no_activity = self.digital_twin.simulate_glucose_trajectory(
            current_glucose=140,
            meal_carbs=60,
            meal_fat=8,
            meal_protein=15,
            activity_level='sedentary',
            hours_ahead=3
        )
        
        # Scenario 2: 30-minute walk
        walk_30min = self.digital_twin.simulate_glucose_trajectory(
            current_glucose=140,
            meal_carbs=60,
            meal_fat=8,
            meal_protein=15,
            activity_level='moderate',
            hours_ahead=3,
            context={'activity_duration': 30}
        )
        
        # Scenario 3: 60-minute walk
        walk_60min = self.digital_twin.simulate_glucose_trajectory(
            current_glucose=140,
            meal_carbs=60,
            meal_fat=8,
            meal_protein=15,
            activity_level='moderate',
            hours_ahead=3,
            context={'activity_duration': 60}
        )
        
        return {
            'scenario_1_no_activity': {
                'peak_glucose': no_activity['peak_glucose'],
                'hypoglycemia_risk': no_activity['hypoglycemia_risk_percentage']
            },
            'scenario_2_30min_walk': {
                'peak_glucose': walk_30min['peak_glucose'],
                'hypoglycemia_risk': walk_30min['hypoglycemia_risk_percentage']
            },
            'scenario_3_60min_walk': {
                'peak_glucose': walk_60min['peak_glucose'],
                'hypoglycemia_risk': walk_60min['hypoglycemia_risk_percentage']
            },
            'recommendation': '30-minute walk is optimal - similar glucose control to 60-min with less hypoglycemia risk'
        }
    
    def scenario_medication_timing(self,
                                  medication_options: List[Dict]) -> Dict:
        """
        Question: "When should I take my diabetes medication?"
        
        Tests different medication timing options
        """
        
        results = {}
        
        for option in medication_options:
            medication_name = option['name']
            timing = option['timing']
            
            result = self.digital_twin.simulate_glucose_trajectory(
                current_glucose=option.get('baseline_glucose', 120),
                meal_carbs=option.get('expected_carbs', 50),
                meal_fat=option.get('expected_fat', 10),
                meal_protein=option.get('expected_protein', 20),
                insulin_bolus=option.get('insulin_units', 4),
                hours_ahead=4,
                context={
                    'medication_name': medication_name,
                    'medication_timing': timing,
                    'medication_peak_time': option.get('peak_time_minutes', 60)
                }
            )
            
            results[medication_name] = {
                'timing': timing,
                'peak_glucose': result['peak_glucose'],
                'time_in_range': result['hypoglycemia_risk_percentage'],
                'effectiveness_score': self._score_outcome(result)
            }
        
        best_option = max(results.items(), key=lambda x: x[1]['effectiveness_score'])
        
        return {
            'scenarios': results,
            'best_option': best_option[0],
            'recommendation': f"Take {best_option[0]} at {best_option[1]['timing']} for optimal glucose control"
        }
```

### 5.2 Early Warning System

```python
class EarlyWarningSystem:
    """
    Detect potential health crises before they occur
    """
    
    def __init__(self, digital_twin: PersonalizedGlucoseDynamicsModel):
        self.digital_twin = digital_twin
    
    def predict_hypoglycemia_risk(self, 
                                 current_state: Dict,
                                 forecast_hours: int = 4) -> Dict:
        """
        Early warning: "You might have low glucose in 2-4 hours"
        
        Factors:
        1. Recent insulin injection
        2. Upcoming physical activity
        3. Extended fasting
        4. Previous similar patterns
        """
        
        trajectory = self.digital_twin.simulate_glucose_trajectory(
            current_glucose=current_state['glucose'],
            meal_carbs=current_state.get('recent_carbs', 0),
            meal_fat=current_state.get('recent_fat', 0),
            meal_protein=current_state.get('recent_protein', 0),
            insulin_bolus=current_state.get('pending_insulin', 0),
            activity_level=current_state.get('activity_level', 'sedentary'),
            hours_ahead=forecast_hours
        )
        
        if trajectory['hypoglycemia_risk_percentage'] > 15:
            return {
                'alert_level': 'high' if trajectory['hypoglycemia_risk_percentage'] > 30 else 'medium',
                'risk_percentage': trajectory['hypoglycemia_risk_percentage'],
                'predicted_min_glucose': trajectory['min_glucose'],
                'recommendations': trajectory['recommendations'],
                'suggested_action': self._generate_action(trajectory['hypoglycemia_risk_percentage'])
            }
        
        return {'alert_level': 'low', 'risk_percentage': 0}
    
    def predict_hyperglycemia_crisis(self,
                                    current_state: Dict) -> Dict:
        """
        Early warning: "Risk of dangerously high glucose"
        
        Triggers:
        1. Missed insulin dose
        2. Infection or illness
        3. High stress
        4. Large high-GI meal
        """
        
        trajectory = self.digital_twin.simulate_glucose_trajectory(
            current_glucose=current_state['glucose'],
            meal_carbs=current_state.get('meal_carbs', 0),
            meal_fat=current_state.get('meal_fat', 0),
            meal_protein=current_state.get('meal_protein', 0),
            context=current_state.get('context', {})
        )
        
        if trajectory['hyperglycemia_risk_percentage'] > 20:
            return {
                'alert_level': 'high' if trajectory['peak_glucose'] > 300 else 'medium',
                'predicted_peak': trajectory['peak_glucose'],
                'time_to_peak_minutes': trajectory['time_to_peak_minutes'],
                'recommendation': 'Contact healthcare provider. May need insulin adjustment.',
                'suggested_actions': [
                    'Drink water to stay hydrated',
                    'Check urine ketones if peak > 300',
                    'Plan light meal instead of heavy food'
                ]
            }
        
        return {'alert_level': 'low', 'peak_glucose': trajectory['peak_glucose']}
    
    def _generate_action(self, hypoglycemia_risk: float) -> str:
        if hypoglycemia_risk > 30:
            return "🔴 URGENT: Consume 15g fast-acting carbs immediately (juice, glucose tablet)"
        elif hypoglycemia_risk > 15:
            return "⚠️ Eat small snack (10g carbs) and re-check in 15 minutes"
        else:
            return "✓ Monitor closely but no immediate action needed"
```

---

## 6. Digital Twin Training & Personalization

### 6.1 Model Calibration Pipeline

```python
class DigitalTwinCalibration:
    """
    Train personalized digital twin from patient's actual data
    
    Process:
    1. Collect 2-4 weeks of daily data (glucose, meals, activity, insulin)
    2. Extract meal-response patterns
    3. Calibrate insulin-carb parameters
    4. Validate accuracy on held-out data
    5. Deploy personalized model
    """
    
    def __init__(self, patient_data: List[Dict], validation_split: float = 0.2):
        """
        Args:
            patient_data: Historical records {timestamp, glucose, carbs, insulin, activity, ...}
            validation_split: Proportion of data for validation (not training)
        """
        
        self.patient_data = patient_data
        self.validation_split = validation_split
        
        # Split into training and validation
        split_idx = int(len(patient_data) * (1 - validation_split))
        self.train_data = patient_data[:split_idx]
        self.val_data = patient_data[split_idx:]
    
    def calibrate_model(self) -> PersonalizedGlucoseDynamicsModel:
        """
        Calibrate all parameters of personalized model
        """
        
        # Extract meal response patterns
        meal_responses = self._extract_meal_responses()
        
        # Calibrate insulin sensitivity
        isf = self._calibrate_isf(meal_responses)
        
        # Calibrate insulin-carb ratio
        icr = self._calibrate_icr(meal_responses)
        
        # Calibrate absorption rates
        absorption_time = self._calibrate_absorption_time(meal_responses)
        
        # Create personalized model
        patient_profile = {
            'weight_kg': self._estimate_weight(),
            'age': self._estimate_age(),
            'insulin_sensitivity_factor': isf,
            'insulin_carb_ratio': icr,
            'total_daily_insulin': self._estimate_tdi(),
            'diabetes_type': 'Type 2'  # Or extracted from data
        }
        
        model = PersonalizedGlucoseDynamicsModel(patient_profile)
        
        # Validate model accuracy
        validation_error = self._validate_model(model)
        
        print(f"✓ Digital Twin Calibrated:")
        print(f"  - ISF: {isf:.1f} mg/dL per unit insulin")
        print(f"  - ICR: {icr:.1f} g carbs per unit insulin")
        print(f"  - Absorption time: {absorption_time:.0f} minutes")
        print(f"  - Validation RMSE: {validation_error:.1f} mg/dL")
        
        return model
    
    def _extract_meal_responses(self) -> List[Dict]:
        """
        Find clear meal-response patterns in data
        """
        
        meal_responses = []
        
        for i in range(len(self.train_data) - 4):  # Look ahead 4 records (20 min)
            record = self.train_data[i]
            
            # Identify meals (carbs logged)
            if record.get('carbs', 0) > 20:  # Clear meal
                response = {
                    'meal_carbs': record['carbs'],
                    'meal_fat': record.get('fat', 0),
                    'meal_protein': record.get('protein', 0),
                    'baseline_glucose': record['glucose'],
                    'peak_glucose': max(r['glucose'] for r in self.train_data[i:i+4]),
                    'insulin_given': record.get('insulin', 0),
                    'time_to_peak_min': self._find_peak_time(i)
                }
                meal_responses.append(response)
        
        return meal_responses
    
    def _validated_on_held_out_data(self) -> float:
        """
        Calculate model accuracy on validation data
        
        Returns: Root Mean Square Error (RMSE) in mg/dL
        """
        
        predictions = []
        actuals = []
        
        for record in self.val_data:
            predicted = self.model.simulate_glucose_trajectory(
                current_glucose=record['glucose'],
                meal_carbs=record.get('carbs', 0),
                meal_fat=record.get('fat', 0),
                meal_protein=record.get('protein', 0),
                hours_ahead=1
            )
            
            predictions.append(predicted['glucose_trajectory'][-1])
            
            # Find actual glucose 1 hour later
            actual_index = self.val_data.index(record) + 12  # 12 * 5min = 60min
            if actual_index < len(self.val_data):
                actuals.append(self.val_data[actual_index]['glucose'])
        
        rmse = np.sqrt(np.mean((np.array(predictions) - np.array(actuals)) ** 2))
        return rmse
```

---

## 7. Multi-Model Digital Twins

### 7.1 Ensemble Approach

```python
class EnsembleDigitalTwin:
    """
    Combine multiple prediction models for robust forecasting
    
    Models:
    1. Compartmental Model (mechanistic)
    2. Machine Learning (data-driven)
    3. Gaussian Process (uncertainty quantification)
    4. Recurrent Neural Network (temporal patterns)
    """
    
    def __init__(self, patient_data: List[Dict]):
        
        # Initialize diverse models
        self.compartmental_model = PersonalizedGlucoseDynamicsModel(patient_data[0])
        self.ml_model = self._train_ml_model(patient_data)
        self.gp_model = self._train_gp_model(patient_data)
        self.rnn_model = self._train_rnn_model(patient_data)
        
        self.model_weights = {
            'compartmental': 0.4,  # Trust mechanistic model
            'ml': 0.3,
            'gp': 0.2,
            'rnn': 0.1
        }
    
    def predict_ensemble(self, current_state: Dict, hours_ahead: int) -> Dict:
        """
        Ensemble prediction combining all models
        
        Advantages:
        1. Robustness: No single model failure
        2. Uncertainty quantification: Confidence intervals
        3. Better generalization
        """
        
        # Get predictions from all models
        pred_compartmental = self.compartmental_model.simulate_glucose_trajectory(
            current_glucose=current_state['glucose'],
            meal_carbs=current_state.get('carbs', 0),
            hours_ahead=hours_ahead
        )
        
        pred_ml = self.ml_model.predict(current_state, hours_ahead)
        pred_gp = self.gp_model.predict(current_state, hours_ahead)
        pred_rnn = self.rnn_model.predict(current_state, hours_ahead)
        
        # Ensemble average
        ensemble_trajectory = (
            self.model_weights['compartmental'] * np.array(pred_compartmental['glucose_trajectory']) +
            self.model_weights['ml'] * np.array(pred_ml['trajectory']) +
            self.model_weights['gp'] * np.array(pred_gp['mean']) +
            self.model_weights['rnn'] * np.array(pred_rnn['trajectory'])
        )
        
        # Calculate confidence intervals from all models
        all_predictions = np.array([
            pred_compartmental['glucose_trajectory'],
            pred_ml['trajectory'],
            pred_gp['mean'],
            pred_rnn['trajectory']
        ])
        
        uncertainty = np.std(all_predictions, axis=0)
        
        return {
            'ensemble_trajectory': ensemble_trajectory.tolist(),
            'confidence_lower_bound': (ensemble_trajectory - 1.96 * uncertainty).tolist(),
            'confidence_upper_bound': (ensemble_trajectory + 1.96 * uncertainty).tolist(),
            'ensemble_peak': float(np.max(ensemble_trajectory)),
            'model_agreement': float(1 - np.mean(uncertainty) / np.mean(ensemble_trajectory))
        }
```

---

## 8. Digital Twin Applications in Diabetes Management

### 8.1 Personalized Meal Planning

```
User: "What should I eat for lunch?"

SYSTEM:
1. Simulates 5 different lunch options using digital twin
2. Predicts glucose response for each
3. Ranks by health outcomes
4. Provides recommendations

Output:
"Recommended: Grilled chicken with brown rice and steamed vegetables
 - Predicted peak glucose: 145 mg/dL (good)
 - Time in range: 95% (excellent)
 - Better than: Pizza (peak 240), White rice (peak 190), Burger (peak 220)"
```

### 8.2 Medication Optimization

```
Doctor: "Patient needs insulin adjustment"

SYSTEM:
1. Runs digital twin with current insulin doses
2. Tests 3-5 different dosing scenarios
3. Projects glucose control month ahead
4. Recommends optimal dose

Output:
"Current dose (20U): 18% time hypoglycemic
 Recommended dose (22U): 8% time hypoglycemic, 92% in range
 Benefit: Safer, better control, reduced hypo risk"
```

### 8.3 Lifestyle Intervention Planning

```
Patient: "I want to start exercising. How will it affect my diabetes?"

SYSTEM:
1. Simulates daily glucose with current routine
2. Adds planned exercise: 30-min jog 3x/week
3. Shows predicted glucose changes
4. Adjusts insulin recommendations

Output:
"With 3x/week jogging:
 - Fasting glucose: -8 mg/dL (improvement)
 - HbA1c: -0.4% (significant benefit)
 - New insulin requirement: Reduce morning dose by 1-2 units"
```

---

## 9. Privacy & Security in Digital Twins

### 9.1 Data Protection

```python
class DigitalTwinPrivacy:
    """
    Ensure digital twin data is secure and private
    """
    
    PROTECTIONS = {
        'data_storage': 'Encrypted at rest (AES-256)',
        'data_transmission': 'Encrypted in transit (TLS 1.3)',
        'access_control': 'Role-based access (patient, doctor, researcher)',
        'anonymization': 'Remove identifiers before sharing for research',
        'audit_logging': 'Track all access to digital twin model',
        'retention_policy': 'Delete data after 7 years or on patient request',
        'consent_management': 'Patient controls what data is used for simulation'
    }
```

---

## 10. Integration with Federated Learning

### 10.1 Digital Twins + Federated Learning

```
WORKFLOW:

1. TRAIN DIGITAL TWIN LOCALLY
   - Hospital trains personalized model on their patients' data
   - Digital twin stays private in hospital

2. EXTRACT INSIGHTS (FEDERATED LEARNING)
   - Train global food-glucose response model federally
   - All hospitals contribute learnings without sharing patient data
   
3. ENHANCE LOCAL TWINS
   - Hospital receives updated global model
   - Local digital twins benefit from learnings across institutions
   - Continues iterating

BENEFIT: Individual digital twins improve through multi-institutional collaboration
```

---

## 11. Validation & Clinical Outcomes

### 11.1 Digital Twin Accuracy Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Prediction RMSE** | < 15 mg/dL | Difference between predicted and actual glucose |
| **Peak Detection** | ±15 minutes | Accuracy of predicting when glucose peaks |
| **Hypo Detection** | > 95% sensitivity | Correctly identify hypoglycemia risk |
| **Hyper Detection** | > 90% specificity | Correctly identify safe glucose ranges |
| **Personalization Benefit** | > 80% vs generic | Better outcomes with personalized vs standard model |

---

**End of Digital Twins Architecture Document**
