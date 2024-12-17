# IvfSuccessCalculator

Build the CDC IVF Success Calculator found here: https://www.cdc.gov/art/ivf-success-estimator/index.html

Calculating a patient's IVF success rate is an integral part of how we calculate the patient's [customized refund](https://joinsunfish.com/success_program/). This is an opportunity to showcase your software engineering skills while implementing a real project at Sunfish.

## Purpose

The purpose of this exercise is to build a web server in the programming language of your choice that:

1. Parses the provided `ivf_success_formulas.csv`
2. Provides an HTTP endpoint which receives inputs to the formula
3. Selects the correct formula to calculate the IVF success rate based on those inputs
4. Performs the IVF success calculation based on the inputs
5. Responds with the IVF success rate back to the user

The final implementation should produce the same results as the CDC's IVF Success Calculator linked above.

## `ivf_success_formulas.csv`

The `ivf_success_formulas.csv` file contains 6 formulas for calculating the IVF success rate of a particular patient. The first three columns are parameters for choosing the correct formula, and their names are prefixed with `param_`.

The fourth column indicates which formula to choose among the [CDC Success Estimator formulas](https://www.cdc.gov/art/ivf-success-estimator/formulas_new.html). The values in the CSV are taken directly from the CDC's own formula tables; you are free to use the above page as a reference if you find it easier to read.

The remaining 28 columns are used to perform the calculation, and their names are prefixed with `formula_`.

The next section explains what the inputs are, and the following section explains the calculations.

## User Input

### Formula Selection Parameters

The user needs to provide the following parameters in order to select the correct formula:

- Are they planning to use their own eggs in the IVF process?
  - TRUE: The patient is using their own eggs to create embryos.
  - FALSE: The patient will be leveraging a donor's eggs to create embryos.
- Have they used IVF in the past? (TRUE, FALSE, or N/A)
  - If N/A, the answer to this question is not relevant in the formula selection. This is only N/A when the previous question is FALSE.
- Do they know the reason for their own infertility? (TRUE or FALSE)

**Note:** For testing against the actual CDC IVF Success Calculator: in the project, "Have you attempted IVF previously?" answer is either TRUE or FALSE, but on the CDC site, you can choose one of 0, 1, 2, or 3+. For testing your code against this form, choose "2" on the CDC site.

### Formula Calculation Parameters

The user needs to provide the following parameters for the formula calculation:

- Age (20-50)
- BMI, calculated from:
  - Weight (80 - 300 lbs)
  - Height, in feet and inches. For example, 5'8".
- Various possible reasons for inferility:
  - Tubal Factor (TRUE or FALSE)
  - Male Factor Infertility (TRUE or FALSE)
  - Endometriosis (TRUE or FALSE)
  - Ovulatory Disorder (TRUE or FALSE)
  - Diminished Ovarian Reserve (TRUE or FALSE)
  - Uterine Factor (TRUE or FALSE)
  - Other Reason (TRUE or FALSE)
- Or, the reason for infertility is unexplained (TRUE or FALSE)
  - This is equivalent to "none of the above."
  - _NOTE_: This is **not** the same as "not knowing the reason for their infertility" in the previous section. In the above case, the patient has not yet seen a medical professional regarding infertility issues. In this case, they have seen a medical professional, and the medical professional is unable to explain the cause of their infertility.
- Number of Prior Pregnancies (0 or more)
  - _Note:_ In the [IVF Success Estimator formulas](https://www.cdc.gov/art/ivf-success-estimator/formulas_new.html), this parameter is called "Gravida"
- Number of Live Births (0 or more)
  - The number of live births cannot exceed the number of prior pregnancies (even in the case of twins)

### Receiving the input

You have a few different choices for receiving the input:

1. Via a POST request containing an HTML form
2. As URL query parameters
3. Contained within a document in the format of your choice (JSON, XML, gRPC / Protobuf, ...)

If using (1), please include the HTML page containing the form. If using (2) or (3), please provide sample inputs to demonstrate usage.

## Formula Calculations

### BMI

BMI is calculated as `user_weight_in_lbs / (user_height_in_feet ✕ 12 + user_height_in_inches)² ✕ 703`.

#### Example

A 5'8" person weighing 150 lbs would have a BMI of approximately 22.8:

- `user_weight_in_lbs` = 150
- `user_height_in_ft` = 5
- `user_height_in_inches` = 8

5 ✕ 12 + 8 = 68\
68² = 4624\
150 / 4624 ✕ 703 = 22.8

### IVF Success Formula

The IVF success formula is calculated as a score based on the questions in the above form. The score is converted to a success rate by calculating eˢᶜᵒʳᵉ / (1 + eˢᶜᵒʳᵉ).

The following sections explain each individual piece of the calculation.

#### Intercept

The `formula_intercept` is a constant added to the rest of the formula.

#### Age

Age is calculated with a linear component as well as a polynomial component:

`formula_age_linear_component` ✕ `user_age` + `formula_age_power_coefficient` ✕ (`user_age` ^ `formula_age_power_factor`)

### BMI

BMI is calculated with a linear component as well as a polynomial component:

`formula_bmi_linear_coefficient` ✕ `user_bmi` + `formula_bmi_power_coefficient` ✕ (`user_bmi` ^ `formula_bmi_power_factor`)

### Infertility Reasons

Each infertility reason (tubal factor, male factor infertility, endometriosis, ovulatory disorder, diminished ovarian reserve, uterine factor, other reason, and unexplained infertility) has two separate values: one if the infertility reason is TRUE and one if the infertility reason is FALSE. In the [CDC IVF Success Estimator formulas](https://www.cdc.gov/art/ivf-success-estimator/formulas_new.html), these are indicated as "Yes" or "No" respectively.

If the infertility reason is TRUE, use the `formula_*_true_value`. If FALSE, use the `formula_*_false_value`.

### Prior Pregnancies

_Note:_ In the [IVF Success Estimator formulas](https://www.cdc.gov/art/ivf-success-estimator/formulas_new.html), this parameter is called "Gravida"

The formula only has three values:

- A value if the user has not had any prior pregnancies (`formula_prior_pregnancies_0_value`)
- A value if the user had exactly one prior pregnancy (`formula_prior_pregnancies_1_value`)
- A value if the user had two or more prior pregnancies (`formula_prior_pregnancies_2+_value`)

You are free to either use radio buttons (or an enumeration field) for these three choices, or a text field asking the user to enter in a number.

In the latter case, if the user selects either 0 or 1 prior pregnancies, use the corresponding value. If the user selects a number greater than or equal to two, use the third value.

### Live Births

Similar to the Prior Pregnancies section above, while the user can input any number greater than zero, the formula only has three values:

- A value if the user has not had any prior pregnancies (`formula_prior_live_births_0_value`)
- A value if the user had exactly one prior pregnancy (`formula_prior_live_births_1_value`)
- A value if the user had two or more prior pregnancies (`formula_prior_live_births_2+_value`)

Again, you may either use a radio button / enumeration field or a text input.

In the latter case, if the user selects either 0 or 1 live births, use the corresponding value. If the user selects a number greater than or equal to two, use the third value.

### Final Calculation

The overall formula is:

`score` =\
 `formula_intercept` +\
`formula_age_linear_component` ✕ `user_age` + `formula_age_power_coefficient` ✕ (`user_age` ^ `formula_age_power_factor`) +\
`formula_bmi_linear_coefficient` ✕ `user_bmi` + `formula_bmi_power_coefficient` ✕ (`user_bmi` ^ `formula_bmi_power_factor`) +\
`formula_tubal_factor_value` +\
`formula_male_factor_infertility_value` +\
`formula_endometriosis_value` +\
`formula_ovulator_disorder_value` +\
`formula_diminished_ovarian_reserve_value` +\
`formula_uterine_factor_value` +\
`formula_other_reason_value` +\
`formula_unexplained_infertility_value` +\
`formula_prior_pregnancies_value` +\
`formula_live_births_value`

#### Example: Using Own Eggs / Did Not Previously Attempt IVF / Known Infertility Reason

Consider a request with the following inputs:

- Using Own Eggs: TRUE
- Previously Attempted IVF: FALSE
- Reason for Infertility Known: TRUE
- Age: 32
- Height: 5'8"
- Weight: 150 lbs
- Tubal Factor: FALSE
- Male Factor Infertility: FALSE
- Endometriosis: TRUE
- Ovulatory Disorder: TRUE
- Diminished Ovarian Reserve: FALSE
- Uterine Factor: FALSE
- Other Infertilty Reason: FALSE
- Unexplained Infertility: FALSE
- Prior Pregnancies: 1
- Prior Live Births: 1

From the answers to first three questions, select [CDC formula "1-3"](https://www.cdc.gov/art/ivf-success-estimator/formulas_new.html) (on the second row in the CSV). The calculation is then:

`score` =\
 -6.8392144 (intercept)\
 \+ 0.3347309 ✕ 32 + -0.0003249 ✕ 32²·⁷⁶³³¹³ (age)\
 \+ 0.06997997 ✕ 22.8 + -0.0015045 ✕ 22.8² (BMI)\
 \+ 0 (tubal factor)\
 \+ 0 (male factor infertility)\
 \+ 0.02773216 (endometriosis)\
 \+ 0.27949598 (ovulatory disorder)\
 \+ 0 (diminished ovarian reserve)\
 \+ 0 (uterine factor)\
 \+ 0 (other infertility reason)\
 \+ 0 (unexplained infertility)\
 \+ 0.03514055 (prior pregnancies)\
 \+ 0.15787934 (prior live births)\
= 0.498270

`success_rate` = e⁰·⁴⁹⁸²⁷⁰ / (1 + e⁰·⁴⁹⁸²⁷⁰) = 1.64587 / 2.64587 = 62.21%

You can confirm this by going to the [IVF Success Calculator](https://www.cdc.gov/art/ivf-success-estimator/index.html) and see if you get the same results for the same inputs.

#### Example: Using Own Eggs / Did Not Previously Attempt IVF / Unknown Infertility Reason

- Using Own Eggs: TRUE
- Previously Attempted IVF: FALSE
- Reason for Infertility Known: FALSE
- Age: 32
- Height: 5'8"
- Weight: 150 lbs
- Tubal Factor: FALSE
- Male Factor Infertility: FALSE
- Endometriosis: FALSE
- Ovulatory Disorder: FALSE
- Diminished Ovarian Reserve: FALSE
- Uterine Factor: FALSE
- Other Infertilty Reason: FALSE
- Unexplained Infertility: FALSE
- Prior Pregnancies: 1
- Prior Live Births: 1

From the answers to first three questions, select [CDC formula "4-6"](https://www.cdc.gov/art/ivf-success-estimator/formulas_new.html) (on the third row in the CSV). The calculation is then:

`score` =\
-7.5545223 (intercept)\
\+ 0.37931798 ✕ 32 + -0.0003752 ✕ 32²·⁷⁶³³¹³ (age)\
\+ 0.08057661 ✕ 22.8 + -0.0015304 ✕ 22.8² (BMI)\
\+ 0 (tubal factor)\
\+ 0 (male factor infertilty)\
\+ 0 (endometriosis)\
\+ 0 (ovulatory disorder)\
\+ 0 (diminished ovarian reserve)\
\+ 0 (uterine factor)\
\+ 0 (other infertility reason)\
\+ 0 (unexplained infertility reason)\
\+ 0.02240271 (prior pregnancies)\
\+ 0.16421628 (prior live births)\
= 0.398540

`success_rate` = e⁰·³⁹⁸⁵⁴⁰ / (1 + e⁰·³⁹⁸⁵⁴⁰) = 59.83%

#### Example: Using Own Eggs / Previously Attempted IVF / Known Infertility Reason

- Using Own Eggs: TRUE
- Previously Attempted IVF: TRUE
- Reason for Infertility Known: TRUE
- Age: 32
- Height: 5'8"
- Weight: 150 lbs
- Tubal Factor: TRUE
- Male Factor Infertility: FALSE
- Endometriosis: FALSE
- Ovulatory Disorder: FALSE
- Diminished Ovarian Reserve: TRUE
- Uterine Factor: FALSE
- Other Infertilty Reason: FALSE
- Unexplained Infertility: FALSE
- Prior Pregnancies: 1
- Prior Live Births: 1

From the answers to first three questions, select [CDC formula "7-8"](https://www.cdc.gov/art/ivf-success-estimator/formulas_new.html) (on the fourth row in the CSV). The calculation is then:

`score` =\
-8.102508 (intercept)
\+ 0.37506646 ✕ 32 + -0.0003171 ✕ 32²·⁷⁸⁴⁶¹⁹ (age)
\+ 0.04565965 ✕ 22.8 + -0.0008793 ✕ 22.8² (BMI)
\+ 0.06858044 (tubal factor)
\+ 0 (male factor inferility)
\+ 0 (endometriosis)
\+ 0 (ovulatory disorder)
\+ -0.4806452 (diminished ovarian reserve)
\+ 0 (uterine factor)
\+ 0 (other inferility reason)
\+ 0 (unexplained infertility)
\+ 0.15884291
\+ 0.32698183
= -0.368348

`success_rate` = e⁻⁰·³⁶⁸³⁴⁸ / (1 + e⁻⁰·³⁶⁸³⁴⁸) = 40.89%

**Note**: If comparing to the [CDC IVF Success Calculator](https://www.cdc.gov/art/ivf-success-estimator/index.html), use "2" for the number of previous IVF cycles question.

## Response

The response can come in the form of an HTML page, plain text, or some other user-understandable format.

## Submission

Please create a public repository on GitHub (or your favorite Git hosting platform) and email the link to mpigott@joinsunfish.com. The README should include how to run your code and instructions for how to create a request for calculating an IVF success rate, sending that request, and receiving the response.
