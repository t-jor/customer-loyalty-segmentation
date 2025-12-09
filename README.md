# Customer Loyalty Segmentation (TravelTide Case Study)

This project analyzes customer behavior for **TravelTide**, an online travel booking platform, to support the design of a **personalized loyalty program**.  
Using SQL-based feature engineering and Tableau visualizations, customers were segmented into actionable groups and matched with initial perk hypotheses to improve retention and campaign targeting.

---

## 🎯 Business Objective

TravelTide has grown rapidly since 2021 but continues to struggle with **customer retention**.  
Marketing aims to launch a personalized rewards program and needs clear, data-driven answers to:

1. **Which distinct customer segments exist?**  
2. **How do behavior, demographics, and travel patterns differ by segment?**  
3. **Which perks are likely to resonate with each group?**  
4. **How should a loyalty program be structured to increase repeat bookings?**

Since TravelTide operates only with historical behavioral data (no experimental data available), this project provides **analytically supported segment insights** and **initial perk assignments**, which require validation through controlled testing.

---

## 🧠 Approach & Methodology

### **1. Data Understanding & Cleaning**

- Exploration of user, session, flight, and hotel datasets  
- Standardization of date fields, durations, prices, and booking flags  
- Removal or correction of invalid records (e.g., negative nights)

### **2. Feature Engineering (SQL)**

Creation of user-level behavioral and pricing metrics, including:

- Booking frequency & cancellation patterns  
- Session interactions (clicks, minutes, booking speed)  
- Price sensitivity indicators  
- Travel characteristics (distance, trip length, weekend vs. weekday travel)  
- Demographic attributes (age categories)

### **3. Segmentation Logic**

A weighted scoring model evaluates each user across behavior and travel attributes.  
Users are assigned to one of several meaningful groups, including:

- Business Traveler  
- Family Traveler  
- Frequent Traveler  
- Premium Traveler  
- Budget / Deal Hunter  
- Dreamer  
- New / Young Customers  
- Others (low-signal users)

This segmentation is **fully data-driven**, based on engineered features and behavioral scoring.

### **4. Perk Mapping (Educated Hypothesis)**

The perk assignments are based on behavioral insights from the segmentation and reflect strategic recommendations derived from the observed customer patterns:

- segment characteristics  
- typical industry patterns in travel loyalty programs  
- inferred customer needs  

Since no experimental validation was possible within this project setup, these recommendations should be considered as **initial hypotheses** and would typically be validated through controlled **A/B testing** in a real-world rollout, as outlined in the recommended next steps.

---

## 📊 Example Outputs

- Session-based metrics including clicks, booking behavior, trip length, seat booking patterns  
- SQL-based segmentation into 10 customer groups with initial perk hypotheses  
- Tableau dashboards illustrating user behavior, segment distribution, and feature patterns  
- Deliverables:  
  - **[Full Report](reports/TravelTide_FullReport_Final.pdf)** – detailed analysis covering all segments  
  - **[Executive Summary](reports/TravelTide_ExecutiveSummary_Final.pdf)** – one-page management overview  
  - **[Segment Definitions](docs/TravelTide_Segment_Definitions.xlsx)** – feature summary and scoring logic

---

## 🛠 Tools & Technologies

- **SQL (Postgres)** – feature engineering, segmentation scoring, KPIs  
- **Tableau** – exploratory analysis, behavior profiles, segment visualization  
- **Google Sheets / Docs** – stakeholder reporting, executive summary  

---

## 📁 Repository Structure

```text
/
├── sql/                            # Segmentation & feature engineering queries
├── dashboard/                      # Tableau workbook and dashboard exports
├── docs/                           # Executive summary, segment documentation
├── img/                            # Project visuals (segments, perk mapping, EDA)
└── README.md                       # Overview and project documentation
```

A detailed explanation of the SQL logic, transformations, and scoring model can be found in the  
👉 **Technical README (`TECHNICAL_README.md`)**.

---

## 🚀 Next Steps

These steps outline what would realistically follow in a real-world analytics setting and extend the project beyond its fictional constraints:

- **🧪 Validate perk–segment hypotheses with A/B experiments**  
  (Not possible within this educational project but essential for real deployment.)

- **🔄 Refine & stabilize segment definitions with additional user cohorts**  
  As new data becomes available, behavioral thresholds and percentiles should be recalibrated.

- **🎯 Build and test a tiered loyalty program focused on the four core segments**  
  Start with high-confidence segments (e.g., Business, Family, Frequent, Premium) for initial rollout.

- **📈 Monitor behavioral changes over time**  
  Segment drift and shifts in travel patterns should be incorporated into regular review cycles.

---

## 👤 Author

**Thomas Jortzig**  
Customer Loyalty Segmentation – TravelTide Case Study (09/2025)
