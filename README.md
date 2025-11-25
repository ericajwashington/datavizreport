# Surgical Site Infections Among Colorectal Cancer Patients
# datavizreport
## Erica Washington, MPH, CPH, CIC, CPHQ, FAPIC


## Note that the data from this set are not publicly available due to the privacy agreement with Louisiana Tumor Registry.
## See Moodle upload for a copy of my signed data use agreement.
## This code is available for building graphs for your own use.

### The aim is to review associations between covariates in a retrospective cohort set that entials patients who underwent sugery to excise colorectal cancer. Overall survival after acquiring a surgical site infection is the primary goal of overall research; however, investigation of covariates is important to review because they will be revealing and noteworthy for  regression analysis and mindfulness of colinearity.

### The first portion of this code entails Python code. The second part of the code is SAS coding that was used during the initial presentation.

## Import Matplotlib and Pandas
### Import commands to create figures
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import requests
import io
from google.colab import drive
drive.mount('REDACTED DUE TO PRIVACY AGREEMENT')
print('DataFrame Info:')
df.info()
print('\nDescriptive Statistics:')
df.describe()

## Confirm that the data frame loaded correctly by loading the first few lines of code
csv_file_path = '/content/gdrive/MyDrive/LA CRC cases with definitive surgery (no fac-name, 2015-2022) 11-05-2025.csv'
df = pd.read_csv(csv_file_path, low_memory=False)
print('DataFrame loaded successfully.')
df.head()

## Create some graphics that show associations between the outcome and covariates.

## Relationship Between Poverty Status and Insurance
#Is there an association between poverty and insurance status?
import matplotlib.pyplot as plt
import seaborn as sns

## Ensure 'poverty' and 'insurance' are treated as categorical
df['poverty'] = df['poverty'].astype('category')

## Map numerical insurance categories to descriptive names
insurance_mapping = {1: 'Private', 2: 'Medicare/Other', 3: 'Medicaid', 9: 'Unknown'}
df['insurance'] = df['insurance'].astype('category').cat.rename_categories(insurance_mapping)

## Create a cross-tabulation of 'poverty' and 'insurance'
### Then normalize by 'poverty' to show proportions within each poverty group
pivot_df = df.groupby(['poverty', 'insurance'], observed=False).size().unstack(fill_value=0)

# Plotting the grouped bar chart
ax = pivot_df.plot(kind='bar', figsize=(12, 7), width=0.8)
plt.title('Distribution of Insurance Status by Poverty Level')
plt.xlabel('Poverty Level')
plt.ylabel('Count')
plt.xticks(rotation=45, ha='right')
plt.legend(title='Insurance Status')
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns

# Create a pivot table: rows are years, columns are stages, values are mean of readmission_30d
heatmap_data = df.pivot_table(values='readmission_30d', index='dx_yr', columns='AJCC_stage', aggfunc='mean')

# Plot the heatmap
plt.figure(figsize=(12, 8))
sns.heatmap(heatmap_data, annot=True, cmap='viridis', fmt=".2f", linewidths=.5)
plt.title('Average 30-Day Readmission Rate by Diagnosis Year and AJCC Stage')
plt.xlabel('AJCC Stage')
plt.ylabel('Diagnosis Year')
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns

# Create a new column with mapped summary stages
def map_summary_stage(stage):
    if stage == 1:
        return 'Localized'
    elif stage in [2, 3, 4, 5]:
        return 'Regional'
    elif stage == 7:
        return 'Distant'
    elif stage == 9:
        return 'Unstaged'
    elif stage == 0: # Assuming stage 0 is an unspecified category
        return 'Unspecified'
    else:
        return 'Other' # Catch any other unexpected stages

df['mapped_summary_stage'] = df['summary_stage'].apply(map_summary_stage)
df['mapped_summary_stage'] = df['mapped_summary_stage'].astype('category')

# Group by diagnosis year and mapped summary stage, then count the number of cases
cases_by_year_stage = df.groupby(['dx_yr', 'mapped_summary_stage'], observed=False).size().reset_index(name='count')

