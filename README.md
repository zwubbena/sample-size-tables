# Sample Size Analysis for Special Education IEP Compliance Auditing

## Introduction

This Python script generates and compares sample size tables for auditing compliance of special education Individualized Education Programs (IEPs). The program helps identify and adopt appropriate sample size tables for selecting IEP folders for review based on adjustable confidence levels and margins of error, ensuring statistically valid audits while optimizing resource allocation.

> [!NOTE]
> The script generates:
> - Multi-scenario comparison charts showing sample sizes under different parameters
> - Sample size tables for populations ranging from 1 to 20,000 students
> - A summary comparison of all scenarios with key statistics
> - An Excel workbook with scenario-specific worksheets and embedded visualizations
> - Automatic file download for immediate use

## Data Source

The script generates its own data based on:
- **Population Range**: 1 to 20,000 (covers individual students to large district populations)
- **Multiple Scenarios**: Three default scenarios with different margins of error
- **Statistical Formula**: Finite population correction formula for accurate sampling
- **No External Data Required**: All calculations use established statistical theory

## Goal

To identify and adopt a sample size table for auditing compliance of special education Individualized Education Programs (IEPs) that balances statistical validity with practical resource constraints for school districts with different size special education populations.

## Business Rules

- **BR-1.** IEP Audit Requirements:
    - **BR-1.1.** Sample sizes must be statistically valid for compliance determinations
    - **BR-1.2.** Audits must be feasible within available resources
    - **BR-1.3.** Results must be defensible for federal and state reporting

- **BR-2.** Statistical Parameters:
    - **BR-2.1.** Confidence level fixed at 80% (Z=1.28) for all scenarios
    - **BR-2.2.** Margin of error varies by scenario (13%, 15%, 16%)
    - **BR-2.3.** Population proportion set at 0.50 for conservative estimates

- **BR-3.** Sample Size Constraints:
    - **BR-3.1.** Sample size cannot exceed population size
    - **BR-3.2.** All sample sizes rounded up to ensure adequate coverage
    - **BR-3.3.** Minimum sample size is 1 (for populations of 1)

- **BR-4.** Scenario Comparison:
    - **BR-4.1.** Multiple scenarios analyzed simultaneously
    - **BR-4.2.** Each scenario visually distinguished by line style and color
    - **BR-4.3.** Legend displays actual calculated parameters

- **BR-5.** Output Requirements:
    - **BR-5.1.** Excel workbook must contain all scenarios for comparison
    - **BR-5.2.** Each scenario gets dedicated worksheet with grouped ranges
    - **BR-5.3**. Visualizations embedded for immediate reference
    - **BR-5.4.** Files automatically downloaded upon completion

## Calculations

### Sample Size Formula

The script uses the finite population correction formula:

```
Step 1: Calculate initial sample size (infinite population)
n₀ = (Z² × p × (1-p)) / e²

Where:
- Z = Z-score for desired confidence level (1.28 for 80%)
- p = Expected population proportion (0.50)
- e = Margin of error (0.13, 0.15, or 0.16)

Step 2: Apply finite population correction
n = n₀ / (1 + ((n₀ - 1) / N))

Where:
- N = Population size (number of IEPs)
- n = Adjusted sample size

Step 3: Ensure practical constraints
n_final = min(ceiling(n), N)
```

### Common Z-Score Reference
- Z = 1.28 → 80% confidence
- Z = 1.645 → 90% confidence
- Z = 1.96 → 95% confidence
- Z = 2.576 → 99% confidence

## Step-by-Step Process

### Step 1: Import Required Libraries and Install Dependencies

**Business Rules**:
1.1. Must use Google Colab compatible libraries
1.2. Installation must be silent to avoid cluttering output
1.3. All libraries must be available in the Colab environment

**Python Code**:
```python
import os
import math
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats
from matplotlib.lines import Line2D

# Silent installation of Excel writer
!pip install -q xlsxwriter
```

**Explanation**: The script uses standard data science libraries available in Google Colab. The `xlsxwriter` library is installed silently (`-q` flag) to keep the output clean. This library enables advanced Excel formatting and image embedding, essential for creating professional reports for IEP compliance auditing.

### Step 2: Define Scenario Parameters for IEP Auditing

