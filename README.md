**Andrew Xia**

>
## Introduction

League of Legends is a competitive team-based game where two teams compete to gain advantages and ultimately destroy the opposing team's base (nexus tower). Throughout a match, teams earn gold in order to purchase stronger items. Because of this, the difference in gold between two teams is an important and great way to
measure which team is ahead at a given point in the game.

In this project, I analyze match data provided by the Oracle Elixir 2025 esports match data. The original dataset contains 120,492 rows and 165 columns. Each game contains individual player rows as well as
team-level summary rows.

The main question I investigate is:

> **How does a team's gold deficit at 15 minutes relate to its likelihood of
> coming back and winning a professional League of Legends match?**

This is an interesting question because a team that is behind in gold early  can still recover and win. In order to understand how strong early game disadvantes are related to the final outcome of a professional match, we can examine how comeback rates change as the 15 minute gold deficit becomes larger.

### Relevant Columns

These were the columns that were relevant in my analsis:

| Column | Description |
| --- | --- |
| `gameid` | Unique identifier for each professional match. |
| `datacompleteness` | Indicates whether the recorded match data is complete or partial. |
| `league` | The professional league in which the match was played. |
| `teamname` | The name of the team represented by the row. |
| `side` | Whether the team played on Blue Side or Red Side. |
| `result` | The final match result, where `1` represents a win and `0` represents a loss. |
| `golddiffat15` | The team's gold difference relative to its opponent at 15 minutes. A negative value means the team is behind. |
| `xpdiffat15` | The team's experience difference relative to its opponent at 15 minutes. |
| `killsat15` | The number of kills the team has at 15 minutes. |
| `deathsat15` | The number of deaths the team has at 15 minutes. |

>
## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
The orignal dataset has both individaul player rows and team level summary rows. My question focuses on whether or not a **team** can come back from and early gold deficit so I kept only the team summary rows. The reduced the data to 20,082 team rows.

I then looked at the missing values in `golddiffat15` which had 1,640 rows with missing 15 minute gold differences. All of these rows were marked as `partial` in the `datacompleteness` column whereas the 18,442 rows with recorded 15-minute gold difference were marekd as `complete`. I only kept complete games for my analysis so that every row had the information needed at 15 minutes.

Since I am specifically studying comebakcs I only kept teams who had a negative `golddiffat15`, meaning they were behind in gold at 15 minutes.

Lastly, I made mulitplied `golddiffat15` by -1 to create `gold_deficit_at_15`. This makes it easier to interpret as instead of a value like -2000 for a 2000 gold defecit, the new column 2000 as a positive number to represent the deficit.

Below are the first five rows of the cleaned data used for the comeback analysis:

| teamname | golddiffat15 | gold_deficit_at_15 | result |
| --- | ---: | ---: | ---: |
| IziDream | -3837 | 3837 | 0 |
| Skillcamp | -5069 | 5069 | 0 |
| Project Conquerors | -118 | 118 | 1 |
| Zerance | -4439 | 4439 | 0 |
| Karmine Corp Blue Stars | -5490 | 5490 | 0 |

### Univariate Analysis

The following histogram shows the distribution of gold deficits among teams
that were behind at 15 minutes. Most teams have relatively smaller deficits,
while extremely large gold deficits are much less common.

This histogram below shows the distributions of gold defecits among teams that were behind at 15 minutes. Most teams have a smaller deficit while extremlely large gold deffcitis are actually a lot less common.

<div style="text-align: center;">
  <iframe
    src="assets/gold-deficit-distribution.html"
    width="100%"
    height="550"
    frameborder="0"
    style="max-width: 900px;"
  ></iframe>
</div>

### Bivariate Analysis

This box plot below compares the two distributions of 15 minute gold deficits of teams that ended up losing vs teams that came back to win. For comeback wins the plot is more concentrated around smaller deficits. As for losses, the plot is more conentrated around larger deficits, which suggests that larger early gold deficits are more difficult to comeback from.

<div style="text-align: center;">
  <iframe
    src="assets/gold-deficit-by-outcome.html"
    width="100%"
    height="550"
    frameborder="0"
    style="max-width: 900px;"
  ></iframe>
</div>

### Interesting Aggregates

To directly examine the relationship between gold deficit at 15 minutes and result I grouped the teams into ranges based on size of their gold dficit at 15 minutes. I then calculted the comback win rate for each range.


