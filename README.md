# Vehicle Inventory Aging Analytics Dashboard

<img src="https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" alt="Power BI"> <img src="https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logoColor=white" alt="DAX"><img src="https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logoColor=white" alt="Power Query"><img src="https://img.shields.io/badge/SQL-336791?style=for-the-badge&logoColor=white" alt="SQL"><img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">

An interactive Power BI dashboard for monitoring vehicle inventory age, identifying aging units, and evaluating operational progress through allocation, completion, release, and shipment.

## Dashboard Preview

<img width="1062" height="981" alt="image" src="https://github.com/user-attachments/assets/f22d88c6-1082-418c-b3aa-3ccc049b9f22" />

## Project Overview

Managing vehicle inventory requires more than knowing how many units are currently on the ground. Operations teams also need to understand how long vehicles have remained in inventory, where they are in the processing lifecycle, and which facilities or vehicle programs are contributing to aging inventory.

This dashboard provides a historical, snapshot-based view of vehicle inventory. Users can select a reporting date and analyze the inventory that was on the ground as of that date rather than only viewing the current inventory position.

The report combines executive KPIs, age distributions, percentile analysis, vehicle-program summaries, and VIN-level detail into a single interactive dashboard.

## Business Questions

The dashboard was designed to answer questions such as:

- How many vehicles were on the ground on a selected date?
- What was the average age of the inventory?
- How many vehicles had not been allocated?
- How many vehicles had not completed processing?
- How many vehicles had not been released?
- What percentage of inventory exceeded a specific age threshold?
- Which facilities or vehicle programs were driving aging inventory?
- How has average inventory age changed over time?
- Which individual units require operational attention?

## Inventory Views

Users can switch between four inventory populations:

| View | Definition |
| --- | --- |
| **All Units** | Every vehicle that was on the ground as of the selected snapshot date |
| **Not Allocated** | Vehicles that had not been allocated as of the snapshot date |
| **Not Completed** | Vehicles that had not completed processing as of the snapshot date |
| **Not Released** | Vehicles that had not been released as of the snapshot date |

Each selection dynamically updates the KPI cards, distribution chart, summary matrix, and VIN-level detail table.

## Key Performance Indicators

| KPI | Description |
| --- | --- |
| **Average Inventory Age** | Average number of days vehicles had been on the ground as of the selected snapshot date |
| **50th Percentile** | Inventory age at or below which 50% of eligible vehicles fall |
| **85th Percentile** | Inventory age at or below which 85% of eligible vehicles fall |
| **7-Day Comparison** | Percentage change compared with the previous seven-day period |
| **30-Day Comparison** | Percentage change compared with the previous 30-day period |
| **Weekly Trend** | Current-week inventory age compared with the previous three weeks |
| **MTD** | Month-to-date average inventory age |
| **QTD** | Quarter-to-date average inventory age |
| **YTD** | Year-to-date average inventory age |

Percentile metrics supplement the average by showing both typical inventory age and the longer-running portion of the inventory population.

## Dashboard Features

- Historical inventory snapshots
- Dynamic snapshot-date selection
- Average inventory-age KPIs
- Median and 85th-percentile analysis
- Seven-day and 30-day performance comparisons
- Current-week and previous-week summaries
- MTD, QTD, and YTD metrics
- Dynamic inventory-status selection
- Inventory-age histogram
- Cumulative-percentage analysis
- Age-bucket filtering
- Facility filtering
- Vehicle-program filtering
- Age distribution by vehicle program
- VIN-level drill-down details
- Clear Filters button for resetting the report

## Snapshot-Based Inventory Logic

The dashboard reconstructs the inventory population as it existed on the selected reporting date.

A vehicle is considered on the ground when:

1. Its arrival date is on or before the selected snapshot date.
2. Its shipment date is blank or later than the selected snapshot date.

This design allows the report to recreate historical inventory conditions instead of only displaying the units that are currently on the ground.

For each operational view, allocation, completion, and release dates are evaluated relative to the selected snapshot date. A vehicle remains in a pending category when the applicable milestone date is blank or occurs after the snapshot date.

## Example DAX Logic

The following simplified DAX measure demonstrates the historical snapshot pattern used to calculate average inventory age:

```DAX
Average Age of Inventory =
VAR SnapshotDate =
    MAX('Dates'[Date])

VAR UnitsOnGround =
    FILTER(
        CALCULATETABLE(
            VINInventory,
            REMOVEFILTERS('Dates')
        ),
        NOT ISBLANK(VINInventory[Arrival Date])
            && VINInventory[Arrival Date] <= SnapshotDate
            && (
                ISBLANK(VINInventory[Ship Date])
                    || VINInventory[Ship Date] > SnapshotDate
            )
    )

RETURN
    AVERAGEX(
        UnitsOnGround,
        DATEDIFF(
            VINInventory[Arrival Date],
            SnapshotDate,
            DAY
        )
    )
```

