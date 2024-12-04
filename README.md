# Introduction 

Cookie Cats is a hugely popular mobile puzzle game developed by Tactile Entertainment. It's a classic "connect three" style puzzle game where the player must connect tiles of the same color in order to clear the board and win the level. It also features singing cats.

As players progress through the game they will encounter gates that force them to wait some time before they can progress or make an in-app purchase. In this project, I analyzed the result of an A/B test where the first gate in Cookie Cats was moved from level 30 to level 40. In particular, I analyzed how this change impacts on player retention and game rounds.

# Background

The dataset is prepared by Aurelia Sui.
https://www.kaggle.com/datasets/yufengsui/mobile-games-ab-testing

The data is from 90,189 players that installed the game while the AB-test was running. The variables are:

- **userid** - a unique number that identifies each player.
- **version** - whether the player was put in the control group (gate_30 - a gate at level 30) or the test group (gate_40 - a gate at level 40).
- **sum_gamerounds** - the number of game rounds played by the player during the first week after installation
- **retention_1** - did the player come back and play 1 day after installing?
- **retention_7** - did the player come back and play 7 days after installing?
When a player installed the game, he or she was randomly assigned to either gate_30 or gate_40.

# Analysis
## Data
Dataset consists of 90189 rows(unique users), and 5 variables. Luckily, there is no missing value in the dataset.

![image](https://github.com/user-attachments/assets/8ed8b01f-0878-4b10-9ddd-b84d9a7e1398) 

![image](https://github.com/user-attachments/assets/e3e7f2b2-65a0-4adf-ace6-9fd14c699c6b)

Each player is assigned to either gate_30 or gate_40 in the game. The number of players for each group is approximately the same in the control and test group.

```python
df.groupby("version").sum_gamerounds.agg(["count", "median", "mean", "std", "max"])
```

![image](https://github.com/user-attachments/assets/6a3999a5-8094-4f5c-9207-5988a6a95e98)

## Outliers
The outlier for the control group(gate_30) can be seen in the graph below:

![image](https://github.com/user-attachments/assets/527ee7c9-1edd-45ec-8247-ac505d750c8b)

In the standard IQR method, the %25-%75 quartiles are typically used. However, based on the visual inspection of the outliers, I adjusted the range to be broader by using IQR with %1-%99 percentiles. This approach allows to accommodate a wider range of data points.
After removing outliers, distribution of sum_gamerounds is shown as follows:

![image](https://github.com/user-attachments/assets/7e18a235-268c-4d43-8278-1fad52b0c4f1)

## Important Statistics

As you can see, 50% of players played less than 16 rounds during the first week, and 75% of players played less than 51 rounds.

![image](https://github.com/user-attachments/assets/854c5da5-8116-4a01-bf12-8c5b45cda3be)

Almost 4000 players did not even play a single round after installation.

Additionaly, number of players decrease as the levels progress.

![image](https://github.com/user-attachments/assets/4939f1c0-81aa-4d39-a5c7-82d41785551e)

Looking at the summary statistics, the control(gate_30) and test groups(gate_40) seem similar, but it should be investigated if these groups are statistically significant. 

![image](https://github.com/user-attachments/assets/a9a5a9d2-6f66-4986-892a-380121f95388)     

Retention variables show player retention details. %55 percent of the players didn't play the game 1 day after installing, while %81 percent of the players didn't play the game 7 day after installing.

![image](https://github.com/user-attachments/assets/23a74bb2-714c-4482-95f0-6890738a2780)

Looking at the summary statistics of retention variables(after 1 day and 7 days) by version and comparing with sum_gamerounds, it can be seen there are similarities between groups. However, it still should be considered if there is a statistically significant difference.

![image](https://github.com/user-attachments/assets/31b929af-d441-491a-9611-a81ca3bcb16e)  ![image](https://github.com/user-attachments/assets/7109f5f4-9390-407f-9da7-e2a1d8770b93)

I created a new "Retention" variable to show active players, who played the game after 1 day and 7 days.
It shows that 14% of the total players are active players.
Again, control and test groups seems similar in case of player retention. As the final step, I performed an A/B test if these groups are statistically significant.

```python
df["Retention"] = np.where((df.retention_1 == True) & (df.retention_7 == True), 1,0)
df.groupby(["version", "Retention"])["sum_gamerounds"].agg(["count", "median", "mean", "std", "max"])
```

![image](https://github.com/user-attachments/assets/52ade668-ea83-46f3-8491-ae90a34cbe80)

## A/B Testing

### Procedure:
- Apply Shapiro Test for normality
- If parametric apply Levene Test for homogeneity of variances
- If Parametric + homogeneity of variances apply T-Test
- If Parametric - homogeneity of variances apply Welch Test
- If Non-parametric apply Mann Whitney U Test directly

1)Create hypothesis - H0: A = B (There is no significant difference between groups)

2)Check hypothesis
- Normal distribution
- Homogeneity of variance

```python
test_stat, p_value = shapiro(group_A)
print("Test Stat = %.4f, p-value = %.4f" %(test_stat,p_value))
```
Test Stat = 0.5104, p-value = 0.0000
```python
test_stat, p_value = shapiro(group_B)
print("Test Stat = %.4f, p-value = %.4f" %(test_stat,p_value))
```
Test Stat = 0.5044, p-value = 0.0000

3)Apply hypothesis
As p-value < 0.05 for both groups, **normal distribution** is **rejected**. Mann Whitney U Test can be applied directly:

```python
test_stat, p_value = mannwhitneyu(group_A,group_B)
print("Test Stat = %.4f, p-value = %.4f" %(test_stat,p_value))
```

Test Stat = 1024331275.0000, p-value = 0.0502

4)Evaluate results with p-value
As p-value > 0.05, it is concluded that hypothesis is accepted and there is no significant difference between A/B groups. 
   
# Conclusion

As players progress through the game they will encounter gates that force them to wait some time before they can progress or make an in-app purchase. In this project, we analyzed the result of an A/B test where the first gate in Cookie Cats was moved from level 30 to level 40. In particular, we analyzed the impact on player retention and game rounds.

Firstly, we investigated relationships and structures in the data. There was no missing value but there was an outlier problem in the data. Summary stats and plots assisted to understand the data and problem.

Before A/B Testing, we shared some details and statistics about game, players and problems.

After applying A/B Testing, Shapiro Test rejected H0 for normality assumption. Therefore we needed to apply a Non-parametric Mann Whitney U Test to compare two groups. As a result, H0 hypothesis is accepted which means A/B groups are statistically significant.

Briefly, moving first gate from level 30 to level 40 creates no significant difference between control and test groups.
It means player retention is not affected at all, and we can not enhance player retention with this level change.


```python
ab.groupby("version").retention_1.mean(), ab.groupby("version").retention_7.mean()
```
![image](https://github.com/user-attachments/assets/2322c757-7229-422e-a60a-057ae122cf07)

1-day and 7-day average retention are a bit higher when the gate is at level 30 than when it is at level 40.

Consequently, even if there is a slight numerical difference , the gate can be remained at level 30.





