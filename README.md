# Churn Analysis and Retention ROI from Real Spotify Customer Reviews

Python, SQL and machine learning project that uses **2,000 real public customer reviews of Spotify** to find why unhappy customers talk about leaving, predict who shows a churn signal, estimate the revenue at risk and test whether retention offers would pay off.

> Not affiliated with Spotify. Spotify is a trademark of its owner. The reviews are public online reviews that I collected myself.

## Question

Why do customers talk about leaving a streaming service, and if the company could reach those customers with a retention offer, would it be worth the cost?

## Data

- **2,000 public customer reviews**, Sep 2021 to Nov 2026, collected with a browser data-scraper extension.
- Fields: star rating, title, review text, review date, experience date, country.
- Reviewer names, profile links and avatars were removed. `Churn_analysis_Spotify_public.csv` contains no personal data or links.
- Reviews have **no billing, usage or cancellation records**, so this project measures a **churn signal**, not Spotify's real churn rate.

**Churn signal** = an unhappy reviewer (1 or 2 stars) who also talks about cancelling, refunds, being charged, switching or leaving.

## Method

1. **Clean** the scraped data with pandas: rename columns, parse dates, handle missing text, remove duplicates, drop personal data.
2. **Engineer features**: churn signal, seven complaint themes (billing, cancellation, support, account, ads, bugs, algorithm), a 0 to 100 churn score.
3. **Analyse** with KPIs, pivot tables, SQL (SQLite) and Matplotlib / Seaborn charts.
4. **Predict** the churn signal with TF-IDF and logistic regression, using 5-fold cross-validation and calibrated probabilities.
5. **Explain** each prediction with the words that drive it.
6. **Price the risk**: revenue at risk, retention offer per complaint theme, expected ROI, and a sensitivity analysis.

## Key findings

### What the reviews say

- **The sample is very negative.** The average rating is 1.73 out of 5 and 81% of reviews are 1 or 2 stars. Review sites attract unhappy customers, so these numbers describe complaining reviewers, not all customers.
- **26.3% of reviews (526) carry a churn signal.**
- **Money and cancellation drive exits.** Billing is mentioned in 69% of churn-signal reviews (35% of all reviews) and cancellation in 52% of churn-signal reviews (14% overall). A typical story is being charged after a free trial or after cancelling.
- **Support and account problems make it worse.** Support appears in 43% of churn-signal reviews and account problems in 37%.
- **Ads and playlist complaints are common but are not tied to leaving.** They appear less often in churn-signal reviews (13% and 12%) than across all reviews (18% and 19%).
- **By country (15+ reviews):** Canada (39%) and the UK (33%) show the highest churn-signal rates, the US is 24%.

### The prediction model

- **AUC 0.80** on held-out folds. AUC is the chance that the model gives a randomly picked churn-signal review a higher risk score than a randomly picked normal review (0.5 = coin flip, 1.0 = perfect). Average precision is 0.64 against 0.26 for random guessing.
- Among the **100 highest-scored reviews, 91%** really carry the churn signal.
- **Probabilities are calibrated**: the top-scored 10% of reviews average 77% predicted against 78% actual.
- **No circular logic.** The churn signal is defined by certain words, so a model that reads those words just relearns the rule (AUC 0.91). I removed those words before modelling and report the honest 0.80.
- Words that push a review towards a churn signal: *subscription, account, trial, card, charge, refused, paying*. Words that push it away: *good, love, best, music, songs, ads*.

### Revenue at risk and retention ROI

- Revenue at risk is **$49,669 per year** across 860 reviewers who sound like paying subscribers (assumed price $12.99 per month).
- A targeted retention campaign of **445 offers costs $4,139** and is expected to save **$6,990**, a **net benefit of $2,851 (ROI 0.7x)**.
- **Cheap, targeted offers pay back best** under these assumptions. Priority account recovery (1.6x) beats discount-based offers such as a refund review plus a free month (0.6x) or a retention call with 25% off (0.4x).
- **The result is sensitive to the assumptions.** The net benefit at the assumed success rates is $2,851. If every offer works half as well it falls to $287. At a lower price of $4 per month it is only $181. If offers work 1.5x better it rises to $6,756.

**Suggested actions (if advising the company):** make cancellation self-serve and obvious, confirm cancellations clearly and stop charges right away, clarify what happens when a free trial ends, make refunds and billing support easier to reach, and track the monthly churn-signal rate as an early-warning KPI.

## Assumptions

These are **not measured**. They sit in one place in the notebook and can be changed.

- Subscription price: $12.99 per month (US Premium Individual list price, Nov 2026). Real average revenue per subscriber is lower.
- Offer costs and success rates for each retention offer.
- Whether a reviewer is a paying subscriber (a text check for words like premium, subscription, renew).

## Limits

- **The label is a proxy.** A high probability means a review *sounds like* a customer who is leaving, not that the customer cancelled.
- **Reviewers are not a random sample of customers.** Do not scale the dollar figures up to all customers.
- **No backtest against real outcomes**, because the data has no cancellation records.
- **The model finds who is at risk, not who would respond to an offer.** Measuring that needs an A/B test.
- Some reviews may be fake or AI-written (not checked here).

## How to run

1. Open [Google Colab](https://colab.research.google.com) and upload `Churn_analysis_Spotify_.ipynb` (File, then Upload notebook).
2. Upload `Churn_analysis_Spotify_public.csv` in the Files panel on the left. The file name must match exactly.
3. Click **Runtime, then Run all**.

The notebook creates `exported_churn_data.csv`, `scored_reviews.csv` and a temporary `reviews.db` in the Files panel.
