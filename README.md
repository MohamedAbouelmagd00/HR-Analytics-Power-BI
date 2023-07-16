# README

The datasets are in 5 files `EducationLevel.csv`, `Employee.csv`, `PerformanceRating.csv`, `RatingLevel.csv`, and `SatisfiedLevel.csv` 

# Data Modeling and EDA

## Data Modeling

### Creating DimDate table.

```xml
DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(

    CALENDAR(

                DATE(_minYear,1,1),

                DATE(_maxYear,12,31)

),

"Year",YEAR([Date]),
"Year Start",DATE( YEAR([Date]),1,1),
"YearEnd",DATE( YEAR([Date]),12,31),
"MonthNumber",MONTH([Date]),
"MonthStart",DATE( YEAR([Date]), MONTH([Date]), 1),
"MonthEnd",EOMONTH([Date],0),
"DaysInMonth",DATEDIFF(DATE( YEAR([Date]), MONTH([Date]), 1),EOMONTH([Date],0),DAY)+1,
"YearMonthNumber",INT(FORMAT([Date],"YYYYMM")),
"YearMonthName",FORMAT([Date],"YYYY-MMM"),
"DayNumber",DAY([Date]),
"DayName",FORMAT([Date],"DDDD"),
"DayNameShort",FORMAT([Date],"DDD"),
"DayOfWeek",WEEKDAY([Date]),
"MonthName",FORMAT([Date],"MMMM"),
"MonthNameShort",FORMAT([Date],"MMM"),
"Quarter",QUARTER([Date]),
"QuarterName","Q"&FORMAT([Date],"Q"),
"YearQuarterNumber",INT(FORMAT([Date],"YYYYQ")),
"YearQuarterName",FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
"QuarterStart",DATE( YEAR([Date]), (QUARTER([Date])*3)-2, 1),
"QuarterEnd",EOMONTH(DATE( YEAR([Date]), QUARTER([Date])*3, 1),0),
"WeekNumber",WEEKNUM([Date]),
"WeekStart", [Date]-WEEKDAY([Date])+1,
"WeekEnd",[Date]+7-WEEKDAY([Date]),
"FiscalYear",if(_fiscalStart=1,YEAR([Date]),YEAR([Date])+ QUOTIENT(MONTH([Date])+ (13-_fiscalStart),13)),
"FiscalQuarter",QUARTER( DATE( YEAR([Date]),MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1,1) ),
"FiscalMonth",MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1
)
```

### Creating The Snowflake schema model

- Connect  `DimDate`, let's connect it to the `FactPerformanceRating` table.
- Connect the `DimDate` table to the `DimEmployee` table. This relationship will be inactive because Power BI cannot have more than one active relationship between the same tables at once.
- Connect the `DimEducationLevel` table to the `DimEmployee` table.
- In the `FactPerformanceRating` table we have four different types of performance ratings: `EnvironmentSatisfaction`, `JobSatisfaction`, `RelationshipSatisfaction`, and `WorkLifeBalance`.
These all have numbers from one to five, but we have no context on what these ratings mean.
Connect these columns with `DimSatisfiedLevel` and use `EnvironmentSatisfaction` as the active relationship, using the *Manage Relationships* menu.
- In the `FactPerformanceRating` table we have two different types of overall performance ratings: `SelfRating` and `ManagerRating`.
These all have numbers from one to five, but we have no context on what these ratings mean.
Connect these columns with `DimRatingLevel` and use `SelfRating` as the active relationship, using the *Manage Relationships* menu.

## EDA

### **Building the *Overview* Page**