**Business Rules**:
2.1. Default scenarios must represent practical audit approaches
2.2. Single scenario mode available for focused analysis
2.3. Display parameters optimized for typical IEP population sizes
2.4. Each scenario must have distinct visual properties

**Python Code**:
```python
# Scenario Control
RUN_SINGLE = False  # Set True to run only one scenario
SINGLE_SCENARIO_INDEX = 0  # Index of scenario to run if RUN_SINGLE = True

# Display Settings
POP_MIN = 1             # Minimum for individual IEP reviews
POP_MAX = 250           # Typical special education program size
SAMPLE_CAP = 30         # Maximum for practical auditing
X_TICK_INTERVAL = 10
Y_TICK_INTERVAL = 5
OUTPUT_DIR = "/content/"

# IEP Audit Scenarios
SCENARIOS = [
    {
        "Name": "Scenario A (80%, ±13%)",  # More precise
        "Z": 1.28,
        "Margin": 0.13,
        "Proportion": 0.50,
        "LineStyle": "-",
        "LineColor": "blue",
    },
    {
        "Name": "Scenario B (80%, ±15%)",  # Balanced
        "Z": 1.28,
        "Margin": 0.15,
        "Proportion": 0.50,
        "LineStyle": "--",
        "LineColor": "green",
    },
    {
        "Name": "Scenario C (80%, ±16%)",  # More efficient
        "Z": 1.28,
        "Margin": 0.16,
        "Proportion": 0.50,
        "LineStyle": ":",
        "LineColor": "red",
    },
]
```

**Explanation**: Three scenarios represent different audit approaches for IEP compliance. Scenario A (±13%) provides higher precision for critical audits, Scenario B (±15%) offers a balanced approach, and Scenario C (±16%) maximizes efficiency when resources are limited. The visual distinctions ensure clarity in printed reports.

### Step 3: Create Sample Size Calculation Function

**Business Rules**:
3.1. Function must handle all population sizes from 1 to 20,000
3.2. Results must be deterministic and reproducible
3.3. Calculation errors must be caught and reported

**Formula**:
```
n₀ = (Z² × p × (1-p)) / e²
n = n₀ / (1 + ((n₀ - 1) / N))
n_final = min(ceiling(n), N)
```

**Python Code**:
```python
def calculate_sample_size(N, e, p, z):
    """
    Calculate sample size for IEP compliance auditing.
    Uses finite population correction for accuracy.
    """
    if N < 1:
        raise ValueError("Population must be >= 1")

    # Initial calculation (infinite population)
    numerator = (z ** 2) * p * (1 - p)
    denominator = e ** 2
    n_0 = numerator / denominator
    
    # Apply finite population correction
    adjusted_n = n_0 / (1 + ((n_0 - 1) / N))
    
    # Ensure whole number, never exceeding population
    return min(math.ceil(adjusted_n), N)
```

**Explanation**: This function calculates the exact number of IEPs to audit from any given population size. The finite population correction is crucial for small special education programs, where the standard formula would overestimate required samples. The ceiling function ensures we always meet or exceed statistical requirements.

### Step 4: Generate Sample Size Tables for All Scenarios

**Business Rules**:
4.1. Must calculate sample sizes for populations 1 to 20,000
4.2. Each scenario processed independently for comparison
4.3. Summary statistics calculated for decision-making
4.4. Data structured for both analysis and visualization

**Python Code**:
```python
scenarios_to_run = [SCENARIOS[SINGLE_SCENARIO_INDEX]] if RUN_SINGLE else SCENARIOS
plot_df = pd.DataFrame()
scenario_summary = []

for scenario in scenarios_to_run:
    name = scenario["Name"]
    z = scenario["Z"]
    e = scenario["Margin"]
    p = scenario["Proportion"]
    
    print(f"▶ Processing {name}...")
    
    # Generate complete sample size table
    df = pd.DataFrame({
        "Population_Size": range(1, 20001)
    })
    df["Sample_Size"] = df["Population_Size"].apply(
        lambda N: calculate_sample_size(N, e, p, z)
    )
    df["Truncated_Sample_Size"] = df["Sample_Size"].apply(
        lambda x: min(x, SAMPLE_CAP)
    )
    
    # Calculate summary statistics
    confidence = round(2 * stats.norm.cdf(z) - 1, 4)
    max_sample = df["Sample_Size"].max()
    
    scenario_summary.append({
        "Scenario": name,
        "Z-Score": z,
        "Confidence_Level": f"{confidence*100:.1f}%",
        "Margin_of_Error": f"±{e*100:.1f}%",
        "Max_Sample_Size": max_sample
    })
    
    # Prepare visualization data
    temp = df[df["Population_Size"].between(POP_MIN, POP_MAX)].copy()
    temp["Scenario"] = name
    temp["LineStyle"] = scenario["LineStyle"]
    temp["LineColor"] = scenario["LineColor"]
    plot_df = pd.concat([plot_df, temp], ignore_index=True)
```

