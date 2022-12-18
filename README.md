# SmileGate-Synthetic-Labelling
# Synthetic Labelling for Anomalous Records Using Isolation Forest
## < Nicole An > 
## Overview

The goal of this project is to use Isolation Forest as a surrogate labelling strategy and evaluate the classification model using synthetic labels with the baseline model to examine if this approach brings any improvement. There are 2 models built, one trained on synthetic labels and one trained actual labels. Both models will be evaluate on multiple metrics that showcase their abilities in predicting for fraudulent transactions.  


## Methodology

A.	Data partitioning
a.	The dataset is splitted randomly by 70/30 where 70% were training data and 30% were testing data.
B.	Data preprocessing
a.	Transform transaction_timestamp to numerical variable which represents the days since transaction
b.	Generate synthetic labels using IsoFor with specification
i.	Hyperparameters: sample_size=2048; num_trees=100; max_depth=12
ii.	Model: ~ registration_deposit +  mean_deposit + mean_txn + monetary_returns_5day + monetary_returns_15day + monetary_returns_30day + game_cash_count_3day + distinct_account_3day + days_since_last_pmnt
iii.	Threshold: anomaly_score = 0.6085; avg_tree_depth = ___
C.	Model specification  
a.	Random Forest Model with synthetic labelling
i.	Hyperparamter: tree = 200; min_n = 10
ii.	Features: synthetic_target ~ registration_deposit +  mean_deposit + mean_txn + monetary_returns_5day + monetary_returns_15day + monetary_returns_30day + game_cash_count_3day + distinct_account_3day + days_since_last_pmnt
b.	Random Forest Model with actual event labelling
i.	Hyperparameter: trees = 10; min_n = 5; mtry = 5
ii.	Features: event_label ~ registration_deposit +  mean_deposit + mean_txn + monetary_returns_5day + monetary_returns_15day + monetary_returns_30day + game_cash_count_3day + distinct_account_3day + days_since_last_pmnt


## The Overall Approach

Using the Isolation Forest model with the above specification, the model create new labelling data with “fraud”/“legit” based on each training sample’s anomaly score. The next step is to inject the anomalies back to the data with a new column. The operational rule is to perform at 10% TPR, so a threshold of 0.6085 is developed to further create synthetic labels. This means that if the transaction has a 60.85% chance of being a fraud or above, the isolation forest will detect it as fraudulent. By operating at 10% TPR, the system catches at least 10% of frauds. 
After synthetic labels are created, a classification model is built with the above specification and recipe. The Random Forest model using synthetic labelling predicts both the synthetic labels and the actual labels, and the performance is shown below. It showcases how well the surrogate model performs on predicting synthetic labels and how accurate it is at predicting actual labels. 
A baseline model is constructed as a benchmark. The Random Forest model using actual event label will be compared to the surrogate model to see how the surrogate model performs against the standard. 

## Overview of Performance 

Random Forest Model using synthetic labels as target (Surrogate Model)
i)	Precision and recall
•	Predicting Synthetic Labels
.metric
<chr>	.estimator
<chr>	.estimate
<dbl>	part
<chr>	
precision	binary	0.9956190	training	
precision	binary	0.8606171	testing	

.metric
<chr>	.estimator
<chr>	.estimate
<dbl>	part
<chr>	
recall	binary	0.9668166	training	
recall	binary	0.6882438	testing	
The surrogate model performs very well, operating at 0.86 precision and 0.69 recall. That is, out of 1000 fraudulent transactions predicted, 860 of them are correct. It also detects 690 fraudulent transactions from 1000 true synthetically created fraud labels. 
•	Predicting Event Labels
.metric
<chr>	.estimator
<chr>	.estimate
<dbl>	part
<chr>	
precision	binary	0.1376893	training	
precision	binary	0.1422868	testing	

.metric
<chr>	.estimator
<chr>	.estimate
<dbl>	part
<chr>	
recall	binary	0.10658915	training	
recall	binary	0.08767614	testing	

