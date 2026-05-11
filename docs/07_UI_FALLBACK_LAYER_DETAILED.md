# 07 - UI Fallback Layer (Detailed Architecture)

## 1. Overview
The UI Fallback Layer serves as the minimal graphical interface for the voice-first diabetes management system. It's intentionally kept lean—used only when voice interaction is unavailable, user prefers visual input, or detailed visualization is beneficial for health monitoring and historical analysis.

**Key Principle**: Voice is primary, UI is fallback—not the other way around

---

## 2. UI Components Architecture

### 2.1 Core UI Components

#### 1. Dashboard (Home Screen)

**Purpose**: At-a-glance view of current health status

```python
class Dashboard:
    """
    Minimal, information-dense dashboard
    Shows only critical information
    """
    
    DASHBOARD_SECTIONS = {
        'glucose_status': {
            'elements': [
                'current_glucose_value',      # 145 mg/dL
                'glucose_trend_arrow',         # ↗ (rising)
                'trend_description',          # "Slowly rising"
                'time_since_last_reading',    # "5 minutes ago"
                'predicted_glucose_1hr',      # "May reach 185 mg/dL"
            ],
            'design': 'Large, color-coded: red (danger), yellow (caution), green (good)'
        },
        'quick_stats': {
            'elements': [
                'carbs_today',                # "120g logged"
                'activity_today',             # "4500 steps"
                'meals_logged_today',         # "2 of 3 logged"
                'medication_status',          # "On track"
            ],
            'design': 'Small cards, quick overview'
        },
        'next_action': {
            'elements': [
                'primary_recommendation',     # "Log meal?" or "Time to walk?"
                'action_button',              # Voice or manual input
                'time_until_next_check',      # "Check in 2 hours"
            ],
            'design': 'Prominent, action-oriented'
        },
        'alerts': {
            'elements': [
                'critical_alerts',            # Hypoglycemia, extreme hyperglycemia
                'important_reminders',        # Missed medication, no activity
                'informational_messages',     # Pattern insights
            ],
            'design': 'Color-coded priority badges'
        }
    }
    
    def render_dashboard(self, user_context):
        """
        Render dashboard based on current context
        """
        
        dashboard_data = {
            'glucose_section': {
                'value': f"{user_context['current_glucose']} mg/dL",
                'trend': user_context['glucose_trend_arrow'],
                'color': self._get_glucose_color(user_context['current_glucose']),
                'prediction': f"~{user_context['predicted_glucose_1hr']} in 1 hour",
                'recommendation': self._get_primary_action(user_context)
            },
            'quick_stats_cards': [
                {'label': 'Carbs Today', 'value': f"{user_context['carbs_today']}g"},
                {'label': 'Steps', 'value': f"{user_context['steps_today']:,}"},
                {'label': 'Meals Logged', 'value': f"{user_context['meals_logged']}/3"},
                {'label': 'Medication', 'value': 'On Track' if user_context['on_medication'] else 'Due'}
            ],
            'alerts': self._get_active_alerts(user_context)
        }
        
        return dashboard_data


class GlucoseStatusCard:
    """
    Prominent glucose status display
    """
    
    GLUCOSE_ZONES = {
        'critical_low': {'range': (0, 70), 'color': '#FF0000', 'icon': '⚠️', 'text': 'CRITICAL LOW'},
        'low': {'range': (70, 100), 'color': '#FFA500', 'icon': '⚠️', 'text': 'LOW'},
        'target': {'range': (100, 180), 'color': '#00AA00', 'icon': '✓', 'text': 'Normal'},
        'high': {'range': (180, 240), 'color': '#FFD700', 'icon': '⚠️', 'text': 'High'},
        'critical_high': {'range': (240, 400), 'color': '#FF0000', 'icon': '🚨', 'text': 'CRITICAL HIGH'}
    }
    
    TREND_ARROWS = {
        'rapidly_rising': '↑↑',
        'rising': '↑',
        'stable': '→',
        'falling': '↓',
        'rapidly_falling': '↓↓'
    }
    
    def get_glucose_display(self, glucose_value, trend):
        """
        Determine how to display glucose value
        """
        
        zone = self._determine_zone(glucose_value)
        
        return {
            'value': f"{glucose_value}",
            'unit': 'mg/dL',
            'color': zone['color'],
            'icon': zone['icon'],
            'status_text': zone['text'],
            'trend_arrow': self.TREND_ARROWS[trend],
            'last_update': 'now',
            'size': 'large'  # Prominent display
        }
```

#### 2. Food Logging Screen