| deficit_group   |    result |
|:----------------|----------:|
| 0-500           | 0.468378  |
| 500-1000        | 0.388999  |
| 1000-2000       | 0.290233  |
| 2000-3000       | 0.196774  |
| 3000-5000       | 0.077592  |
| 5000+           | 0.0166667 |

From this table we see a clear downward trend in win rate. As gold deficit at 15 minutes increases the comback win rate decreases. Teans behind 500 gold or less won around 47% of their games while teams behind 5000 or more gold came back only around 2% of the time. This suggests that the gold deficit at 15 minutes is strongly associated with the team's chance at obtaining a comeback win.

>
## Assessment of Missingness

### MNAR Analysis

After investagating the missingness of `golddiffat15` I did not have enogh evidence to conclude that `golddiffat15` is **MNAR**. My analysis shows the missingness is strongly associated with the `league` column. This provides evidence for the missingness to possibly be MAR but it doesnt not rule out the possibility of MNAR either.

A possible MNAR mechanism would be if games with extremely large gold differences at 15 minutes were more likely to have their gold difference values missing. For this case the missingness of `golddiffat15` would depend on the value of `golddiff15` itself.

Additional information that could explain the missingness could be whether or not tracking was used for each game. Another piece of information that could help would be  based on the region whether or not that region has their league statistics public through official API's.


### Missingness Dependency

In order to test whether missingness of `golddiffat15` depends on other observed variables in the dataset, I did permutation tests using `league` and `side`.

#### Missingness and League

**Null Hypothesis:** The missingness of `golddiffat15` does not depend on
whether a game is from the LPL. Any observed difference in missingness rates
is due to random chance.

**Alternative Hypothesis:** The missingness of `golddiffat15` does depend on
whether a game is from the LPL.

**Test Statistic:** The difference in the proportion of missing
`golddiffat15` values between LPL and non-LPL games.

The observed difference ofr missingness rates was roughly **0.998**. After doing 1000 permutations, none of the simulated differences were as extreme as the observed difference which gave a p-value of **0.00** (or less than 0.001)

