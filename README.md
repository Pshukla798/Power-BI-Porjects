**Healthcare Data Analysis Project
**
This project focuses on analyzing healthcare data related to patient visits, wait times, satisfaction scores, demographics, and appointment types. 
Using Power BI and DAX measures, this analysis provides insights into various factors impacting healthcare operations and patient experiences.

 
**Project Structure**

├── LICENSE                         <- Project license (MIT, GPL, etc.).
├── README.md                       <- Project documentation for using and understanding the analysis.
├── data                            <- Dataset used in the project.
│   └── patients_data.csv           <- The dataset containing patient visit data.
├── reports                         <- Folder for final reports.
│   └── Healthcare_Analysis_Report.pdf <- The final PDF report summarizing the analysis.
├── dax_measures                    <- DAX measures for the analysis.
│   ├── dax_measures.sql            <- SQL-like DAX code for calculated measures.
│   └── dax_visualizations.sql      <- DAX code for visualizations and insights.
├── src                             <- Source files for custom calculations and analysis.
│   └── data_dictionary.csv         <- Data dictionary describing the dataset.
├── PowerBI                         <- Folder with the .pbix Power BI report file.
│   └── healthcare_analysis.pbix    <- The Power BI report file containing all visualizations.
└── LICENSE                         <- Project license file.

**Dataset Overview
**
The dataset contains the following information about patient visits to a healthcare facility:
**
Patient Demographics:** Includes gender, age, race, and patient ID.
**Appointment Details**: Information about whether the appointment was administrative or not, the referral type, and the wait time.
**Patient Experience: **Includes satisfaction scores and the moment of the day the appointment took place.

Problem Statement
The healthcare industry constantly strives to optimize its operations, enhance patient satisfaction, and improve overall service efficiency. However, understanding the complex dynamics of healthcare visits and patient experience can be challenging. This project aims to leverage data to answer key questions that can provide valuable insights into operational improvements and customer satisfaction.

The primary goal of this analysis is to evaluate healthcare operational metrics, focusing on the following aspects:

Average Wait Time: How long do patients typically wait before their appointments? Identifying trends and patterns in wait times is crucial for improving scheduling and overall service efficiency. This analysis will help to uncover times of the day, week, or year when patient wait times are higher than average.

Patient Satisfaction: Understanding the average patient satisfaction score and identifying the factors that contribute to a positive or negative patient experience is essential for any healthcare facility. By exploring patient satisfaction trends across various factors (age, gender, type of appointment, etc.), this analysis helps pinpoint areas where patient satisfaction can be improved.

Total Patient Visits Monthly: Knowing how many patients visit the healthcare facility each month and identifying patterns in the data helps forecast demand, optimize staffing, and ensure proper resource allocation. This analysis can highlight busy months, as well as identify quieter periods when operational adjustments can be made.

Administrative vs. Non-Administrative Appointments: Administrative appointments (those that involve paperwork, insurance validation, etc.) often differ in terms of wait times and patient satisfaction when compared to non-administrative appointments. This analysis will help distinguish between these appointment types and determine their impact on patient experience.

Referrals and Walk-In Patients: The balance between referred patients (those who come to the healthcare facility through a referral) and walk-in patients (those who come without prior appointments) can significantly impact wait times and patient satisfaction. This analysis aims to explore the relationship between referral status and patient experience.

Patient Visits by Age Group and Race: Healthcare needs and experiences can vary significantly across different demographic groups. Analyzing patient visits by age group and race will uncover any disparities in healthcare access or outcomes, helping to ensure equitable care for all patient groups.

