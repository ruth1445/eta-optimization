# eta-optimization
# ETA Prediction & Cost Optimization for Transit App

> **Note:** This case study describes ML work I did on a real transit product. Due to NDA obligations, I've removed company names and specific metrics.
> I'm sharing only the technical approach and general business logic to demonstrate my problem-solving methodology.

## Problem

A campus transit tracking app was displaying real-time bus ETAs from a third-party API but facing two challenges:

1. **Prediction accuracy varied wildly** by time, location, and conditions — users stopped trusting ETAs and just watched live location instead
2. **API costs were unsustainable** — expensive traffic and weather data calls were draining resources without proportional value

As a user of the app myself, I experienced this. The ETA would say "5 minutes" and the bus would arrive in 12.

## Solution

I built two statistical models to solve this:

### 1. Inaccuracy Index
Quantified when third-party ETA predictions were unreliable:
- Calculated prediction error (predicted arrival - actual arrival) by route, time of day, day of week, and weather conditions
- Found that certain routes/times had systematic biases (e.g., Route EE at 8 AM in rain averaged +6 min error, Route A at 2 PM sunny averaged +1 min error)
- Built a decision rule: if inaccuracy exceeds threshold X, call expensive real-time APIs; otherwise, trust the cheaper baseline

### 2. Dwell Time Threshold Detection
Identified when buses were stuck at stops (indicating delays to downstream predictions):
- Analyzed historical dwell time distributions per stop
- Set threshold: if dwell time > mean + 2σ, flag as "unusual delay"
- This caught common issues: busy stops during rush hour, lunch time, etc.

## Technical Approach

**Data:** 6 DynamoDB tables containing GPS traces, API predictions, traffic/weather conditions, and system logs.

**Methods:**
- Exploratory data analysis: heatmaps of error by route/time/weather
- Statistical thresholds: mean + standard deviation on dwell times
- Baseline modeling: linear regression comparing prediction error to traffic/weather features
- Cost-benefit analysis: estimated 73% reduction in expensive API calls by using decision logic

## Impact

- Identified specific routes/times where predictions fail consistently
- Provided clear decision logic for API spend: "trust third-party predictions when inaccuracy < 2 min, call real APIs otherwise"
- Estimated cost savings: ~73% reduction in traffic/weather API calls while maintaining accuracy
- Model ready for deployment to staging environment

## Key Insight

**ETA predictions don't account for variable dwell time at busy stops.** Once I identified this pattern, the solution became clear:
1. Define "dwelling" using historical patterns
2. Detect dwelling in real-time when it occurs
3. Adjust downstream ETAs accordingly

The hardest part wasn't the ML—it was understanding the business constraint (API costs) and identifying the root cause (dwell time variance). That's engineering.

## What I Learned

This project taught me that **the best ML work starts with understanding constraints**. The team didn't need the perfect ETA model. They needed a model that was *good enough* while minimizing cost. Constraint breeds creativity.
