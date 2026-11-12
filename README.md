# Loan Portfolio and Default Risk Analysis

A Python analysis of 307,511 consumer loans: data cleaning, a portfolio summary written for senior management, a comparison of customers who fall behind on payments against those who do not, and a baseline model that estimates the probability of default.

This was a course project in the MS Information Systems program at Baruch College (November 2023). The client, "Amalgamated Bank Corp (ABC)", is a fictional lender invented for the case. The data is the public [Home Credit Default Risk](https://www.kaggle.com/competitions/home-credit-default-risk) dataset from Kaggle.

Everything is in one notebook: [`loan_default_analysis.ipynb`](loan_default_analysis.ipynb).

## Key findings

| Question from management | Answer |
| :--- | :--- |
| How much do we have out in loans? | About $179.8 billion across 301,991 loans (after cleaning) |
| How much is in default or late? | 24,441 loans, about $13.6 billion |
| What share of the business is that? | 8.09% of loans, 7.56% of the amount |

Who has trouble repaying:

- **Men more than women.** 10.2% of male borrowers have payment difficulties against 7.0% of female borrowers, although women hold about two thirds of the loans.
- **Single and civil-marriage borrowers are over-represented.** Single borrowers are 14.4% of healthy accounts but 17.8% of troubled ones. Married and widowed borrowers are under-represented.
- **Middle incomes carry slightly more risk.** Borrowers earning roughly 80,000 to 200,000 make up a larger share of troubled accounts than of healthy ones. Above 200,000 the pattern reverses.
- **Younger borrowers and borrowers with more children score as higher risk** in the model. Age has the largest effect of the non-financial factors tested.

![Portfolio summary: default by count, amount and percent](screenshot1.png)

## Method

**1. Cleaning.** Six columns had missing values, and each was handled on its merits.

| Column | Missing | Treatment | Reason |
| :--- | ---: | :--- | :--- |
| `OCCUPATION_TYPE` | 96,391 | Filled with "Unknown" | Too many to drop; keeps the rows usable |
| `OWN_CAR_AGE` | 202,929 | Filled with -1 | Marks "not available" (mostly borrowers without a car) and is easy to filter |
| `NAME_TYPE_SUITE` | 1,292 | Filled with "Unknown" | Low-value field, not worth losing rows over |
| `AMT_GOODS_PRICE`, `AMT_ANNUITY`, `CNT_FAM_MEMBERS` | 278, 12, 2 | Rows dropped | Essential loan fields; very few records |

**2. Outliers.** Each numeric column was checked with a box plot, a scatter plot, the 1.5 × IQR rule and the 99th and 99.9th percentiles, using two small reusable functions (`find_outlier`, `remove_outlier`). Cut-offs were set by combining the statistics with common sense, for example removing a recorded income of 117 million as a data-entry error, and treating the placeholder value 365,243 in `DAYS_EMPLOYED` as "not employed" instead of deleting 55,000 rows. Across cleaning and outlier removal 5,520 records (1.8%) were dropped. Day counts (age, employment, registration) were converted to years.

**3. Portfolio and customer profile.** Summary charts of exposure, default rate, income type, education, family status and housing, plus income by education and age band.

![Customer profile](screenshot2.png)
![Income by education, family status and age band](screenshot3.png)

**4. Default analysis.** Side-by-side comparisons of troubled and healthy accounts by gender, car and property ownership, number of children, family status and income.

![Default by gender](screenshot4.png)
![Default by family status](screenshot5.png)
![Default by income band](screenshot6.png)

**5. Predictive model.** A logistic regression (scikit-learn, `class_weight='balanced'`) on ten numeric features: children, income, credit amount, annuity, age, years employed, years since registration, years since ID issue, car age and family size.

| | Predicted OK | Predicted default |
| :--- | ---: | ---: |
| **Actual OK** | 205,764 | 71,786 |
| **Actual default** | 16,190 | 8,251 |

- Accuracy: 70.9%
- Recall on defaults: 33.8% (the model catches about one in three problem loans)
- Precision on defaults: 10.3%

A second version without the three money columns scored lower (57.4%) but made the effect of each remaining factor easier to read, which is how the age, children and family-size findings above were obtained.

## Limitations

This is a baseline to show management what is possible, not a production credit model.

- The model is scored on the same data it was trained on, as the assignment specified. A train/test split would give an honest out-of-sample figure.
- Features were not scaled, so the large money columns dominate the small year columns. Standardizing them is the first improvement to make.
- Only numeric features were used. Occupation, education and income type are likely to add predictive power once encoded.
- With 8% of loans in default, accuracy alone is misleading; recall, precision and AUC are the better measures.

## Running it

```bash
pip install -r requirements.txt
jupyter notebook loan_default_analysis.ipynb
```

The data file is not included in this repository. Download `application_train.csv` from the [Kaggle competition page](https://www.kaggle.com/competitions/home-credit-default-risk/data) and place it next to the notebook. The course version of the file was reduced to the 36 columns below; keep only these to reproduce the figures above.

<details>
<summary>Columns used</summary>

`SK_ID_CURR`, `NAME_CONTRACT_TYPE`, `CODE_GENDER`, `FLAG_OWN_CAR`, `FLAG_OWN_REALTY`, `CNT_CHILDREN`, `AMT_INCOME_TOTAL`, `AMT_CREDIT`, `AMT_ANNUITY`, `AMT_GOODS_PRICE`, `NAME_TYPE_SUITE`, `NAME_INCOME_TYPE`, `NAME_EDUCATION_TYPE`, `NAME_FAMILY_STATUS`, `NAME_HOUSING_TYPE`, `DAYS_BIRTH`, `DAYS_EMPLOYED`, `DAYS_REGISTRATION`, `DAYS_ID_PUBLISH`, `OWN_CAR_AGE`, `FLAG_MOBIL`, `FLAG_EMP_PHONE`, `FLAG_WORK_PHONE`, `FLAG_CONT_MOBILE`, `FLAG_PHONE`, `FLAG_EMAIL`, `OCCUPATION_TYPE`, `CNT_FAM_MEMBERS`, `REG_REGION_NOT_LIVE_REGION`, `REG_REGION_NOT_WORK_REGION`, `LIVE_REGION_NOT_WORK_REGION`, `REG_CITY_NOT_LIVE_CITY`, `REG_CITY_NOT_WORK_CITY`, `LIVE_CITY_NOT_WORK_CITY`, `ORGANIZATION_TYPE`, `TARGET`

</details>

The notebook writes a cleaned copy, `application_train_clean.csv`, which the analysis and model sections read back in.

## Repository contents

| File | What it is |
| :--- | :--- |
| `loan_default_analysis.ipynb` | The full analysis with outputs |
| `home-credit-columns-description.csv` | Data dictionary from Home Credit |
| `screenshot1.png` to `screenshot6.png` | Charts used in this README |
| `requirements.txt` | Python packages |

## Author

Sijie Hu, MS Information Systems (Data Analytics), Baruch College. [LinkedIn](https://linkedin.com/in/thesjhu)
