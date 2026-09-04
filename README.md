**Andrew Xia**

## Introduction

League of Legends is a competitive team-based game in which two teams compete to gain advantages and ultimately destroy the opposing team's base. Throughout a match, teams earn gold in order to purchase stronger items. Because of this, the difference in gold between two teams is an important and great way to
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