**Purpose**: Manual food entry when voice is unavailable

```python
class FoodLoggingUI:
    """
    Minimal UI for food logging - supports voice input first
    """
    
    INPUT_METHODS = {
        'image_upload': {
            'description': 'Take photo of food',
            'automation': 'AI analyzes automatically',
            'accuracy': 'High'
        },
        'food_search': {
            'description': 'Search food database',
            'automation': 'Manual selection',
            'accuracy': 'Medium'
        },
        'quick_selection': {
            'description': 'Favorite meals (pre-saved)',
            'automation': 'One-click logging',
            'accuracy': 'High'
        },
        'custom_entry': {
            'description': 'Manually input nutrition',
            'automation': 'User entry',
            'accuracy': 'User-dependent'
        }
    }
    
    def render_food_logging(self, session_context):
        """
        Show food logging interface
        """
        
        ui = {
            'header': {
                'title': 'Log Your Meal',
                'voice_option': 'Or say "I ate..."',
                'microphone_button': True  # Always available
            },
            'input_tabs': [
                {
                    'tab': 'Camera',
                    'icon': '📷',
                    'action': self.capture_food_image(),
                    'hint': 'Take a photo for instant analysis'
                },
                {
                    'tab': 'Search',
                    'icon': '🔍',
                    'action': self.search_food_database(),
                    'hint': 'Find food from extensive database'
                },
                {
                    'tab': 'Favorites',
                    'icon': '⭐',
                    'action': self.show_favorite_meals(),
                    'hint': 'Your recent meals'
                }
            ],
            'food_items': [
                # Dynamically populated
            ],
            'nutrition_summary': {
                'calories': 0,
                'carbs': 0,
                'protein': 0,
                'fat': 0
            },
            'portion_adjustment': {
                'slider': True,
                'buttons': ['Increase', 'Decrease'],
                'current_portion': '1 cup'
            },
            'action_buttons': {
                'log_meal': True,
                'cancel': True,
                'swap_food': 'Change detected food'
            }
        }
        
        return ui
    
    def capture_food_image(self):
        """
        Allow user to capture food image
        """
        return {
            'type': 'image_capture',
            'instructions': 'Take clear photo from above',
            'tips': [
                'Include whole meal',
                'Good lighting helps',
                'Avoid shadows'
            ],
            'next_step': 'auto_analyze'
        }
    
    def search_food_database(self):
        """
        Search food database
        """
        return {
            'type': 'search',
            'placeholder': 'Search for food (e.g., "rice", "chicken curry")',
            'recent_searches': self.get_recent_searches(),
            'categories': ['Grains', 'Proteins', 'Vegetables', 'Snacks', 'Beverages'],
            'autocomplete': True
        }


class MealComposer:
    """
    Build multi-item meal from components
    """
    
    def add_food_item_to_meal(self, food_item, portion):
        """
        Add food item to current meal
        """
        
        meal = {
            'items': [
                {'food': 'rice', 'portion': '1 cup', 'calories': 206, 'carbs': 45},
                {'food': 'curry', 'portion': '1.5 cups', 'calories': 270, 'carbs': 12}
            ],
            'total': {
                'calories': 476,
                'carbs': 57,
                'protein': 18,
                'fat': 13
            },
            'glucose_impact': {
                'estimated_peak': '1-2 hours',
                'estimated_magnitude': '+65 mg/dL'
            }
        }
        
        return meal
```

#### 3. Activity Logging Screen

```python
class ActivityLoggingUI:
    """
    Minimal activity logging interface
    """
    
    def render_activity_logging(self):
        """
        Show activity logging options
        """
        
        ui = {
            'header': 'Log Activity',
            'voice_option': 'Or say "I walked for 30 minutes"',
            'quick_activities': [
                {
                    'activity': 'Walking',
                    'intensity': ['Light', 'Moderate', 'Brisk'],
                    'icon': '🚶',
                    'typical_duration': '15-60 min'
                },
                {
                    'activity': 'Jogging',
                    'intensity': ['Easy', 'Moderate', 'Fast'],
                    'icon': '🏃',
                    'typical_duration': '20-45 min'
                },
                {
                    'activity': 'Cycling',
                    'intensity': ['Light', 'Moderate', 'Vigorous'],
                    'icon': '🚴',
                    'typical_duration': '20-60 min'
                },
                {
                    'activity': 'Swimming',
                    'intensity': ['Light', 'Moderate', 'Vigorous'],
                    'icon': '🏊',
                    'typical_duration': '15-45 min'
                },
                {
                    'activity': 'Strength Training',
                    'intensity': ['Light', 'Moderate', 'Intense'],
                    'icon': '🏋️',
                    'typical_duration': '20-40 min'
                }
            ],
            'wearable_sync': {
                'available': True,
                'connected_devices': ['Fitbit', 'Apple Watch'],
                'message': 'Your activity has been logged from your device'
            },
            'custom_activity': {
                'enabled': True,
                'placeholder': 'Custom activity (e.g., "Dancing", "Gardening")',
                'duration_input': True,
                'intensity_slider': True
            }
        }
        
        return ui
    
    def sync_wearable_data(self):
        """Connect to fitness wearables"""
        
        return {
            'supported_devices': [
                'Apple Watch',
                'Fitbit',
                'Garmin',
                'Samsung Galaxy Watch',
                'Google Fit',
                'Strava'
            ],
            'connection_status': 'Not Connected',
            'action': 'Tap to connect'
        }
```

