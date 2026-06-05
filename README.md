# Farming vs Fighting: Can CS Offset a Kill Deficit in League of Legends?

By David Yu

## Introduction

In League of Legends, players often judge the state of a game by looking at the kill score. If a team is down several kills by 15 minutes, it is easy to assume that the game is slipping away. However, kills are not the only way a team builds an advantage. A team can fall behind in fights while still farming well, catching waves, and staying close in resources through creep score, or CS.

This project investigates whether a farming advantage can offset an early kill deficit in professional League of Legends. More specifically, I look at whether teams that are **down in kills but ahead in CS at 15 minutes** perform similarly to teams that are **up in kills but even or behind in CS**.

The dataset used in this project is the 2025 League of Legends esports match data from Oracle’s Elixir. Each row contains information about either a player or a team in a professional match. For this project, I focus mainly on the team-level rows, since my question is about whether a team’s overall farming advantage can compensate for its overall kill deficit.

The most relevant columns for this analysis are:

| Column         | Description                                                                                |
| :------------- | :----------------------------------------------------------------------------------------- |
| `result`       | Whether the team won the game. A value of 1 means the team won, and 0 means the team lost. |
| `killsat15`    | Number of kills the team had at 15 minutes.                                                |
| `deathsat15`   | Number of deaths the team had at 15 minutes.                                               |
| `csat15`       | Total team creep score at 15 minutes.                                                      |
| `csdiffat15`   | Team CS difference at 15 minutes compared to the opposing team.                            |
| `golddiffat15` | Team gold difference at 15 minutes compared to the opposing team.                          |
| `side`         | Whether the team played on blue side or red side.                                          |
| `league`       | The professional league the match came from.                                               |

My central question is:

**Can a team’s CS advantage at 15 minutes meaningfully offset being behind in kills?**

In this project, I define “offset” as a comparison between two different types of early-game states:

| Game State             | Meaning                                                                   |
| :--------------------- | :------------------------------------------------------------------------ |
| Down kills, up CS      | The team is losing the kill score but has a farming advantage.            |
| Up kills, even/down CS | The team is winning the kill score but does not have a farming advantage. |

If the first group wins at a similar rate to the second group, that would suggest that CS can partially make up for an early kill deficit.

## Data Cleaning and Exploratory Data Analysis

To prepare the data, I first filtered the dataset to only include team-level rows. The original dataset includes both individual player rows and team rows, but player rows are harder to compare directly because CS expectations differ heavily by role. For example, a support and an ADC should not be judged by the same raw CS value. Since this project focuses on overall team game state, team-level rows are more appropriate.

I then selected columns related to the state of the game at 15 minutes. This was important because I wanted the analysis to reflect information that would be available during the game, not information that is only known after the game ends. For that reason, I avoided using final-game statistics like final dragons, barons, towers, total gold, or total damage as main predictive features. Those columns may describe how the game ended, but they should not be used to explain what could have been known at 15 minutes.

I created a new column called `state15`, which groups each team into one of five early-game states:

| State at 15 Minutes      | Description                                                              |
| :----------------------- | :----------------------------------------------------------------------- |
| Up kills, up CS          | The team is ahead in both kills and CS.                                  |
| Up kills, even/down CS   | The team is ahead in kills but not ahead in CS.                          |
| Down kills, up CS        | The team is behind in kills but ahead in CS.                             |
| Down kills, even/down CS | The team is behind in both kills and CS.                                 |
| Even kills or other      | The team is even in kills or does not fit cleanly into the other groups. |

The summary below shows the win rate for each game state.

| state15                  | win_rate | count | avg_killdiff | avg_csdiff | avg_golddiff |
| :----------------------- | -------: | ----: | -----------: | ---------: | -----------: |
| Up kills, up CS          | 0.820548 |  5110 |       3.5229 |    33.8532 |       2694.3 |
| Up kills, even/down CS   | 0.509018 |  2994 |      2.63794 |   -23.2418 |      634.597 |
| Even kills or other      |      0.5 |  2234 |            0 |          0 |            0 |
| Down kills, up CS        | 0.495346 |  2901 |     -2.63668 |    23.9869 |     -609.075 |
| Down kills, even/down CS | 0.182587 |  5203 |     -3.50778 |   -33.2481 |     -2671.71 |

This table already shows an important pattern. Teams that were ahead in both kills and CS had the highest win rate, winning about 82.1% of the time. Teams that were behind in both kills and CS had the lowest win rate, winning only about 18.3% of the time. This makes sense: when a team is losing both fights and farm, they are usually in a very difficult position.

