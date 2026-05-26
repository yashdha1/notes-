# Linear Regression 
1. Simple linear regressions -> one independent variable    y = mx + b 
2. Muliple linear regression -> multi independent variable   

#### Evaluation metrics for the LR : 
- [****Mean Squared Error (MSE)****](https://www.geeksforgeeks.org/python/python-mean-squared-error/)****:**** Measures the average squared difference between actual and predicted values to avoid cancellation of errors.
- [****Mean Absolute Error (MAE):****](https://www.geeksforgeeks.org/python/how-to-calculate-mean-absolute-error-in-python/) Calculate the accuracy of a regression model. MAE measures the average absolute difference between the predicted values and actual values.
- [****Root Mean Squared Error (RMSE)****](https://www.geeksforgeeks.org/r-language/root-mean-square-error-in-r-programming/)****:**** Square root of the residuals variance is RMSE. It describes how well the observed data points match the expected values or the model's absolute fit to the data.
- [****R-Squared****](https://www.geeksforgeeks.org/maths/r-squared/)****:**** Indicates how much variation the model explains. Its value is typically between 0 and 1, but it can be negative if the model performs worse than a simple baseline model (e.g., predicting the mean).
- [****Adjusted R-square****](https://www.geeksforgeeks.org/machine-learning/ml-adjusted-r-square-in-regression-analysis/)****:**** Measures the proportion of variance explained by the model while adjusting for the number of predictors and penalizing irrelevant features.

#### Regularization Techniques 
adding a penalty to large coefficients (weights) in Linear Regression to keep the model simple and stable.
1. ridge regularisation 
2. lasso regressions 
3. Elastic Net 

### Polynomial Regression 
	using the non-linear features using the Relu and other non linear feat layers. 

--- 
# Classification 
KNN, SVM , Decision Trees, Logistic Regression   