#### 4. Health Metrics & History Screen

```python
class HealthMetricsUI:
    """
    View historical metrics and trends
    """
    
    def render_metrics_dashboard(self, user_id):
        """
        Show historical health data
        """
        
        ui = {
            'time_periods': [
                {'label': 'Today', 'days': 1, 'default': True},
                {'label': 'Week', 'days': 7},
                {'label': 'Month', 'days': 30},
                {'label': '3 Months', 'days': 90}
            ],
            'metrics_tabs': [
                {
                    'tab': 'Glucose',
                    'charts': [
                        {
                            'type': 'line_graph',
                            'title': 'Glucose Over Time',
                            'data': 'glucose_readings_by_time',
                            'zones': 'show_target_ranges'
                        },
                        {
                            'type': 'bar_chart',
                            'title': 'Average Glucose by Meal',
                            'data': 'avg_glucose_by_meal_type'
                        },
                        {
                            'type': 'distribution',
                            'title': 'Time in Target Range',
                            'data': 'percentage_in_zone'
                        }
                    ],
                    'statistics': ['Average', 'Min', 'Max', 'Std Dev']
                },
                {
                    'tab': 'Nutrition',
                    'charts': [
                        {
                            'type': 'bar_chart',
                            'title': 'Carbs Intake Trend',
                            'data': 'daily_carbs'
                        },
                        {
                            'type': 'pie_chart',
                            'title': 'Macronutrient Distribution',
                            'data': 'carbs_protein_fat_ratio'
                        },
                        {
                            'type': 'frequency_chart',
                            'title': 'Most Eaten Foods',
                            'data': 'top_10_foods'
                        }
                    ]
                },
                {
                    'tab': 'Activity',
                    'charts': [
                        {
                            'type': 'line_graph',
                            'title': 'Daily Activity Minutes',
                            'data': 'activity_minutes_daily'
                        },
                        {
                            'type': 'bar_chart',
                            'title': 'Activity Type Distribution',
                            'data': 'activity_breakdown'
                        }
                    ]
                }
            ],
            'insights_panel': {
                'title': 'Trends & Insights',
                'items': [
                    'Your glucose control has improved 15% this month',
                    'Most active day: Saturday',
                    'Rice causes 20% higher glucose response for you',
                    'Exercise timing: Best results with afternoon walks'
                ]
            }
        }
        
        return ui
```

#### 5. Settings & Profile Screen

```python
class SettingsUI:
    """
    User settings and profile management
    """
    
    def render_settings(self):
        """
        Show settings options
        """
        
        ui = {
            'sections': [
                {
                    'section': 'Profile',
                    'items': [
                        {'label': 'Age', 'value': '32', 'editable': True},
                        {'label': 'Weight', 'value': '75 kg', 'editable': True},
                        {'label': 'Diabetes Type', 'value': 'Type 2', 'editable': True},
                        {'label': 'Medications', 'value': 'Metformin 1000mg', 'editable': True}
                    ]
                },
                {
                    'section': 'Preferences',
                    'items': [
                        {
                            'label': 'Preferred Language',
                            'value': 'English',
                            'options': ['English', 'Sinhala', 'Tamil'],
                            'editable': True
                        },
                        {
                            'label': 'Tone',
                            'value': 'Encouraging',
                            'options': ['Encouraging', 'Neutral', 'Cautious'],
                            'editable': True
                        },
                        {
                            'label': 'Notification Frequency',
                            'value': 'Moderate',
                            'options': ['Frequent', 'Moderate', 'Minimal'],
                            'editable': True
                        }
                    ]
                },
                {
                    'section': 'Data & Privacy',
                    'items': [
                        {'label': 'Data Backup', 'action': 'Backup Now', 'toggle': True},
                        {'label': 'Delete Old Data', 'description': 'KeepData older than 90 days', 'action': 'Configure'},
                        {'label': 'Privacy Settings', 'action': 'Manage', 'link': '/privacy'}
                    ]
                },
                {
                    'section': 'Connected Devices',
                    'items': [
                        {
                            'device': 'Fitbit',
                            'status': 'Connected',
                            'action': 'Disconnect'
                        },
                        {
                            'device': 'Glucose Meter',
                            'status': 'Not Connected',
                            'action': 'Connect'
                        }
                    ]
                },
                {
                    'section': 'About',
                    'items': [
                        {'label': 'App Version', 'value': '2.1.0'},
                        {'label': 'Last Updated', 'value': 'March 22, 2026'},
                        {'label': 'Support', 'action': 'Contact Support'}
                    ]
                }
            ]
        }
        
        return ui
```

