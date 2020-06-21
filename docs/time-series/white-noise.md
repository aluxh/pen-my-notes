# White Noise

- White noise is a time-series that is purely random. (No trends or patterns found)
- Test for white noise can be carried out using Ljung-Box test or ACF plot
  - Ljung-Box test can perform group test based on the first h autocorrelation values together.
  - An example, `Box.test(pigs, lag=24, fitdf=0, type='Lj')` using `R`.
  - If the result for the test has small and significant p-values, it indicates that the data are probably not white noise. i.e. result for the above, is `p-value < 2.2e-16`.