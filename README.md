# lab2clean
R Package for Automation and Standardization of Cleaning Retrospective Clinical Lab Data
  <br>
  
## 1. Introduction
Navigating the shift of clinical laboratory data from primary everyday clinical use to secondary research purposes presents a significant challenge. Given the substantial time and expertise required to preprocess and clean this data and the lack of all-in-one tools tailored for this need, we developed our algorithm "lab2clean" as an open-source R-package. "lab2clean" package is set to automate and standardize the intricate process of cleaning clinical laboratory results. With a keen focus on improving the data quality of laboratory result values, while assuming compliance with established standards like LOINC and UCUM for test identifiers and units, our goal is to equip researchers with a straightforward, plug-and-play tool, making it smoother for them to unlock the true potential of clinical laboratory data in clinical research and clinical machine learning (ML) model development.

The lab2clean package contains two key functions: `clean_lab_result()` and `validate_lab_result()`. The `clean_lab_result()` function cleans and standardizes the laboratory results, and the `validate_lab_result()` function performs validation to ensure the plausibility of these results. This vignette aims to explain the theoretical background, usage, and customization of these functions.
<br>

## 2. Setup
### Install and load the `lab2clean` package
You can install the latest version of `lab2clean` directly from GitHub using the `remotes` package. If you do not have the `remotes` package installed, you can install it using the following command:

```{r install github, eval=FALSE}
install.packages("remotes")
remotes::install_github("Ahmed-Medhat-Zayed/lab2clean")
```
After installation, load the package:
```{r read library,}
library(lab2clean)
```

## 3. Function 1: Clean and Standardize results


The `clean_lab_result()` has five arguments:

* `lab_data` : A dataset containing laboratory data

* `raw_result` : The column in `lab_data` that contains raw result values to be cleaned

* `locale` : A string representing the locale for the laboratory data. Defaults to "NO"

* `report` : A report is generated. Defaults to "TRUE".

* `n_records` : In case you are loading a grouped list of distinct results, then you can assign the n_records to the column that contains the frequency of each distinct result. Defaults to NA


Let us demonstrate the `clean_lab_result()` function using `Function_1_dummy` and inspect the first six rows:

```{r Function_1_dummy, }
data("Function_1_dummy", package = "lab2clean")
head(Function_1_dummy,6)
```

This dataset -for demonstration purposes- contains two columns: `raw_result` and the `frequency`. The `raw_result` column holds raw laboratory results, and `frequency` indicates how often each result appeared. Let’s explore the `report` and `n_records` arguments:

```{r function with report, results='markup'}
cleaned_results <- clean_lab_result(Function_1_dummy, raw_result = "raw_result", report = TRUE, n_records = "frequency")
```

The `report` provides a detailed report on how the whole process of cleaning the data is done, and offers some descriptive insights of the process. The `n_records` argument adds percentages to each of the aforementioned steps to enhance the reporting. For simplicity, we will use `report = FALSE` in the rest of this tutorial:

```{r function with report 1, }
cleaned_results <- clean_lab_result(Function_1_dummy, raw_result = "raw_result", report = FALSE)
cleaned_results
```


This function creates three different columns:

1- `clean_result`: The cleaned version of the `raw_result` column. For example, "?" is converted to <NA>, "3.14159 * 10^30" to "3.142", and "+++" to "3+".

2- `scale_type` : Categorizes the cleaned results into specific types like Quantitative (Qn), Ordinal (Ord), or Nominal (Nom), with further subcategories for nuanced differences, such as differentiating simple numeric results (Qn.1) from inequalities (Qn.2), range results (Qn.3), or titer results (Qn.4) within the Quantitative scale.

3- `cleaning_comments`: Provides insights on how the results were cleaned. 

The process above provided a generic description on how the `clean_lab_result()` function operates. It would be useful to delve into more details on the exact way that some of the specific raw results are cleaned:

* `Locale` variable:

In the `clean_lab_result()` function, we have an argument named locale. It addresses the variations in number formats with different decimal and thousand separators that arise due to locale-specific settings used internationally. We chose to standardize these varying languages and locale-specific settings to have the cleaned results in English, US. If the user did not identify the locale of the dataset, the default is `NO`, which means not specified. For example for rows 71 and 72, there is a locale_check in the `cleaning_comments`, and the results are 1.015 and 1,060 respectively. That means that either "US" or "DE" locale should be specified to identify this result value. If we specified the locale as `US` or `DE`, we can see different values as follows:

```{r locale, warning=FALSE, message=FALSE}
Function_1_dummy_subset <- Function_1_dummy[c(71,72),, drop = FALSE]
cleaned_results <- clean_lab_result(Function_1_dummy_subset, raw_result = "raw_result", report = FALSE, locale = "US")
cleaned_results
cleaned_results <- clean_lab_result(Function_1_dummy_subset, raw_result = "raw_result", report = FALSE, locale = "DE")
cleaned_results
```


* `Language` in `common words`:

In the `clean_lab_result()` function, we support 19 distinct languages in representing frequently used terms such as "high," "low," "positive," and "negative. For example, the word `Pøsitivo` is included in the common words and will be cleaned as `Pos`.

Let us see how this data table works in our function:
```{r common words, warning=FALSE, message=FALSE}
data("common_words", package = "lab2clean")
common_words
```