---

## 3. UI Display Technologies

### 3.1 Responsive Design

```css
/* Mobile-First Design */

@media (max-width: 480px) {
    /* Smartphone layout */
    .dashboard {
        single-column layout
        Large touch targets (48px minimum)
        Simplified color scheme
        Large fonts for accessibility
    }
}

@media (481px - 768px) {
    /* Tablet layout */
    .dashboard {
        two-column layout
        Slightly compact
        More information visible
    }
}

@media (769px - 1280px) {
    /* Desktop layout */
    .dashboard {
        three-column layout
        Comprehensive visualization
        Detailed charts
    }
}
```

### 3.2 Accessibility Features

```python
ACCESSIBILITY_FEATURES = {
    'color_blind_mode': {
        'supported': True,
        'modes': ['Normal', 'Deuteranopia', 'Protanopia', 'Tritanopia'],
        'description': 'Alternative color schemes for color blindness'
    },
    'high_contrast': {
        'enabled': True,
        'description': 'Dark mode and light mode with high contrast'
    },
    'text_scaling': {
        'supported': True,
        'range': '0.8x to 2.0x',
        'description': 'Adjust font size for comfortable reading'
    },
    'screen_reader': {
        'compatible': True,
        'standards': 'WCAG 2.1 AA',
        'description': 'Full screen reader support'
    },
    'keyboard_navigation': {
        'supported': True,
        'description': 'Navigate entire app with keyboard'
    },
    'haptic_feedback': {
        'supported': True,
        'description': 'Vibration feedback for action confirmation (mobile)'
    }
}
```

---

## 4. UI Integration with Voice System

### 4.1 Voice-First Interaction Pattern

```python
class VoiceUIIntegration:
    """
    UI supports and enhances voice interaction
    """
    
    def show_processing_indicator(self):
        """
        Show that system is listening/processing
        """
        return {
            'visual': 'Animated microphone icon',
            'animation': 'Pulsing waveform',
            'text': 'Listening...',
            'dismiss': 'Auto-dismiss when complete'
        }
    
    def show_voice_transcription(self, transcribed_text):
        """
        Show what the system heard
        """
        return {
            'message': f'You said: "{transcribed_text}"',
            'actions': [
                {'action': 'Confirm', 'key': 'Enter'},
                {'action': 'Repeat', 'key': 'R'},
                {'action': 'Edit Manually', 'key': 'E'}
            ],
            'tone': 'neutral',
            'display_duration': '3 seconds then auto-execute'
        }
    
    def show_voice_response(self, audio_response, text_response):
        """
        Show AI's spoken response and transcript
        """
        return {
            'audio': audio_response,  # TTS output
            'text': text_response,    # Readable transcript
            'actions': [
                {'action': 'Play Again'},
                {'action': 'Continue'},
                {'action': 'Show Details'}
            ]
        }
    
    def show_clarification_options(self, options):
        """
        Show voice clarification options
        """
        return {
            'question': 'Did you mean A or B?',
            'options': [
                {'label': 'White Rice', 'voice_keyword': 'white'},
                {'label': 'Brown Rice', 'voice_keyword': 'brown'},
                {'label': 'Other', 'voice_keyword': 'other'}
            ],
            'can_tap': True,
            'can_say': True
        }
```

---

## 5. Offline Mode Support

