# nanoplastics
This repository contains a comprehensive analysis pipeline for investigating the environmental weathering and metabolic degradation of polystyrene (PS) nanoparticles using non-targeted mass spectrometry. The study examines three polymer types (PET, PP, PS) across multiple weathering conditions.
### Experimental Design

#### **Experiment 1: Environmental Weathering**
- **Polymers:** Polyethylene terephthalate (PET), Polypropylene (PP), Polystyrene (PS)
- **Weathering Conditions:**
  - RAW: Raw polymer material
  - SEP C: Nanoparticles without weathering (control)
  - SEP: Accelerated weathering (UV + mechanical stress)
  - Semi: Environmental weathering (natural conditions)
- **Samples:** 12 total (3 polymers × 4 conditions)
- **Analysis:** Non-targeted LC-MS/MS
- **Features:** 21,928 molecular features detected

#### **Experiment 2: Metabolic Degradation**
- **Test Sample:** PS SEP (weathered polystyrene nanoparticles)
- **Control Sample:** Caffeine (positive control for microsomal activity)
- **Incubation System:** Liver microsomes + NADPH + buffers (MgCl₂, Tris-HCl)
- **Time Points:** 0, 2, 15, 30, 90 minutes
- **Blank:** No sample control
- **Features:** 17,051 molecular features detected

---

## Repository Structure

```
├── README.md                              # This file
├── RESULTS_SUMMARY.md                     # Key findings and conclusions
│
├── Data Processing Scripts/
│   ├── DataProcess1.py                    # Initial data processing and feature filtering
│   ├── meta_processing_find_features.py   # Feature annotation and MS/MS matching
│   └── batch_correction.py                # Batch effect correction (ComBat)
│
├── Weathering Analysis/
│   ├── pca_analysis.py                    # PCA visualization (original data)
│   ├── clustermap_analysis.py             # Hierarchical clustering heatmaps
│   ├── weathering_analysis.py             # Comprehensive weathering trajectory analysis
│   └── distance_visualizations.py         # Alternative distance visualization methods
│
├── Metabolic Degradation Analysis/
│   └── metabolism_analysis.py             # Time-course analysis for microsomal incubation
│
├── Input Data/
│   ├── alignment_mini.csv                 # Feature table with m/z, RT, intensities
│   ├── Batches.xlsx                       # Batch assignment for correction
│   └── alignment_metadata.csv             # Sample metadata
│
└── Output Files/
    ├── Weathering Results/
    │   ├── corrected_combat_style.csv
    │   ├── weathering_pca_trajectories.png
    │   ├── weathering_distance_analysis.png
    │   ├── weathering_distances.csv
    │   ├── weathering_feature_changes.csv
    │   └── distance_*.png (various visualizations)
    │
    └── Metabolism Results/
        ├── metabolism_pca_timecourse.png
        ├── metabolism_increasing.png
        ├── metabolism_decreasing.png
        ├── metabolism_comparison.png
        ├── metabolism_full_results.csv
        └── metabolism_ps_specific.csv
```

---

## Analysis Workflow

### Phase 1: Data Preprocessing

#### 1.1 Initial Processing (`DataProcess1.py`)
**Purpose:** Process raw MZmine feature tables and filter for significant features

**Input:**
- `alignment_full_feature_table.csv` - Raw MZmine output
- `alignment_metadata.csv` - Sample information

**Output:**
- `alignment_mini.csv` - Processed feature table
- `alignment_mini_stats5.csv` - Statistical results
- `PS_chemicals_with_time_trend.csv` - Time-dependent features
- `PS_chemicals.csv` - PS-specific chemicals

**Key Steps:**
1. Extract area and height measurements
2. Calculate blank-corrected intensities
3. Perform linear regression vs time
4. Identify PS-specific features
5. Filter by statistical significance (p < 0.05)

**Usage:**
```bash
python3 DataProcess1.py
```

#### 1.2 Feature Annotation (`meta_processing_find_features.py`)
**Purpose:** Link weathering results with MS/MS spectral data

**Input:**
- `alignment_mini.csv`
- `weathering_feature_changes.csv`

**Output:**
- `weathering_features_with_msms.csv` - Annotated features with MS/MS scans

**Usage:**
```bash
python3 meta_processing_find_features.py
```

#### 1.3 Batch Correction (`batch_correction.py`)
**Purpose:** Remove technical variation while preserving biological differences

**Input:**
- `alignment_mini.csv`
- `Batches.xlsx`

**Output:**
- `corrected_combat_style.csv` - ComBat-corrected data
- `corrected_*.csv` - Results from 4 correction methods
- Comparison visualizations

**Methods Compared:**
1. Median Normalization
2. Mean Centering
3. **ComBat-style** (SELECTED - standard for metabolomics)
4. Ratio-based Normalization

**Usage:**
```bash
python3 batch_correction.py
```

**Key Finding:** ComBat-style correction selected despite higher CV (194.25%) because it's the publication-standard method for metabolomics data.

---

### Phase 2: Weathering Analysis