The production report expands this pattern to separately evaluate vehicles that were not allocated, not completed, or not released as of the selected date.

## Age Distribution

Inventory is grouped into operational age ranges:

| Bucket | Age Range |
| --- | --- |
| **0–5** | 0 to 5 days |
| **6–10** | 6 to 10 days |
| **11–15** | 11 to 15 days |
| **16–20** | 16 to 20 days |
| **21–25** | 21 to 25 days |
| **26–30** | 26 to 30 days |
| **31–40** | 31 to 40 days |
| **41–50** | 41 to 50 days |
| **51–60** | 51 to 60 days |
| **61–90** | 61 to 90 days |
| **91–120** | 91 to 120 days |
| **121–150** | 121 to 150 days |
| **151–180** | 151 to 180 days |
| **181–210** | 181 to 210 days |
| **211–240** | 211 to 240 days |
| **241–270** | 241 to 270 days |
| **271+** | More than 270 days |

The histogram displays the number of vehicles in each bucket. The cumulative-percentage line shows how much of the total inventory falls at or below each age threshold.

## Report Sections

### Executive KPI Cards

The top section summarizes average age, median age, 85th-percentile age, recent changes, and cumulative reporting-period results for each inventory status.

### Age Distribution

The histogram shows the number of vehicles within each age bucket and includes a cumulative-percentage line for threshold analysis.

### KPI Summary Matrix

The matrix breaks down the selected inventory population by vehicle program and age bucket, making it easier to identify the programs contributing to aging inventory.

### VIN Inventory Detail

The detail table provides record-level information including:

- Facility
- Vehicle program
- VIN
- Vehicle location
- Model
- Current status
- Arrival date
- Allocation date
- Completion date
- Release date
- Shipment date
- Calculated inventory age

## Technical Implementation

| Technology | Purpose |
| --- | --- |
| **Power BI Desktop** | Data modeling, report development, and visual design |
| **DAX** | Snapshot calculations, inventory populations, percentiles, comparisons, and time intelligence |
| **Power Query** | Data extraction, transformation, cleansing, and type enforcement |
| **SQL** | Source validation, date verification, and inventory reconciliation |
| **Python** | Supplemental data profiling, validation, and exploratory analysis |
| **Bookmarks and Report Controls** | Inventory-view selection, navigation, and filter-reset behavior |

## Data Model

The analytical model includes:

- Vehicle inventory records
- Vehicle arrival dates
- Allocation dates
- Processing-completion dates
- Release dates
- Shipment dates
- Vehicle status information
- Facility and vehicle-program dimensions
- A dedicated calendar table
- Disconnected parameter tables for inventory views and age buckets

The model is designed so that the selected snapshot date determines both the eligible vehicle population and the calculated age of each vehicle.

## Data Quality and Validation

Validation checks were performed for:

- Missing arrival dates
- Missing milestone dates
- Shipment dates occurring before arrival
- Allocation, completion, or release dates occurring before arrival
- Duplicate VIN records
- Invalid placeholder dates
- Negative inventory ages
- Inconsistent vehicle-status records
- Historical inventory-count reconciliation
- Consistent filtering across summary and detail visuals

SQL and Power BI validation measures were used to confirm historical vehicle counts and milestone-date behavior.

## Skills Demonstrated

- Power BI dashboard development
- Historical snapshot modeling
- Advanced DAX
- Power Query transformation
- SQL data validation
- Python data analysis
- Time-intelligence calculations
- Percentile analysis
- Dynamic histogram development
- Interactive report navigation
- Operational inventory analysis
- Data-quality reconciliation
- Executive dashboard design

## Repository Structure

```text
.
├── README.md
├── inventory-aging-dashboard.png
├── Age of Inventory Dashboard.pbix
├── dax/
│   └── inventory-measures.dax
├── sql/
│   └── inventory-validation.sql
└── python/
    └── inventory-validation.py
```

Only include the DAX, SQL, and Python folders if those files are actually included in the repository.

## Privacy and Data Security

This repository is intended to demonstrate business-intelligence development, data modeling, and inventory-analysis skills.

The public version should not include:

- Complete VINs
- Customer information
- Proprietary operational data
- Company credentials
- Database connection strings
- Internal server names
- Confidential facility information

Screenshots and sample datasets should be anonymized, aggregated, or created using synthetic data.

## Author

**Joel Perez**

Data Analytics · Business Intelligence · Power BI · Power Platform
