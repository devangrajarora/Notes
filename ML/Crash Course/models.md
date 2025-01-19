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

#### Types of Loss
1. L<sub>1</sub> loss
    * Sum of absolute differences
    * Σ |actual - predicted|
2.  Mean absolute error (MAE)
    * Mean of L<sub>1</sub> loss across data points
    * L<sub>1</sub> loss / N, or (1 / N) * Σ |actual - predicted|
3. L<sub>2</sub> loss
    * Sum of squares of absolute differences
    * Σ |actual - predicted|<sup>2</sup>
4. Mean squared error (MSE)
    * Mean of L<sub>2</sub> loss across data points
    * L<sub>2</sub> loss / N
  
Squaring in L<sub>2</sub> and MSE, causes larger losses (>1) to become even bigger and smaller losses (<1) to become even smaller.\
A model trained to minimise MSE moves closer to outliers, since presence of outliers is penalized more with L<sub>2</sub> than L<sub>1</sub>.

### Gradient descent
A model training technique to iteratively find the bias and weights to minimise loss.\
Lower loss ~ better predictions\
Gradient descent literally means change in downward direction, i.e. iterative decrement in loss value.

Steps:
  * Initialize bias = 0, weights = 0
  * Compute loss value based on decided loss function
  * Determine direction to move for bias and each weight
  * Adjust bias and weight, repeat step 0

**Model convergence:** State where model has reached it's minimal loss. Any further iterations of adjusting weights and bias will have little to no reduction in loss.

### Hyperparameters
Hyperparameters are parameters which are used to control the process of model training. Example:
  * Batch size
  * Learning rate
  * Epoch

Hyperparameters control the training process. Parameters (feature weights, bias) are computed during training.
