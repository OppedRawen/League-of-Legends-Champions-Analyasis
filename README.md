# Can Farming Offset a Kill Deficit in Professional League of Legends?

By David Yu

## Introduction

Write your dataset introduction and your main question here.

## Data Cleaning and Exploratory Data Analysis

Explain how you cleaned the data.

Show a cleaned DataFrame preview.

Embed your first plot.

<iframe
 src="assets/state_winrate.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

## Assessment of Missingness

Explain NMAR and your missingness permutation tests.

## Hypothesis Testing

Explain your hypotheses, test statistic, p-value, and conclusion.

<iframe
 src="assets/permutation_test.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

| state15                  |   win_rate |   count |   avg_killdiff |   avg_csdiff |   avg_golddiff |
|:-------------------------|-----------:|--------:|---------------:|-------------:|---------------:|
| Up kills, up CS          |   0.820548 |    5110 |        3.5229  |      33.8532 |       2694.3   |
| Up kills, even/down CS   |   0.509018 |    2994 |        2.63794 |     -23.2418 |        634.597 |
| Even kills or other      |   0.5      |    2234 |        0       |       0      |          0     |
| Down kills, up CS        |   0.495346 |    2901 |       -2.63668 |      23.9869 |       -609.075 |
| Down kills, even/down CS |   0.182587 |    5203 |       -3.50778 |     -33.2481 |      -2671.71  |

## Framing a Prediction Problem

Explain what you are predicting, what type of prediction problem it is, what the response variable is, and what information is available at prediction time.

## Baseline Model

Explain your baseline model, features, encodings, metric, and performance.

## Final Model

Explain your added features, model choice, hyperparameter tuning, and performance improvement.

## Fairness Analysis

Explain your fairness groups, metric, hypotheses, p-value, and conclusion.