# lab2clean
R Package for Automation and Standardization of Cleaning Retrospective Clinical Lab Data
  <br>
  
## 1. Introduction
Navigating the shift of clinical laboratory data from primary everyday clinical use to secondary research purposes presents a significant challenge. Given the substantial time and expertise required to preprocess and clean this data and the lack of all-in-one tools tailored for this need, we developed our algorithm "lab2clean" as an open-source R-package. "lab2clean" package is set to automate and standardize the intricate process of cleaning clinical laboratory results. With a keen focus on improving the data quality of laboratory result values, while assuming compliance with established standards like LOINC and UCUM for test identifiers and units, our goal is to equip researchers with a straightforward, plug-and-play tool, making it smoother for them to unlock the true potential of clinical laboratory data in clinical research and clinical machine learning (ML) model development.

There are two functions inside the `lab2clean` R package, namely `clean_lab_result` and `validate_lab_result`. The
`clean_lab_result` function cleans and standardizes the laboratory results, and the `validate_lab_result` function validates the reults through flags, duplicates, and more.
<br>

## 2. Setup
### Install and load the `lab2clean` package
You can install the latest version of `lab2clean` directly from GitHub using the `remotes` package. If you do not have the `remotes` package installed, you can install it using the following command:

```{r install github, eval=FALSE}
install.packages("remotes")
remotes::install_github("Ahmed-Medhat-Zayed/lab2clean")
```

```{r read library,}
library(lab2clean)
```

## 3. Function 1: Clean and Standardize results


The `clean_lab_result` has five arguments:

* `lab_data` : A dataset containing laboratory data

* `raw_result` : The column in `lab_data` that contains raw result values to be cleaned

* `locale` : A string representing the locale for the laboratory data. Defaults to "NO"

* `report` : A report is generated. Defaults to "TRUE".

* `n_records` : In case you are loading a grouped list of distinct results, then you can assign the n_records to the column that contains the frequency of each distinct result. Defaults to NA


This function creates three different columns:

* `clean_result`: It cleans the raw_result column. For example, if the entry is `?`, the result is `<NA>`, if the entry is `3.14159 * 10^30`, the result is `3.142`, if the entry is `+++`, the result is `3+`, and so on.

* `scale_type` : This column categorizes distinct result format into specific result types, adhering to LOINC’s standard scale types such as Quantitative (Qn), Ordinal (Ord), Nominal (Nom). Within these scale types, our function does further subcategorization, such as differentiating simple numeric results (Qn.1) from inequalities (Qn.2), range results (Qn.3), or titer results (Qn.4) within the Quantitative scale.

* `cleaning_comments`: This column provides comments onto how the results were cleaned. 

The process above provided a generic description on how the `clean_lab_result` function operates. It would be useful to delve into more details on the exact way that some of the specific raw results are cleaned:

* `Locale understaning`: In the `clean_lab_result` function, we have an argument named locale. It addresses the variations in number formats with different decimal and thousand separators that arise due to locale-specific settings used internationally. The default is to standardize these varying languages and locale-specific settings to English (EN).

* `Languages used in common words`: In the `clean_lab_result` function, we support 19 distinct languages in representing frequently used terms such as "high," "low," "positive," and "negative. For example, the word `Pøsitivo` is included in the common words and will be cleaned as `Pos`.

* `Flags creation`: When a common word is combined with a numeric value, we keep the numeric value, but we create a flag to inform the user that further checks might be requested. For example, the word `Négatif 0.3` is cleaned as `0.3` and the word `33 Normal` is cleaned as `33`. In addition, when there is a space between a numeric value and a character, we will also create a flag. For example,  word `- 5` is cleaned as `5` with a flag, but the word `-5` is cleaned as `-5`, and no flag is created because we can assume it was a negative value.  

Finally let us discuss how this table is used to further clean our function:
```{r common words, warning=FALSE, message=FALSE}
data("common_words", package = "lab2clean")
common_words
```

As we can see, there are 19 languages for 8 common words. If the words are positive or negative, then the result will either be cleaned to Pos or Neg unless if it is proceeded by a number, therefore a flag will be created. If the words are High, Low and Medium, there will be flags. If it is not detected, then it will go to Neg. Finally, if it is Sample or Specimen, then a comment will pop-up mentioning that the test was not performed.

## 4. Function 2: Validate results

The `validate_lab_result` has six arguments:

* `lab_data` : A data frame containing laboratory data

* `result_value` : The column in lab_data with quantitative result values for validation

* `result_unit` : The column in lab_data with result units in a UCUM-valid format

* `loinc_code` : The column in lab_data indicating the LOINC code of the laboratory test

* `patient_id` : The column in lab_data indicating the identifier of the tested patient.

* `lab_datetime` : The column in lab_data with the date or datetime of the laboratory test.

Let us also explain two tables that we used for the validation function. Let us begin with the reportable interval table.
```{r reportable_interval, warning=FALSE, message=FALSE}
data("reportable_interval", package = "lab2clean")
reportable_interval_subset <- reportable_interval[reportable_interval$interval_loinc_code == "2160-0", ]
reportable_interval_subset
```

* Patient 14499007 has a flag named `low_unreportable`. As we can see, for the "2160-0" loinc_code, his result was 0.0 which was not in the acceptable reportable limit (0.0001, 120). In a similar note, patient 17726236 has a `high_unreportable`.


Let us continue with the logic rules table
```{r logic_rules, warning=FALSE, message=FALSE}
data("logic_rules", package = "lab2clean")
logic_rules <- logic_rules[logic_rules$rule_id == 3, ]
logic_rules
```

* Patient 10000003 has both `logic_flag` and `duplicate`. The `duplicate` means that this patient has a duplicate row, whereas the `logic_flag` should be interpreted as follows. For the loinc_code "2093-3", which is cholesterol, we need that the "2093-3" > "2085-9" + "13457-7", or equivalently cholesterol > hdl cholesterol + ldl cholesterol (from the logic rules table). Therefore for patient 10000003, we have a logic flag because his ldl ("13457-7") equals 100.0 and his hdl ("2085-9") equals 130.0. His total cholesterol ("2093-3") equals 230. Therefore we see that the rule "2093-3" > "2085-9" + "13457-7" is not satidfied because we have 230 > 100+130, i.e. 230>230, which is clearly false, and thus a logic flag is created.

We suggest that the flags being created to be checked from an expert to ensure...