- **Exploring the data**
    1.  Create a new empty table called `_Measures`.
        
        ```python
        _Measures = Row("Column", BLANK())
        ```
        
    2. Create a `TotalEmployees` measure inside the `_Measures` table, which takes the count of all employees. Display this measure on a card visual.
        
        ```sql
        TotalEmployees = DISTINCTCOUNT(DimEmployee[EmployeeID])
        ```
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled.png)
        
    3. Create two more measures: `ActiveEmployees` and `InactiveEmployees` inside the `_Measures` table, which takes the count of all employees that are currently active or inactive, respectively. (You can use `DimEmployee[Attrition]` to determine whether an employee is active or inactive: a "Yes" means the employee is inactive, a "No" means the employee is active.)
    Display both measures on card visuals.
        
        ```jsx
        ActiveEmployees = CALCULATE(DISTINCTCOUNT(DimEmployee[EmployeeID]), DimEmployee[Attrition] = "NO")
        ```
        
        ```sql
        InactiveEmployees = CALCULATE(DISTINCTCOUNT(DimEmployee[EmployeeID]), DimEmployee[Attrition] = "YES")
        ```
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%201.png)
        
    4. Calculate `% Attrition Rate` based on the previous measures that you have created. Format this measure as a percentage with 1 decimal place.
    Display this measure on a card visual.
        
        ```sql
        % Attrition Rate = [InactiveEmployees] / [TotalEmployees]
        or 
        % Attrition Rate = DIVIDE([InactiveEmployees], [TotalEmployees])
        ```
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%202.png)
        
    
    ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%203.png)
    
- **Hiring Trends over time.**
    
    They would like to start with analyzing their hiring trends over time to see where they see the biggest growth in new employees.
    In this exercise, we'll be looking to activate relationships between tables. In this case, `DimDate` already has an active relationship with `FactPerformanceRating`. Therefore, we'll need to utilize `USERELATIONSHIP()` to count the number of employees by date.
    
    1. Create a stacked column chart to show `TotalEmployees` by `Date`.
    Hmm… that didn't look quite right! That's because it has an inactive relationship. Let's activate it!
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%204.png)
        
    2. Create a new measure called `TotalEmployeesDate` that uses the `CALCULATE()` function on our previous measure and the `USERELATIONSHIP()` function in the filter.
        
        ```sql
        TotalEmployeesDate = CALCULATE([TotalEmployees], USERELATIONSHIP(DimDate[Date], DimEmployee[HireDate]))
        ```
        
        - Replace `TotalEmployees` in our chart with our newly created `TotalEmployeesDate`.
            
            ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%205.png)
            
        - Add `Attrition` to the chart to see the split of employees by active vs inactive.
            
            ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%206.png)
            
        - Change the X-axis from 'Continuous' to 'Categorical'.
            
            ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%207.png)
            
        - Rename the chart "Employee Hiring Trends"
            
            ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%208.png)
            
        
- **Analyzing departments and job roles.**
    1. Create a Clustered bar chart to show `ActiveEmployees` by `Department`. Rename this chart "Active Employees by Department”.
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%209.png)
        
    2. Create another appropriate visualization that displays `ActiveEmployees` by `Department` and `JobRole`. Rename this chart "Active Employees by Department and Job Role".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2010.png)
        
    

![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2011.png)

![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2012.png)

### Building *Demographics* Page

- **********************************************************Demographics: Age and Gender**********************************************************
    1. Create two card visuals that displays the minimum and maximum values for `Age`.
    Remove the category label for both visuals and rename the cards "Youngest Employee"/"Oldest Employee".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2013.png)
        
    2.  In *Power Query*, create a conditional column called `AgeBins` that separates employees ages by bins in the following structure: <\20, 20-29, 30-39, 40-49, 50>.
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2014.png)
        
        • Next, select an appropriate chart that displays `TotalEmployees` by `AgeBins`. Ensure that the chart is sorted by `AgeBins` ascending.
        • Rename the chart "Employees by Age"
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2015.png)
        
    3. • Duplicate the "Employees by Age" chart.
    • Change to an appropriate chart that shows the `TotalEmployees` value distribution across `AgeBins` and `Gender`.
    • Rename the chart "Employees by Age and Gender"
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2016.png)
        
    4. Add a page level filter that enables you to look at the report page based on whether an employee is current active or inactive.
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2017.png)
        
