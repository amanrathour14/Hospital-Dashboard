
# Hospital Dashboard System

## Project Overview

This project provides a comprehensive hospital dashboard and data analysis solution. The **Web Application** is a React-based frontend with a Node.js/Express backend that reads patient data from Excel files and offers interactive visualizations (e.g. line, bar, and donut charts) using Chart.js. A complementary **Backend System** is implemented in Flask (Python) with a MySQL database. The Flask backend loads and normalizes the initial Excel datasets into tables (Patients, FollowUps, Deliveries) to reduce redundancy and exposes REST API endpoints for patient lookup, analytics, and data entry. Finally, a **Patient Data Analysis** module (Python) merges the three Excel files using Pandas and performs exploratory data analysis (EDA) on demographics, delivery outcomes, and medical conditions, generating insights (e.g. high C-section rate and common comorbidities) to aid decision-making. In summary, the stack includes React + Chart.js for the frontend, Node/Express for one API implementation, Flask + MySQL for another backend, and Pandas/NumPy/Seaborn for analysis.

## Folder Structure

The repository is organized into major components:

* `frontend/` – **React** application (dashboard UI, charts, patient forms).
* `node-backend/` – **Node.js/Express** server (REST API, reads Excel data).
* `flask-backend/` – **Flask** server (API, MySQL integration).
* `analysis/` – **Python** scripts/notebooks (EDA with Pandas, NumPy, Matplotlib).
* `data/` – Excel files (`basic_information.xlsx`, `followup_data.xlsx`, `delivery_information.xlsx`).
* `README.md` – this documentation.

Each folder contains its own code and configuration (e.g. `frontend/package.json`, `node-backend/server.js`, `flask-backend/app.py`, `analysis/*.py`).

## Prerequisites

* **Node.js & npm** (v16+ recommended) for the React and Express components.
* **Python 3.8+** with `pip` for the Flask app and data analysis (libraries: Flask, pandas, numpy, matplotlib, seaborn, flask\_mysqldb, etc.).
* **MySQL Server** (v5.7+/8.0+) for the Flask backend database.
* Install Chart.js integration for React: run `npm install chart.js react-chartjs-2` in the frontend.
* (Optional) Git for cloning the repo.

## Setup Instructions

1. **Clone the repository:**

   ```bash
   git clone https://github.com/yourusername/hospital-dashboard.git
   cd hospital-dashboard
   ```

2. **React Frontend:**

   * Navigate to `frontend/`, install dependencies, and start the app:

     ```bash
     cd frontend
     npm install
     npm start
     ```
   * The React app will run (by default) at `http://localhost:3000`. It fetches data from the backend APIs and renders charts (using Chart.js) and patient profile pages. The example charts below show typical dashboard visuals (time-series line charts, bar charts, and pie/donut charts) as implemented in the app.

   *Example dashboard charts (React + Chart.js): these could display metrics like blood pressure over time, categorical breakdowns, etc.*

