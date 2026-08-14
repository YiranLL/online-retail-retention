# online-retail-retention
# Predicting 90-Day Repeat Purchasing

## Business question

Given information available at the completion of a customer's first valid order, estimate the probability that the customer places another valid order within 90 days.

## Unit of analysis

One eligible customer.

## Prediction time

The completion of the customer's first valid order.

## Target

`repeat_within_90d = 1` if the customer places a second valid order within 90 days of the first order, and `0` otherwise.

## Feature constraint

Features may only use information available by the completion of the first valid order.

## Intended decision

Use predicted probabilities to stratify customers for a future retention experiment under limited marketing capacity.

## Important limitation

This is a predictive analysis. It does not identify which customers would change their behaviour because of a marketing intervention.

# Data

Dataset: UCI Online Retail II

The raw dataset is not stored in this repository.

Place the downloaded Excel file in this directory before running the notebooks.