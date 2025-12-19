# 📚 คู่มือ DataOps Foundation Jenkins Pipeline

## สารบัญ
- [1. ภาพรวมโปรเจค](#1-ภาพรวมโปรเจค)
- [2. โครงสร้างโปรเจค](#2-โครงสร้างโปรเจค)
- [3. Functions - ฟังก์ชันหลัก](#3-functions---ฟังก์ชันหลัก)
- [4. Tests - การทดสอบ](#4-tests---การทดสอบ)
- [5. ETL Pipeline - โปรแกรมหลัก](#5-etl-pipeline---โปรแกรมหลัก)
- [6. Jenkinsfile - ระบบ CI/CD](#6-jenkinsfile---ระบบ-cicd)
- [7. การไหลของข้อมูล](#7-การไหลของข้อมูล)
- [8. วิธีใช้งาน](#8-วิธีใช้งาน)
- [9. การแก้ไขปัญหา](#9-การแก้ไขปัญหา)

---

## 1. ภาพรวมโปรเจค

### 🎯 โปรเจคนี้คืออะไร?

**DataOps Foundation Jenkins** เป็นระบบ ETL (Extract, Transform, Load) Pipeline ที่ทำงานแบบอัตโนมัติผ่าน Jenkins โดยมีหน้าที่:

| ขั้นตอน | รายละเอียด |
|---------|-----------|
| **Extract** | อ่านข้อมูล Loan จากไฟล์ CSV |
| **Transform** | ทำความสะอาด กรอง และแปลงข้อมูล |
| **Load** | ส่งข้อมูลเข้า SQL Server Database |

### ✨ คุณสมบัติหลัก

- ✅ ประมวลผลข้อมูลขนาดใหญ่ (14,000+ records)
- ✅ ทำความสะอาดข้อมูลอัตโนมัติ
- ✅ มี Unit Test ตรวจสอบคุณภาพ
- ✅ สร้าง Star Schema (Fact + Dimension Tables)
- ✅ CI/CD Pipeline ผ่าน Jenkins
- ✅ รัน Tests แบบ Parallel เพื่อความเร็ว

---

## 2. โครงสร้างโปรเจค

```
📁 dataops-foundation-jenkins-new/
│
├── 📂 functions/                      # ฟังก์ชันสำหรับประมวลผลข้อมูล
│   ├── __init__.py                    # Export ฟังก์ชันทั้งหมด
│   ├── guess_column_types.py          # เดาประเภทข้อมูลจาก CSV
│   ├── filter_issue_date_range.py     # กรองข้อมูลตามช่วงปี
│   └── clean_missing_values.py        # ลบคอลัมน์ที่มี null มาก
│
├── 📂 tests/                          # ไฟล์ทดสอบ Unit Test
│   ├── guess_column_types_test.py     # ทดสอบการเดาประเภท
│   ├── filter_issue_date_range_test.py # ทดสอบการกรองวันที่
│   └── clean_missing_values_test.py   # ทดสอบการลบ null
│
├── 📂 data/                           # ไฟล์ข้อมูล
│   └── LoanStats_web_small.csv        # ข้อมูล Loan (14,422 rows)
│
├── 📂 day2/                           # Jupyter Notebooks (สำหรับเรียนรู้)
│
├── 📄 etl_pipeline.py                 # โปรแกรมหลัก ETL
├── 📄 Jenkinsfile                     # คำสั่ง Jenkins Pipeline
├── 📄 requirements.txt                # Python dependencies
├── 📄 setup_data.sh                   # Script เตรียมข้อมูล
├── 📄 manual.md                       # คู่มือเดิม
└── 📄 README.md                       # คำอธิบายโปรเจค
```

### 📊 ความสัมพันธ์ระหว่างไฟล์

```
┌─────────────────────────────────────────────────────────────────┐
│                        JENKINSFILE                              │
│                    (ผู้ควบคุมทั้งหมด)                            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌───────────┐   ┌───────────┐   ┌─────────────┐
    │  tests/   │   │ functions/│   │etl_pipeline │
    │           │   │           │   │    .py      │
    │ ทดสอบว่า  │   │ ฟังก์ชัน   │   │             │
    │ functions │──▶│ ประมวลผล  │◀──│ import และ  │
    │ ถูกต้อง   │   │ ข้อมูล    │   │ เรียกใช้    │
    └───────────┘   └───────────┘   └─────────────┘
                           │               │
                           └───────┬───────┘
                                   ▼
                         ┌─────────────────┐
                         │  data/*.csv     │
                         │ (ข้อมูลต้นทาง)   │
                         └─────────────────┘
```

---

## 3. Functions - ฟังก์ชันหลัก

### 3.1 📋 guess_column_types.py

**หน้าที่:** วิเคราะห์ไฟล์ CSV และเดาประเภทข้อมูลของแต่ละคอลัมน์

```python
# การใช้งาน
from functions import guess_column_types

success, result = guess_column_types("data/LoanStats_web_small.csv")

# ผลลัพธ์ที่ได้
# {
#     'id': 'datetime64',
#     'loan_amnt': 'integer',
#     'funded_amnt': 'integer',
#     'funded_amnt_inv': 'floating',
#     'term': 'string',
#     'int_rate': 'string',
#     ...
# }
```

**ประเภทข้อมูลที่ตรวจจับได้:**

| ประเภท | ตัวอย่าง |
|--------|---------|
| `integer` | 1, 2, 100, -50 |
| `floating` | 1.5, 3.14, -2.7 |
| `string` | "hello", "abc" |
| `boolean` | True, False |
| `date` | 2023-01-15 |
| `datetime64` | 2023-01-15 14:30:45 |

**รองรับ Delimiter หลายแบบ:**
- Comma (,)
- Semicolon (;)
- Tab (\t)
- Pipe (|)

---

### 3.2 📅 filter_issue_date_range.py

**หน้าที่:** กรองข้อมูลให้เหลือเฉพาะปี 2016-2019

```python
# การใช้งาน
from functions import filter_issue_date_range

df_filtered = filter_issue_date_range(df)

# ก่อนกรอง: 14,422 rows (มีข้อมูลปี 2015, 2020 ปนอยู่)
# หลังกรอง: 9,424 rows (เฉพาะ 2016-2019)
```

**กฎการกรอง:**

```
┌─────────────────────────────────────────────────────────┐
│                    Timeline                             │
│                                                         │
│  2015        2016        2017        2018        2019        2020
│    │           │           │           │           │           │
│    ├───────────┼───────────┼───────────┼───────────┼───────────┤
│    │    ❌     │     ✅    │     ✅    │     ✅    │     ✅    │    ❌
│    │  ลบออก   │    เก็บ   │    เก็บ   │    เก็บ   │    เก็บ   │  ลบออก
│                                                         │
│  Boundary:                                              │
│  • 2015-12-31 → ❌ ลบ                                   │
│  • 2016-01-01 → ✅ เก็บ                                 │
│  • 2019-12-31 → ✅ เก็บ                                 │
│  • 2020-01-01 → ❌ ลบ                                   │
└─────────────────────────────────────────────────────────┘
```

**รองรับรูปแบบวันที่:**
- `2023-01-15` (ISO format)
- `Jan-2023` (Month-Year)
- `15/01/2023` (DD/MM/YYYY)

---

### 3.3 🧹 clean_missing_values.py

**หน้าที่:** ลบคอลัมน์ที่มีค่า null มากเกินไป (default: >30%)

```python
# การใช้งาน
from functions import clean_missing_values

df_cleaned = clean_missing_values(df, threshold=0.3)

# ก่อน: 144 คอลัมน์
# หลัง: 100 คอลัมน์ (ลบ 44 คอลัมน์ที่มี null > 30%)
```

**ตัวอย่างการทำงาน:**

```
┌────────────────┬──────────────┬─────────────┐
│    Column      │  % Null      │   Action    │
├────────────────┼──────────────┼─────────────┤
│ good_col       │    0%        │   ✅ เก็บ   │
│ ok_col         │   15%        │   ✅ เก็บ   │
│ bad_col        │   40%        │   ❌ ลบ    │
│ very_bad_col   │   90%        │   ❌ ลบ    │
└────────────────┴──────────────┴─────────────┘

Threshold = 30%
• ≤ 30% null → เก็บไว้
• > 30% null → ลบทิ้ง
```

**สามารถปรับ Threshold ได้:**
```python
clean_missing_values(df, threshold=0.1)  # ลบถ้า null > 10%
clean_missing_values(df, threshold=0.5)  # ลบถ้า null > 50%
```

---

## 4. Tests - การทดสอบ

### 🧪 ภาพรวมการทดสอบ

แต่ละฟังก์ชันมี Test File ที่สอดคล้องกัน:

| Function File | Test File | Test Cases |
|--------------|-----------|------------|
| `guess_column_types.py` | `guess_column_types_test.py` | 4 |
| `filter_issue_date_range.py` | `filter_issue_date_range_test.py` | 4 |
| `clean_missing_values.py` | `clean_missing_values_test.py` | 4 |

### 4.1 guess_column_types_test.py

```
┌─────────────────────────────────────────────────────────────┐
│  Test Case 1: การตรวจจับประเภทข้อมูลพื้นฐาน                  │
│  ─────────────────────────────────────────────────────────  │
│  • ทดสอบ: integer, float, string, boolean, mixed            │
│  • Expected: ตรวจจับได้ถูกต้องทั้ง 5 ประเภท                  │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 2: ทดสอบการตรวจจับรูปแบบวันที่และเวลา             │
│  ─────────────────────────────────────────────────────────  │
│  • ทดสอบ: date (2023-01-15), datetime (2023-01-15 14:30:45) │
│  • Expected: date_col='date', datetime_col='datetime64'     │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 3: ทดสอบตัวแบ่งที่แตกต่างกัน                      │
│  ─────────────────────────────────────────────────────────  │
│  • ทดสอบ: comma, semicolon, tab, pipe                       │
│  • Expected: parse ได้ทั้ง 4 แบบ                            │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 4: Edge Cases                                    │
│  ─────────────────────────────────────────────────────────  │
│  • 4a: ไฟล์ไม่มีอยู่ → return (False, error_message)        │
│  • 4b: ไฟล์ว่าง → handle gracefully                         │
│  • Result: ✅ PASS                                          │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 filter_issue_date_range_test.py

```
┌─────────────────────────────────────────────────────────────┐
│  Test Case 1: การกรองพื้นฐาน                                │
│  ─────────────────────────────────────────────────────────  │
│  • Input: 24 records (ปี 2015-2020)                         │
│  • Expected: 16 records (เฉพาะ 2016-2019)                   │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 2: ทดสอบขอบเขต (Boundary Testing)               │
│  ─────────────────────────────────────────────────────────  │
│  • Input: 2015-12-31, 2016-01-01, 2019-12-31, 2020-01-01   │
│  • Expected: เก็บเฉพาะ 2016-01-01 และ 2019-12-31           │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 3: ทดสอบรูปแบบข้อมูล String                      │
│  ─────────────────────────────────────────────────────────  │
│  • Input: "Dec-2015", "Jan-2016", "Jun-2017", "Jan-2020"   │
│  • Expected: เก็บ Jan-2016, Jun-2017, Dec-2019             │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 4: Edge Cases                                    │
│  ─────────────────────────────────────────────────────────  │
│  • 4a: ไม่มีข้อมูลในช่วง 2016-2019 → return 0 records      │
│  • 4b: ไม่มีคอลัมน์ issue_d → return original DataFrame    │
│  • Result: ✅ PASS                                          │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 clean_missing_values_test.py

```
┌─────────────────────────────────────────────────────────────┐
│  Test Case 1: การทำความสะอาดพื้นฐาน                         │
│  ─────────────────────────────────────────────────────────  │
│  • Input: 4 columns (0%, 15%, 40%, 90% null)               │
│  • Threshold: 30%                                           │
│  • Expected: เก็บ 2 columns (0% และ 15%)                   │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 2: ทดสอบ threshold ที่แตกต่างกัน                 │
│  ─────────────────────────────────────────────────────────  │
│  • Threshold 10%: เก็บ 2 columns                           │
│  • Threshold 25%: เก็บ 3 columns                           │
│  • Threshold 35%: เก็บ 4 columns                           │
│  • Threshold 50%: เก็บ 6 columns                           │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 3: ทดสอบการรักษาประเภทข้อมูล                     │
│  ─────────────────────────────────────────────────────────  │
│  • int64 → int64 ✅                                         │
│  • float64 → float64 ✅                                     │
│  • object → object ✅                                       │
│  • Result: ✅ PASS                                          │
├─────────────────────────────────────────────────────────────┤
│  Test Case 4: Edge Cases                                    │
│  ─────────────────────────────────────────────────────────  │
│  • 4a: DataFrame ว่าง → return empty DataFrame             │
│  • 4b: ทุกคอลัมน์มี null เกิน threshold → return 0 cols   │
│  • 4c: ไม่มี null เลย → return original DataFrame          │
│  • Result: ✅ PASS                                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. ETL Pipeline - โปรแกรมหลัก

### 📄 etl_pipeline.py

**หน้าที่:** รวมฟังก์ชันทั้งหมดเข้าด้วยกัน ประมวลผลข้อมูล และสร้าง Star Schema

### 5.1 ขั้นตอนการทำงาน

```python
# Step 1: Import functions
from functions import guess_column_types, filter_issue_date_range, clean_missing_values

# Step 2: วิเคราะห์ประเภทข้อมูล
success, column_types = guess_column_types("data/LoanStats_web_small.csv")
# → วิเคราะห์ 144 คอลัมน์

# Step 3: โหลดข้อมูล
df = pd.read_csv("data/LoanStats_web_small.csv", low_memory=False)
# → 14,422 rows, 144 columns

# Step 4: ทำความสะอาดข้อมูล
df = clean_missing_values(df, threshold=0.3)
# → ลบ 44 คอลัมน์ เหลือ 100 คอลัมน์

# Step 5: กรองตามช่วงวันที่
df = filter_issue_date_range(df)
# → เก็บเฉพาะปี 2016-2019

# Step 6: สร้าง Star Schema
# → Dimension Tables + Fact Table

# Step 7: Deploy to Database (ถ้ามี --deploy flag)
# → ส่งข้อมูลไป SQL Server
```

### 5.2 Star Schema ที่สร้างขึ้น

```
                    ┌─────────────────────┐
                    │   issue_d_dim       │
                    │   _yourname         │
                    ├─────────────────────┤
                    │ • issue_d_id (PK)   │
                    │ • issue_d           │
                    │ • month             │
                    │ • year              │
                    │ • quarter           │
                    │ (30 records)        │
                    └──────────┬──────────┘
                               │
┌─────────────────────┐        │        ┌─────────────────────┐
│  home_ownership     │        │        │   loan_status       │
│  _dim_yourname      │        │        │   _dim_yourname     │
├─────────────────────┤        │        ├─────────────────────┤
│ • home_ownership_id │        │        │ • loan_status_id    │
│   (PK)              │        │        │   (PK)              │
│ • home_ownership    │        │        │ • loan_status       │
│ (4 records)         │        │        │ (6 records)         │
│ - OWN               │        │        │ - Fully Paid        │
│ - RENT              │        │        │ - Current           │
│ - MORTGAGE          │        │        │ - Charged Off       │
│ - OTHER             │        │        │ - Late              │
└──────────┬──────────┘        │        └──────────┬──────────┘
           │                   │                   │
           │    ┌──────────────┴──────────────┐    │
           │    │                             │    │
           └────┤      loans_fact_yourname    ├────┘
                │                             │
                ├─────────────────────────────┤
                │ • fact_id (PK)              │
                │ • loan_amnt                 │
                │ • funded_amnt               │
                │ • term                      │
                │ • int_rate                  │
                │ • installment               │
                │ • home_ownership_id (FK)    │
                │ • loan_status_id (FK)       │
                │ • issue_d_id (FK)           │
                │ (9,424 records)             │
                └─────────────────────────────┘
```

### 5.3 วิธีรันโปรแกรม

```bash
# รันแบบทดสอบ (ไม่ส่งเข้า Database)
python etl_pipeline.py

# รันแบบ Deploy เข้า Database
python etl_pipeline.py --deploy
```

### 5.4 ผลลัพธ์ที่ได้

```
📊 Loan Amount Statistics:
   - Min: $1,000.00
   - Max: $40,000.00
   - Average: $15,506.00
   - Total: $146,128,525.00

📈 Fact Table Sample:
   fact_id  loan_amnt  funded_amnt  int_rate
   1        10000      10000        5.32%
   2        25000      25000        11.99%
   3        5500       5500         11.99%
   4        20000      20000        8.39%
   5        19800      19800        14.46%
```

---

## 6. Jenkinsfile - ระบบ CI/CD

### 📄 โครงสร้าง Jenkinsfile

```groovy
pipeline {
    agent any                          // รันบน Jenkins agent ใดก็ได้
    
    environment {
        DB_PASSWORD = credentials('db-password')  // ดึง password จาก Jenkins
    }
    
    options {
        timeout(time: 20, unit: 'MINUTES')       // จำกัดเวลา 20 นาที
        timestamps()                              // แสดงเวลาใน log
    }
    
    stages {
        stage('🔄 Checkout & Setup') { ... }
        stage('🐍 Python Environment') { ... }
        stage('🧪 Unit Tests') { ... }           // รัน parallel
        stage('🔍 ETL Validation') { ... }
        stage('🔄 ETL Processing') { ... }
        stage('📤 Deploy to Database') { ... }
    }
    
    post {
        always { /* cleanup */ }
        success { /* แจ้งสำเร็จ */ }
        failure { /* แจ้งล้มเหลว */ }
    }
}
```

### 6.1 Stage Details

#### Stage 1: 🔄 Checkout & Setup
```groovy
stage('🔄 Checkout & Setup') {
    steps {
        script {
            echo "=== Simple ETL CI/CD Pipeline Started ==="
            echo "Build: ${BUILD_NUMBER}"
            echo "Branch: ${GIT_BRANCH}"
            echo "Workspace: ${WORKSPACE}"
        }
        script {
            // ตรวจสอบโครงสร้างโปรเจค
            if (fileExists('functions') && fileExists('tests')) {
                echo "✅ Project structure verified"
            }
        }
    }
}
```

#### Stage 2: 🐍 Python Environment
```groovy
stage('🐍 Python Environment') {
    steps {
        sh '''
            rm -rf venv
            python3 -m venv venv
            . venv/bin/activate
            python -m pip install --upgrade pip
            pip install pandas numpy sqlalchemy pymssql
            pip install pytest pytest-cov
            python -c "import pandas, numpy, sqlalchemy; print('✅ Core packages installed')"
        '''
    }
}
```

#### Stage 3: 🧪 Unit Tests (Parallel)
```groovy
stage('🧪 Unit Tests') {
    parallel {
        stage('Test: guess_column_types') {
            steps {
                sh '''
                    . venv/bin/activate
                    cd tests
                    python guess_column_types_test.py
                '''
            }
        }
        stage('Test: filter_issue_date_range') {
            steps {
                sh '''
                    . venv/bin/activate
                    cd tests
                    python filter_issue_date_range_test.py
                '''
            }
        }
        stage('Test: clean_missing_values') {
            steps {
                sh '''
                    . venv/bin/activate
                    cd tests
                    python clean_missing_values_test.py
                '''
            }
        }
    }
}
```

#### Stage 4: 🔍 ETL Validation
```groovy
stage('🔍 ETL Validation') {
    steps {
        sh '''
            . venv/bin/activate
            python -c "
from functions import guess_column_types, filter_issue_date_range, clean_missing_values
print('✅ All functions imported successfully')
"
            [ -f data/LoanStats_web_small.csv ] && echo "✅ Data file found"
        '''
    }
}
```

#### Stage 5: 🔄 ETL Processing
```groovy
stage('🔄 ETL Processing') {
    steps {
        sh '''
            . venv/bin/activate
            python etl_pipeline.py
        '''
    }
}
```

#### Stage 6: 📤 Deploy to Database
```groovy
stage('📤 Deploy to Database') {
    steps {
        sh '''
            . venv/bin/activate
            python etl_pipeline.py --deploy
        '''
    }
}
```

#### Post Actions
```groovy
post {
    always {
        sh '''
            rm -rf venv
            find . -name "*.pyc" -delete
            find . -name "__pycache__" -type d -exec rm -rf {} +
        '''
    }
    success {
        echo "✅ ETL Pipeline completed successfully!"
    }
    failure {
        echo "❌ ETL Pipeline failed!"
    }
}
```

### 6.2 Pipeline Flow Diagram

```
┌────────────────────────────────────────────────────────────────┐
│                    JENKINS PIPELINE                            │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│  Stage 1: 🔄 Checkout & Setup                                  │
│  • Git clone repository                                        │
│  • Verify project structure                                    │
│  • Display build info                                          │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│  Stage 2: 🐍 Python Environment                                │
│  • Create virtual environment                                  │
│  • Install dependencies (pandas, numpy, sqlalchemy, etc.)      │
│  • Verify installations                                        │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│  Stage 3: 🧪 Unit Tests (PARALLEL)                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐           │
│  │    Test 1    │ │    Test 2    │ │    Test 3    │           │
│  │ guess_column │ │ filter_issue │ │ clean_missing│           │
│  │   _types     │ │ _date_range  │ │   _values    │           │
│  │   4/4 ✅     │ │    4/4 ✅    │ │    4/4 ✅    │           │
│  └──────────────┘ └──────────────┘ └──────────────┘           │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│  Stage 4: 🔍 ETL Validation                                    │
│  • Import all functions                                        │
│  • Check data file exists                                      │
│  • Test file readability                                       │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│  Stage 5: 🔄 ETL Processing                                    │
│  • Run etl_pipeline.py                                         │
│  • Process data: 14,422 → 9,424 rows                          │
│  • Create Star Schema                                          │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│  Stage 6: 📤 Deploy to Database                                │
│  • Run etl_pipeline.py --deploy                                │
│  • Connect to SQL Server                                       │
│  • Create tables & insert data                                 │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│  Post Actions                                                  │
│  • Cleanup: remove venv, __pycache__                          │
│  • Report: success/failure notification                        │
└────────────────────────────────────────────────────────────────┘
```

---

## 7. การไหลของข้อมูล

### 📊 Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        INPUT                                    │
│  data/LoanStats_web_small.csv                                   │
│  • 14,422 rows                                                  │
│  • 144 columns                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1: guess_column_types()                                   │
│  ─────────────────────────────────────────────────────────────  │
│  • วิเคราะห์ประเภทข้อมูลแต่ละคอลัมน์                            │
│  • Output: dictionary ของ column types                         │
│  • ตัวอย่าง: loan_amnt=integer, term=string                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2: pd.read_csv()                                          │
│  ─────────────────────────────────────────────────────────────  │
│  • โหลดข้อมูลเข้า DataFrame                                     │
│  • 14,422 rows × 144 columns                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3: clean_missing_values(threshold=0.3)                    │
│  ─────────────────────────────────────────────────────────────  │
│  • ลบคอลัมน์ที่มี null > 30%                                    │
│  • ลบ 44 คอลัมน์                                                │
│  • Output: 14,422 rows × 100 columns                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4: filter_issue_date_range()                              │
│  ─────────────────────────────────────────────────────────────  │
│  • เก็บเฉพาะปี 2016-2019                                        │
│  • Output: 9,424 rows × 100 columns                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP 5: Create Star Schema                                     │
│  ─────────────────────────────────────────────────────────────  │
│  • home_ownership_dim (4 records)                              │
│  • loan_status_dim (6 records)                                 │
│  • issue_d_dim (30 records)                                    │
│  • loans_fact (9,424 records)                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        OUTPUT                                   │
│  SQL Server Database (104.199.169.50/TestDB)                   │
│  • 3 Dimension Tables                                          │
│  • 1 Fact Table                                                │
└─────────────────────────────────────────────────────────────────┘
```

### 📈 Data Transformation Summary

| Stage | Rows | Columns | Action |
|-------|------|---------|--------|
| Input | 14,422 | 144 | Load CSV |
| After clean_missing_values | 14,422 | 100 | Remove 44 columns |
| After filter_issue_date_range | 9,424 | 100 | Remove ~5,000 rows |
| Final Fact Table | 9,424 | 9 | Select key columns |

---

## 8. วิธีใช้งาน

### 8.1 ติดตั้ง Jenkins ด้วย Docker

```bash
# รัน Jenkins container
docker run -d \
  --name jenkins-python \
  --restart unless-stopped \
  -e TZ=Asia/Bangkok \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  amornpan/jenkins-python:latest

# ดู initial password
docker logs jenkins-python
```

### 8.2 สร้าง Jenkins Job

1. เปิด http://localhost:8080
2. กด **New Item** → **Pipeline**
3. ตั้งชื่อ: `dataops-foundation-jenkins-new`
4. เลือก **Pipeline script from SCM**
5. SCM: **Git**
6. Repository URL: `https://github.com/amornpan/dataops-foundation-jenkins-new.git`
7. Script Path: `Jenkinsfile`
8. Save

### 8.3 ตั้งค่า Credentials

1. ไปที่ **Manage Jenkins** → **Credentials**
2. เพิ่ม **Secret text** สำหรับ `db-password`
3. ใส่ password ของ SQL Server

### 8.4 รัน Pipeline

1. กด **Build Now**
2. ดูผลที่ **Console Output**
3. รอจนทุก stage เสร็จสิ้น

### 8.5 รันแบบ Manual (ไม่ผ่าน Jenkins)

```bash
# Clone repository
git clone https://github.com/amornpan/dataops-foundation-jenkins-new.git
cd dataops-foundation-jenkins-new

# สร้าง virtual environment
python3 -m venv venv
source venv/bin/activate  # Linux/Mac
# หรือ venv\Scripts\activate  # Windows

# ติดตั้ง dependencies
pip install pandas numpy sqlalchemy pymssql pytest

# รัน tests
cd tests
python guess_column_types_test.py
python filter_issue_date_range_test.py
python clean_missing_values_test.py
cd ..

# รัน ETL pipeline
python etl_pipeline.py           # ทดสอบเฉยๆ
python etl_pipeline.py --deploy  # deploy จริง
```

---

## 9. การแก้ไขปัญหา

### 🔴 Problem 1: Jenkins ไม่สามารถ clone repository

```
Error: Could not resolve host: github.com
```

**วิธีแก้:**
```bash
# ตรวจสอบ network ใน WSL
wsl -d Ubuntu-22.04 ping -c 3 github.com

# แก้ DNS
wsl -d Ubuntu-22.04
sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo chattr +i /etc/resolv.conf
exit

# Restart Docker
wsl --shutdown
# เปิด Docker Desktop ใหม่
```

### 🔴 Problem 2: Database Connection Failed

```
Error: Unable to connect: Adaptive Server is unavailable (104.199.169.50)
Connection refused (111)
```

**วิธีแก้:**

**Option A: แก้ Jenkinsfile ให้ไม่ fail**
```groovy
stage('📤 Deploy to Database') {
    steps {
        script {
            try {
                sh '''
                    . venv/bin/activate
                    python etl_pipeline.py --deploy
                '''
            } catch (Exception e) {
                echo "⚠️ Database deployment skipped: ${e.message}"
                // Pipeline ยังคง SUCCESS
            }
        }
    }
}
```

**Option B: ตรวจสอบ Database Server**
```bash
# ทดสอบ connection จาก Jenkins container
docker exec -it jenkins-python bash
apt-get update && apt-get install -y telnet
telnet 104.199.169.50 1433
```

**Option C: ตรวจสอบ Firewall Rules**
- เปิด port 1433 บน SQL Server
- ตรวจสอบว่า IP ของ Jenkins ได้รับอนุญาต

### 🔴 Problem 3: Unit Tests Failed

```
Error: ModuleNotFoundError: No module named 'functions'
```

**วิธีแก้:**
```bash
# ตรวจสอบ PYTHONPATH
export PYTHONPATH=$PYTHONPATH:$(pwd)

# หรือเพิ่มใน Jenkinsfile
sh '''
    . venv/bin/activate
    export PYTHONPATH=$PYTHONPATH:$(pwd)
    cd tests
    python guess_column_types_test.py
'''
```

### 🔴 Problem 4: Data File Not Found

```
Error: FileNotFoundError: data/LoanStats_web_small.csv
```

**วิธีแก้:**
```bash
# ตรวจสอบว่ามีไฟล์
ls -la data/

# ถ้าไม่มี ให้ download หรือสร้าง sample data
bash setup_data.sh
```

### 🔴 Problem 5: Docker Desktop WSL Integration Failed

```
Error: WSL integration with distro 'Ubuntu-22.04' unexpectedly stopped
```

**วิธีแก้:**
```bash
# 1. Shutdown WSL
wsl --shutdown

# 2. Update WSL
wsl --update

# 3. Restart computer

# 4. ถ้ายังไม่หาย reset Docker Desktop
# Docker Desktop → Settings → Troubleshoot → Reset to factory defaults
```

---

## 📚 อ้างอิง

- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Python pandas Documentation](https://pandas.pydata.org/docs/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [Docker Documentation](https://docs.docker.com/)

---

## 📝 Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2024 | Initial release |

---

*สร้างโดย: DataOps Foundation Team*
*อัปเดตล่าสุด: ธันวาคม 2024*
