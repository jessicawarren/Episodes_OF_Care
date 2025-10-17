import sqlite3
import pandas as pd
import matplotlib.pyplot as plt

# Connect to your local SQL database
conn = sqlite3.connect('your_database.db')  # Replace with your database path or connection string

# Define SQL queries to fetch data from relevant tables
# Adjust table and column names accordingly

# Fetch provider info
providers_query = "SELECT * FROM provider_table"
providers_df = pd.read_sql(providers_query, conn)

# Fetch patient headers (demographics)
headers_query = "SELECT * FROM header_table"
headers_df = pd.read_sql(headers_query, conn)

# Fetch patient lines/diagnoses
lines_query = "SELECT * FROM line_table"
lines_df = pd.read_sql(lines_query, conn)

# Close the connection if needed
conn.close()

# Merge dataframes as necessary
# For example, merge headers with lines based on patient ID
merged_df = pd.merge(headers_df, lines_df, on='patient_id')

# Filter for diabetes diagnoses
# Assume 'diagnosis_code' is the column with diagnosis codes
# and 'DRG' is the diagnosis-related group
diabetes_codes = ['<list_of_diabetes_codes>']  # Populate with actual codes
filtered_df = merged_df[merged_df['diagnosis_code'].isin(diabetes_codes)]

# Filter for patients followed for 2 years after diagnosis
# Assume 'diagnosis_date' and 'followup_date' columns
filtered_df['diagnosis_date'] = pd.to_datetime(filtered_df['diagnosis_date'])
filtered_df['followup_date'] = pd.to_datetime(filtered_df['followup_date'])
filtered_df['followup_duration'] = (filtered_df['followup_date'] - filtered_df['diagnosis_date']).dt.days

two_year_followup_df = filtered_df[filtered_df['followup_duration'] >= 730]

# Count diagnosis codes per patient
diag_counts = two_year_followup_df.groupby('patient_id')['diagnosis_code'].count().reset_index()
diag_counts.rename(columns={'diagnosis_code': 'diagnosis_count'}, inplace=True)

# Calculate number of trips to doctors or ER
# Assume 'visit_type' column indicates 'Doctor' or 'ER'
visits_df = two_year_followup_df[two_year_followup_df['visit_type'].isin(['Doctor', 'ER'])]

trip_counts = visits_df.groupby('patient_id').size().reset_index(name='trip_count')

# Merge trip counts with patient age
# Assume 'age' column exists in headers_df
patient_ages = headers_df[['patient_id', 'age']]
analysis_df = pd.merge(diag_counts, trip_counts, on='patient_id', how='left')
analysis_df = pd.merge(analysis_df, patient_ages, on='patient_id')

# Divide into age ranges
bins = [0, 18, 35, 50, 65, 80, 100]
labels = ['0-17', '18-34', '35-49', '50-64', '65-79', '80+']
analysis_df['age_range'] = pd.cut(analysis_df['age'], bins=bins, labels=labels, right=False)

# Aggregate data by age ranges
age_group_stats = analysis_df.groupby('age_range').agg({
    'diagnosis_count': 'mean',
    'trip_count': 'mean'
}).reset_index()

# Plotting
fig, ax = plt.subplots(figsize=(10,6))
ax.bar(age_group_stats['age_range'], age_group_stats['diagnosis_count'], label='Avg Diagnoses')
ax.set_xlabel('Age Range')
ax.set_ylabel('Average Diagnoses per Patient')
ax2 = ax.twinx()
ax2.plot(age_group_stats['age_range'], age_group_stats['trip_count'], color='orange', label='Avg Trips to Dr/ER')
ax2.set_ylabel('Average Trips per Patient')

plt.title('Diabetes Patients Follow-up and Visits by Age Range')
ax.legend(loc='upper left')
ax2.legend(loc='upper right')
plt.show()