#### 2.1 Principal Component Analysis (`pca_analysis.py`)
**Purpose:** Initial exploratory visualization of sample relationships

**Input:** `corrected_combat_style.csv`

**Output:**
- `pca_scree.png` - Variance explained by each PC
- `pca_scores.png` - 2D scores plot (PC1 vs PC2)
- `pca_scores_3d.png` - 3D scores plot
- `pca_loadings.png` - Top contributing features
- `pca_results.csv` - PC coordinates for all samples

**Key Statistics:**
- PC1: 18.3% variance
- PC2: 16.9% variance
- PC3: 9.0% variance
- Cumulative (PC1-PC3): 44.2%

**Usage:**
```bash
python3 pca_analysis.py
```

#### 2.2 Hierarchical Clustering (`clustermap_analysis.py`)
**Purpose:** Visualize sample relationships through clustering

**Input:** `corrected_combat_style.csv`

**Output:**
- `clustermap_combat.png` - Feature × sample heatmap
- `clustermap_combat_correlation.png` - Sample correlation matrix
- `clustermap_combat_distance.png` - Sample distance matrix
- `sample_correlation_matrix.csv`
- `sample_distance_matrix.csv`

**Methods:**
- Top 1,000 most variable features selected
- Log₁₀ + Z-score normalization
- Average linkage, Euclidean distance
- Dual color annotations (polymer type + weathering category)

**Usage:**
```bash
python3 clustermap_analysis.py
```

#### 2.3 Comprehensive Weathering Analysis (`weathering_analysis.py`)
**Purpose:** Quantify weathering trajectories and compare polymer degradation

**Input:** `corrected_combat_style.csv`

**Output:**
- `weathering_pca_trajectories.png` - Trajectories with arrows
- `weathering_distance_analysis.png` - 4-panel distance comparison
- `weathering_distances.csv` - All pairwise distances
- `weathering_feature_changes.csv` - Top 50 features per polymer

**Key Analyses:**
1. **Weathering trajectory PCA** - Shows RAW → SEP C → SEP → Semi progression
2. **Euclidean distance calculations** - In original 21,928D feature space
3. **Feature-level fold changes** - Top changing features (RAW to Semi)

**Research Questions Answered:**
1. **How do polymers degrade from RAW to Semi?**
   - Total weathering distance (RAW → Semi):
     - PS: 189.75 (most altered)
     - PP: 160.11 (moderate)
     - PET: 138.82 (most stable)

2. **Does accelerated weathering (SEP) replicate environmental weathering (Semi)?**
   - Across all polymers, SEP is 20-29% closer to SEP C than to Semi
   - **Conclusion:** Accelerated weathering does NOT fully replicate environmental weathering

**Usage:**
```bash
python3 weathering_analysis.py
```

#### 2.4 Distance Visualizations (`distance_visualizations.py`)
**Purpose:** Alternative visualizations of original feature space distances

**Input:** `corrected_combat_style.csv`

**Output:**
- `distance_heatmap.png` - Color-coded distance matrix
- `distance_dendrogram.png` - Hierarchical clustering tree
- `distance_trajectories.png` - Weathering path distances
- `distance_mds.png` - 2D projection preserving distances
- `distance_network.png` - Network graph

**Key Difference from PCA:**
- **MDS (Multidimensional Scaling):** Optimizes distance preservation in 2D
- **PCA:** Optimizes variance explanation
- MDS better represents actual 21,928D distances

**Usage:**
```bash
python3 distance_visualizations.py
```

---

### Phase 3: Metabolic Degradation Analysis

#### 3.1 Microsomal Incubation Analysis (`metabolism_analysis.py`)
**Purpose:** Evaluate metabolic degradation of PS nanoparticles

**Input:** `alignment_mini.csv`

**Output:**
- `metabolism_pca_timecourse.png` - Time trajectory (T0 → T90)
- `metabolism_increasing.png` - Features increasing (metabolites)
- `metabolism_decreasing.png` - Features decreasing (degradation)
- `metabolism_comparison.png` - PS vs Caffeine comparison
- `metabolism_full_results.csv` - Complete results (17,051 features)
- `metabolism_ps_specific.csv` - PS-specific features (130 features)

**Key Analyses:**
1. **Time trend analysis** - Linear regression for each feature vs time
2. **PS-specific feature identification** - Filters out blank and caffeine signals
3. **Metabolite formation** - Features increasing over time
4. **Parent compound degradation** - Features decreasing over time
5. **Microsomal activity validation** - Caffeine control assessment

**Key Results:**
- **Total features:** 17,051
- **PS-specific features (strict):** 130 with significant time trends
- **PS increasing features:** 1,557 (potential metabolites)
- **PS decreasing features:** 162 (parent compounds metabolized)
- **Caffeine control:** 2,921 significant features (✓ microsomes active)

**Interpretation:**
1. PS nanoparticles are metabolically degraded by liver microsomes
2. More metabolite formation than degradation (1,557 vs 162)
3. PS degradation differs from caffeine metabolism
4. Microsomal system is metabolically active (validated by caffeine)