**Explanation**: This loop generates comprehensive sample size tables for IEP auditing under each scenario. The tables cover all possible population sizes from individual case reviews to large district-wide audits. Summary statistics help administrators choose the most appropriate scenario based on their audit goals and resources.

### Step 5: Create Grouped Reference Tables

**Business Rules**:
5.1. Consecutive populations with same sample size must be grouped
5.2. Groups displayed as ranges for space efficiency
5.3. Single populations shown individually when unique

**Python Code**:
```python
# Within the scenario loop:
# Group population ranges by sample size
grouped = df.groupby("Sample_Size")["Population_Size"].agg(["min", "max"]).reset_index()

# Create readable range descriptions
grouped["Population_Range"] = grouped.apply(
    lambda row: f"{row['min']}" if row["min"] == row["max"]
    else f"{row['min']}-{row['max']}", 
    axis=1
)

# Select final columns
grouped = grouped[["Sample_Size", "Population_Range"]]

# Store for Excel export
scenario["DataFrame_Grouped"] = grouped
```

**Explanation**: Grouping creates practical reference tables for audit teams. For example, if a district has 150 IEPs and the table shows "Sample Size: 25, Population Range: 100-200", the audit team knows to review 25 IEPs. This condensed format makes field reference quick and error-free.

### Step 6: Create Comparative Visualization

**Business Rules**:
6.1. All scenarios must appear on one graph for comparison
6.2. Legend must show actual calculated parameters
6.3. Graph suitable for inclusion in audit reports
6.4. Visual design must be accessible and professional

**Python Code**:
```python
print("▶ Creating line graph...")
plt.figure(figsize=(10, 6))

for scenario in scenarios_to_run:
    temp = plot_df[plot_df["Scenario"] == scenario["Name"]]
    
    # Calculate actual parameters for accurate legend
    confidence_level = 2 * stats.norm.cdf(scenario["Z"]) - 1
    label = f"±{scenario['Margin']*100:.1f}%, CL={round(confidence_level * 100)}%"
    
    # Plot with distinct visual style
    plt.plot(
        temp["Population_Size"],
        temp["Truncated_Sample_Size"],
        label=label,
        linestyle=scenario["LineStyle"],
        color=scenario["LineColor"],
        linewidth=1.5
    )

# Configure professional appearance
plt.xlabel("Population Size")
plt.ylabel(f"Sample Size (capped at {SAMPLE_CAP})")
plt.title("Sample Size vs Population Size")
plt.xticks(range(POP_MIN, POP_MAX+1, X_TICK_INTERVAL))
plt.yticks(range(0, SAMPLE_CAP+1, Y_TICK_INTERVAL))
plt.ylim(0, SAMPLE_CAP)
plt.xlim(POP_MIN, POP_MAX)
plt.grid(True, alpha=0.3)
plt.legend(loc='upper right')
plt.tight_layout()

# Save high-resolution image
graph_path = os.path.join(OUTPUT_DIR, "inline_plot.png")
plt.savefig(graph_path, dpi=300, bbox_inches='tight')
plt.show()
```

**Explanation**: This visualization directly supports decision-making for IEP audit planning. The legend shows exact margins of error and confidence levels, eliminating confusion about scenario parameters. The graph clearly illustrates how different precision requirements affect sample sizes, helping administrators balance accuracy with available resources.

### Step 7: Export to Structured Excel Workbook

**Business Rules**:
7.1. Summary sheet must be first for executive overview
7.2. Each scenario gets dedicated worksheet
7.3. Graphs embedded in multiple locations for reference
7.4. Professional formatting applied automatically