As seen in this data, there are 19 languages for 8 common words. If the words are positive or negative, then the result will either be cleaned to `Pos` or `Neg` unless if it is proceeded by a number, therefore the word is removed and a flag is added to the `cleaning_comments`. For example, the word `Négatif 0.3` is cleaned as `0.3` and the word `33 Normal` is cleaned as `33`. Finally, if the result has one of those words "Sample" or "Specimen", then a comment will pop-up mentioning that `test was not performed`.


* `Flag` creation:

In addition to the common words, when there is a space between a numeric value and a minus character, this also creates a flag. For example,  result `- 5` is cleaned as `5` with a flag, but the result `-5` is cleaned as `-5`, and no flag is created because we can assume it was a negative value.  


## 4. Function 2: Validate results

The `validate_lab_result()` has six arguments:

* `lab_data` : A data frame containing laboratory data

* `result_value` : The column in lab_data with quantitative result values for validation

* `result_unit` : The column in lab_data with result units in a UCUM-valid format

* `loinc_code` : The column in lab_data indicating the LOINC code of the laboratory test

* `patient_id` : The column in lab_data indicating the identifier of the tested patient.

* `lab_datetime` : The column in lab_data with the date or datetime of the laboratory test.


Let us check how our package validates the results using the `validate_lab_result()` function. Let us consider the `Function_2_dummy` data that contains 86,863 rows and inspect its first 6 rows;

```{r Function_2_dummy dataset, warning=FALSE, message=FALSE}
data("Function_2_dummy", package = "lab2clean")
head(Function_2_dummy, 6)
```


Let us apply the `validate_lab_result()` and see its functionality:
```{r apply validate_lab_result, warning=FALSE, message=FALSE}
validate_results <- validate_lab_result(Function_2_dummy, 
                                        result_value="result_value",
                                        result_unit="result_unit",
                                        loinc_code="loinc_code",
                                        patient_id = "patient_id" , 
                                        lab_datetime="lab_datetime1")

```


The `validate_lab_result()` function generates a `flag` column, with different checks:
```{r flag column creation, warning=FALSE, message=FALSE}
head(validate_results, 6)
levels(factor(validate_results$flag))
```

We can now subset specific patients to explain the flags:
```{r flag explain by subseting patients, warning=FALSE, message=FALSE}
subset_patients <- validate_results[validate_results$patient_id %in% c("14236258", "10000003", "14499007"), ]
subset_patients
```


* Patient 14236258 has both `delta_flag_8_90d` and `delta_flag_7d` that is calculated by lower and upper percentiles set to 0.0005 and 0.9995 respectively. While the delta check is effective in identifying potentially erroneous result values, we acknowledge that it may also flag clinically relevant changes. Therefore, it is crucial that users interpret these flagged results in conjunction with the patient's clinical context.

Let us also explain two tables that we used for the validation function. Let us begin with the reportable interval table.
```{r reportable_interval, warning=FALSE, message=FALSE}
data("reportable_interval", package = "lab2clean")
reportable_interval_subset <- reportable_interval[reportable_interval$interval_loinc_code == "2160-0", ]
reportable_interval_subset
```

* Patient 14499007 has a flag named `low_unreportable`. As we can see, for the "2160-0" loinc_code, his result was 0.0 which was not in the reportable range (0.0001, 120). In a similar note, patient 17726236 has a `high_unreportable`.


Logic rules ensure that related test results are consistent:
```{r logic_rules, warning=FALSE, message=FALSE}
data("logic_rules", package = "lab2clean")
logic_rules <- logic_rules[logic_rules$rule_id == 3, ]
logic_rules
```

* Patient 10000003 has both `logic_flag` and `duplicate`. The `duplicate` means that this patient has a duplicate row, whereas the `logic_flag` should be interpreted as follows. For the loinc_code "2093-3", which is cholesterol, we need that the "2093-3" > "2085-9" + "13457-7", or equivalently cholesterol > hdl cholesterol + ldl cholesterol (from the logic rules table). Therefore for patient 10000003, we have a logic flag because LDL ("13457-7") equals 100.0 and HDL ("2085-9") equals 130.0. Total cholesterol ("2093-3") equals 230. Therefore we see that the rule "2093-3" > "2085-9" + "13457-7" is not satisfied because we have 230 > 100+130, i.e. 230>230, which is clearly false, and thus a logic flag is created.

## 5. Customization

We fully acknowledge the importance of customization to accommodate diverse user needs and tailor the functions to specific datasets. To this end, the data in `logic_rules`, `reportable_interval`, and `common_words` are not hard-coded within the function scripts but are instead provided as separate data files in the "data" folder of the package. This approach allows users to benefit from the default data we have included, which reflects our best knowledge, while also providing the flexibility to append or modify the data as needed.

For example, users can easily customize the `common_words` RData file by adding phrases that are used across different languages and laboratory settings. This allows the `clean_lab_result()` function to better accommodate the specific linguistic and contextual nuances of their datasets. Similarly, users can adjust the `logic_rules` and `reportable_interval` data files for `validate_lab_result()` function to reflect the unique requirements or standards of their research or clinical environment.

By providing these customizable data files, we aim to ensure that the `lab2clean` package is not only powerful but also adaptable to the varied needs of the research and clinical communities.
