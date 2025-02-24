# Clinical Database for Genetic Testing Center

> **Disclaimer**: This project is based on a fictional genetic testing company and was created as a final project for a Clinical Database Design course. The database design, data, and queries are intended solely for educational purposes.

## Background

A genetic testing company specializes in analyzing an individual's DNA to provide insights into various aspects of their genetic makeup. The primary goal is to uncover pathogenic variants that cause disease phenotypes. This process can be streamlined by comparing the patient's DNA against a family member’s and looking at the differences between their DNA sequence. Variants that are found in both the patient and the familial control’s sequence can be eliminated, while variants that are only in the patients sequence can be further investigated for pathogenicity. 

Genetic testing can reveal potential health risks based on variations in specific genes. This information may include predispositions to certain diseases or conditions, allowing individuals to make informed decisions about their lifestyle, healthcare, and preventive measures. In addition to this, genetic testing helps individuals identify whether they carry specific genetic mutations associated with hereditary conditions. This information is particularly relevant for family planning, as it can indicate the risk of passing on certain genetic disorders to future generations.

Patients are referred to this company through genetics counselors to ensure they are receiving the proper test for their phenotype. Genetic counselors provide support, address concerns, and help interpret the implications of genetic information. The specific type of genetic testing is chosen based on specific interests and concerns, whether related to ancestry, health, or other factors. 

This company performs DNA extraction from whole blood, bone marrow, and saliva samples using Qiagen DNA extraction kits. DNA libraries are prepared for sequencing using Illumina library prep kits and sequenced on Illumina Next-Generation Sequencers.


---

## Clinical Database Requirements

Patient and family information will be recorded when they are referred from their clinician or genetic counselor. Patient demographic data (first name, last name, sex, age, ethnicity, address, and phone number) will be associated with a unique patient ID (PID). Relatives that participate as familial controls will have unique family IDs (FID) for their information (first name, last name, sex, age, ethnicity). Patients can have multiple familial controls but each familial control will only be associated with one patient. 

Each patient and familial control will provide a specimen for genetic testing. Specimen data (specimen type, collection date, volume, stability) will have a unique specimen ID (SID). All specimens will be associated with a patient, but will only be associated with a familial control ID if it came from a family member and not a patient.

Once specimens are provided, they will have tests ordered under unique test IDs (TID). Test data (turn-around time, and methodology) will be tied to specimen IDs. In addition to this, each specimen will be provided with a result once testing is finished. Results that identify variants are tied to unique variant IDs (VID) which are associated with variant data (gene, c.dot, p.dot, and classification). Results that are inconclusive will have unique inconclusive IDs (IID) which are associated with the corresponding inconclusive result data (notes).

## Conceptual Data Model

![Conceptual Data Model](Images/ERModel.png)

<sub>**Figure 1.** Conceptual data model for genetic testing company. Entities are represented as rectangles, relations as diamonds, and attributes as ovals. Identify attributes are underlined and multivalued attributes are in double ovals.One-to-many cardinality is represented by a 1 and M between relationship entities. A double line connected to a circle with single line branches represents total specialization; ie: a specimen's result can have either a variant or be inconclusive but not both.</sub>

## Logical Database Design Model

![Logical Database Model](Images/LogicalDatabase.png)

<sub>**Figure 2.** Logical Database Design for genetic testing company. Entities are represented as relation tables. Foreign keys are labeled as FK and primary keys are labeled as PK.</sub>

## Database Design Language (DDL) Implementation

### Patient Table
```sql
CREATE TABLE [PATIENT](
  [PID] varchar(15) NOT NULL, 
  [FirstName] varchar(15), 
  [LastName] varchar(25), 
  [Sex] varchar(6), 
  [Age] integer, 
  [Ethnicity] varchar(25), 
  [Address] varchar(50), 
  [Phone] integer, 
  CONSTRAINT [PATIENT_PK] PRIMARY KEY([PID])
);
```

### Familial Control Table
```sql
CREATE TABLE [FAMCONTROL](
  [FID] varchar(15) NOT NULL, 
  [PID] varchar(15) NOT NULL, 
  [Relation] varchar(15), 
  [F_FirstName] varchar(15), 
  [F_LastName] varchar(25), 
  [F_Sex] varchar(6), 
  [F_Age] integer, 
  [F_Ethnicity] varchar(25), 
  CONSTRAINT [FAMCONTROL_PK] PRIMARY KEY([FID]), 
  CONSTRAINT [FAMCONTROL_fk1] FOREIGN KEY([PID]) REFERENCES [PATIENT]([PID])
);
```

### Specimen Table
```sql
CREATE TABLE [SPECIMEN](
  [SID] varchar(15) NOT NULL, 
  [PID] varchar(15) NOT NULL, 
  [FID] varchar(15), 
  [Type] varchar(25), 
  [Collection_Date] date, 
  [Volume] varchar(15), 
  [Stability] varchar(25), 
  CONSTRAINT [SPECIMEN_PK] PRIMARY KEY([SID]), 
  CONSTRAINT [SPECIMEN_fk1] FOREIGN KEY([PID]) REFERENCES [PATIENT]([PID]), 
  CONSTRAINT [SPECIMEN_fk2] FOREIGN KEY([FID]) REFERENCES [FAMCONTROL]([FID])
);
```

