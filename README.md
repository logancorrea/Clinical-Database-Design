# Clinical Database for Genetic Testing Center

## Background

A genetic testing company specializes in analyzing an individual's DNA to provide insights into various aspects of their genetic makeup. The primary goal is to uncover pathogenic variants that cause disease phenotypes. This process is streamlined by comparing the patient's DNA against a family member’s DNA and analyzing differences in their DNA sequences. Variants found in both the patient and familial control sequences are eliminated, while unique patient-specific variants are further investigated for pathogenicity.

Genetic testing provides potential health insights, including:
- **Predispositions to diseases**: Informing individuals about potential health risks.
- **Identification of hereditary conditions**: Helping families assess risks of passing genetic disorders to future generations.

Patients are referred to this company through genetic counselors, ensuring the proper test is selected for the patient's phenotype. The counselors also provide support, address concerns, and help interpret genetic test results.

**Sample Processing**:
- DNA extraction is performed using Qiagen DNA extraction kits.
- DNA libraries are prepared using Illumina library prep kits.
- Sequencing is conducted on Illumina Next-Generation Sequencers.

---

## Clinical Database Requirements

The database records patient and family information, specimen data, test orders, and results. Key features include:
- **Patient Information**: Each patient has a unique Patient ID (PID) and demographic details.
- **Family Controls**: Relatives have unique Family IDs (FID) and are associated with a single patient.
- **Specimen Data**: Each specimen has a unique Specimen ID (SID) and is linked to the patient and optionally to familial controls.
- **Test Data**: Each test order has a unique Test ID (TID) and is associated with a specimen.
- **Results**: Results include unique Variant IDs (VID) for positive findings and unique Inconclusive IDs (IID) for inconclusive results.

---

## Conceptual Data Model

![Conceptual Data Model](path/to/conceptual-data-model-image)

- **Entities**: Represented as rectangles.
- **Relations**: Represented as diamonds.
- **Attributes**: Represented as ovals.
- **Specialization**: Results are either variants or inconclusive but not both.

---

## Logical Database Design

![Logical Database Design](path/to/logical-database-design-image)

- **Primary Keys (PK)**: Uniquely identify records in tables.
- **Foreign Keys (FK)**: Establish relationships between tables.

---

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