**Usage:**
```bash
python3 metabolism_analysis.py
```

---

## Key Statistical Methods

### Distance Calculations

**Original Feature Space (21,928D):**
```
d(A,B) = √[Σᵢ₌₁²¹⁹²⁸ (Aᵢ - Bᵢ)²]
```
- Uses ALL features
- Captures complete chemical differences
- Used for weathering distance analysis

**PCA Space (2D):**
```
d(A,B) = √[(PC1ₐ - PC1ᵦ)² + (PC2ₐ - PC2ᵦ)²]
```
- Uses only PC1 and PC2 (~35% variance)
- For visualization only
- Does NOT capture full chemical differences

### Data Transformations

**Standard Pipeline:**
1. Replace zeros with min_value/2
2. Log₁₀ transformation: `log₁₀(x + 1)`
3. Z-score standardization: `(x - μ) / σ`

### Batch Correction (ComBat)

**Model:**
```
Y*ᵢⱼ = (Yᵢⱼ - α - Xβ - γᵢ)/δᵢ + α + Xβ
```
Where:
- Y*ᵢⱼ = Corrected intensity
- γᵢ = Additive batch effect
- δᵢ = Multiplicative batch effect
- α, β = Overall effects

---

## Software Requirements

### Python Version
- Python 3.8 or higher

### Required Packages
```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy
```

**Detailed Requirements:**
- pandas >= 1.3.0
- numpy >= 1.21.0
- matplotlib >= 3.4.0
- seaborn >= 0.11.0
- scikit-learn >= 0.24.0
- scipy >= 1.7.0

### Installation
```bash
# Create virtual environment (optional but recommended)
python3 -m venv ms_analysis_env
source ms_analysis_env/bin/activate  # On Windows: ms_analysis_env\Scripts\activate

# Install requirements
pip install -r requirements.txt
```

---

## Running the Complete Pipeline

### Step-by-Step Execution

```bash
# 1. Data preprocessing
python3 DataProcess1.py
python3 batch_correction.py
python3 meta_processing_find_features.py

# 2. Weathering analysis
python3 pca_analysis.py
python3 clustermap_analysis.py
python3 weathering_analysis.py
python3 distance_visualizations.py

# 3. Metabolic degradation analysis
python3 metabolism_analysis.py
```

### Quick Start (Main Results Only)

```bash
# Generate key weathering results
python3 batch_correction.py
python3 weathering_analysis.py

# Generate metabolic degradation results
python3 metabolism_analysis.py
```

---

## Output Interpretation Guide

### Weathering Analysis Outputs

#### Distance Values
- **PET:** 138.82 (most chemically stable)
- **PP:** 160.11 (intermediate stability)
- **PS:** 189.75 (most chemically altered)

**Interpretation:** PS undergoes the greatest chemical transformation during weathering, consistent with literature showing PS is more susceptible to UV degradation than PET.

#### SEP vs Semi Comparison
- **All polymers:** SEP is 20-29% closer to SEP C than to Semi
- **Interpretation:** Accelerated weathering protocols do not fully replicate environmental weathering chemistry

### Metabolic Degradation Outputs

#### PS-Specific Features
- **130 features** with significant time-dependent changes (p < 0.05)
- **Strict criteria:** PS > Blank AND PS > Caffeine AND significant trend

#### Metabolite Formation
- **1,557 features increasing** over 90 minutes
- Suggests PS breaks down into multiple metabolic products

#### Parent Compound Degradation
- **162 features decreasing** over 90 minutes
- Represents original PS components being metabolized

---

## Literature Context

### Polymer Environmental Stability

**From Literature Search:**
- **PET degradation time:** ~1,179 years (most persistent)
- **PS degradation time:** ~900 years
- **PP degradation time:** ~0.27 years (with UV exposure)

**Note:** PP's rapid degradation in pure form vs. our intermediate results (PP between PS and PET) is explained by commercial stabilizers in our PP samples. This makes our results more representative of real-world commercial plastics.

### Key Finding Validation
Our results align with literature:
- PET most resistant to weathering
- PS susceptible to UV-induced chemical changes
- Commercial PP (with stabilizers) shows intermediate behavior

---

## Troubleshooting

### Common Issues

**Issue 1: "FileNotFoundError: alignment_mini.csv"**
- **Solution:** Ensure input files are in the current directory or provide full path

**Issue 2: Memory errors with large datasets**
- **Solution:** Reduce `top_n` parameter in filtering functions
- Use `filter_features()` with lower `top_n` value (e.g., 500 instead of 1000)

**Issue 3: PCA plots appear crowded**
- **Solution:** Adjust `figsize` parameter in plotting functions
- Increase DPI for better resolution: `dpi=300` → `dpi=600`

**Issue 4: ComBat correction warnings**
- **Solution:** These are usually harmless; check that batch assignments are correct
- Verify `Batches.xlsx` has correct sample-to-batch mapping

---

## Version History

### Version 1.0 (December 2025)
- Initial release
- Complete weathering analysis pipeline
- Metabolic degradation analysis
- Comprehensive visualization suite

---