# Plot the line graph
plt.figure(figsize=(14, 7))
sns.lineplot(data=cases_by_year_stage, x='dx_yr', y='count', hue='mapped_summary_stage', marker='o')
plt.title('Number of Cases by Mapped Summary Stage Over Years')
plt.xlabel('Diagnosis Year')
plt.ylabel('Number of Cases')
plt.xticks(rotation=45)
plt.legend(title='Summary Stage Category', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.grid(True, linestyle='--', alpha=0.6)
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns

# Create a pivot table: rows are years, columns are Charlson Scores, values are mean of BMI
bmi_heatmap_data = df.pivot_table(values='BMI', index='dx_yr', columns='Charlson_score', aggfunc='mean')

# Remove columns for Charlson scores 7, 8 and 9
bmi_heatmap_data = bmi_heatmap_data.drop(columns=[7, 8, 9], errors='ignore')

# Reverse the order of years in the index for plotting
bmi_heatmap_data = bmi_heatmap_data.iloc[::-1]

# Plot the heatmap
plt.figure(figsize=(14, 8))
sns.heatmap(bmi_heatmap_data, annot=False, cmap='coolwarm', fmt=".1f", linewidths=.5)
plt.title('Average BMI by Diagnosis Year and Charlson Score (Excluding 7, 8 and 9)')
plt.xlabel('Charlson Score')
plt.ylabel('Diagnosis Year')
plt.tight_layout()
plt.show()

import numpy as np

# Define bin edges and labels
bin_edges = [0, 12, 24, 36, np.inf]
bin_labels = ['0-12 months', '13-24 months', '25-36 months', '>36 months']

# Create the 'binned_survival_months' column using pd.cut()
df['binned_survival_months'] = pd.cut(
    df['surv_mos_active_fup'],
    bins=bin_edges,
    labels=bin_labels,
    right=False # Ensure 0-12 months includes 0 but excludes 12
)

# Ensure the column is an ordered categorical type
df['binned_survival_months'] = pd.Categorical(
    df['binned_survival_months'],
    categories=bin_labels,
    ordered=True
)

print("Created 'binned_survival_months' column with ordered categorical bins. Displaying head and value counts:")
print(df[['surv_mos_active_fup', 'binned_survival_months']].head())
print("\nValue counts for 'binned_survival_months':")
print(df['binned_survival_months'].value_counts().sort_index())

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Group by Charlson_score and binned_survival_months and count occurrences
stacked_bar_data = df.groupby(['Charlson_score', 'binned_survival_months'], observed=False).size().unstack(fill_value=0)

print("Prepared data for stacked bar chart:")
print(stacked_bar_data.head())

import matplotlib.pyplot as plt
import seaborn as sns

# Filter out Charlson scores 7, 8, and 9
filtered_stacked_bar_data = stacked_bar_data.drop(index=[7, 8, 9], errors='ignore')

# Plotting the stacked bar chart
ax = filtered_stacked_bar_data.plot(kind='bar', stacked=True, figsize=(14, 8), cmap='viridis')

# Add total numbers above each bar
totals = filtered_stacked_bar_data.sum(axis=1) # Sum of all categories for each Charlson score

# Iterate over the list of Charlson scores (x-axis labels)
for i, charlson_score_label in enumerate(filtered_stacked_bar_data.index):
    # Get the total height for this Charlson score
    total_count = totals.loc[charlson_score_label]

    # Find the position to place the text. We can use the x-coordinate of the bottom patch
    # of the first container, as all patches for a given x-label share the same x-coordinate.
    x_pos = ax.containers[0].patches[i].get_x() + ax.containers[0].patches[i].get_width() / 2
    y_pos = total_count # Place the text at the total height

    ax.text(x_pos, y_pos, str(int(total_count)), ha='center', va='bottom', fontsize=9)

plt.title('Distribution of Binned Survival Months by Charlson Score (Excluding 7, 8, 9)')
plt.xlabel('Charlson Score')
plt.ylabel('Number of Patients')
plt.xticks(rotation=0) # Keep x-axis labels horizontal for readability
plt.legend(title='Survival Months', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns

# Create the scatter plot - Survival Months Active Follow-Up vs. Age at Diagnosis
plt.figure(figsize=(14, 8))
sns.scatterplot(
    data=df,
    x='surv_mos_active_fup', # Swapped x-axis
    y='age_at_diagnosis',    # Swapped y-axis
    s=50, # Set a fixed size for the points
    alpha=0.6 # Set transparency for better visibility of overlapping points
)

plt.title('Survival Months Active Follow-up vs. Age at Diagnosis')
plt.xlabel('Survival Months Active Follow-up') # Updated x-axis label
plt.ylabel('Age at Diagnosis')                 # Updated y-axis label
plt.grid(True, linestyle='--', alpha=0.6)
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns

# Calculate the mean readmission rate for each Charlson score
readmission_rate_by_charlson = df.groupby('Charlson_score')['readmission_30d'].mean().reset_index()

# Remove categories 7, 8, and 9
readmission_rate_by_charlson = readmission_rate_by_charlson[~readmission_rate_by_charlson['Charlson_score'].isin([7, 8, 9])]

# Plot the bar chart for 30-Day Readmission Rate by Charlson Score (Excluding 7, 8, and 9)
plt.figure(figsize=(12, 7))
sns.barplot(data=readmission_rate_by_charlson, x='Charlson_score', y='readmission_30d', hue='Charlson_score', palette='viridis', legend=False)
plt.title('30-Day Readmission Rate by Charlson Score (Excluding 7, 8, and 9)')
plt.xlabel('Charlson Score')
plt.ylabel('Average 30-Day Readmission Rate')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.xticks(rotation=0)
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt

# Count the occurrences of each race
race_counts = df['race'].value_counts()

# Plot the pie chart
plt.figure(figsize=(10, 8)) # Increased figure size to accommodate legend
plt.pie(race_counts, labels=None, autopct='%1.1f%%', startangle=140) # labels=None to prevent duplicate labels
plt.title('Distribution of Cases by Race')
plt.axis('equal') # Equal aspect ratio ensures that pie is drawn as a circle.
plt.legend(race_counts.index, title="Race", bbox_to_anchor=(1, 0.5), loc="center left") # Move legend to the right
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns

# Create a bar plot using sns.barplot to show Average 30-Day Readmission Rate by Charlson Score
plt.figure(figsize=(10, 6))
sns.barplot(
    data=readmission_rate_by_charlson_overall,
    x='Charlson_score',
    y='readmission_30d',
    palette='viridis'
)

# Add title and labels
plt.title('Average 30-Day Readmission Rate by Charlson Score')
plt.xlabel('Charlson Score')
plt.ylabel('Average 30-Day Readmission Rate')
plt.xticks(rotation=0)
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns

# Create a scatter plot for 30-Day Readmission Rate by Charlson Score (Scatter Plot)
plt.figure(figsize=(10, 6))
sns.scatterplot(
    data=readmission_rate_by_charlson_overall,
    x='Charlson_score',
    y='readmission_30d',
    s=200, # Increased point size for better visibility
    hue='Charlson_score', # Color points by Charlson score
    palette='viridis', # Use a color palette
    legend=False # No need for a separate legend if hue is x variable
)

# Add title and labels
plt.title('Average 30-Day Readmission Rate by Charlson Score (Scatter Plot)')
plt.xlabel('Charlson Score')
plt.ylabel('Average 30-Day Readmission Rate')
plt.xticks(rotation=0)
plt.grid(axis='both', linestyle='--', alpha=0.7) # Add grid for both x and y axis
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

# Map numerical 'chf' column to descriptive labels
chf_mapping = {0: 'No CHF', 1: 'CHF Present'}
df['chf_status'] = df['chf'].map(chf_mapping).astype('category')

# Ensure the order of categories for consistent plotting
df['chf_status'] = pd.Categorical(df['chf_status'], categories=['No CHF', 'CHF Present'], ordered=True)

# Create the box plot to show Distribution of Survival Months by CHF Status
plt.figure(figsize=(8, 6))
sns.boxplot(
    data=df,
    x='chf_status',
    y='surv_mos_active_fup',
    hue='chf_status', # Assign x to hue to resolve FutureWarning
    palette={'No CHF': 'lightgreen', 'CHF Present': 'salmon'},
    legend=False # Set legend to False as hue is x variable
)

plt.title('Distribution of Survival Months by CHF Status')
plt.xlabel('CHF Status')
plt.ylabel('Survival Months Active Follow-up')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

# Map numerical 'cvd' column to descriptive labels
cvd_mapping = {0: 'No CVD', 1: 'CVD Present'}
df['cvd_status'] = df['cvd'].map(cvd_mapping).astype('category')

# Ensure the order of categories for consistent plotting
df['cvd_status'] = pd.Categorical(df['cvd_status'], categories=['No CVD', 'CVD Present'], ordered=True)

# Create the box plot by Survival Months and CVD Status
plt.figure(figsize=(8, 6))
sns.boxplot(
    data=df,
    x='cvd_status',
    y='surv_mos_active_fup',
    hue='cvd_status', # Assign x to hue to resolve FutureWarning
    palette={'No CVD': 'lightgreen', 'CVD Present': 'darkred'},
    legend=False # Set legend to False as hue is x variable
)

plt.title('Distribution of Survival Months by CVD Status')
plt.xlabel('CVD Status')
plt.ylabel('Survival Months Active Follow-up')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

# Map numerical 'chf' column to descriptive labels
chf_mapping = {0: 'No CHF', 1: 'CHF Present'}
df['chf_status'] = df['chf'].map(chf_mapping).astype('category')

# Ensure the order of categories for consistent plotting
df['chf_status'] = pd.Categorical(df['chf_status'], categories=['No CHF', 'CHF Present'], ordered=True)

# Create the box plot for CHF and Survival Months
plt.figure(figsize=(8, 6))
sns.boxplot(data=df, x='chf_status', y='surv_mos_active_fup', palette={'No CHF': 'lightgreen', 'CHF Present': 'salmon'})

plt.title('Distribution of Survival Months by CHF Status')
plt.xlabel('CHF Status')
plt.ylabel('Survival Months Active Follow-up')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

# The original coding for this project was SAS. See below for the code.

*3. Create a new dataset based on the variables that need to be modified.;
data work.surgery1; set work.surgery;
if age_at_diagnosis <18 then delete;
if length_of_stay=0 then delete; 
/*create a hispanic variable based on NHIA categories*/
if nhia in (1,2,3,4,5,6,7,8) then hispanic=1;
else if nhia = 0 then hispanic=0;
/*create race categories basedon census categories*/
if race_1 = 1 then race_fin = 1; /*white*/
else if race_1 = 2 then race_fin = 2; /*black*/
else if race_1 = 3 then race_fin = 3; /*American Indian or Alaska Native*/
else if race_1 in (4,5,6,8,10,11,13,14,15,16,17,96) then race_fin = 4; /*Asian*/
else if race_1 in (7,20,21,27,97) then race_fin = 5; /*Native Hawaiian or Other Pacific Islander*/
else if race_1 = 98 then race_fin = 6; /*Other*/
else if race_1 = 99 then race_fin = 9; /*Unknown*/
run;
proc means data=work.surgery1;
var age_at_diagnosis length_of_stay; run;
proc contents data=work.surgery1; run;
proc contents data=work.linked; run;

*4. Run means statements for each of the continuous variables;
proc contents data=work.surgery1; run;
proc means data=work.surgery1;
var BMI LENGTH_OF_STAY age_at_diagnosis cer_height cer_weight readmit_day;
run;

*5. Run frequency statements for each categorical variable;
proc freq data=work.surgery1;
tables insurance marital_status_at_dx rx_summ_surg_rad_seq
hispanic race_fin Charlson_score behavior_icdo3 census_tr_poverty_indictr
ruca_2010 rx_summ_surg_oth_reg_dis rx_summ_reason_for_no_rad
rx_summ_systemic_surg_seq sequence_number summary_stage
uric_2010 BMI_cat rx_summ_scope_reg_ln_sur smoking_status sex ulcers urban_rural
vital_status_recode AJCC_stage acos_flag aids census_tr_poverty_indictr chf
cpd cvd dementia diabetes diabetes_comp diagnosis liver_disease paralysis poverty pvd 
summary_stage; 
run;

/*Stage 5 tumors aren't typically described. Who are the three people with 
Stage 5 tumors? How old are these patients?*/
proc print data=work.surgery1;
where summary_stage = 5; run; *They are ages 56, 89, and 94. 
It is no longer a valid code for cases diagnosed from January 1, 2018, and forward. 
Delete these three observations.;

*6. Delete the three observations with Stage 5 cancer.;
data work.surgery2; set work.surgery1;
if summary_stage = 5 then delete; run;

*7. Re-run your frequencies and means statements using work.surgery2;
proc means data=work.surgery2;
var BMI LENGTH_OF_STAY age_at_diagnosis cer_height cer_weight readmit_day; run;
proc freq data=work.surgery2;
tables insurance marital_status_at_dx rx_summ_surg_rad_seq
hispanic race_fin Charlson_score behavior_icdo3 census_tr_poverty_indictr
ruca_2010 rx_summ_surg_oth_reg_dis rx_summ_reason_for_no_rad
rx_summ_systemic_surg_seq sequence_number summary_stage
uric_2010 BMI_cat rx_summ_scope_reg_ln_sur smoking_status sex ulcers urban_rural
vital_status_recode AJCC_stage acos_flag aids census_tr_poverty_indictr chf
cpd cvd dementia diabetes diabetes_comp diagnosis liver_disease paralysis poverty pvd 
summary_stage; run;