### Test Table
```sql
CREATE TABLE [TEST](
  [TID] varchar(15) NOT NULL, 
  [SID] varchar(15) NOT NULL, 
  [TAT] varchar(25), 
  [Methodology] varchar(25), 
  CONSTRAINT [TEST_PK] PRIMARY KEY([TID]), 
  CONSTRAINT [TEST_fk1] FOREIGN KEY([SID]) REFERENCES [SPECIMEN]([SID])
);
```

### Variant Table
```sql
CREATE TABLE [VARIANT](
  [VID] varchar(25) NOT NULL, 
  [SID] varchar(25) NOT NULL, 
  [Gene] varchar(25), 
  [cdot] varchar(25), 
  [pdot] varchar(25), 
  [Classification] varchar(25), 
  CONSTRAINT [VARIANT_PK] PRIMARY KEY([VID]), 
  CONSTRAINT [VARIANT_fk1] FOREIGN KEY([SID]) REFERENCES [SPECIMEN]([SID])
);
```

### Inconclusive Table
```sql
CREATE TABLE [INCONCLUSIVE](
  [IID] varchar(15) NOT NULL, 
  [SID] varchar(15) NOT NULL, 
  [Notes] varchar(50), 
  CONSTRAINT [INCONCLUSIVE_PK] PRIMARY KEY([IID]), 
  CONSTRAINT [INCONCLUSIVE_fk1] FOREIGN KEY([SID]) REFERENCES [SPECIMEN]([SID])
);
```
Attribute values were inserted into data tables using INSERT VALUES SQL functions. The data was fabricated using random patient data and variants that are commonly found in general DNA testing.

## Example Queries

### Query to find patients with variants
```sql
SELECT DISTINCT
  PATIENT.FirstName,
  PATIENT.LastName,
  VARIANT.Gene,
  VARIANT.cdot,
  VARIANT.pdot,
  VARIANT.Classification
FROM PATIENT
JOIN SPECIMEN ON PATIENT.PID = SPECIMEN.PID
JOIN VARIANT ON SPECIMEN.SID = VARIANT.SID;
```
### Query to find familial controls with variants
```sql
SELECT DISTINCT
  FAMCONTROL.F_FirstName,
  FAMCONTROL.F_LastName,
  VARIANT.Gene,
  VARIANT.cdot,
  VARIANT.pdot,
  VARIANT.Classification
FROM FAMCONTROL
JOIN SPECIMEN ON FAMCONTROL.FID = SPECIMEN.FID
JOIN VARIANT ON SPECIMEN.SID = VARIANT.SID;
```

### Query to find patients and familial controls with gemrline variants
```sql
SELECT DISTINCT
  PATIENT.FirstName || 'and' || FAMCONTROL.F_FirstName AS SharedFirstName,
  COALESCE(PATIENT.LastName, FAMCONTROL.F_LastName) AS SharedLastName,
    VARIANT.Gene,
    VARIANT.cdot,
    VARIANT.pdot,
    VARIANT.Classification
FROM VARIANT
JOIN SPECIMEN ON VARIANT.SID = SPECIMEN.SID
JOIN PATIENT ON SPECIMEN.PID = PATIENT.PID
JOIN FAMCONTROL ON SPECIMEN.FID = FAMCONTROL.FID;
```

### Query to find patients with somatic variants
```sql
SELECT
  PATIENT.FirstName,
  PATIENT.LastName,
  VARIANT.Gene,
  VARIANT.cdot,
  VARIANT.pdot,
  VARIANT.Classification
FROM PATIENT
JOIN SPECIMEN ON PATIENT.PID = SPECIMEN.PID
JOIN VARIANT ON SPECIMEN.SID = VARIANT.SID;
GROUP BY PATIENT.FirstName, PATIENT.LastName, VARIANT.Gene, VARIANT.cdot, VARIANT.pdot, VARIANT.Classification
HAVING COUNT(VARIANT.SID) = 1;
```

### Query to find specimen with the earliest turnaround time
```sql
SELECT
  TEST.SID AS SpecimenID,
  SPECIMEN.Type,
  SPECIMEN.Collection_Date,
  TEST.TAT AS TurnaroundTime
FROM TEST
JOIN SPECIMEN ON TEST.SID = SPECIMEN.SID
ORDER BY TEST.TAT
LIMIT 1;
```

## Discussion

The most significant challenge for this database construction was identifying how to associate patient results with their familial control results. I found the easiest solution to this problem was to associate the results with each specimen that was provided and then tie the specimen back to the patient and familial controls. In addition to this, I found it difficult to modify attribute types using SQLite. When correcting mistakes that were made in column types, SQLite didn’t seem to support MODIFY COLUMN on ALTER TABLE. This led to me needing to drop the column and recreate it entirely. In the future, I would use a more advanced SQL software to code this database. 

Given more time and resources, I would improve upon this project by including connections to referring clinicians and genetic counselors so their data could be included in the database. Ideally, these clinicians would have their own databases where they gather patient demographics and determine the appropriate panel to order for genetic testing.