The most interesting comparison is between **Down kills, up CS** and **Up kills, even/down CS**. Teams that were down in kills but up in CS won about **49.5%** of their games. Teams that were up in kills but even or behind in CS won about **50.9%** of their games. These win rates are extremely close.

This suggests that a CS advantage may be able to offset a kill deficit, at least in some early-game situations. A team that is losing the kill score but farming better is not necessarily in a worse position than a team that is winning fights but falling behind in farm.

<iframe
 src="assets/state_winrate.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

The comparison also shows why kill score alone can be misleading. Being up kills but down CS is not as dominant as it may look on the scoreboard. Likewise, being down kills but up CS may not be as doomed as it feels in-game.

## Assessment of Missingness

Before building models or testing hypotheses, I looked at missing values in the columns most relevant to my question. The main columns I cared about were `killsat15`, `deathsat15`, `csat15`, `csdiffat15`, `golddiffat15`, and `result`.

For the NMAR analysis, I considered whether any column may be **NMAR**, meaning that the missingness of the column depends on the missing value itself. In this dataset, many missing values appear to come from whether certain match statistics were available or fully recorded, rather than from the actual unobserved value. For example, if a 15-minute statistic is missing, it may be due to data collection limitations or incomplete game records. However, without additional information about Oracle’s Elixir’s data collection process for each league and match, it is difficult to prove the exact missingness mechanism.

I then performed permutation tests to check whether the missingness of a selected 15-minute column was related to other observed columns. The goal was to see whether missingness appeared to depend on other features such as league, side, or data completeness.

The missingness tests help ensure that the later analysis is not blindly assuming that all missing values are random. This matters because if missing rows are concentrated in certain leagues or match types, the final conclusions may represent some parts of professional play better than others.

<iframe
 src="assets/missingness_test.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

## Hypothesis Testing

My hypothesis test focuses directly on the idea of “offsetting.” Instead of only asking whether more CS is associated with more wins, I compare two specific game states:

1. Teams that are **down in kills but up in CS** at 15 minutes.
2. Teams that are **up in kills but even or down in CS** at 15 minutes.

This comparison is important because it asks whether a farming advantage can substitute for a fighting advantage. If the two groups have similar win rates, then CS may be doing enough to offset the difference in kill score.

### Null Hypothesis

The win rate of teams that are down in kills but up in CS at 15 minutes is lower than the win rate of teams that are up in kills but even or down in CS. In other words, a CS lead does not offset the kill deficit.

### Alternative Hypothesis

The win rate of teams that are down in kills but up in CS at 15 minutes is similar to the win rate of teams that are up in kills but even or down in CS. In other words, a CS lead may meaningfully offset the kill deficit.

### Test Statistic

The test statistic is the difference in win rates:

```text
win rate of teams down kills but up CS
-
win rate of teams up kills but even/down CS
```

From the observed data:

```text
0.495346 - 0.509018 = -0.013672
```

The observed difference is about **-1.37 percentage points**. This means the farm-offset group won only slightly less often than the kill-lead-but-no-CS group.

This result is meaningful because the two game states look very different from a player’s perspective. One team is losing the kill score but farming better. The other team is winning the kill score but not farming better. Despite that difference, their win rates are nearly the same.

After running a permutation test, the resulting p-value was:

```text
0.0
```

Using a significance level of 0.05, I concluded:

```text
The alternative hypothesis is significant
```

Overall, the observed win rates suggest that CS advantage can meaningfully offset a kill deficit. This does not mean that farming always makes up for dying or losing fights. It means that, in professional games, a team that is behind in kills but ahead in CS can be in a much more playable position than the scoreboard alone suggests.

<iframe
 src="assets/hypothesis_test.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

## Framing a Prediction Problem

For the prediction section, I framed the task as a binary classification problem:

**Can we predict whether a team will win using information available at 15 minutes?**

The response variable is `result`, where 1 means the team won and 0 means the team lost.

I chose this prediction problem because it connects naturally to the main question. If CS can offset kill deficits, then a model using 15-minute kill and CS differences should be able to learn that kill score alone is not enough. The model should account for both fighting advantage and farming advantage.

The most important rule in this step was avoiding data leakage. Since the prediction is supposed to happen at 15 minutes, I only used features that would be known at or before 15 minutes. I did not use final-game statistics like final towers, final barons, final dragons, final total gold, or final damage, because those values are influenced by the outcome of the game.

The main evaluation metric I used was accuracy. Since the response variable is binary and the classes are balanced at the team level, accuracy is a reasonable first metric. I also looked at model behavior across specific game states during the fairness analysis.

## Baseline Model

My baseline model was a logistic regression classifier that used two quantitative features:

