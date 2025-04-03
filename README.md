# MLFlowProject Testing Guide 🚀

Welcome to the **MLFlowProject Testing Guide**. This document outlines the end-to-end process for testing our project—from code and data creation to versioning with Git and DVC. Follow along with the detailed steps, conditions, and examples.

---

## 📌 Overview

This guide covers:
- **Project Setup & Git Integration** 🛠️
- **Data Management with DVC** 📊
- **Testing Procedures & Conditions** 🧪
- **Troubleshooting & Verification** 🔍

---

## ⚙️ 1. Initial Project Setup

### **Clone the Repository**
```bash
git clone https://github.com/Waqas56jb/MLOps-DVC-DataVersion.git
cd MLOps-DVC-DataVersion
```

### **Set Up Your Environment**
- (Optional) Create and activate a virtual environment:
  ```bash
  python -m venv .venv
  # Windows:
  .venv\Scripts\activate
  # macOS/Linux:
  source .venv/bin/activate
  ```
- Install required packages:
  ```bash
  pip install -r requirements.txt
  pip install dvc
  ```

---

## 📝 2. Code and Data Creation

### **Create `mlflowproject.py`**

This script creates a CSV file in a new `data` folder. When run:
- If the CSV **does not exist**, it creates one with initial data.
- If the CSV **exists**, it appends a new row.

#### **mlflowproject.py**
```python
import os
import pandas as pd

def create_or_update_csv(csv_path: str) -> None:
    """
    Creates a CSV file if it doesn't exist.
    If it does exist, appends a new row.
    """
    new_row = {
        "Name": ["Waqas"],
        "Age": [28],
        "City": ["SampleCity"]
    }

    if not os.path.exists(csv_path):
        # Create initial CSV with sample data
        initial_data = {
            "Name": ["Alice", "Bob", "Charlie"],
            "Age": [25, 30, 35],
            "City": ["New York", "Los Angeles", "Chicago"]
        }
        df = pd.DataFrame(initial_data)
        df.to_csv(csv_path, index=False)
        print(f"Created new CSV at {csv_path}")
    else:
        # Append new row to existing CSV
        df_existing = pd.read_csv(csv_path)
        df_new_row = pd.DataFrame(new_row)
        df_appended = pd.concat([df_existing, df_new_row], ignore_index=True)
        df_appended.to_csv(csv_path, index=False)
        print(f"Appended new row to existing CSV at {csv_path}")

def main():
    # Ensure the 'data' folder exists
    data_folder = "data"
    os.makedirs(data_folder, exist_ok=True)
    csv_path = os.path.join(data_folder, "sample_data.csv")
    create_or_update_csv(csv_path)

if __name__ == "__main__":
    main()
```

---

## 🗃️ 3. Versioning with Git & DVC

### **Before DVC Initialization (Git-only)**
1. Stage and commit your code:
   ```bash
   git add .
   git commit -m "Initial commit: Added mlflowproject.py"
   git push origin main
   ```

### **Initialize DVC**
1. Initialize DVC:
   ```bash
   dvc init
   ```
2. Create a directory for remote storage simulation:
   ```bash
   mkdir S3
   ```
3. Add and set the DVC remote:
   ```bash
   dvc remote add -d myremote S3
   ```

### **Track Data with DVC**
1. **First Version of Data (V1):**
   - Remove Git tracking of the `data/` folder:
     ```bash
     git rm -r --cached data
     git commit -m "Stop tracking data with Git (DVC will handle it)"
     ```
   - Add data folder to DVC:
     ```bash
     dvc add data/
     ```
   - Stage the changes and commit:
     ```bash
     git add .gitignore data.dvc
     git commit -m "Track data folder with DVC (V1)"
     ```
   - Commit and push data to remote storage:
     ```bash
     dvc commit
     dvc push
     ```
   - Push Git changes:
     ```bash
     git push origin main
     ```

2. **Subsequent Versions (V2, V3, ...):**
   - Update `mlflowproject.py` to append new rows.
   - Run the script:
     ```bash
     python mlflowproject.py
     ```
   - Check changes:
     ```bash
     dvc status
     ```
   - Commit and push updated data:
     ```bash
     dvc commit
     dvc push
     ```
   - Stage and commit the code changes:
     ```bash
     git add .
     git commit -m "Update data to V2"
     git push origin main
     ```

---

## 🧪 4. Testing Procedures & Conditions

### **Testing Process Conditions**
- **Stage 1: Initial Data (V1)**
  - **Action:** Run the script; verify that the CSV is created.
  - **Expected Outcome:** CSV with 3 initial rows.
  - **Validation:** Check file content and run `dvc status`.

- **Stage 2: Updated Data (V2)**
  - **Action:** Run the script again; a new row should be appended.
  - **Expected Outcome:** CSV now contains an extra row.
  - **Validation:** Use `dvc diff` to see differences and verify via `dvc status`.

- **Stage 3: Further Updates (V3, etc.)**
  - **Action:** Modify and re-run the script.
  - **Expected Outcome:** Data evolves as expected.
  - **Validation:** Check `git log` and `dvc status` for a consistent history.

### **Example Unit Test for CSV Creation**
Below is an example using Python’s `unittest` framework to test CSV creation:
```python
import os
import unittest
import pandas as pd
from mlflowproject import create_or_update_csv

class TestCSVCreation(unittest.TestCase):
    def setUp(self):
        self.test_data_folder = 'test_data'
        os.makedirs(self.test_data_folder, exist_ok=True)
        self.csv_path = os.path.join(self.test_data_folder, 'sample_data.csv')
        if os.path.exists(self.csv_path):
            os.remove(self.csv_path)

    def tearDown(self):
        if os.path.exists(self.csv_path):
            os.remove(self.csv_path)
        os.rmdir(self.test_data_folder)

    def test_create_csv(self):
        create_or_update_csv(self.csv_path)
        self.assertTrue(os.path.exists(self.csv_path))
        df = pd.read_csv(self.csv_path)
        self.assertEqual(len(df), 3)  # Expecting 3 rows initially

if __name__ == '__main__':
    unittest.main()
```

---

## 🔍 5. Troubleshooting & Verification

- **Git Checks:**
  - Run `git status` to see uncommitted changes.
  - Use `git log --oneline` to review commit history.

- **DVC Checks:**
  - Run `dvc status` to see if data is up to date.
  - Use `dvc diff <old-commit> <new-commit>` to compare data versions.

- **Logs:**
  - Review console outputs for errors and verify that data operations complete as expected.

---

## 📚 Additional Resources
- [Git Documentation](https://git-scm.com/doc) 📖
- [DVC Documentation](https://dvc.org/doc) 📖
- [MLflow Documentation](https://mlflow.org/docs/latest/index.html) 📖

---

Enjoy building and testing your ML project, and happy coding! 🚀✨
```

---
 