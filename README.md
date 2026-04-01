# eta-optimization
# ETA Prediction & Cost Optimization for Transit App

 **Note:** This case study describes ML work I did on a real transit product. Due to NDA obligations, I've removed company names and specific metrics. I'm sharing only the technical approach and general business logic to demonstrate my problem-solving methodology.

## Problem

A campus transit tracking app was displaying real-time bus ETAs from a third-party API but facing two challenges:

1. Prediction accuracy varied and users (including myself) stopped trusting ETAs and just watched live location of the buses instead.
2. API costs were unsustainable and so. Expensive traffic and weather data calls were draining our resources.

## Solution

I built two statistical models to solve this:

### 1. Inaccuracy Index
Quantified when third-party ETA predictions were unreliable:
- Calculated prediction error (predicted arrival - actual arrival) by route, time of day, day of week, and weather conditions
- Found that certain routes/times had biases (a certain route at 8 AM in the rain averaged +6 min error whereas another route at 2 PM sunny averaged +1 min error)
- Built a decision rule that said if inaccuracy exceeds threshold X, call expensive real-time APIs; otherwise, trust the cheaper baseline

### 2. Dwell Time Threshold Detection
Identified when buses were stuck at stops (indicating delays to downstream predictions):
- Analyzed historical dwell time distributions per stop
- Set threshold: if dwell time > mean + 2σ, flag as "unusual delay"
- This caught common issues like busy stops during rush hour, lunch time, etc.

## Technical Approach

**Data:** 6 DynamoDB tables containing GPS traces, API predictions, traffic/weather conditions, and system logs.

**Methods:**
- Exploratory data analysis: heatmaps of error by route/time/weather
- Statistical thresholds: mean + standard deviation on dwell times
- Baseline modeling: linear regression comparing prediction error to traffic/weather features
- Cost-benefit analysis: estimated 73% reduction in expensive API calls by using decision logic

## Impact

- Identified specific routes/times where predictions fail consistently
- Provided clear decision logic for API spend
- Model ready for deployment to staging environment