3. **Node.js/Express Backend:**

   * Navigate to `node-backend/`, install dependencies:

     ```bash
     cd ../node-backend
     npm install
     ```
   * Ensure the Excel data files (`.xlsx`) are placed in a data folder or the same directory as the server script. The backend uses a library like [`xlsx`](https://www.npmjs.com/package/xlsx) to read these files. For instance:

     ```js
     const reader = require('xlsx');
     const wb = reader.readFile('data/basic_information.xlsx');
     const basicInfo = reader.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]]);
     ```

     This approach (using `xlsx.readFile`) converts the Excel sheets to JSON objects.
   * Start the Express server:

     ```bash
     node server.js
     ```
   * The server listens (e.g. on port 4000) and provides REST endpoints (e.g. `GET /api/patients`, `GET /api/patients/:id`, `GET /api/deliveries`, etc.) to support the React app. Node/Express is well-suited for fast, stateless APIs.

4. **Flask Backend (MySQL):**

   * Create a MySQL database (e.g. `hospital_db`) and user for the Flask app.
   * Navigate to `flask-backend/`, create a virtual environment and install requirements:

     ```bash
     cd ../flask-backend
     pip install -r requirements.txt
     ```
   * Update the Flask config (e.g. `app.config`) with your MySQL credentials. For example, using **Flask-MySQLdb** requires:

     ```python
     app.config['MYSQL_HOST'] = 'localhost'
     app.config['MYSQL_USER'] = 'root'
     app.config['MYSQL_PASSWORD'] = 'password'
     app.config['MYSQL_DB'] = 'hospital_db'
     ```

     Then initialize:

     ```python
     from flaskext.mysql import MySQL
     mysql = MySQL()
     mysql.init_app(app)
     cursor = mysql.get_db().cursor()  # to execute queries
     ```

     This setup (via `mysql.get_db().cursor()`) enables executing SQL directly from Flask.
   * **Database schema:** Populate the database with normalized tables. For example, create a `patients` table (with `patient_id` as PK and columns for demographics, condition, etc.), a `followups` table (`patient_id` FK + visit dates/BP/weight), and a `deliveries` table (`patient_id` FK + delivery date/outcome/weights). This normalization eliminates redundancy and enforces data integrity.
   * **Load initial data:** Use Python (pandas or xlrd) to read the Excel files and insert rows into MySQL. For example, one can use `pandas.read_excel()` and loop over records, executing `INSERT` queries. (GeeksforGeeks shows how a Flask route might use `pandas.read_excel(file)` to parse uploaded Excel data.) Alternatively, write a one-off Python script with `xlrd`/`mysqlclient` that iterates through sheets and does `cursor.execute(query, values)` for each row.
   * Start the Flask API server:

     ```bash
     flask run
     ```
   * The Flask app will serve endpoints (e.g. `GET /api/patient/<id>`, `POST /api/followup`, `GET /api/analytics/delivery-rate`, etc.) for patient lookup and analytics. It is integrated with the React frontend for form submissions.

5. **Python Data Analysis:**

   * In the `analysis/` folder, ensure you have `pandas`, `numpy`, `matplotlib`, `seaborn` installed.
   * Write/run analysis scripts or Jupyter notebooks that load and merge the three Excel datasets:

     ```python
     import pandas as pd
     basic = pd.read_excel('../data/basic_information.xlsx')
     delivery = pd.read_excel('../data/delivery_information.xlsx')
     followup = pd.read_excel('../data/followup_data.xlsx')
     merged = basic.merge(delivery, on='patient_id').merge(followup, on='patient_id')
     ```

     Pandas is ideal for loading and manipulating such tabular data.
   * Perform EDA: compute statistics, correlations, and create visualizations (histograms, scatter plots, trend lines). Libraries like Matplotlib/Seaborn provide rich plotting for insights (e.g. weight vs. delivery outcome, distribution of ages, etc.). EDA is “an essential step” to uncover patterns and anomalies in healthcare data.
   * Document key findings in the notebook or export them (e.g. as charts or CSV summaries). For example: the analysis might reveal that \~60% of deliveries were C-sections, a prevalence of PCOD and diabetes among patients, and typical prenatal weight gain patterns. (By WHO guidelines, C-section rates above \~10% offer no additional mortality benefit, so such a high local rate may warrant review.)

## Example API Endpoints

Below are some sample endpoints provided by the two backend systems (React/Node vs. Flask). The exact routes may vary by implementation:

* **Node/Express API:**

  * `GET /api/patients` – return list of all patients.
  * `GET /api/patients/:id` – return data for a specific patient (basic info, follow-ups, deliveries).
  * `GET /api/patients/:id/followups` – return follow-up visits for a patient.
  * `GET /api/deliveries` – summary stats (e.g., overall C-section rate).

* **Flask API (MySQL):**

  * `GET /api/patient/<patient_id>` – look up patient by ID.
  * `GET /api/analytics/delivery_rate` – compute delivery trends (e.g. monthly C-section rate).
  * `POST /api/patient` – add a new patient record.
  * `POST /api/followup` – submit a new follow-up visit record.

These RESTful endpoints follow standard HTTP verbs (GET, POST) for resource access, as recommended for API design.

## SQL Schema Overview

The MySQL database is normalized into separate tables to improve integrity. For example:

* `patients(patient_id, ht, wt, education, income, marriage_age, first_preg_age, district, occupation, diet, condition, ...)` – one row per patient.
* `deliveries(delivery_id, patient_id, date_of_delivery, age_at_delivery, weight_at_delivery, haemoglobin, placental_weight, term, delivery_type)` – linked to `patients` by `patient_id`.
* `followups(followup_id, patient_id, visit_no, date, bp_systolic, bp_diastolic, weight)` – multiple visits per patient.

Foreign keys (`patient_id`) link these tables. Normalized schema like this “minimizes redundancy, enhances data integrity, and facilitates easier maintenance”.

## Key EDA Insights

The merged dataset of 96 patients allows various analyses. For example:

* **Demographics:** Patients had an average of \~10.7 years of education and mean age at first pregnancy \~24.5 years. Nearly all patients come from three rural districts (Raigad, Ratnagiri, Sindhudurg), reflecting the service region.
* **Medical Conditions:** Common conditions include PCOD (17 cases) and diabetes (16 cases). About 17 patients had no notable condition. These suggest focusing on metabolic and reproductive health programs.
* **Delivery Outcomes:** 76% of deliveries were full-term; 58% were C-sections (LSCS). This C-section rate (\~56/96) far exceeds the WHO’s suggested range (rates above \~10% confer no additional mortality benefit), indicating a need to review delivery practices or patient risk factors.
* **Follow-Up Trends:** Most patients gained weight between prenatal visits, and systolic blood pressure tended to rise as pregnancy progressed. A few data outliers (e.g. an anomalous weight drop) highlight the importance of data cleaning.
* **Correlations:** Age at marriage and first pregnancy are strongly correlated (r≈0.85), meaning most women conceive soon after marriage. (This pattern is typical in similar populations.) No strong correlation was found between income and maternal weight.

These findings (derived via Pandas/Numpy analysis and visualized with Matplotlib/Seaborn) can inform hospital decisions, such as allocating resources for high-risk pregnancies, tailoring nutritional programs, or investigating the high C-section rate.

## Credits and References

* **React & Chart.js:** Frontend uses React (Facebook) and Chart.js. See *“Creating a dashboard with React and Chart.js”* for an overview.
* **Node.js/Express:** Backend follows REST principles and uses the `xlsx` npm package for Excel I/O.
* **Flask & MySQL:** Backend uses Flask (Python) with `flask-mysqldb` for database access. The schema follows normalization best practices.
* **Data Analysis:** Python EDA is implemented with Pandas, NumPy, Matplotlib, and Seaborn. EDA is “an essential step” to discover patterns.
* **Health Guidelines:** The WHO FAQ on C-section rates is cited regarding interpretation of delivery statistics.