| Feature        | Type         | Why it was included                                                    |
| :------------- | :----------- | :--------------------------------------------------------------------- |
| `killdiffat15` | Quantitative | Measures whether the team is ahead or behind in kills at 15 minutes.   |
| `csdiffat15`   | Quantitative | Measures whether the team is ahead or behind in farming at 15 minutes. |

This baseline model directly matches the core question of the project. `killdiffat15` represents the fighting advantage, while `csdiffat15` represents the farming advantage.

The baseline model achieved an accuracy of:

```text
0.7340742748712388
```

I consider this a useful baseline because it uses only two simple features that most League players already understand. However, the model is limited because League games are not decided by kills and CS alone. Other factors, such as gold difference, XP difference, side selection, and regional league differences, may also affect the outcome.

The baseline model is still valuable because it gives a simple starting point: how much can we predict from kill difference and CS difference alone?

## Final Model

For the final model, I added more features and engineered new columns that better represent the idea of offsetting.

The added features included:

| Feature               | Why it helps                                                                                     |
| :-------------------- | :----------------------------------------------------------------------------------------------- |
| `golddiffat15`        | Captures the actual economic advantage behind kills, CS, plates, and other sources of gold.      |
| `xpdiffat15`          | Captures level advantages, which can matter even when gold and kills do not tell the full story. |
| `side`                | Accounts for possible blue-side and red-side differences.                                        |
| `league`              | Accounts for differences in professional regions and competitive environments.                   |
| `farm_offset_state`   | Indicates whether a team is down in kills but ahead in CS.                                       |
| `kill_cs_interaction` | Captures whether the value of CS changes depending on the kill difference.                       |

The most important engineered feature was the interaction between kill difference and CS difference. This matters because my question is not simply whether CS is good. The question is whether CS matters differently when a team is behind in kills. An interaction feature helps the model capture this relationship.

For the final model, I used:

```text
Decision Tree Classifier
```

I tuned the following hyperparameters using cross-validation:

```text
max_depth, min_samples_leaf, criterion
```

The final model achieved an accuracy of:

```text
0.7410926365795725
```



Even if the improvement is small, the final model is more thoughtful because it includes features that better reflect the actual game state at 15 minutes. In League, two teams with the same kill difference can be in very different positions depending on CS, gold, XP, and side.

<iframe
 src="assets/model_performance.html"
 width="800"
 height="600"
 frameborder="0"
></iframe>

## Fairness Analysis

For the fairness analysis, I asked whether the final model performs differently for teams in different early-game states.

The two groups I compared were:

| Group   | Description                                               |
| :------ | :-------------------------------------------------------- |
| Group X | Teams that were down in kills but up in CS at 15 minutes. |
| Group Y | Teams that were not in that farm-offset state.            |

This comparison is relevant because the farm-offset group is the main focus of the project. If the model performs much worse for teams that are down in kills but ahead in CS, then the model may not be capturing the exact type of game state this project cares about.

### Null Hypothesis

The model is fair across these two groups. Its accuracy for farm-offset teams and non-farm-offset teams is roughly the same, and any observed difference is due to random chance.

### Alternative Hypothesis

The model is unfair across these two groups. Its accuracy for farm-offset teams is lower than its accuracy for non-farm-offset teams.

### Test Statistic

The test statistic is:

```text
accuracy for non-farm-offset teams
-
accuracy for farm-offset teams
```

The observed test statistic was:

```text

win_rate	count	avg_killdiff	avg_csdiff
comparison_group				
Down kills, up 30+ CS	0.646857	875	-2.234286	46.730286
Up kills, even/down CS	0.487875	2763	2.278683	-23.336953
```

The p-value from the permutation test was:

```text
0.0
```



This fairness analysis is important because my project is centered on a specific kind of game: the team is losing fights, but staying alive through farm. A good model should not only perform well overall. It should also handle this specific game state reasonably well.

## Conclusion

This project started from a familiar League question: is the kill score really enough to tell who is winning?

The analysis suggests that the answer is no. Teams that were down in kills but ahead in CS at 15 minutes won about **49.5%** of their games, while teams that were up in kills but even or behind in CS won about **50.9%** of their games. These win rates are nearly identical.

This does not mean that kills are unimportant. Teams that were ahead in both kills and CS had a very high win rate, while teams that were behind in both had a very low win rate. But the central comparison shows that farming can change how we interpret a kill deficit. A team that is down in kills but ahead in CS may still be in a very playable position.

For players and viewers, the main takeaway is simple:

**A kill lead without a farming lead may not be as strong as it looks, and a kill deficit with a farming lead may not be as bad as it feels.**
