# Practical Solo Data Collection Guide
## Sri Lankan Food Recognition - Home-Based, One Month Timeline

**For: Solo collector (you) with 2 phones, home kitchen, 1 month**

---

## Table of Contents

1. [Essential Data to Collect](#essential-data-to-collect)
2. [Free Tools Required](#free-tools-required)
3. [Step-by-Step Daily Workflow](#step-by-step-daily-workflow)
4. [Professional Photography Protocol](#professional-photography-protocol)
5. [Quick Measurement System](#quick-measurement-system)
6. [File Organization](#file-organization)
7. [Daily Schedule (4 Curries + 500-600 Images)](#daily-schedule)
8. [One-Month Collection Plan](#one-month-collection-plan)
9. [Annotation Workflow (Simple & Efficient)](#annotation-workflow-simple--efficient)
10. [Quality Assurance (Solo)](#quality-assurance-solo)
11. [Backup & Safety](#backup--safety)

---

## Essential Data to Collect

### MUST-HAVE Data (Non-negotiable)

```
For EACH image, you MUST record:

1. IMAGE FILES
   ✓ Filename: curry_name_batch_angle_001.jpg
   ✓ Resolution: Automatic (phone captures this)
   ✓ Store as: JPG (high quality)

2. DISH INFORMATION
   ✓ Curry name (e.g., "Chicken Curry")
   ✓ Batch number (1 or 2 for that curry today)
   ✓ Photography angle (see angle numbers below)

3. PORTION INFORMATION
   ✓ Weight in grams (weighed with scale)
   ✓ Approximate volume in ml (cup measurement)
   → Write immediately on paper with image number

4. CAMERA SETTINGS (Automatic capture)
   ✓ Phone captures automatically in EXIF data
   ✓ You just need to note: "Daylight" OR "Lamplight"
   ✓ Flash: YES or NO

5. REFERENCE OBJECT
   ✓ Which object used: Card OR Coin OR Ruler
   ✓ Is it visible in frame? YES/NO
   → Critical for scale calibration

---

NICE-TO-HAVE Data (If time permits):
├ Lighting quality description
├ Any special notes about the curry
└ Alternative portion variation

---

SKIP/SIMPLIFY (Too complex for solo):
├ ✗ Micronutrient details (use standard values later)
├ ✗ Ingredient-level breakdown
├ ✗ Detailed texture descriptions
└ ✗ Complex metadata fields
```

### Minimal Required Metadata

```
Create SIMPLE CSV file during collection:

image_id,curry_name,batch,angle,weight_g,volume_ml,reference_object,photos_taken
curry_chicken_batch1_45deg_001,Chicken Curry,1,45°,180,200,card,yes
curry_chicken_batch1_top_002,Chicken Curry,1,Top,180,200,card,yes
curry_chicken_batch1_side_003,Chicken Curry,1,Side,180,200,card,yes
...

That's it! Simple 8-column CSV.
```

---

## Free Tools Required

### Photography Tools (PHONE)

```
What you ALREADY HAVE:
✓ iPhone 12 Pro - Excellent camera (12MP, auto HDR)
✓ Android phone - Good backup
✓ Natural lighting at home

RECOMMENDED FREE PHONE APPS:

App 1: Pro Camera (for manual control)
├ iOS: "Camera Pro" (free version available)
├ Android: "Manual Camera" (free on Play Store)
├ Why: Allows aperture/ISO/shutter adjustment
└ ~5 min setup time

App 2: Mobile Object Lasso (optional)
├ For post-shot cropping consistency
├ Or use built-in phone photo editor
└ Optional - can skip if tight on time

TIP: iPhone 12 Pro built-in Camera app is actually EXCELLENT
├ Just use it in Natural Light mode
├ No need for fancy app
└ Saves you learning curve time
```

### Measurement & Documentation Tools

```
Tool 1: Digital Kitchen Scale
├ Cost: FREE (borrow from neighbor/family)
├ Or: ~$10-15 on Amazon (if buying)
├ Accuracy: ±0.1g (essential)
└ Where: Use for weighing ALL curries

Tool 2: Measuring Cups
├ Cost: FREE (you have in kitchen)
├ Use: Volume estimation
└ Accuracy: ~5% (acceptable)

Tool 3: Calibration Objects (FREE)
├ Credit card (has you): 85.6 × 53.98mm
├ Smartphone coin: ~5-10mm reference
└ Ruler: If you have paper ruler
```

### Data Organization Tools (COMPUTER)

```
Tool 1: Google Sheets (FREE)
├ Access: google.com/sheets
├ Create: "Curry_Collection_Metadata"
├ Import: CSV from phone
├ Sync: Automatic cloud backup
└ Time: 5 min to learn

Tool 2: Google Drive (FREE)
├ Access: google.com/drive
├ Storage: 15GB free (enough for 10,000+ photos)
├ Auto-backup: Enable for phone photos
└ Benefit: Never lose data

Tool 3: VS Code (FREE, already have)
├ Use: Edit metadata CSV locally
├ Use: Organize file naming
└ Use: Quick data validation

Tool 4: Python Scripts (Optional, FREE)
├ Purpose: Batch file renaming
├ Purpose: Metadata validation
├ Time: ~30 min to write/run
└ Benefit: Automate repetitive tasks
```

### Free Photo Import/Organization

```
Option 1: Google Photos (RECOMMENDED for beginners)
├ Download: Free app
├ Benefit: Auto-upload to cloud
├ Storage: 15GB free (enough)
├ Web access: View from computer
└ Backup: Automatic

Option 2: USB to Computer (Simple)
├ Use: USB cable to transfer photos daily
├ Backup: To external drive (if you have)
└ Manual but reliable
```

---

## Professional Photography Protocol
### For ONE PERSON, HOME-BASED

### Essential Setup (15 minutes)

```
Location: Kitchen table or similar
├ Clean, uncluttered surface
├ Natural light source (window preferred)
└ Stable shooting position

Plate Setup:
├ White ceramic plate (any size)
├ Clean surface, no reflections
├ Credit card placed beside (not overlapping)
└ Same position for all shots

Lighting Setup:
├ Best: Natural window light (morning/afternoon)
├ Worst: Direct harsh sun (creates shadows)
├ Backup: Turn on room lights (overhead lamp)
├ Tip: Soft, diffused light works best (cloudy day ideal)

Phone Mount/Stability:
├ Option 1: Books/stack of objects to stabilize phone
├ Option 2: Use phone prop stand ($5-10)
├ Option 3: Hold steady in hand (if tripod not available)
└ Critical: No camera shake (use both hands, rest on table edge)
```

### Camera Settings (iPhone 12 Pro - SIMPLEST)

```
DON'T OVERTHINK THIS. Use defaults:

1. Open Camera app
2. Select "Photo" mode
3. Tap on plate center to focus
4. Take photo
5. Done!

iPhone handles everything:
✓ Automatic exposure
✓ White balance
✓ HDR enhancement
✓ ISO adjustment
✓ Shutter speed optimization

Result: Professional-looking photos
```

### Camera Settings (Android - SIMPLE)

```
If using Manual Camera app:

Aperture: Leave auto (typically f/2.0)
Shutter Speed: Leave auto (typically 1/100)
ISO: 200-400 (depending on light)
White Balance: Daylight (5600K)

Honestly: Use AUTO mode unless having issues
├ Phone cameras are smart
├ Auto mode is 90% of professional quality
└ Manual adjustments help only 10%
```

### Step-by-Step Photography Angles (PREDEFINED)

```
For EACH CURRY, take these 6 ANGLES:

ANGLE 1: STRAIGHT DOWN (90° overhead)
├ Position: Phone directly above plate
├ Height: ~30-40cm above food
├ What to include: Full plate, credit card visible
├ Purpose: Volume measurement, top view
├ Image filename: curry_name_batch_01_top.jpg
│
├ Why this angle: Shows full dish area for calculation
└ Time: 30 seconds (2-3 photos, pick best)

ANGLE 2: 45° BIRD'S EYE VIEW
├ Position: 45° angle from horizontal
├ Height: ~30-40cm away, tilted
├ What to include: Full dish, depth visible
├ Purpose: Natural viewing angle, realistic perspective
├ Image filename: curry_name_batch_01_45deg.jpg
│
├ Why this angle: How food looks in reality
└ Time: 30 seconds

ANGLE 3: HORIZONTAL SIDE VIEW
├ Position: Phone at level with middle of plate
├ Distance: ~25-30cm from plate
├ What to include: Full height of curry, plate base visible
├ Purpose: Height measurement, depth cues
├ Image filename: curry_name_batch_01_side.jpg
│
├ Why this angle: Shows how tall the curry is
└ Time: 30 seconds

ANGLE 4: CLOSE-UP DETAIL
├ Position: Zoom in to show texture
├ What to include: Portion of curry showing ingredients
├ Purpose: Texture and ingredient identification
├ Image filename: curry_name_batch_01_closeup.jpg
│
├ Why this angle: Food recognition (can see components)
└ Time: 30 seconds

ANGLE 5: WITH BOWL/CUP REFERENCE
├ Position: Same as 45° but with measuring cup/bowl nearby
├ What to include: Show portion size relative to vessel
├ Purpose: Portion size visualization
├ Image filename: curry_name_batch_01_withcup.jpg
│
├ Why this angle: Realistic meal appearance
└ Time: 30 seconds

ANGLE 6: ALTERNATIVE PORTION (if different size)
├ Portion: Make 1.5× or 0.5× the standard portion
├ Position: Same angles as above (1 comprehensive angle)
├ Why: Show portion variation
├ Image filename: curry_name_batch_01_largeportion.jpg
│
├ Or SKIP if time is tight (focus on standard portion)
└ Time: Optional - 15 seconds if doing

TOTAL TIME PER CURRY PER BATCH:
├ Setup: 2-3 minutes
├ 6 angles × 30 seconds: 3 minutes
├ Measurement recording: 2-3 minutes
└ TOTAL: ~8-10 minutes per batch
```

### Photography Workflow (FASTEST METHOD)

```
TIMELINE FOR ONE CURRY (2 batches) = ~45 minutes total:

9:00-9:02   Prepare curry in measuring cup
            ├ Weigh with scale: ~180-200g
            ├ Write in notebook: "Batch 1: 187g"
            └ Pour into plate

9:02-9:10   BATCH 1 - Photography
            ├ Place credit card next to plate
            ├ Take 6 angles (30 sec each)
            ├ Review photos on phone (discard blurry)
            └ Note in CSV: curry_name_batch_1_angles_1-6

9:10-9:12   Prepare BATCH 2 (different size)
            ├ Weigh: ~150g (smaller portion)
            ├ Write note: "Batch 2: 152g"
            └ Pour into plate

9:12-9:20   BATCH 2 - Photography
            ├ Same 6 angles
            ├ Review photos
            └ Note in CSV: curry_name_batch_2_angles_1-6

9:20-9:25   Quick Documentation
            ├ Enter angles in CSV
            ├ Note any issues
            ├ File batch complete
            └ Backup photos to Google Drive

9:25-9:45   BREAK (20 min rest)
            ├ Eat lunch
            ├ Rest eyes
            ├ Prepare for next curry
            └ Stretch!

Expected OUTPUT per 45-min cycle:
├ 12 high-quality photos (6 angles × 2 batches)
├ 2 portion weights recorded (batch 1 + batch 2)
├ Metadata complete in CSV
└ Photos uploaded to cloud backup
```

---

## Quick Measurement System

### Weight Measurement (CRITICAL)

```
Equipment: 1 Digital Scale (~$10-15, or borrow)

Procedure (Takes 1 minute per batch):

Step 1: Tare (zero) the scale with plate
├ Put empty plate on scale
├ Press "tare" or "zero" button
└ Scale shows: 0g

Step 2: Add curry to plate
├ Scoop curry from pot
├ Place on plate
└ Scale shows weight (e.g., "187g")

Step 3: Record the weight
├ Write immediately in notebook
├ Before you forget!
├ Write: "Curry_Batch1_187g"
└ Later: Transfer to CSV

Accuracy: ±0.1g
├ This is good enough for dataset
├ More precision unnecessary
└ Do NOT worry about exact values
```

### Volume Measurement (QUICK ESTIMATE)

```
Equipment: Measuring cup from kitchen

Option A: Measure with Cup (2 minutes)
├ After weighing, transfer curry to cup
├ Fill cup with curry
├ Read ml marking
├ Example: "200ml"
├ Note: For liquidy curries (dhal, fish curry)
└ Not critical if weight is recorded

Option B: Visual Estimation (30 seconds)
├ Skip actual measurement if pressed for time
├ Estimate from plate:
│  ├ Small bowl (~75% full) = ~150ml
│  ├ Medium bowl (~100% full) = ~200ml
│  └ Large bowl (~100% full) = ~250ml
├ Record: "~200ml estimated"
└ Fine for initial dataset

Accuracy: ±10% (acceptable)
├ Volume less critical than weight
├ For AI training: weight matters most
└ Volume helps with cross-checking only
```

### Reference Object Documentation (MANDATORY)

```
For EVERY photo, record:

Reference Object Used: _______________
├ Option 1: "Credit Card" (85.6mm × 53.98mm)
├ Option 2: "Coin" (₨10: ~30mm, ₨50: ~25mm)
└ Option 3: "None" (if not in photo, note it)

Visibility in Photo: YES / NO
├ YES: Card fully visible, not overlapping food
├ NO: Card out of frame (risky, but ok if needed)
└ PARTIALLY: Card partially visible (acceptable)

Quick Rule:
✓ Try to include reference in 80%+ of photos
✓ Essential for scale calibration
✓ If missing from photo: note in metadata
└ This is why it's CRITICAL to record it
```

---

## File Organization

### Simple Daily Folder Structure

```
Create THIS structure on phone/computer:

Main_Folder: "SriLankan_Curries_Collection"
│
├── Metadata
│   ├── collection_log.csv (Main tracking spreadsheet)
│   ├── measurements.txt (Manual notes)
│   └── daily_summary.txt (End-of-day notes)
│
├── Day_01_29Mar (Example: March 29)
│   ├── curry_chicken_batch1_01_top.jpg
│   ├── curry_chicken_batch1_02_45deg.jpg
│   ├── curry_chicken_batch1_03_side.jpg
│   ├── curry_chicken_batch1_04_closeup.jpg
│   ├── curry_chicken_batch2_01_top.jpg
│   ├── curry_chicken_batch2_02_45deg.jpg
│   └── ... (continuing pattern)
│
├── Day_02_30Mar
│   ├── curry_fish_batch1_01_top.jpg
│   ├── curry_fish_batch1_02_45deg.jpg
│   └── ...
│
├── Day_03_31Mar
│   └── ... (similar structure)
│
└── Backups
    ├── backup_week1_compressed.zip
    └── backup_week2_compressed.zip
```

### Filename Convention (MUST BE CONSISTENT)

```
NAMING RULE (simple & clear):

Format: curry_[name]_batch[num]_angle[num]_[dayofmonth].jpg

Examples:
├ curry_chicken_batch1_01_29.jpg  ← angle 01 (top), batch 1, March 29
├ curry_chicken_batch1_02_29.jpg  ← angle 02 (45°), batch 1
├ curry_chicken_batch2_01_29.jpg  ← angle 01 (top), batch 2
├ curry_fish_batch1_01_30.jpg     ← angle 01, batch 1, March 30
└ curry_dhal_batch1_03_31.jpg     ← angle 03 (side), batch 1, March 31

ANGLE NUMBERS:
├ 01 = top (90°)
├ 02 = 45° bird's eye
├ 03 = side (horizontal)
├ 04 = closeup
├ 05 = with cup/reference
└ 06 = (optional) alternative portion

WHY THIS NAMING:
✓ Unique filename (won't conflict)
✓ Sortable by date (ends with day)
✓ Batch tracking (which preparation)
✓ Angle tracking (which view)
√ Curry name clear
└ Easy to find specific images later
```

### CSV Metadata File (Keep SIMPLE)

```
Create on phone/computer (Google Sheets = easiest):

File: collection_log.csv

Columns (8 columns only):

1. Image_ID
   └ curry_chicken_batch1_01_29

2. Curry_Name
   └ Chicken Curry

3. Batch_Number
   └ 1 (or 2)

4. Angle
   └ Top (or 45deg, Side, Closeup, etc.)

5. Weight_Grams
   └ 187

6. Volume_ML
   └ 200 (or leave blank if too time-consuming)

7. Reference_Object
   └ Card (or Coin or None)

8. Notes
   └ "Lighting: bright" or "Reference card partially visible"

EXAMPLE ROW:
curry_chicken_batch1_01_29 | Chicken Curry | 1 | Top | 187 | 200 | Card | Bright natural light

THAT'S IT! 8 columns, super simple.
```

---

## Daily Schedule (4 Curries + 500-600 Images)

### Daily Target Breakdown

```
TARGET: 500-600 IMAGES IN ONE DAY

Math:
├ 4 curries × 2 batches each = 8 total batches
├ 8 batches × 6 angles per batch = 48 core angles
├ 48 angles × 10-12 shots per angle (different settings/focus) = 480-576 images
├ WITH variations + alternative sizes = 500-600 images
└ TIME: 7-8 hours of work

Expected breakdown:
├ Curry 1 (Chicken): 150 images (6 angles × 2 batches × 12 shots average)
├ Curry 2 (Dhal): 150 images
├ Curry 3 (Fish): 150 images
├ Curry 4 (Vegetable): 150 images
└ TOTAL: ~600 images
```

### Sample Daily Schedule (7:00 AM - 3:00 PM)

```
7:00-7:15   Morning Setup
            ├ Gather equipment
            ├ Clean kitchen table
            ├ Set up lighting
            ├ Place credit card in frame
            └ Phone battery check (charge if needed)

7:15-9:00   CURRY 1 (Chicken Curry) - 105 min
            ├ 9:15-9:20    Prep Batch 1 (weigh, measure)
            ├ 9:20-9:35    Photography Batch 1 (6 angles, ~12 shots)
            ├ 9:35-9:40    Record notes
            ├ 9:40-9:45    Rest & reload
            ├ 9:45-9:50    Prep Batch 2 (different size)
            ├ 9:50-10:05   Photography Batch 2 (6 angles, ~12 shots)
            ├ 10:05-10:10  Record notes
            └ Result: ~144 images, 2 batches documented

9:10-9:15   QUICK BACKUP
            ├ Photos to Google Drive (1-2 min if WiFi good)
            └ If no WiFi: backup at end of day

9:15-11:00  CURRY 2 (Dhal Curry) - 105 min
            ├ Same procedure as Curry 1
            └ Result: ~144 images

11:00-11:15 LUNCH BREAK (15 min)
            ├ Eat something
            ├ Stretch
            ├ Drink water
            └ Rest eyes from phone screen

11:15-1:00  CURRY 3 (Fish Curry) - 105 min
            ├ Same procedure
            └ Result: ~144 images

1:00-1:30   MID-AFTERNOON BREAK (30 min)
            ├ Longer break
            ├ Review photos from morning
            ├ Check for any issues
            └ Rest eyes (important!)

1:30-3:15   CURRY 4 (Vegetable Curry) - 105 min
            ├ Same procedure
            └ Result: ~144 images

3:15-3:30   END-OF-DAY TASKS (15 min)
            ├ Final backup to cloud
            ├ Update CSV file
            ├ Quick quality check (review sample of photos)
            ├ Organize file folders
            └ Prepare tomorrow's list

TOTAL TIME: ~8 hours (realistic with breaks)
TOTAL IMAGES: ~576 images
QUALITY: Professional standard
EFFORT: Manageable for one person
```

---

## One-Month Collection Plan

### Calendar Overview (30 Days)

```
MONTH: April 2026 (30 days)
GOAL: Collect 4,800-7,200 images of 10-15 different curries
PACE: 4 curries/day, 2 batches each, 6 angles each

Week 1 (Apr 1-7): Foundation Week
├ Day 1 (Tue): Chicken Curry [600 images]
├ Day 2 (Wed): Dhal Curry [600 images]
├ Day 3 (Thu): Fish Curry [600 images]
├ Day 4 (Fri): Vegetable Curry [600 images]
├ Weekend (Sat-Sun): REST + backup + review
└ Subtotal: 2,400 images (4 curries)

Week 2 (Apr 8-14): Expansion Week
├ Day 6 (Mon): Chicken Curry (alternative prep) [600 images]
├ Day 7 (Tue): Beef Curry [600 images]
├ Day 8 (Wed): Prawn Curry [600 images]
├ Day 9 (Thu): Mutton Curry [600 images]
├ Day 10 (Fri): Jackfruit Curry [600 images]
├ Weekend: REST + backup
└ Subtotal: 3,000 images (5 more curries = 9 total)

Week 3 (Apr 15-21): Variation Week
├ Day 12 (Mon): Chickpea Curry [600 images]
├ Day 13 (Tue): Pumpkin Curry [600 images]
├ Day 14 (Wed): Mixed Vegetable Curry [600 images]
├ Day 15 (Thu): Lentil Curry (red/yellow) [600 images]
├ Day 16 (Fri): Spinach Curry (Palak) [600 images]
├ Weekend: REST + backup
└ Subtotal: 3,000 images (5 more curries = 14 total)

Week 4 (Apr 22-30): Final Week
├ Day 19 (Mon): Brinjal Curry [600 images]
├ Day 20 (Tue): Green Bean Curry [600 images]
├ Day 21 (Wed): Cabbage Curry [600 images]
├ Day 22 (Thu): Carrot Curry [600 images]
├ Day 23 (Fri): Choose favorite curry for extra batch [600 images]
├ Days 24-30: Buffer for re-shooting + organization
└ Subtotal: 3,000 images (5 more curries = 19 total)

FINAL TOTALS:
├ Total images: 11,400 images (if every day successful)
├ Realistic: 8,400-9,600 images (accounting for rest days, issues)
├ Curries covered: 15-19 different curries
├ Batches per curry: 2 (standard + alternative)
└ Success Rate: Very high with this plan

CONTINGENCY:
├ Days 24-30: 7 buffer days for:
│  ├ Re-shoots if any curry turned out poorly
│  ├ Additional curries if inspiration strikes
│  ├ Extra angles for favorite curries
│  └ Data organization & quality check
└ This gives flexibility to aim for 15+ curries with high quality
```

### Daily Checklist (COPY THIS & USE EVERY DAY)

```
✓ MORNING CHECKLIST (7:00 AM)
  ├ ☐ Phone battery: >80% charge
  ├ ☐ iPhone 12 Pro ready + backup Android
  ├ ☐ Digital scale working (test with known weight)
  ├ ☐ Credit card in frame
  ├ ☐ Kitchen table clean & organized
  ├ ☐ Measuring cups washed & ready
  ├ ☐ Notebook open for quick notes
  ├ ☐ Google Drive app open on phone (backup ready)
  └ ☐ Review yesterday's photos (quick look)

✓ PRE-CURRY CHECKLIST (before each curry)
  ├ ☐ Ingredients ready to cook
  ├ ☐ Pots/pans clean
  ├ ☐ Measuring cups nearby
  ├ ☐ Scale ready to use
  ├ ☐ Plate positioned consistently (same spot)
  ├ ☐ Credit card placed beside plate
  ├ ☐ Phone camera app open & ready
  └ ☐ Lighting check (brightness adequate?)

✓ DURING-PHOTOGRAPHY CHECKLIST (each angle)
  ├ ☐ Food in frame (fully visible)
  ├ ☐ Credit card visible (not in food)
  ├ ☐ Focus sharp (use tap-to-focus)
  ├ ☐ Exposure looks good (not too dark/bright)
  ├ ☐ Take 2-3 shots per angle (pick best)
  ├ ☐ Try one with different lighting if available
  └ ☐ No camera shake (steady hold or use stand)

✓ POST-CURRY CHECKLIST (after both batches)
  ├ ☐ Weight recorded in notebook
  ├ ☐ Volume recorded (if measuring)
  ├ ☐ CSV file updated with batch info
  ├ ☐ All 12 angles (6 per batch) captured
  ├ ☐ Sample photos reviewed on phone (delete obvious blurs)
  ├ ☐ Reference object noted for each set
  └ ☐ Photos ready for upload

✓ END-OF-DAY CHECKLIST (3:00 PM)
  ├ ☐ All 4 curries completed
  ├ ☐ Total image count: ~600 (confirm by folder size)
  ├ ☐ CSV updated with all metadata
  ├ ☐ Photos uploaded to Google Drive (verify sync)
  ├ ☐ Backup spreadsheet downloaded (as Excel)
  ├ ☐ Review 10-15 random photos for quality
  ├ ☐ Delete any severely blurry/bad photos
  ├ ☐ Make notes in daily_summary.txt
  ├ ☐ Plan tomorrow's curries
  ├ ☐ Celebrate! Tell someone you completed another day
  └ ☐ REST - you earned it!
```

---

## Annotation Workflow (Simple & Efficient)

### MINIMALIST ANNOTATION (Solo approach)

```
Most Important Insight: 
  You don't need a nutritionist review initially.
  Use standard nutritional values from reliable source.
  You can update later if needed.

Annotation Process (3 phases):
```

### Phase 1: Immediate (Same Day) - 30 minutes

```
While food is still fresh in memory:

Step 1: CSV Entry (Google Sheets)
├ Open collection_log.csv
├ Enter all 8 columns:
│  ├ Image_ID: curry_chicken_batch1_01_29
│  ├ Curry_Name: Chicken Curry
│  ├ Batch_Number: 1
│  ├ Angle: Top
│  ├ Weight_Grams: 187
│  ├ Volume_ML: 200 (or leave blank)
│  ├ Reference_Object: Card
│  └ Notes: "Good lighting, card visible"
│
└ TIME: 3 minutes per curry (12 rows for 2 batches × 6 angles)

Step 2: Quick Quality Check
├ Open phone gallery
├ Flip through today's 600 images
├ Delete obviously bad ones (blurry, not in frame, exposed wrong)
├ Keep: all others (even if slightly imperfect)
├ TIME: 10-15 minutes

Step 3: Cloud Backup
├ Photos uploaded to Google Drive
├ CSV uploaded & saved
├ Verify sync complete
└ TIME: 5 minutes (if WiFi good)

TOTAL: ~30 minutes per day
```

### Phase 2: Weekly Summary (30 minutes on Sunday)

```
Every Sunday evening (takes 30 min):

Step 1: Review Week's Data
├ Open Google Sheets
├ Look at all 25-30 entries for the week
├ Check for missing data (flag any gaps)
└ Verify curry names spelled consistently

Step 2: Backup (Double Backup)
├ Download CSV as Excel file
├ Download photos to external drive (if you have one)
├ Or: just verify Google Drive has all photos
└ Multiple copies now exist

Step 3: Create Weekly Summary
├ Create text file: "Week_April1-7_Summary.txt"
├ Write:
│  ├ Curries completed this week (4 curries)
│  ├ Total images (2,400)
│  ├ Any issues faced ("Weather cloudy on Friday - used lamplight")
│  ├ Quality assessment (1-10 rating)
│  └ Notes for next week ("Improve lighting setup")
│
└ Store summary in Metadata folder

TOTAL: ~30 minutes per week
```

### Phase 3: Final Annotation (After collection completes) - 3-4 hours

```
After 30 days of collection (all ~9,600 images collected):

Step 1: Nutritional Data Addition (most time-consuming)
├ For each unique curry, add:
│  ├ Carbohydrates per 100g
│  ├ Protein per 100g
│  ├ Fat per 100g
│  ├ Calories per 100g
│  ├ GI (Glycemic Index)
│  └ GL (Glycemic Load)
│
├ SOURCE: Use 1 reliable source for all
│  ├ Option 1: USDA FoodData Central (free online)
│  ├ Option 2: Government nutrition tables
│  └ Option 3: Published research paper
│
├ HOW: Google "{curry_name} nutritional value per 100g"
├ Add to new column in CSV: "Nutrition_Source"
└ TIME: ~5-10 minutes per unique curry × 15 curries = 90 minutes

Step 2: Image Quality Review (full dataset)
├ Sample 10% of images (~960 images)
├ Rate each 1-5 for quality
├ Keep: 4-5 ratings
├ Delete: 1-2 ratings (obvious errors)
├ Identify: Issues and patterns
└ TIME: 60-90 minutes

Step 3: Final CSV Cleanup
├ Ensure curl names consistent (capitalize same way)
├ Fill any missing reference_object info
├ Add angles consistently (use: "Top", "45°", "Side", etc.)
├ Verify weights (any obvious errors? 0g is not real)
└ TIME: 30-60 minutes

TOTAL ANNOTATION TIME: ~3-4 hours (one-time, after collection)
```

### Simplified Nutritional Data Template

```
Create file: "Curry_Nutrition_Reference.csv"

curry_name,carbs_per_100g,protein_per_100g,fat_per_100g,calories_per_100g,gi_index,source
Chicken Curry,2.5,18.0,8.5,165,15,USDA_FoodData_Central
Dhal Curry,20.0,8.0,2.5,130,32,Government_Nutrition_Table
Fish Curry,3.0,20.0,6.0,155,12,Research_Paper_2023
Vegetable Curry,8.0,2.5,2.0,50,20,USDA_FoodData_Central
Beef Curry,2.0,22.0,10.0,200,15,USDA_FoodData_Central
[... etc for all 15 curries]

WHY THIS:
✓ Same nutritional values used for ALL images of same curry
✓ No need to enter per-image (efficient!)
✓ Can update in one place if source changes
✓ Can calculate "nutrition per portion" = values × (weight/100)
```

---

## Quality Assurance (Solo)

### Simple Quality Standards

```
PASS/FAIL CRITERIA (you judge these yourself):

Image Quality Score: 1-5

✓ 5 = Excellent (keep definitely)
  ├ Sharp focus (entire dish in focus)
  ├ Good exposure (not too bright or dark)
  ├ Credit card visible & not overlapping
  ├ Plate centered in frame
  └ Professional-looking

✓ 4 = Good (keep)
  ├ Slightly soft focus but center clear
  ├ Exposure ok (recoverable if needed)
  ├ All key elements visible
  └ Good enough for training data

✓ 3 = Acceptable (keep but note)
  ├ Some focus issues at edges
  ├ Exposure slightly off
  ├ Minor framing issues
  └ Still usable with notice

✓ 2 = Poor (consider deleting)
  ├ Very soft/blurry
  ├ Bad exposure (too dark or blown)
  ├ Credit card not visible
  ├ Framing cut off important parts

✓ 1 = Bad (delete)
  ├ Out of focus entirely
  ├ Motion blur
  ├ No credit card visible
  ├ Plate not in frame

YOUR PROCESS:
├ Day-of: Delete obvious 1-2 ratings
├ Weekly: Review 10-20% of photos, rate each
├ End: Keep 4-5 only, or keep 3-5 if quantity needed
└ Final dataset: ~8,000-9,000 high-quality images
```

### Weekly Quality Report (Template)

```
File: Weekly_QA_Report_Apr1-7.txt

Week: April 1-7 (Week 1)
Days completed: 4 (Mon-Thu)
Curries: Chicken, Dhal, Fish, Vegetable
Total images: 2,400

QUALITY BREAKDOWN:
├ Rating 5 (Excellent): 1,200 images (50%)
├ Rating 4 (Good): 900 images (37.5%)
├ Rating 3 (Acceptable): 240 images (10%)
├ Rating 2 (Poor): 45 images (2%)
├ Rating 1 (Bad): 15 images (1%)

ISSUE LOG:
├ April 3: Cloudy weather → used lamp lighting (worked ok)
├ April 5: 30 images blurry due to hand shake → used stand afterward
├ April 7: Reference card partially out of frame in 25 photos (noted)

IMPROVEMENTS FOR NEXT WEEK:
├ Use tripod/stand more consistently
├ Clean credit card before each batch (was slightly bent)
├ Better lighting transition (lamplight → sunlight caused color shift)

RATING: Week 1: 9/10 ✓
└ Very happy with quality and consistency!
```

---

## Backup & Safety

### Backup Strategy (CRITICAL - Don't Lose Your Data!)

```
BACKUP RULE: 3 Locations for All Images

Location 1: Phone (Original)
├ Your iPhone 12 Pro
├ Android backup phone
└ Advantage: Always with you

Location 2: Cloud (Google Drive - MOST IMPORTANT)
├ Enable auto-photo sync in Google Photos app
├ Or: Manually upload to Google Drive weekly
├ Storage: 15GB free (enough for ~10,000 photos)
├ Advantage: If phone breaks, data safe
└ Set reminder: Check sync status every Monday

Location 3: Computer/External Drive (If you have)
├ Weekly download photos from Google Drive
├ Store on external USB drive
├ Advantage: Offline backup
└ Alternative: Dropbox or OneDrive (free 5-10GB)

BACKUP SCHEDULE:
├ Daily: Google Drive auto-sync enabled
├ Sunday (weekly): Download & verify photos on computer
├ Before bed: Quick check "Google Drive has today's photos"

VERIFICATION:
├ Monthly: Check file count matches (should increase by ~2,400/week)
├ Monthly: Spot-check random old photo (can you open it?)
├ Never: Delete originals until you have 2 other backups
```

### Emergency Procedures

```
IF PHONE BREAKS:
├ Step 1: Don't panic
├ Step 2: Access Google Drive from computer
├ Step 3: Download all photos
├ Step 4: Get new phone, restore from backup
├ Result: No data loss!

IF GOOGLE ACCOUNT COMPROMISED:
├ Change password immediately
├ All data still accessible from computer backup
└ No data loss (you have external backup)

IF COMPUTER FAILS:
├ Photos still on: Phone + Google Drive
├ Re-download to new computer from Google Drive
└ No data loss!

This is why you MUST have multiple backups!
```

---

## Complete Daily Workflow (SUMMARY)

### ACTUAL STEP-BY-STEP (Start to Finish = 8 hours)

```
7:00-7:15 AM     MORNING SETUP (15 min)
                 ├ Battery check
                 ├ Equipment ready
                 ├ Table cleaned
                 └ Mental focus

7:15 AM          START CURRY 1: CHICKEN CURRY

7:15-7:25        [+10 min] Prep Batch 1
                 ├ Cook curry (if needed) or reheat
                 ├ Scoop into plate
                 ├ Weigh: write weight
                 └ Place on shooting table

7:25-7:40        [+15 min] Photography Angles (Batch 1)
                 ├ Angle 1 (20 sec): Phone straight above
                 ├ Angle 2 (20 sec): 45° angle
                 ├ Angle 3 (20 sec): Side view
                 ├ Angle 4 (20 sec): Close-up
                 ├ Angle 5 (20 sec): With cup reference
                 └ Angle 6 (optional 15 sec): Extra

7:40-7:45        [+5 min] Record Notes
                 ├ CSV file: angles 1-6 for batch 1
                 └ Notes: "Lighting: good", "Card: visible"

7:45-7:50        [+5 min] Rest + Reload

7:50-8:00        [+10 min] Prep Batch 2 (different size)
                 ├ Different portion size
                 ├ Weigh: write weight
                 └ Position on table

8:00-8:15        [+15 min] Photography (Batch 2)
                 ├ Same 6 angles
                 └ Same 2-3 minutes per angle

8:15-8:20        [+5 min] Record Notes
                 └ CSV: angles 1-6 for batch 2

                 CURRY 1 COMPLETE: 65 min
                 Result: ~144 images, full documentation

8:20-8:25        [+5 min] BACKUP
                 ├ Upload photos to Google Drive
                 └ Verify sync shows "Completed"

8:25-9:30 AM     CURRY 2: DHAL CURRY (~65 min)
                 └ EXACT SAME PROCESS

                 (Repeat every 65 minutes for each curry)

9:30-10:35 AM    CURRY 3: FISH CURRY (~65 min)

10:35-11:00 AM   BREAKS (25 min during day)
                 ├ Lunch/snack
                 ├ Stretch
                 ├ Drink water
                 └ Rest eyes

11:00-12:05 PM   CURRY 4: VEGETABLE CURRY (~65 min)

12:05-3:00 PM    INCLUDING LONGER BREAK (mid-afternoon)
                 ├ Extended break at lunch
                 ├ Review morning photos
                 └ Check for any issues

3:00-3:30 PM     FINAL TASKS (30 min)
                 ├ Final backup to cloud
                 ├ Update CSV spreadsheet
                 ├ Delete any obviously bad photos
                 ├ Quick quality review (10-20 photos)
                 ├ Count total images taken
                 ├ File organization
                 └ Write down tomorrow's plan

3:30 PM          DAY COMPLETE! 🎉
                 ├ ~600 images collected
                 ├ 4 curries documented
                 ├ Full metadata recorded
                 ├ All backed up safely
                 └ REST & celebrate!
```

---

## Quick Reference Cards

### CARD 1: Angle Codes (Keep on Phone)

```
Print this or screenshot:

01 = TOP (90° straight down)
02 = 45° ANGLE (bird's eye)
03 = SIDE (horizontal)
04 = CLOSE-UP (zoom detail)
05 = WITH CUP (portion reference)
06 = ALT PORTION (different size)

Reference Objects:
C = Card
O = Coin  
R = Ruler
N = None
```

### CARD 2: Daily Checklist (Take Photo)

```
Take photo of this checklist & keep on phone!

BEFORE EACH CURRY:
☐ Scale working
☐ Plate clean
☐ Card placed
☐ Phone ready
☐ Lighting ok

BEFORE EACH ANGLE:
☐ Food in frame
☐ Card visible
☐ Phone steady
☐ Focus sharp
☐ Take 2-3 shots

AFTER BATCH:
☐ Weight written
☐ CSV updated
☐ Photos reviewed
☐ Backup started
```

### CARD 3: Measurement Quick Guide

```
WEIGHING:
- Tare scale (zero with plate)
- Add curry
- Write number immediately
- CRITICAL: Don't forget!

VOLUME:
- Fill measuring cup
- Read ml marking
- Write it down
- Or: Skip if too time-consuming

REFERENCE:
- Credit card: 85.6 × 53.98mm
- Is it visible? YES/NO
- Record in CSV
```

---

## Final Summary: What You'll Have After 1 Month

```
COMPLETE OUTPUT (April 30):

✓ 8,400-9,600 High-Quality Images
  ├ 14-19 different curries
  ├ 2 batches per curry
  ├ 6 angles per batch
  └ Professional standard

✓ Complete Metadata
  ├ collection_log.csv (all images)
  ├ Measurements (weights in grams)
  ├ Reference objects documented
  ├ Camera settings logged
  └ Notes for any issues

✓ Nutritional Data
  ├ Per-100g values for each curry
  ├ Glycemic Index included
  ├ Sources documented
  └ Ready for dataset

✓ Organized Files
  ├ Logical folder structure (by day)
  ├ Consistent naming convention
  ├ Cloud backup (Google Drive)
  ├ External backup (if available)
  └ Easy to find anything

✓ Quality Assurance
  ├ Weekly quality reports
  ├ Problem log
  ├ 4-5 star rating system applied
  ├ Bad images removed
  └ ~90% high-quality images remaining

✓ Ready for Next Phase
  ├ Images ready for AI training
  ├ Metadata ready for annotation tools
  ├ Organized for augmentation
  └ Professional dataset structure

TOTAL EFFORT: One person, 30 days, realistic pace
TOTAL DATA: Ready for research/AI development
NEXT STEPS: Can start building AR portion estimation model!
```

---

## Troubleshooting (Quick Fixes)

```
PROBLEM 1: Photos are blurry
├ Solution: Use tripod/stand (stabilize phone)
├ Solution: Tap to focus on plate center
├ Solution: Use better lighting
└ PREVENT: Practice steady hands

PROBLEM 2: Reference card not visible in photo
├ Solution: Position card fully in frame corner
├ Solution: Don't let food cover it
├ Solution: Use plain white card (not transparent)
└ PREVENT: Check reference visible before photographing

PROBLEM 3: Lighting is poor (photos too dark)
├ Solution: Use natural window light (best)
├ Solution: Turn on room lights
├ Solution: Photograph during daytime (9-4pm good)
├ Solution: Use white plate (reflects light)
└ PREVENT: Scout location before shooting

PROBLEM 4: Can't remember which batch is which
├ Solution: Label plate with batch number before shooting
├ Solution: Write immediately after weighing
├ Solution: Use consistent batch order (always small→large)
└ PREVENT: Update CSV immediately, not later

PROBLEM 5: Photos keep failing to upload to Google Drive
├ Solution: Check WiFi connection
├ Solution: Try uploading later (off-peak hours)
├ Solution: Manually upload via computer
├ Solution: Use mobile data as backup
└ PREVENT: Test upload on Day 1

PROBLEM 6: Running out of phone storage (photos fill memory)
├ Solution: Upload photos immediately after curry session
├ Solution: Delete from phone after verified in cloud
├ Solution: Use second phone for backup camera
└ PREVENT: Enable auto-delete after backup on phone
```

---

## Timeline at a Glance

```
✓ TODAY (Day 1): 
  ├ Complete all setup
  ├ Collect 4 curries, 600 images
  └ Backup to cloud

✓ THIS WEEK (Days 2-7):
  ├ Repeat daily: 4 curries → 600 images
  ├ Weekly backup (Sunday)
  └ Subtotal: ~4,200 images

✓ THIS MONTH (Days 8-30):
  ├ Weeks 2-4: Same daily process
  ├ End-of-month: Annotation + final QA
  └ Final Total: ~9,600 images + full metadata

✓ MAY 1:
  ├ Dataset ready for AI training
  ├ Metadata complete
  ├ All backed up safely
  └ Ready to build AR model!

THIS IS DOABLE. YOU CAN DO THIS! 🎯
```

