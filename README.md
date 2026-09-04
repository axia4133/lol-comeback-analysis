**Andrew Xia**

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


## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
The orignal dataset has both individaul player rows and team level summary rows. My question focuses on whether or not a **team** can come back from and early gold deficit so I kept only the team summary rows. The reduced the data to 20,082 team rows.

I then looked at the missing values in `golddiffat15` which had 1,640 rows with missing 15 minute gold differences. All of these rows were marked as `partial` in the `datacompleteness` column whereas the 18,442 rows with recorded 15-minute gold difference were marekd as `complete`. I only kept complete games for my analysis so that every row had the information needed at 15 minutes.

Since I am specifically studying comebakcs I only kept teams who had a negative `golddiffat15`, meaning they were behind in gold at 15 minutes.

Finally, I created a new column called `gold_deficit_at_15` by multiplying
`golddiffat15` by -1. This makes the deficit easier to interpret: instead of
a value such as -2000 representing a 2000-gold deficit, the new column
represents that deficit as a positive value of 2000.

Lastly, I made mulitplied `golddiffat15` by -1 to create `gold_deficit_at_15`. This makes it easier to interpret as instead of a value like -2000 for a 2000 gold defecit, the new column simply has 200 as a positive number to represent the deficit.

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

This histogram below shows the distirbution of gold defecits among teams that were behind at 15 minutes. Most teams have a smaller deficit while extremlely large gold deffcitis are actually a lot less common.

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

This box plot below compares the two distirbutions of 15 minute gold deficits of teams that ended up losing vs teams that came back to win. For comeback wins the plot is more concentrated around smaller deficits. As for losses, the plot is more conentrated around larger deficits, which suggests that larger early gold deficits are more difficult to comeback from.

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