```python
class OfflineUI:
    """
    Graceful degradation when offline
    """
    
    def render_offline_state(self):
        """
        Show UI modifications for offline
        """
        
        offline_ui = {
            'banner': {
                'message': 'Offline mode - limited features',
                'actions': ['Retry', 'Continue']
            },
            'available_features': [
                'View cached glucose data',
                'Log food (saved locally)',
                'Log activity (saved locally)',
                'Read past recommendations',
                'View offline charts'
            ],
            'unavailable_features': [
                'Real-time glucose sync',
                'Cloud backup',
                'Wearable device sync',
                'AI predictions'
            ],
            'data_sync': {
                'status': 'Pending sync',
                'last_sync': '3 hours ago',
                'items_waiting': 5
            }
        }
        
        return offline_ui
```

---

## 6. Data Visualization

### 6.1 Glucose Trend Visualization

```python
class GlucoseVisualization:
    """
    Visual representation of glucose data
    """
    
    def render_glucose_graph(self, time_period='today'):
        """
        Render glucose over time
        """
        
        graph = {
            'type': 'line_chart',
            'x_axis': 'Time (hourly)',
            'y_axis': 'Glucose (mg/dL)',
            'data_points': [
                {'time': '07:00', 'glucose': 145},
                {'time': '08:00', 'glucose': 180},
                {'time': '09:00', 'glucose': 172},
                # ... more points
            ],
            'reference_lines': {
                'upper_limit': 180,
                'target_range': (100, 180),
                'lower_limit': 70
            },
            'shaded_zones': {
                'critical_low': (0, 70, 'red'),
                'low': (70, 100, 'orange'),
                'target': (100, 180, 'green'),
                'high': (180, 240, 'yellow'),
                'critical_high': (240, 400, 'red')
            },
            'interactive': {
                'hover_detail': True,
                'zoom': True,
                'export': True
            }
        }
        
        return graph
```

---

## 7. Notification Center

```python
class NotificationUI:
    """
    Unified notification display
    """
    
    def show_notification(self, notification):
        """
        Display notification
        """
        
        notification_ui = {
            'type': notification['type'],  # 'alert', 'reminder', 'suggestion', 'celebration'
            'priority': notification['priority'],  # 'critical', 'high', 'normal', 'low'
            'icon': self._get_icon(notification['type']),
            'title': notification['title'],
            'message': notification['message'],
            'action': {
                'primary': notification.get('primary_action'),
                'secondary': notification.get('secondary_action')
            },
            'duration': self._get_display_duration(notification['priority']),
            'sound': self._should_play_sound(notification['priority']),
            'vibration': self._should_vibrate(notification['priority'])
        }
        
        return notification_ui
```

---

## 8. Implementation Checklist

- [ ] **Dashboard**
  - [ ] Glucose status card
  - [ ] Quick stats cards
  - [ ] Alert banner
  - [ ] Next action recommendation

- [ ] **Food Logging**
  - [ ] Image capture integration
  - [ ] Food search interface
  - [ ] Portion adjustment
  - [ ] Nutrition summary

- [ ] **Activity Logging**
  - [ ] Quick activity selection
  - [ ] Wearable sync UI
  - [ ] Custom activity entry
  - [ ] Duration/intensity input

- [ ] **Metrics & History**
  - [ ] Glucose trend chart
  - [ ] Nutrition breakdown charts
  - [ ] Activity statistics
  - [ ] Insights panel

- [ ] **Settings**
  - [ ] Profile editing
  - [ ] Preference selection
  - [ ] Privacy settings
  - [ ] Connected devices

- [ ] **Accessibility**
  - [ ] Color-blind modes
  - [ ] High contrast mode
  - [ ] Text scaling
  - [ ] Screen reader support
  - [ ] Keyboard navigation

- [ ] **Testing**
  - [ ] Responsive design tests
  - [ ] Accessibility compliance
  - [ ] Voice UI integration tests
  - [ ] Offline functionality tests

---

## 9. Technology Stack

```
┌─────────────────────────────────────────────────────────────┐
│ UI FALLBACK LAYER                                           │
├─────────────────────────────────────────────────────────────┤
│ Frontend Framework │ React / React Native (cross-platform)  │
│ Mobile            │ iOS (Swift) + Android (Kotlin)          │
│ Web               │ React + TypeScript                      │
│ Charts            │ Chart.js / D3.js / Recharts             │
│ Styling           │ Tailwind CSS / Material-UI              │
│ State Management  │ Redux / Context API                     │
│ Offline Support   │ SQLite (mobile) / IndexedDB (web)       │
│ Camera/Wearable   │ Native platform APIs                    │
└─────────────────────────────────────────────────────────────┘
```

---

**Last Updated**: March 22, 2026
**Status**: Complete
**Next**: 08_INTEGRATED_ARCHITECTURE.md (Master Document)
