# Hospital-Records-Analysis

This project provides a set of SQL queries for analyzing hospital encounter data. The focus is on understanding patient admissions and readmissions, average length of stay, encounter costs, and insurance coverage of procedures. The dataset follows a normalized healthcare schema, including tables for encounters, procedures, patients, payers, and organizations.

## 📊**Analysis Overview**

### 1. **Patient Admission & Readmission Over Time**
This query analyzes patient encounters over time to identify trends in total admissions and estimates potential readmissions by counting repeat patient IDs.

```sql
select  
    to_char(start, 'YYYY') as year,
    to_char(start, 'Month') as month,
    count(distinct e.Id) as total_admissions,
    count(e.Patient) - count(distinct e.Patient) as possible_readmissions
from encounters e
where encounter_class in ('inpatient', 'emergency')
group by year, month, extract(month from start)
order by year, extract(month from start);
```

### 2. **Readmissions Within 30 Days**
This query calculates the number of readmissions that occur within 30 days of a previous encounter for the same patient.

```sql
with patient_encounters as (
  select 
    patient,
    start,
    lag(start) over (partition by patient order by start) as previous_start
  from encounters
)
select
  count(*) as readmissions_within_30_days
from patient_encounters
where previous_start is not null
  and start - previous_start <= interval '30 days';
```

### 3. Average Length of Stay
This query computes the average duration of hospital stays across all encounters where both a start and stop time are available.

```sql
select 
  avg(stop - start) as avg_length_of_stay
from encounters
where stop is not null and start is not null;
```

### 4. Average Length of Stay by Encounter Type
This version of the LOS query breaks down the average stay by encounter class (e.g., inpatient, emergency).

```sql
select 
  encounter_class,
  avg(stop - start) as avg_lenofstay
from encounters
where stop is not null and start is not null
group by encounter_class;
```

### 5. Average Cost per Visit
This query calculates the average total cost per patient encounter.

```sql
select 
  avg(total_class) as avg_cost_per_visit
from encounters;

```
### 6. Average Cost by Encounter Class
This version breaks down the average cost by the type of encounter.

```sql
select
  encounter_class,
  avg(total_class) as avg_total_cost
from encounters
group by encounter_class;
```

### 7. Procedures Covered by Insurance
This query counts how many procedures are tied to encounters with payer coverage greater than 0.

```sql
select
  count(*) as insured_procedure_count
from procedures pr
join encounters e on pr.encounter = e.id
where e.payer_coverage > 0;
```

### 8. Percentage of Procedures Covered by Insurance
Calculates the percentage of all procedures that were covered by insurance.

```sql
select 
  round(
    100.0 * count(case when e.payer_coverage > 0 then 1 end) / count(*),
    2
  ) as percent_procedures_covered_by_insurance
from procedures pr
join encounters e on pr.encounter = e.id;
```