- **Demographics: marital status and ethnicity**
    1. • Select an appropriate visualization that displays a count of all employees by `MaritalStatus`.
    • Rename the chart "Employees by Marital Status"
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2018.png)
        
    2. Create an `AverageSalary` measure inside the `_Measures` table, which works out the average salary of all employees.
    Format this measure as a currency with 0 decimal points.
        
        ```sql
        AverageSalary = AVERAGE(DimEmployee[Salary])
        ```
        
    3. Select an appropriate visualization that displays the count of all employees and their average salary by `Ethnicity`.
    Rename the chart "Employees by Ethnicity and Average Salary".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2019.png)
        

![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2020.png)

### Building ********************Performance Tracker******************** Page

- **************************************************************Performance for each individual**************************************************************
    1. Create a calculated column `FullName` in the `DimEmployee` table that combines `FirstName` and `LastName`.
    2. Create a slicer that will be able to filter the report page based on the employee's `FullName`. Rename slicer to "Select employee".
    Ensure that *Single select,* and *Search* are enabled.
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2021.png)
        
    3. Create a card visual that displays the selected employees `HireDate`. Rename the card visual to "Start Date".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2022.png)
        
    4. Create a measure `LastReviewDate` that gets the last performance review for the selected individual.
    If no review has taken place (the date is `BLANK()`), display a message that no review has happened. Format as "mm/dd/yyyy".
    Display this measure on a card visual and rename it to "Last Review".
        
        ```sql
        LastReviewDate = 
        var MaxDate = MAX(FactPerformanceRating[ReviewDate])
        RETURN
            IF(NOT ISBLANK(MaxDate)
            , FORMAT(MaxDate, "mm/dd/yyyy")
            , "No review has happend")
        
        OR
        
        LastReviewDate = 
        IF (
            MAX ( FactPerformanceRating[ReviewDate] ) = BLANK (),
            "No Review Yet",
            MAX ( FactPerformanceRating[ReviewDate] )
        )
        ```
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2023.png)
        
    5. Create a measure `NextReviewDate` that calculates when the next review is due. It should be 365 days after the `LastReviewDate`.
    If no review has occurred yet, it should be 365 days from the `HireDate`. Format as Date (mm/dd/yyyy).
    Display this measure on a card visual and rename it to "Next Review".
        
        ```sql
        NextReviewDate = 
        IF( [LastReviewDate] = "No review has happend"
        , FORMAT(MIN(DimEmployee[HireDate]) + 365, "MM/DD/YYYY")
        , FORMAT([LastReviewDate] + 365, "MM/DD/YYYY"))
        
        OR
        
        NextReviewDate = 
        VAR reviewOrHire =
            IF (
                MAX ( FactPerformanceRating[ReviewDate] ) = BLANK (),
                MAX ( DimEmployee[HireDate] ),
                MAX ( FactPerformanceRating[ReviewDate] )
            )
        RETURN
            reviewOrHire + 365
        ```
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2024.png)
        
    6. Create a `JobSatisfaction` measure inside the `_Measures` table which works out the max `FactPerformanceRating[JobSatisfaction]` level.
        
        ```sql
        JobSatisfaction = MAX(FactPerformanceRating[JobSatisfaction])
        ```
        
        That was easy; however, the other three satisfaction metrics do not currently have an active relationship to `DimSatisfiedLevel`. Let's activate it.
        
    7. Create three new measures `EnvironmentSatisfaction`, `RelationshipSatisfaction`, and `WorkLifeBalance` that use `USERELATIONSHIP()`.
        
        ```sql
        EnvironmentSatisfaction = CALCULATE(MAX(FactPerformanceRating[EnvironmentSatisfaction]), USERELATIONSHIP(DimSatisfiedLevel[SatisfactionID], FactPerformanceRating[EnvironmentSatisfaction]))
        ```
        
        ```sql
        RelationshipSatisfaction = CALCULATE(MAX(FactPerformanceRating[RelationshipSatisfaction]), USERELATIONSHIP(DimSatisfiedLevel[SatisfactionID], FactPerformanceRating[RelationshipSatisfaction]))
        ```
        
        ```sql
        WorkLifeBalance = CALCULATE(MAX(FactPerformanceRating[WorkLifeBalance]), USERELATIONSHIP(DimSatisfiedLevel[SatisfactionID], FactPerformanceRating[WorkLifeBalance]))
        ```
        
    8. Time to visualize! Choose an appropriate chart (or multiple charts) that shows `EnvironmentSatisfaction`, `JobSatisfaction`, `WorkLifeBalance`, and `RelationshipSatisfaction` by year.
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2025.png)
        
    9. Create two more measures, `SelfRating` and `ManagerRating`, that provide the max rating id, using `USERELATIONSHIP()`.
        
        ```sql
        SelfRating = CALCULATE(MAX(FactPerformanceRating[SelfRating]), USERELATIONSHIP(FactPerformanceRating[SelfRating], DimRatingLevel[RatingID]))
        ```
        
        ```sql
        ManagerRating = CALCULATE(MAX(FactPerformanceRating[ManagerRating]), USERELATIONSHIP(FactPerformanceRating[ManagerRating], DimRatingLevel[RatingID]))
        ```
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2026.png)
        

