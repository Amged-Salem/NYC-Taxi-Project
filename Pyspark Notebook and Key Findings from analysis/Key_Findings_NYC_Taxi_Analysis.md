# Key Findings from NYC Taxi Dataset Analysis

## 🚖 **Payment & Customer Behavior**
- **Credit cards dominate** across all times of day (Payment Type 1)
- **Solo travelers tip the most** (~$1.87 avg) and represent 72% of all trips
- **1/3 of all trips have no tip** - significant for driver income

## 🏙️ **Geographic Patterns**
- **Manhattan dominates volume** (937K trips, 91% of total) but has **lowest revenue per trip** ($14.23)
- **Airports are goldmines**: EWR ($85.54/trip) and Staten Island ($77.77/trip) have highest revenue per trip
- **Top driver zones**: JFK ($1.4M total), LaGuardia ($1.2M), Midtown Center (39K trips)

## ⏰ **Time Patterns**
- **Afternoon is peak time** (12-6 PM) for ALL top pickup zones
- **All boroughs follow same pattern** - consistent city-wide behavior

## 🛣️ **Trip Characteristics**
- **Most trips are short**: Average 2.3 miles in Manhattan, 11.4 miles from Queens
- **Longest realistic trips**: ~4 hours, mostly airport runs ($52-70 fares)
- **Data quality issues**: Many 24-hour "trips" (likely parking/errors)

## 💰 **Revenue Insights**
- **Airport routes most profitable** but lower volume
- **Manhattan = volume game**: Low margins, high frequency
- **Inter-borough travel**: Queens ↔ Manhattan most common routes

## 📊 **Business Intelligence**
- **For drivers**: Focus on airports in afternoon for best revenue
- **For fleet management**: Manhattan for consistent volume, airports for premium fares
- **Peak efficiency**: Afternoon shift captures maximum demand across all zones

## 🎯 **Bottom Line**
NYC taxi market shows clear geographic and temporal patterns - **airports for premium**, **Manhattan for volume**, and **afternoon for peak demand** universally.

---

*Analysis based on 1M+ taxi trip records with comprehensive data cleaning and feature engineering using Apache Spark.*
