# Epic 10: Diet/Lifestyle Module

## Epic Goal

Implement comprehensive diet/lifestyle management module that manages calories per day, recommends food swaps, supports specific diet types (low sodium, low potassium, low fat, and more as recommended by doctor), and provides menu recommendations from photos when eating out. This module is critical as 95% of Type 2 diabetes management is diet management. The module integrates with meal logging (Epic 1) and pattern detection (Epic 4) to provide personalized dietary guidance. Doctor setup for diet types and preferences should be accessible on app as well as WhatsApp, allowing doctors to configure patient diet requirements through either interface.

## Story 10.1: Calorie Management

As a patient,
I want to track and manage my daily calorie intake,
so that I can maintain a healthy diet for diabetes management.

**Acceptance Criteria:**
1. Daily calorie target: system sets daily calorie target based on patient profile (doctor can customize)
2. Calorie tracking: system tracks calories from logged meals (integrated with meal logging from Epic 1)
3. Calorie display: patients can view daily calorie intake vs target in mobile app
4. Calorie breakdown: shows calories by meal (breakfast, lunch, dinner, snacks)
5. Calorie progress: visual indicator showing progress toward daily calorie target
6. Calorie alerts: alerts when approaching or exceeding daily calorie target
7. Calorie history: patients can view calorie intake history (daily, weekly, monthly)
8. Calorie adjustment: system adjusts calorie targets based on patient goals and doctor recommendations
9. Calorie recommendations: system provides recommendations to stay within calorie target
10. Calorie reporting: calorie data included in weekly progress reports

## Story 10.2: Food Swap Recommendations

As a patient,
I want food swap recommendations based on my glucose patterns,
so that I can make better food choices that keep my glucose stable.

**Acceptance Criteria:**
1. Food swap engine: system analyzes patient's food-glucose patterns (from Epic 4)
2. Swap identification: identifies foods that spike glucose and suggests better alternatives
3. Swap recommendations: provides personalized food swap suggestions (e.g., "Instead of rice, try roti - it keeps your glucose 40 mg/dL lower")
4. Swap display: food swaps displayed in mobile app with visual comparison
5. Swap acceptance: patients can accept swaps, system tracks if patient tried the swap
6. Swap effectiveness: system tracks glucose outcomes when patient uses recommended swaps
7. Swap learning: system learns which swaps work best for each patient
8. Swap categories: swaps organized by meal type (breakfast, lunch, dinner, snacks)
9. Swap explanations: explains why swap is recommended (glucose impact, nutritional benefits)
10. Swap integration: swaps integrated with meal logging and pattern detection

## Story 10.3: Specific Diet Types Support

As a patient,
I want to follow specific diet types recommended by my doctor (low sodium, low potassium, low fat, etc.),
so that I can manage my diabetes according to my doctor's guidance.

**Acceptance Criteria:**
1. Diet type selection: doctors can set specific diet types for patients (low sodium, low potassium, low fat, diabetic, etc.)
2. Diet type rules: system enforces diet type rules (e.g., low sodium diet limits sodium intake)
3. Diet type recommendations: system provides food recommendations based on diet type
4. Diet type compliance: system tracks patient compliance with diet type requirements
5. Diet type alerts: alerts when patient logs food that violates diet type rules
6. Diet type alternatives: suggests alternatives when patient logs non-compliant food
7. Diet type education: provides education about diet type requirements and benefits
8. Diet type reporting: diet type compliance included in weekly progress reports
9. Multiple diet types: supports patients with multiple diet requirements (e.g., low sodium + diabetic)
10. Diet type customization: doctors can customize diet type rules for individual patients

## Story 10.4: Menu Recommendations from Photos

As a patient,
I want menu recommendations when eating out based on photos of restaurant menus,
so that I can make healthy choices even when dining out.

**Acceptance Criteria:**
1. Menu photo upload: patients can upload photos of restaurant menus via mobile app
2. Menu OCR: system uses OCR to extract menu items from photos
3. Menu analysis: system analyzes menu items for nutritional content and diabetes-friendliness
4. Menu recommendations: system recommends best options from menu based on patient's diet type and patterns
5. Recommendation display: recommendations displayed with explanation (why this option is good)
6. Recommendation ranking: menu items ranked by suitability (best to worst)
7. Recommendation filtering: filter recommendations by diet type, calorie target, glucose impact
8. Menu item details: patients can view details of recommended items (calories, carbs, sodium, etc.)
9. Menu history: system stores menu photos and recommendations for future reference
10. Menu learning: system learns patient's restaurant preferences and improves recommendations