The surrogate model does not do a good job at predicting the actual event labels. Out of 1000 true frauds, the model only correctly identify 88 of them. Only 142 of 1000 fraudulent transactions predicted are actual frauds. 
ii)	Other Evaluation Metrics
•	Predicting Synthetic Labels
.estimator
<chr>	part
<chr>	accuracy
<dbl>	kap
<dbl>	mn_log_loss
<dbl>	roc_auc
<dbl>
binary	training	0.99956	0.9807839	0.005492538	0.9999943
binary	testing	0.99514	0.7624141	0.012538956	0.9978527

•	Predicting Event Labels
.estimator
<chr>	part
<chr>	accuracy
<dbl>	kap
<dbl>	mn_log_loss
<dbl>	roc_auc
<dbl>
binary	training	0.9769871	0.10869202	0.3908580	0.6352585
binary	testing	0.9785267	0.09824946	0.3842525	0.6384003

The surrogate model has a 0.638 AUC, which is decent for a predictive model. 
 
Random Forest Model using actual labelling as target (Baseline Model)
i)	Precision & Recall
.metric
<chr>	.estimator
<chr>	.estimate
<dbl>	part
<chr>	
precision	binary	0.9973345	training	
precision	binary	0.9847393	testing	

.metric
<chr>	.estimator
<chr>	.estimate
<dbl>	part
<chr>	
recall	binary	0.6526163	training	
recall	binary	0.5195706	testing	
The baseline model has a very high precision rate at 98.5%, meaning that 98.5% of the fraud predictions are actual frauds. However, its recall is not as great. That is, the baseline model only catches 52.0% of frauds, creating a large gap for fraudulent transactions on Smile Gate.
ii)	Other Evaluation Metrics

.estimator
<chr>	part
<chr>	accuracy
<dbl>	kap
<dbl>	mn_log_loss
<dbl>	roc_auc
<dbl>
binary	training	0.9948529	0.7864748	0.01182377	0.9999594
binary	testing	0.9927200	0.6769081	0.15292807	0.8272963
The testing AUC for baseline model shows that the model is overfitting with the amount of numerical variables in the model. 

## Local Interpretation on Top 10 True Positives

•	Surrogate Model
  
           
Since the synthetic labelling RF model does not produce very accurate predictions, it is important to review the top 10 True Positive fraud predictions and the variables that make high impacts. The important variables in this model tend to all make negative impact on the prediction, meaning as these variables increase, there is less chance of the sample being fraudulent. The synthetic labelling RF is very good at distinguishing between different distinct_account_3day values, and their anomalous records are significant. 

•	Random Forest Baseline Model
  
 
          

The top 10 true positive fraud detections in the baseline model are shown above. Distinct_account_3day continue to be a significant influencer along with mean transaction amount. Compared to the surrogate model, variables in this model contribute to the prediction less evenly with most variables contribute less than 1% and a few large influencers. Game cash, the most important variable globally, positively contribute to fraud detection, meaning the more time spent in the game, the more likely there will be fraudulent transactions made by this account. 

## Summary & Conclusion

The RF model with synthetic label is performing very well in finding synthetic labels. It has a high AUC of 0.998 with a precision of 86.1%. For unsupervised learning, the IsoFor model is evaluated to be very accurate in predicting anomalous records. On the other hand, with real life data, however, it does not perform greatly evaluating against the event label. It has a low precision and recall rate (both ~10%). The synthetic labels operate at a threshold of 0.6085, which is already tuned to have the best tradeoff between precision and recall where both are at each other’s highest. Therefore, the isolation forest has limited powers in producing robust prediction on actual events. 
In real life where we do not have labels on transactions or the fraud cases are too little, this synthetic approach using IsoFor may come in handy. It has an AUC of 0.64, which is a decent score compared to the baseline of 0.82. Without the actual event label, we can still build a decent model which produce some insights to the data with this ensemble unsupervised machine learning model. To dive deeper, we can conclude that the number of features in the model should be limited to prevent overfitting because isolation forest may be very dependable on other features. If there are irrelevant features to the actual labels, IsoFor is not as robust to sensitive data and will be influence by noises. 


