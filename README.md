# Predict Academic Success and Dropout (KNIME)

DA3131 Data Mining

A KNIME workflow that predicts whether a university student will **drop out**, stay **enrolled**, or **graduate**. The goal is to build an early warning system so universities can support at-risk students sooner.

## Problem

Student dropout affects students, universities and society. If a university can identify at-risk students at enrollment and after the first semester, it can offer early help such as tutoring, mentoring and financial counselling.

This is a **3-class classification** problem.

## Dataset

- **Name:** Predict Students' Dropout and Academic Success
- **Source:** UCI Machine Learning Repository (Realinho et al., 2021)
- **Link:** https://archive.ics.uci.edu/dataset/697/predict+students+dropout+and+academic+success
- **Size:** 4,424 students, 36 features, 1 target column
- **Target classes:** Graduate (2,209), Dropout (1,421), Enrolled (794)
- **Missing values:** none
- **License:** CC BY 4.0

Feature groups: demographics (age, gender, marital status), academic (previous grades, course, first and second semester results), and socioeconomic (parents' education and occupation, debt, scholarship, tuition status).

**Main challenges**
- Class imbalance (Enrolled is the smallest class).
- Many categorical columns are stored as numeric codes, so they must be decoded.

## Repository structure

```
.
├── README.md
├── data/
│   └── dataset.csv                  # original dataset (semicolon separated)
├── workflow/
│   └── Student_Dropout_Group05.knwf # exported KNIME workflow
├── images/
│   └── knime_canvas.png             # screenshot of the KNIME canvas
└── presentation/
    └── Group05_Presentation.pdf     # slide deck
```

Change the names above to match your actual files.

## Tools

- KNIME Analytics Platform 5.5.1
- No programming is needed. The whole pipeline is built with KNIME nodes.

## How to run the workflow

1. Install KNIME Analytics Platform 5.5.1 or newer.
2. In KNIME, choose **File > Import KNIME Workflow** and select `workflow/Student_Dropout_Group05.knwf`.
3. Open the **CSV Reader** node and set the file path to `data/dataset.csv` on your computer.
4. Check that the column delimiter is `;`.
5. If SMOTE is missing, install it with **File > Install KNIME Extensions**, search for SMOTE, then restart KNIME.
6. Run all nodes with **Ctrl+Shift+F7**.
7. Right click each **Scorer** node and open the confusion matrix and accuracy statistics.

## Workflow overview

![KNIME workflow](images/knime_canvas.png)<img width="1447" height="662" alt="Student_Dropout" src="https://github.com/user-attachments/assets/5a1e6f3c-dfd0-4cc0-9124-cbfe7a0834bd" />


### 1. Data reading
- **CSV Reader** reads `dataset.csv` (delimiter `;`).
- **Column Name Replacer** (regex) removes stray quote, tab and file-marker characters from the column names.
- **Data Explorer** checks the columns and data types.

### 2. Exploratory data analysis
Done before and after preprocessing: Statistics, Histogram, Pie Chart, Box Plot, Scatter Plot, Linear Correlation, Value Counter, Bar Chart, Parallel Coordinates Plot and Sunburst Chart.

### 3. Preprocessing
1. **Missing Value:** check, rows with missing values would be removed (none found).
2. **Rule Engine (8 nodes):** decode numeric codes into readable categories and group rare categories. Columns decoded: Marital status, Application mode, Course, Previous qualification, Mother's qualification, Father's qualification, Mother's occupation, Father's occupation.
3. **One to Many:** converts the decoded text columns into 0/1 columns (about 96 features).
4. **Column Filter:** removes `Nacionality` because almost all students have the same nationality.
5. **Normalizer:** Min-Max scaling from 0 to 1.

### 4. Data splitting and balancing
- **Table Partitioner:** 80% train, 20% test, stratified on `Target`, random seed 42.
- **SMOTE:** applied to the **train set only** to balance the classes. The test set is left unchanged.

### 5. Models
| Model | Main settings |
|---|---|
| Decision Tree | Gini index, MDL pruning |
| Random Forest | 100 trees, seed 42 |
| Logistic Regression | Reference class Graduate, Stochastic average gradient solver, Gauss prior (variance 0.1) |
| Gradient Boosted Trees | 100 models, depth 4, learning rate 0.1, seed 42 |

Each model uses a Learner, a Predictor and a Scorer node.

## Results

Results on the test set (885 students, 20% of the data, not used for training or SMOTE).

| Model | Accuracy | Dropout F1 | Graduate F1 | Enrolled F1 |
|---|---|---|---|---|
| Decision Tree | 75.7% | 0.760 | 0.843 | 0.487 |
| Random Forest | 78.5% | 0.801 | 0.864 | 0.468 |
| Logistic Regression | 70.5% | 0.766 | 0.776 | 0.465 |
| Gradient Boosted Trees | **78.9%** | 0.796 | 0.872 | 0.496 |

Precision and recall per class:

| Model | Dropout (P / R) | Graduate (P / R) | Enrolled (P / R) |
|---|---|---|---|
| Decision Tree | 0.820 / 0.708 | 0.798 / 0.894 | 0.510 / 0.465 |
| Random Forest | 0.841 / 0.764 | 0.798 / 0.941 | 0.585 / 0.390 |
| Logistic Regression | 0.840 / 0.704 | 0.797 / 0.756 | 0.395 / 0.566 |
| Gradient Boosted Trees | 0.831 / 0.764 | 0.820 / 0.930 | 0.569 / 0.440 |

**Best model:** Gradient Boosted Trees. It has the highest accuracy (78.9%) and the highest Cohen's kappa (0.646), the best F-measure for Graduate (0.872) and Enrolled (0.496), and it finds 76.4% of actual dropouts (Dropout recall 0.764, tied with Random Forest). Random Forest is very close at 78.5%.

**Notes**
- The Enrolled class is the hardest to predict because it is the smallest class and overlaps with the other two.
- Logistic Regression may show a convergence warning. Increasing the maximum epochs can help.

## Key recommendations

- **At enrollment:** flag students with lower parental education, older age at enrollment, or financial issues. Assign a mentor and offer financial counselling.
- **After the first semester:** run the model with first semester grades. Give intensive support (tutoring, study skills workshops, counselling) to students with a low pass rate.

## Future work

- Real-time dashboard for advisors
- Mid-semester check-ins as extra data
- Testing on data from other institutions
- Personalized intervention recommendations

## Team

Group 05, DA3131 Data Mining

| Name | Student ID |
|---|---|
| | |
| | |
| | |
| | |
| | |

## Reference

Realinho, V., Machado, J., Baptista, L., and Martins, M. V. (2021). *Predict Students' Dropout and Academic Success* [Dataset]. UCI Machine Learning Repository.

## License

The dataset is shared under CC BY 4.0. Add a license for your own work if you wish, for example MIT.