For significance level of 0.05, I reject the null hpothesis. This means there is strong evidence that the missingness of `golddiffat15` depends on if the game is from the LPL league (China's league)

<div style="text-align: center;">
  <iframe
    src="assets/missingness-permutation-test.html"
    width="100%"
    height="550"
    frameborder="0"
    style="max-width: 900px;"
  ></iframe>
</div>

This histogram shows the empirical distribution of the difference in `golddiffat15` missingness rates for LPL and non-LPL games under the null hypothesis. The red line shows the observed difference of about 0.998. Since the observed statistic lies far outside the diestirbution of the permutation, the results show strong evidence that the missingness of `golddiffat15` depends on if a game is form the LPL.


#### Missingness and Side

I then tested whether the missingness of `golddiffat15` depends on whether or not a team plays on Blue or Red Side.

**Null Hypothesis:** The missingness of `golddiffat15` does not depend on whether a team is on Blue or Red Side.

**Alternative Hypothesis:** The missingness of `golddiffat15` does depend on whether a team is on Blue or Red Side.

**Test Statistic:** The difference in the proportion of missing `golddiffat15` values between Blue Side and Red Side teams.

Both Blue and Red Side teams had a missingness rate of approximately
**0.08**, giving an observed difference of **0**. The permutation test
produced a p-value of **1.0**.

Both Blue and Red side teams had a missingness rate of roughly **0.08** which gives an observed difference of **0.00** and the permutation test gave a p-value of **1.0**

Because the p-value is larger than 0.05, we fail to reject the null hypothesis, thus ther eis not sufficienct evidence that missingness of `godldiffat15` depends on the side a team plays on.

>
## Hypothesis Testing

In order to investigate if a team's early gold deficit is associated with the teams chance of making a comeback, I performed a permutation test comparing teams with relatively small and large gold deficits at 15 minutes.

I classified teams as having a small defiict as **less than 1000 gold behind** and teams having a large deficit as **at least 1000 gold behind**.

**Null Hypothesis:** Teams that are less than 1000 gold behind at 15 minutes have the same comeback win rate as teams that are at least 1000 gold behind. Any observed difference is due to random chance.

**Alternative Hypothesis:** Teams that are less than 1000 gold behind at 15 minutes have a higher comeback win rate than teams that are at least 1000 gold behind.

**significance level of 0.05**

**Test Statistic:** The difference in comeback win rates:

**small-deficit comeback rate − large-deficit comeback rate**

I chose this test stastic because my question directly compares the probability of a comeback between the two deficit groups. Positive values mean that teams with a smaller deficit have a higher comeback rate.

I got an observed comeback win rate of **42.97** for teams less than 1000 gold behind and **19.06** for teams that were at least 1000 gold behind. The observed difference was apporximately **0.239** .

After 1000 permutations, none of the simulated differences were as large as the observed difference which gave us a p-value of 0.00 (or p-value of less than 0.001).

Since the p-value is less than the significance level of 0.05 I reject the null hypothesis. The results of the permutation test shows strong evidence taht teams with samller gold deficits at 15 minutes have a higher comeback win rate than teams with larger deficits.

<div style="text-align: center;">
  <iframe
    src="assets/comeback-hypothesis-test-v2.html"
    width="100%"
    height="550"
    frameborder="0"
    style="max-width: 900px;"
  ></iframe>
</div>

This histogram shows the empircal difference for comeback win rates under the null hypothesis. The red line represents the observed difference of about 0.238. This lies far right of the permutation distribution which means it is very unlikely that teams with small or large gold deficits actually have the same comeback win rate. This support sht conclusion that teams with smaller deficits at 15 minutes have a higher chance of coming back and winning.

>
## Framing a Prediction Problem

The question I will answer for the prediction portion of this project:

> **Among teams that are behind in gold at 15 minutes, can we predict whether
> they will come back and win the game?**

This is a **binary classification** problem and the response variable is `result`, where:

- `1` represents a comeback win.
- `0` represents a loss.

I chose `result` because the goal of my model is to predict if a team that is currently behind and will eventually recover and win the match.

### Time of Prediction

The prediction for whether or not a team will comecback is made at 15 minutes into the game. Therefore, the model will only use information that would already be available at that point of the match. It will not use match data from later in the game because those values would not be known when the prediction is being made.

### Evaluation Metric

**F1-score** will be my main evaluation metric.

About 27% of teams that are behind at 15 minutes comeback to win the game, therefore our response variable is imbalanced. Because of this, accuracy can be misleading as a model could achieve relatively high accuracy just by predicting that every team will lose.

We can combine **precision** and **recall** to use F1-score. Recall is important because I want the model to be able to indentify actual comeback wins and precision is also important because I want the model to avoid predicting too many comebacks that don't acutally happen.

I will also report accuracy, precision, and recall to give additional context for the model's performance.

>
## Baseline Model

My baseline model is a **Decision Tree Classifier** with a max depth of 2. The model uses two features that are available at 15 minutes:

| Feature | Type | Description |
| --- | --- | --- |
| `golddiffat15` | Quantitative | The team's gold difference relative to its opponent at 15 minutes. |
| `xpdiffat15` | Quantitative | The team's experience difference relative to its opponent at 15 minutes. |

Both features are quantitative, so no categorical encoding was necessary.
They were passed directly into the Decision Tree Classifier through an
`sklearn` Pipeline.

Since both features are quantitative I did not use any categorical encoding. Both features were passed through the Decision tree Classifier using an `sklearn` Pipeline directly.

The model achieved a training accuracy of approximately **72.75%** and a test
accuracy of approximately **73.32%**. The similar training and test accuracies
suggest that the model is not substantially overfitting.

This model got a training accuracy of roughly **72.75%** and a test accuracy of roughly **73.32%**. Since the trainign and test accuracies are quite similar, this suggests that the model is not substantially overfitting. However this model can be misleading, because we are using accuracy. About 73% of teams that were behind at 15 minutes evnetually lose, so a model can achieve relatively high accuracy just by predicting a loss for pretty much every team.

When treating a comeback win as the positive class, the baseline model had a
**test recall of 0** and an **F1-score of 0**. This means that the model failed
to correctly identify any of the teams that actually came back and won.

The baseline model also had a test recall and F1-score of 0 when having the comeback win as the positive class. This means that the model was not able to correctly identify any of the teams that actually came back and won.

Therefore, I don't think the baseline model to be actually useful for this prediction task. The accuracy does seem relatively high but it was unable to truly identify comeback wins.

>
## Final Model

For my final model, I kept the `golddiffat15` and `xpdiffat15`, and engineered two additional features using the information available at 15 minutes.

### Added Features

The first feature I added is a **small deficit indicator** which indicates whehter a team is less than 1000 gold behind at 15 minutes. Earlier, my exploratory analysis and hypothesis test showed us that teams with deficits smaller than 1000 gold had a much higher comeback rate compared to teams with larger deficits. This feature allows my model to directly capture this difference.

The second new feature added is **combat balance**:

**kills at 15 minutes − deaths at 15 minutes**

Kills and deaths provide informations on how well a team performed in early game fights. For example two teams with similar gold deficits may still be in two very different situations if one team has been performing better in combat. This feature provides the model with additional information on the state fo the game beyong just gold and experience alone.

The final model uses the information from:

| Feature | Description |
| --- | --- |
| `golddiffat15` | Gold difference at 15 minutes |
| `xpdiffat15` | Experience difference at 15 minutes |
| Small-deficit indicator | Whether the team is less than 1000 gold behind |
| Combat balance | Kills minus deaths at 15 minutes |

### Model and Hyperparameter Selection

I experimented with both **Random Forest Classifier** and **Decision Tree Classifiers**, both of which I used 5 fold cross validation using `GridSearchCV and optimized using F1-score.

I searched over several values of:

- `max_depth`
- `min_samples_split`
- `criterion`, using both `gini` and `entropy`

The combinations with the highest validation F1-scores all had very large differences between their training and validation F1-scores which suggests that the models were overfitting.

Because of this I selected a simpler Decision Tree:

- `max_depth = 20`
- `min_samples_split = 20`
- `criterion = 'gini'`

This model had a training F1-score of roughly **0.66** and a validation F1-score of roughly **0.34**. Although the validation F1-score was slightly lower than the highest value that was found during the search, the smaller difference between the training and validation performance suggests less overfit.

### Final Model Performance

After using our model on the unseen test set, the Final model got scores of:

| Metric | Score |
| --- | ---: |
| Accuracy | 0.680 |
| Precision | 0.390 |
| Recall | 0.354 |
| F1-score | 0.371 |

A validation F1-score of roughly **0.34** was pretty close to the test F1-score of about **0.37** which suggests that the model performed simlarly on unseen data.

The final model significantly improves on the baseline model. The baseline had an F1-score and recall of **0** which means it didn't identify any actual comeback wins. The final mdoel had a recall of about **0.35** meaning it successfully indentifies some teams that actually came back to win their game.

Althought the final model does have a lower accuracy compared to the baseline model, accuracy here is not the primary metric because of the imbalanced prediction problem. An increase of F1-score from **0** to approximately **0.37** make sthe final model substantially better and more useful for indentifying comeback wins.

>
## Fairness Analysis

For my fairness analysis, I looked at and examined whether or not the final model performs differently for teams playing **Blue Side** compared to the teams playing on **Red Side**

The two groups are:

- **Group X:** Blue Side teams
- **Group Y:** Red Side teams

My evaluation metric was **recall** since comeback wins are the positive class and recall measure the proportion of actual comeback wins that the model successfully identifies.

### Hypotheses

**Null Hypothesis:** The model is fair with respect to side. Its recall for
Blue Side and Red Side teams is approximately the same, and any observed
difference is due to random chance.

**Alternative Hypothesis:** The model is not fair with respect to side. Its
recall differs between Blue Side and Red Side teams.

**Test Statistic:** The absolute difference in recall between Blue Side and
Red Side teams.

I use absolute difference here for my test statistic since I want to know if the model performs differently between the two sides, regardless of which side has a higher recall.

I used a **significance level of 0.05**.

The model had a recall of about **0.32** for Blue Side teams and **0.38** for Red Side teams. The observed absolute difference for recall is about **0.06**

After performing **1000 permutations**, I obtained a p-value of **0.107**.

Since 0.107 is greater than the significance level of 0.05, I **fail to reject
the null hypothesis**. There is not enough evidence to conclude that the
model's recall differs between Blue Side and Red Side teams.

0.107 is greater than my significance level of 0.05 so I **fail** to reject the null hypothesis. There is not enough evidence to come to the conclusion that the model's recall differes between Blue and Red Side teams. 

However, this does not prove that the model is perfectly fair. It only shows that the permutation test did not provide statistically significant evidence of a difference in recall between the two groups.

<div style="text-align: center;">
  <iframe
    src="assets/fairness-permutation-test-v2.html"
    width="100%"
    height="550"
    frameborder="0"
    style="max-width: 900px;"
  ></iframe>
</div>

This histogram shows the permutation distirbution of the absolute difference in recall between Blue and Red side under the null hypothesis. The red line shows the observed difference of about 0.06. Since the differences at least this large occured reasonbly often under the null, the resulting p-value of 0.107 is not statistically signifcant at the 0.05 significance level.