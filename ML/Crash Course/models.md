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
  * b: bias of the model, also represented as w<sub>0</sub>
  * w<sub>i</sub>: weight of a feature
  * x<sub>i</sub>: feature (input)
  * The bias and weights of linear regression equation get updated during training.

### Loss
Measure of how wrong the predicted label is when compared to the actual label. The training process aims to minimise loss.\
Direction of difference does not matter when calculating loss. Both these conditions have a positive loss value:
 * _predicted_value_ < _actual_value_
 * _predicted_value_ > _actual_value_