![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2027.png)

![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2028.png)

### Building ***********Attrition*********** Page

- **Investigating `% Attrition Rate`**
    1. Copy and paste OR create a new card visual using the `% Attrition Rate` measure.
    2. Select an appropriate visualization that displays the `% Attrition Rate` for each department and job role.
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2029.png)
        
- **Understanding our `% Attrition Rate` based on our `HireDate`.**
    1. First, create a new measure that uses `CALCULATE()` on our `InactiveEmployees` and makes use of the `USERELATIONSHIP` function. Name this measure `InactiveEmployeesDate`.
        
        ```sql
        InactiveEmployeesDate = CALCULATE([InactiveEmployees], USERELATIONSHIP(DimEmployee[HireDate], DimDate[Date]))
        ```
        
    2. Next, create a measure that calculates the rate of attrition based on `InactiveEmployeesDate` and `TotalEmployeesDate`. Name this `% Attrition Rate Date`. Format as a percentage with one decimal place.
        
        ```sql
        % Attirition Rate Date = DIVIDE([InactiveEmployeesDate], [TotalEmployeesDate])
        ```
        
    3. Select an appropriate visualization that displays the `% Attrition Rate Date` over time.
    Rename the chart "Attrition by Hire Date".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2030.png)
        
    
- ****************************************************************************Attrition by Travel Frequency, Overtime, Tenure.****************************************************************************
    1. Select an appropriate visualization that displays the `% Attrition Rate` and `TotalEmployees` for `Business Travel` .
    Rename the chart "Attrition by Travel Frequency".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2031.png)
        
    2. Select an appropriate visualization that displays the `% Attrition Rate` based on whether an employee does overtime or not.
    Rename the chart "Attrition by Overtime Requirement".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2032.png)
        
    3. Select an appropriate visualization that displays the `% Attrition Rate` based on years working for Atlas Labs.
    Rename the chart "Attrition by Tenure".
        
        ![Untitled](README%2089822b6d47a04160924301ee8c2bced5/Untitled%2033.png)
        
    

## Finalizing the Report

1. **************************************Layout Desin improvement.**************************************
2. **Theme formatting**
3. **Page navigation**

![1.Overview.png](README%2089822b6d47a04160924301ee8c2bced5/1.Overview.png)

![2.Demographics.png](README%2089822b6d47a04160924301ee8c2bced5/2.Demographics.png)

![3.Performance Tracker.png](README%2089822b6d47a04160924301ee8c2bced5/3.Performance_Tracker.png)

![4.Attrition.png](README%2089822b6d47a04160924301ee8c2bced5/4.Attrition.png)

![snowflake Schema.png](README%2089822b6d47a04160924301ee8c2bced5/snowflake_Schema.png)