Sample data includes the following columns:
![image](https://github.com/user-attachments/assets/27253d2b-c5b6-4c10-96f8-b88347d709fe)

**Key Analysis Questions
**

This analysis focuses on answering the following questions:

**Average Wait Time:** Discover how long patients typically wait before their appointments. Uncover patterns and trends to highlight inefficiencies in the system.
**Patient Satisfaction:** Explore the average satisfaction scores given by patients. Understand the factors that contribute to a positive patient experience.
**Total Patient Visits Monthly**: Gain an understanding of the flow of patients each month. Analyze trends in patient visits over time.
**Administrative vs. Non-Administrative Appointments:** Compare the impact of administrative versus non-administrative appointments on wait times and satisfaction.
**Referrals and Walk-In Patients:** Examine the balance between referred patients and walk-ins, and their effects on the overall patient experience.
**Patient Visits by Age Group and Race:** Analyze the distribution of patient visits across different age groups and races, uncovering any insights about healthcare preferences and needs.

**DAX Measures and Calculations
**
To answer the analysis questions, the following **DAX measures** were created:

**1. Percentage of Administrative Appointments**
% Administrative Schedule = 
    DIVIDE(
        COUNTROWS(FILTER('Patients Datasets', 'Patients Datasets'[patient_admin_flag] = TRUE())),
        [Total Patients]
    )

**2. Percentage of Female Visits
**
% Female visits = 
    DIVIDE(
        CALCULATE([Total Patients], 'Patients Datasets'[patient_gender] = "F"),
        [Total Patients]
    )

**3. Percentage of Male Visits
**

% Male visits = 
    DIVIDE(
        CALCULATE([Total Patients], 'Patients Datasets'[patient_gender] = "M"),
        [Total Patients]
    )
**4. Percentage of Non-Administrative Appointments
**

% Non_Administrative Schedule = 
    DIVIDE(
        COUNTROWS(FILTER('Patients Datasets', 'Patients Datasets'[patient_admin_flag] = FALSE())),
        [Total Patients]
    )

**5. Percentage of Not Rated (Satisfaction Missing)
**
% Not_rating = 
    VAR Not_ratings = 
        CALCULATE([Total Patients], 'Patients Datasets'[patient_sat_score] = BLANK())
    RETURN
        DIVIDE(Not_ratings, [Total Patients])

**6. Percentage of Referred Patients
**
% Referred patients = 
    VAR _filterpatients = 
        CALCULATE([Total Patients], 'Patients Datasets'[department_referral] <> "none")
    RETURN 
        DIVIDE(_filterpatients, [Total Patients])

**7. Percentage of Unreferred Patients
**
% un_referred patients = 
    VAR _filterpatients = 
        CALCULATE([Total Patients], 'Patients Datasets'[department_referral] = "none")
    RETURN 
        DIVIDE(_filterpatients, [Total Patients])

**8. Percentage of Unknown Visits (Gender = NC)
**
% Unknown visits = 
    DIVIDE(
        CALCULATE([Total Patients], 'Patients Datasets'[patient_gender] = "NC"),
        [Total Patients]
    )

**9. Average Satisfaction Score
**
Avg Satisfaction score = 
    CALCULATE(
        AVERAGE('Patients Datasets'[patient_sat_score]),
        'Patients Datasets'[patient_sat_score] <> BLANK()
    )

**10. Average Wait Time
**

Avg_wait_time = 
    AVERAGE('Patients Datasets'[patient_waittime])

**11. Caption Measure (Dynamic Measure Description)
**

Caption = 
    VAR _selectedMeasure = 
        SELECTEDVALUE(Parameter[Parameter Order])
    RETURN
        IF(_selectedMeasure = 0, 
            "The darkest part denotes low wait time", 
            "The darkest red denotes the low Satisfaction score")

**12. CF Max-Min Point Month
**
CF max_min Point month = 
    VAR _patient_table = 
        CALCULATETABLE(
            ADDCOLUMNS(
                SUMMARIZE('date', 'date'[Month]),
                "@total_patients", [Total Patients]
            ),
            ALLSELECTED()
        )
    VAR Minv = MINX(_patient_table, [@total_patients])
    VAR Maxv = MAXX(_patient_table, [@total_patients])
    VAR Total_patints_ = [Total Patients]
    RETURN
        SWITCH(
            TRUE(),
            Total_patints_ = Minv, 0,
            Total_patints_ = Maxv, 1
        )

**13. CF Max-Min Point Year
**
CF max_min Point Year = 
    VAR _patient_table = 
        CALCULATETABLE(
            ADDCOLUMNS(
                SUMMARIZE('date', 'date'[Year]),
                "total_patients", [Total Patients]
            ),
            ALLSELECTED()
        )
    VAR Miny = MINX(_patient_table, [total_patients])
    VAR Maxy = MAXX(_patient_table, [total_patients])
    VAR Total_patints_ = [Total Patients]
    RETURN
        SWITCH(
            TRUE(),
            Total_patints_ = Miny, 0,
            Total_patints_ = Maxy, 1
        )

**14. Total Patients
**

Total Patients = COUNTROWS('Patients Datasets')

**15. Max-Min Point Year Value
**

value max_min Point Year = 
    VAR _patient_table = 
        CALCULATETABLE(
            ADDCOLUMNS(
                SUMMARIZE('date', 'date'[Year]),
                "total_patients", [Total Patients]
            ),
            ALLSELECTED()
        )
    VAR Miny = MINX(_patient_table, [total_patients])
    VAR Maxy = MAXX(_patient_table, [total_patients])
    VAR Total_patints_ = [Total Patients]
    RETURN
        SWITCH(
            TRUE(),
            Total_patints_ = Miny, Total_patints_,
            Total_patints_ = Maxy, Total_patints_
        )

**16. Max-Min Point Month Value
**

Values max_min Point = 
    VAR _patient_table = 
        CALCULATETABLE(
            ADDCOLUMNS(
                SUMMARIZE('date', 'date'[Month]),
                "@total_patients", [Total Patients]
            ),
            ALLSELECTED()
        )
    VAR Minv = MINX(_patient_table, [@total_patients])
    VAR Maxv = MAXX(_patient_table, [@total_patients])
    VAR Total_patints_ = [Total Patients]
    RETURN
        SWITCH(
            TRUE(),
            Total_patints_ = Minv, Total_patints_,
            Total_patints_ = Maxv, Total_patints_
        )
**17. Parameters for Slicers
**
Parameter = {
    ("Avg Satisfaction score", NAMEOF('Calculations'[Avg Satisfaction score]), 0),
    ("Avg_wait_time", NAMEOF('Calculations'[Avg_wait_time]), 1)
}



# Power BI Report

The Power BI report (healthcare_analysis.pbix) provides the following insights:

**Visualizations:
**
Clustered Bar and Column Charts: Used to visualize trends in patient visits, satisfaction, and wait times.
Slicers: Filters for demographic data, such as gender, age, and referral status, to allow easy exploration of the data.
Cards: Display key metrics like average satisfaction score and average wait time.
Trend Charts: Display monthly or yearly trends for patient visits and wait times, which are key to understanding patterns over time.

Key Calculated Columns and Measures Used:
Age Group Buckets: Helps segment patients into different age groups.
Administrative vs. Non-Administrative Appointments: Differentiates the appointments that have administrative tasks versus those that do not.
Referred vs. Walk-In: Analyzes how the patient referral process affects healthcare experience.

Step-by-Step Guide
1. Clone the Repository
Clone this repository to your local machine to get started:

bash
Copy
git clone https://github.com/yourusername/healthcare-data-analysis.git
2. Set Up Power BI
Open Power BI Desktop.
Import the dataset (patients_data.csv) located in the src/data folder.
Create new columns and measures by copying the provided DAX formulas from dax_measures.
Build the visualizations using the healthcare_analysis.pbix template.
You can also add new insights or custom visualizations based on the data.
3. Explore the Power BI Report
Slicers: Use them to filter by patient gender, age group, or referral status.
Visuals: Examine how administrative appointments impact satisfaction or how patient satisfaction varies with age.
Trend Analysis: Use the trend charts to explore changes in patient visits or satisfaction over time.
4. Customize the Report
You can modify the existing Power BI report, or add new reports as per your needs:

Add new visualizations like stacked bar charts or scatter plots to explore other dimensions.
Create new DAX measures for additional insights or refine existing ones.

Contact
For any questions or suggestions, you can reach out to:

[Prashant Shukla: [shukla12798@gmail.com]
LinkedIn: [https://www.linkedin.com/in/prashant-shukla-8a2690235/overlay/contact-info/]

![image](https://github.com/user-attachments/assets/34153b12-b9cc-43fd-a231-944299a13aac)






















