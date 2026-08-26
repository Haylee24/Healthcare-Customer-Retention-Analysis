# Healthcare-Customer-Retention-Analysis
Predicting hospital churn before it happens, for a healthcare SaaS provider operating in Lagos State, Nigeria.

Developed a Power BI customer retention solution that assessed 2,319 healthcare facilities using a rule-based customer health score, identifying churn patterns, revenue at risk, product adoption gaps and intervention priorities to support data-driven Customer Success decision-making.

## Dashboard Preview
## Introduction
Helium Health provides SaaS platform infrastructure to hospitals and clinics across Nigeria, including hundreds of public and private facilities across Lagos State — from large teaching hospitals to small primary health centres. This project analyses facility-level subscription and engagement data to help the business understand which customers are healthy, which are quietly disengaging, and where churn is concentrated: before a facility cancels rather than after.

## Problem Statement
Helium Health's Customer Success team had no reliable way to tell which subscribed hospitals were at risk of churning until after they had already stopped using the platform. Churn was only visible in hindsight, once revenue had already been lost.

This project answers:

"How can Helium Health identify hospitals that are at risk of churning before they stop using the platform?"

Specifically, it aims to answer:

- Which currently active facilities show early warning signs of disengagement?
- What factors actually predict churn (plan type, facility size, ownership, geography)?
- How much revenue is currently exposed, and where should outreach be prioritized?

## Data Sourcing 
- Base structure: This project uses a publicly available IBM Telco Customer Churn sample dataset, that has been adapted into a simulated healthcare SaaS context for analytical and portfolio purposes. The healthcare-specific variables and business scenario are synthetic and do not represent actual Helium Health data.
- Facility counts: sourced from the Lagos State Ministry of Health (2025, secondary facility count) and a peer-reviewed HEFAMAA-derived facility count study (Adeloye et al., Journal of Multidisciplinary Healthcare, 2023), cross-checked against a third independent source.
- Records: 2,319 facilities, reflecting Lagos State's actual public/private and tertiary/secondary/primary facility mix — not an arbitrary row count.
- Key fields: Facility ID, LGA, Ownership Type, Facility Level, Subscription Plan, Monthly Subscription Fee, Total Staff Licensed, Active Doctors/Nurses/Other Staff, Feature Adoption (%), Days Since Last Login, Customer Health Score, Health Category, Customer Status, Churn Reason, Customer Lifetime Value.
- Naming integrity: Facilities use systematic, clearly labeled generic identifiers rather than invented names.

## Data Transformation & Cleaning 
- Removing all fields irrelevant to a B2B healthcare SaaS context (customer demographics, US-specific geography, telecom product features).
- Replacing US ZIP/Lat-Long geography with Lagos State's 20 Local Government Areas (LGAs).
- Renaming and repurposing telecom fields into healthcare-SaaS equivalents (e.g. Contract → Subscription Plan, Monthly Charges → Monthly Subscription Fee, converted to Naira).
- Reducing row count from 7,043 to 2,319 to match Lagos State's actual facility population.
- Rebuilding engagement metrics so they are internally consistent: 
     - Active Users is a formula equal to Active Doctors + Active Nurses + Active Other Staff, so it can never be smaller than its own components.
     - Feature Adoption (%) is capped at 100% by construction, since active staff can never exceed licensed staff.
     - Health Category is derived directly from Customer Health Score via formula, so the two can never contradict each other.
     - Customer Status (Active/Churned) is generated as an outcome of health score and subscription plan, so churn rate, active-facility count, and revenue-at-risk stay proportionate to one another.
- Adding an Intervention Priority column, mapping Health Category into Customer-Success-facing labels (Critical → Immediate Attention, At Risk → Monitor & Engage, Healthy → Healthy).

## Data Modelling 
The model uses a single fact table (Facility Data) at the facility grain, with no separate dimension tables required — each row already represents one hospital/facility, and categorical fields (LGA, Ownership Type, Facility Level, Subscription Plan, Health Category, Intervention Priority) serve as the slicing dimensions directly within that table.

## Analysis & Measures
Key DAX Measures built for this project: 
``` Total Facilities = 
COUNTROWS('Facility Data') ```

``` Active Facilities = 
CALCULATE(COUNTROWS('Facility Data'), 'Facility Data'[Customer Status] = "Active") ```

``` Churn Rate = 
DIVIDE(
    CALCULATE(COUNTROWS('Facility Data'), 'Facility Data'[Customer Status] = "Churned"),
    COUNTROWS('Facility Data') 
) ```

``` Total Monthly Revenue = 
SUM('Facility Data'[Monthly Subscription Fee (NGN)]) ```

``` Revenue at Risk = 
CALCULATE(
    SUM('Facility Data'[Monthly Subscription Fee (NGN)]),
    'Facility Data'[Customer Status] = "Active",
    'Facility Data'[Health Category] IN {"At Risk", "Critical"}
) ```

``` Active Critical Facilities = 
CALCULATE(
    DISTINCTCOUNT('Facility Data'[Facility ID]),
    'Facility Data'[Customer Status] = "Active",
    'Facility Data'[Health Category] = "Critical"
) ```

``` Active Critical MRR = 
CALCULATE(
    SUM('Facility Data'[Monthly Subscription Fee (NGN)]),
    'Facility Data'[Customer Status] = "Active",
    'Facility Data'[Health Category] = "Critical"
) ```












