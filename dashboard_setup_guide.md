# Power BI Dashboard Setup Guide

## About the `.pbix` File

The `dashboard.pbix` Power BI report file requires **Microsoft Power BI Desktop** to create and open. Power BI Desktop is a Windows application and cannot be generated in a command-line or Linux environment.

## Creating the Dashboard from Scratch

Follow these steps to build the full dashboard using the provided `dataset.xlsx`:

### Step 1 — Import Data

1. Open **Power BI Desktop**
2. Click **Home → Get Data → Excel Workbook**
3. Navigate to `dataset.xlsx` and select it
4. In the Navigator dialog, check all four sheets:
   - ✅ Patients
   - ✅ Doctors
   - ✅ Beds
   - ✅ Billing
5. Click **Transform Data** to open Power Query Editor

### Step 2 — Clean Data in Power Query

Apply the following transformations to each table:

**All tables:**
- Promote first row as headers (if not already done)
- Set correct data types for each column

**Patients table:**
- `PatientID` → Text
- `Age` → Whole Number
- `AdmissionDate` → Date
- `DoctorID` → Text

**Billing table:**
- `Amount` → Decimal Number
- `Date` → Date

Click **Close & Apply** when done.

### Step 3 — Create Relationships (Data Model)

Go to **Model view** and create these relationships:

| From | Column | To | Column | Cardinality |
|---|---|---|---|---|
| Patients | DoctorID | Doctors | DoctorID | Many-to-One |
| Billing | PatientID | Patients | PatientID | Many-to-One |

### Step 4 — Create DAX Measures

Create a new table called `_Measures` and add:

> **Prerequisite — Calendar Table:** Time-intelligence functions (`DATESYTD`, `DATESMTD`, `DATEADD`) require a contiguous Calendar table.
> Create one via **Modeling → New Table**:
> ```dax
> Calendar =
>     CALENDAR(DATE(2023,1,1), DATE(2024,12,31))
> ```
> Then go to **Model view** and create relationships:
> - `Calendar[Date]` → `Billing[Date]` (Many-to-One)
> - `Calendar[Date]` → `Patients[AdmissionDate]` (Many-to-One)

```dax
Total Patients = COUNTROWS(Patients)

Total Revenue = SUM(Billing[Amount])

Bed Occupancy % =
    DIVIDE(
        COUNTROWS(FILTER(Beds, Beds[Status] = "Occupied")),
        COUNTROWS(Beds), 0
    )

Average Billing = AVERAGE(Billing[Amount])

Available Beds = COUNTROWS(FILTER(Beds, Beds[Status] = "Available"))

Revenue YTD =
    CALCULATE([Total Revenue], DATESYTD(Billing[Date]))

Avg Patients per Doctor =
    DIVIDE([Total Patients], DISTINCTCOUNT(Patients[DoctorID]), 0)
```

### Step 5 — Build the Dashboard

**Page 1 — Executive Overview:**
- 4× Card visuals (Total Patients, Total Revenue, Bed Occupancy %, Avg Billing)
- Clustered Bar Chart: Disease on Y-axis, Count on X-axis
- Line Chart: AdmissionDate on X-axis, Total Patients on Y-axis
- Pie Chart: Status (from Beds) as Legend, Count as Values
- Slicers: AdmissionDate (Range), Doctor Name, Disease, Bed Type

**Page 2 — Revenue & Billing:**
- Line Chart: Date vs Total Revenue (monthly)
- Bar Chart: Top 10 patients by billing amount
- Bar Chart: Revenue by Specialization
- Histogram: Billing Amount distribution

**Page 3 — Bed & Doctor Utilisation:**
- Stacked Bar: Bed Type vs Count by Status
- Bar Chart: Top doctors by patient count
- Bar Chart: Patient Age Group distribution
- Card: Available Beds, Beds Under Maintenance

### Step 6 — Publish (Optional)

To share the report:
1. Click **Home → Publish**
2. Sign in to your Power BI account
3. Select a workspace
4. Access the report at [app.powerbi.com](https://app.powerbi.com)

---

*This guide accompanies the Hospital Resource Management Dashboard project.*
*See [README.md](README.md) for full project documentation.*
