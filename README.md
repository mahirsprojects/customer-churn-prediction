# Telco Customer Churn Classifier

## The Business Question

Every month a phone company loses customers to cancellation. The cost of 
acquiring a new customer is significantly higher than retaining an existing one. 
The challenge is that you have thousands of customers and no way to know in 
advance who is about to leave.

This project builds a model that looks at each customer's profile and predicts 
whether they are likely to churn, so a retention team can intervene before it 
is too late.

---

## What This Project Does

1. Cleans and filters the raw dataset using SQL, removing incomplete records 
and computing a derived average monthly spend feature
2. Asks ChatGPT to rank which features it thinks best predict churn before 
any model is built, saving that as a baseline prediction
3. Encodes all customer attributes into numeric form so a decision tree can 
process them
4. Splits the data into training and testing sets, trains a decision tree 
classifier, and evaluates it on customers the model has never seen
5. Compares actual feature importance from the model against ChatGPT's 
predicted ranking and identifies where the data contradicted AI intuition

---

## Why a Decision Tree

A decision tree mirrors how a human would actually think through churn risk. 
It asks a series of yes or no questions about each customer and follows a path 
to a prediction. That makes it interpretable in a way that matters for a 
business context, a manager can follow the logic without understanding 
statistics.

It also produces feature importance scores that show exactly which customer 
attributes drove the predictions, which connects directly to actionable 
retention strategy.

---

## Why Recall Matters More Than Accuracy

The dataset has 73.5% non churners and 26.5% churners. A model that predicts 
No for every single customer would score 73.5% accuracy without learning 
anything useful. Accuracy is the wrong metric here.

The metric that matters is recall on the churn class, meaning out of all 
customers who actually churned, how many did the model catch.

A false negative means the model predicted a customer would stay but they 
left. You did nothing and lost them. A false positive means the model flagged 
a loyal customer as a churn risk and you offered them a retention discount 
they did not need. That costs a small amount but keeps them.

Missing a churner is more expensive than over retaining a loyal customer. 
That is why recall is the metric this model optimises for.

---

## The AI Integration

Before building the model, the full feature list was passed to ChatGPT with 
the following prompt:

*"I am building a decision tree to predict customer churn for a telco company. 
Rank these features from most to least predictive of churn and explain your 
reasoning for the top 5: gender, SeniorCitizen, Partner, Dependents, tenure, 
PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, 
DeviceProtection, TechSupport, StreamingTV, StreamingMovies, Contract, 
PaperlessBilling, PaymentMethod, MonthlyCharges, TotalCharges."*

ChatGPT's ranking is saved in `ai_inputs/chatgpt_feature_ranking.rtf`.

The model's actual feature importance is compared against that ranking in the 
findings below.

---

## Tech Stack

- Python (pandas, scikit-learn, matplotlib, seaborn)
- SQLite via DBeaver
- Decision Tree Classifier
- Dataset: IBM Telco Customer Churn via Kaggle
----

## Model Performance

The model was evaluated on 1409 customers it had never seen during training. ChatGPT predicted right and confirms these findings.

- Overall accuracy: 78%
- Churn recall: 61% (the model caught 230 out of 374 actual churners)
- Churn precision: 60% (of all customers flagged as churners, 60% actually churned)
- False negatives: 144 churners the model missed and made no intervention for
- False positives: 152 loyal customers wrongly flagged for retention outreach

---

## Key Findings

**Contract type is the single strongest predictor of churn by a wide margin.**
Month to month customers churn at a significantly higher rate than customers 
on one or two year contracts. The model assigned this feature an importance 
score of 0.52, more than all other features combined.

**This is where the model diverged from AI intuition.** ChatGPT ranked 
contract type highly but did not predict it would outweigh financial features 
like MonthlyCharges and TotalCharges by this magnitude. Most people assume 
price is the primary driver of cancellation. The data says commitment level 
matters more.

**Fiber optic internet is the second strongest signal.** These customers have 
more alternatives available and higher service expectations. When something 
goes wrong they leave faster than DSL users.

**Tenure and TotalCharges tell the same story from two angles.** Long term 
customers have paid more in total and churn far less. New customers, especially 
those on month to month contracts, are the highest risk segment.

---

## Business Recommendation

Focus retention efforts on three customer segments in this order:

Month to month contract customers in their first 12 months. This is the 
highest risk combination in the dataset. No long term commitment plus low 
tenure means very little switching cost. A proactive offer to move them to 
an annual contract, even at a discounted rate, directly addresses the strongest 
churn signal the model found.

Fiber optic customers showing high monthly charges with no tech support.
These customers are paying premium prices without a support safety net. When 
something goes wrong they have no reason to stay. Adding tech support to their 
plan or proactively checking in on service quality reduces their exit 
motivation.

New customers in the first three months regardless of contract type. Tenure 
is one of the strongest protective factors against churn. Customers who make 
it past the first few months churn at significantly lower rates. An onboarding 
programme that drives early engagement and product adoption addresses churn 
before it becomes likely.

The model's 61% recall means roughly 4 in 10 churners still go undetected. 
Combining model predictions with these three behavioural segments as a manual 
overlay would improve coverage beyond what the model achieves alone.

---

## What I Learned

The train test split is what makes model evaluation honest. The model never 
sees the test answers during training. You reveal them only at the end to 
grade performance on genuinely unseen data.

The more interesting lesson is about metrics. A model that ignores churn 
entirely would score 73% accuracy. That number sounds good and means nothing. 
Precision and recall force you to look at what the model actually gets right 
and wrong for each class separately, which is the only evaluation that 
connects to real business cost.
