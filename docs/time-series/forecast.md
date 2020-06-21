# Forecast using R package

- Residuals should look like white noise
  - They should be uncorrelated
  - They should have mean 0

- Useful properties (for computing forecast intervals)
  - Constant Variance
  - Normally distributed

- Use `checkresiduals()` to check the residuals
  - It will do Ljung-Box test on the residuals. If it is white noise, the p-value should be more than 0.05. And, ACF is within the threshold, showing no relationship with previous values.
  - It should also have gaussian distribution. (Normal Bell-Curve)
  - If you ended up with non-nomality, and some autocorrelation in the residuals, you can probably still use the point forecast. It is the prediction/forecast intervals that might be too wide or too narrow.

## Forecast Errors

- Forecast "errors" = the difference between observed values and its forecast in the test set

- Not equal to residuals
  - Residuals are errors on the training set (vs test set), while forecast errors are on test set.
  - Residuals are based on one-step forecast (vs multiple steps), while forecast errors are based on multiple steps.

- Measure of forecast accuracy
    - MAE - Mean Absolute Error
    - MSE - Mean Square Error
    - MAPE - Mean Absolute Percentage Error
    - MASE - Mean Absolute Scaled Error