# Intro to ML
Machine Learning is the process of training a software (i.e. a **model**) on **data** to make _meaningful predictions_ and _generate new content_.
* Make meaningful predictions:
	* Translate a sentence to a different language
	* Suggest next songs in a playlist
	* Predict the weather over the next 10 days
* Generate new content:
  * Text to image/video etc
  * Summarize a text
  * Write story based on prompt

How to use ML to predict weather data?
1. Provide the ML model a large amount of weather data
2. Model _learns_ the relationship between weather patterns and amount of rain produced
3. Model can _predict_ the amount of rain produced for given weather conditions

**Model**: A model is a <ins>mathematical relationship</ins> derived from data which an ML system uses to make predictions.

## Types of ML systems
Based on techniques used for prediction and content generation:
1. Supervised learning
2. Unsupervised learning
3. Reinforcement learning
4. Generative AI

### Supervised Learning
**Input**:
* Categorized/labeled data.
* The model is provided with a large amount of data with correct answers. 

**Goal**:
* Associate data elements/features to correct answers.
* With these correlations and patterns, the model predicts answers of future inputs.
  
Analogy:
  * Similar to a student studying old exam papers with answers to prepare for new questions.
  * Looking are previous quarter returns to predict next quarter returns

Common use cases
  1. Regression: Predicting numerical values based on historical data across multiple applicable factors
	2. Classification: Categorizing inputs into one or more categories
  	* Binary or Multiclass classification
  	* Examples: Spam detection, classifying if an image is of a man/woman

### Unsupervised Learning
**Input**:
* Unlabeled data. Model is given no hint as to what category a data point belongs to.
* Data does not contain any correct answers associated with it.

**Goal**:
* Based on patterns/features, identify clusters within the data.
	* Data within a cluster has a higher degree of similarity
  * Data in different clusters have a lower degree of similarity
* Categorize incoming data into one of the clusters.
* The model won't name any of the clusters, though we can decide to do so

## References
1. https://developers.google.com/machine-learning/intro-to-ml