**Python Code**:
```python
print("▶ Exporting to Excel...")
excel_path = os.path.join(OUTPUT_DIR, "sample_size_output.xlsx")
summary_df = pd.DataFrame(scenario_summary)

with pd.ExcelWriter(excel_path, engine='xlsxwriter') as writer:
    # Executive summary with comparison
    summary_df.to_excel(writer, sheet_name="Scenario Summary", index=False)
    workbook = writer.book
    summary_ws = writer.sheets["Scenario Summary"]
    summary_ws.set_column('A:E', 20)
    
    # Embed comparison graph
    if os.path.exists(graph_path):
        summary_ws.insert_image("G2", graph_path, {"x_scale": 0.8, "y_scale": 0.8})
    
    # Individual scenario sheets
    for scenario in scenarios_to_run:
        sheet_name = scenario["Name"].split()[1]  # Extract "A", "B", "C"
        grouped = scenario["DataFrame_Grouped"]
        grouped.to_excel(writer, sheet_name=f"Section B - {sheet_name}", index=False)
        
        # Format for readability
        ws = writer.sheets[f"Section B - {sheet_name}"]
        ws.set_column('A:B', 25)
        
        # Add graph for reference
        if os.path.exists(graph_path):
            last_row = len(grouped) + 3
            ws.insert_image(f"A{last_row}", graph_path, {"x_scale": 0.8, "y_scale": 0.8})

print(f"✓ Excel file created: {excel_path}")
```

**Explanation**: The Excel workbook serves as the primary deliverable for IEP audit planning. The summary sheet helps leadership choose between scenarios, while individual sheets provide field teams with easy-to-use reference tables. Embedded graphs ensure visual context is always available, critical for training and documentation.

### Step 8: Complete Analysis and Auto-Download Results

**Business Rules**:
8.1. Provide clear completion confirmation
8.2. Display file locations for manual access
8.3. Automatically download Excel file
8.4. No user interaction required for download

**Python Code**:
```python
# Display completion message
print("\n" + "="*60)
print("Sample size analysis successfully completed!")
print("="*60)
print(f"View graph above or download:")
print(f"Excel: {excel_path}")
print(f"Graph: {graph_path}")

# Automatic download for immediate use
from google.colab import files
files.download(excel_path)
```

**Explanation**: The automatic download ensures IEP audit teams receive the Excel workbook immediately without navigating Colab's file system. This is particularly important for non-technical users who need quick access to the sample size tables for planning their compliance audits.

## Output Files

1. **Excel Workbook** (`sample_size_output.xlsx`):
   - **Scenario Summary**: Executive overview comparing all scenarios
   - **Section B - A**: Detailed table for ±13% margin scenario
   - **Section B - B**: Detailed table for ±15% margin scenario  
   - **Section B - C**: Detailed table for ±16% margin scenario
   - **Embedded Graphs**: Visual reference on multiple sheets

2. **PNG Image** (`inline_plot.png`):
   - Professional multi-scenario comparison graph
   - 300 DPI resolution for reports
   - Shows impact of different precision levels on sample sizes

## Usage Instructions

1. Open the script in Google Colab
2. Review default scenarios (modify if needed):
   - Adjust margins of error in `SCENARIOS` list
   - Set `RUN_SINGLE = True` for single scenario analysis
3. Run all cells sequentially
4. Review the comparison graph displayed inline
5. Excel file downloads automatically to your computer
6. Open Excel file to access sample size tables

## IEP Audit Applications

- **Annual Compliance Reviews**: Determine sample sizes for yearly IEP audits
- **Targeted Investigations**: Calculate samples for specific compliance concerns
- **Multi-District Comparisons**: Use consistent sampling across districts
- **Resource Planning**: Estimate staff time needed for audits
- **Training Materials**: Demonstrate statistical basis for sampling

## Technical Notes

- The 80% confidence level (Z=1.28) balances statistical validity with practical constraints
- Margin of error affects sample size significantly - choose based on audit importance
- Small populations (< 50 IEPs) often require high sampling percentages
- Results are reproducible - same parameters always yield same sample sizes
- The 50% proportion assumption provides maximum (conservative) sample sizes
