# Models

## Linear Regression

Linear regression derives the relationship between features and a label. A model is just a mathematical relation.

Example: Relation between wine age (X axis) and price (Y axis)
  * As wine age increases, price also increases
  * We could draw a best fit line across such data points and that'd be our model, though it may not be very accurate

Linear equation: y = b + mx
  * y: output variable
  * x: input variable
  * m: slope
  * b: intercept

Linear regression equation: y = b + Σw<sub>i</sub>x<sub>i</sub>
  * y: label (output)
  * b: bias of the model, also represented as w<sub>0</sub>, calculated during training
  * w<sub>i</sub>: weight of a feature
  * x<sub>i</sub>: feature (input)
