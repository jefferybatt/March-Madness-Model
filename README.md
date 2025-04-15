# March-Madness-Model
A statistical analysis project with the goal of creating a (near) perfect bracket. I'm utilizing my own intuitive knowledge and experience in basketball, xAI Grok 3 beta, Microsoft Excel 365, Python and Power BI. I plan to iterate on a our composite "Grok Score" each year to improve the performance of the predictive model for the NCAA Men's Basketball Tournament. The foundational Key Performance Indicators (KPIs) I'm focusing on to start are; Possession Differential (PD), Effective Field Goal Percentage (EFG%), Strength of Schedule (SOS) and Average Age Weighted by Minutes (AAWM). 

## Possession Differential (PD)
How many more possessions do you get than your opponent on average? Every possession matters in single elimination tournament basketball. Scores on average are lower in this competition format, two teams facing elimination become more risk averse, games are described as "cagey". To get an approximation of the possession battle (fouls & refs constant), we're simply using a team's season average Rebound Differential (RD) and their season average Turnover Differential (TD). RD + TD = PD. 

## Effective Field Goal Percentage (EFG%)
EFG% weights three pointers 50% more, and is a good measure of shooting or scoring efficiency. EFG % = (FGM + 0.5 x 3PM) / FGA. Note: Next year we'll include Free Throws Made (FTM) to get an even more accurate indicator of shooting efficiency. This also would account for points scored off fouls drawn, which is not currently accounted for in this model. For each 1 Possession Differential you give up to your opponent, you'll need to outshoot them by 0.6%. This means for example if you lose the PD by 5, you'll need to outshoot them by about 3% in EFG to tie the game. 

## Strength of Schedule (SOS)
SOS is included in RPI, which is the primary metric for determining Rank. We want our model to be correlated to Rank (RPI), but not perfectly, otherwise we would never predict any upsets. So in this model we'll just focus simply on SOS. For our purposes, SOS = (Opponent's Winning % / # of Games ). We want to use this to be able to discount those teams to some degree that achieve a high rating in PD and EFG%, because of a lack of competition. 

## Average Age Weighted by Minutes (AAWM)
AAWM is a proxy for experience and maturity essentially. The NIL era has ushered in an older generation of college players, which has increased the disparity between ages on the court, in some cases you have 18 year olds playing against 23 or 24 years olds who still have eligibility. This is a crucial factor, especially in terms of motivation (someone playing maybe the last game of their career vs someone just trying not to get hurt before the NBA draft). There is a counter intutive argument however that the older players in NCAA aren't as good, otherwise they would be playing pro already. In any case, we believe it is still a factor, albeit not as significant as PD, EFG or SOS. 

## Data Gathering and Cleaning
We weren't able to find a single source that had all the data were interested in so we utilized Grok 3 beta and Python (PyCharm) in order to scrape basketball stats from various websites in order to aggregate and merge our data into a single table. Our sources were; ncaa.com for EFG%, RD, TD  and basketball.realgm.com for SOS and AAWM. basketball.realgm.com proved difficult to compile the age data, because of the tricky website navigation, and needing to scrape birthdates and minutes played from seperate tables on the same page for all 365 teams.

## Data Merging and Fuzzy Matching 
There may be no greater fuzzy matching challenge out there than merging NCAA Men's basketball teams stats from various sources. Trying to find the right balance to account for the differencece in terminology; Connecticut vs Uconn for example proved difficult. Another exmaple being that "St." at the beginning of a string means, "Saint", where as "St." at the end of a string means, "State", except for in the case of "Mt. St. Mary's"... We imported packages, 'pandas' and 'fuzzywuzzy' into Pycharm's in order to join on the "Team" column as our Primary Key. There were 9 more teams in the realGM database than in the NCAA website, so that was our target # of teams missing data. In the end we came very close but had to essentially manually match about 20 teams, as there was just no rhyme or reason to the difference between "Texas A&M - CC" and "East Texas A&M". 

## Model 1
I developed a custom "GROK" Score metric, refined a 364-team dataset with 10 manual mappings to match 355 NCAA teams, and validated it with regression analysis in Excel, achieving an R Square of 0.5722

(0.05*(PD/AveragePD)) + (0.5*(EFG/AverageEFG)) + (0.5*(SOS/AverageSOS)) + (0.4*(AAWM/AverageAAWM))

The first model we are using is an additive model. Adding our variables together in order to get a composite score. In simple terms we are aiming to take a teams extra posessions they get vs their opponent and add their shooting efficiency, while also taking into account their strength of schedule and their age or experience level for tournament pressures. We've included a multiplier for each variable in order to balance the equation. We also normalized each metric by dividing it by the average to give us a relative metric. The average was based of the national average, all 355 teams. 

## Model 2
= (((EFG - 0.5158) * 7 + 1) * (((PD - (-13.8)) / (11 - (-13.8))) * 3) * ((SOS - 0.508) * 7 + 1) * (AAWM / 20.5))

The second model we are testing is a multiplicative formula, meaning each term amplifies the others. This design reflects the philosophy that basketball success—especially in high-stakes settings like March Madness—requires strength across multiple dimensions: shooting efficiency, possession control, schedule toughness, and experience. Let’s dive into each term. 

Subtract the average EFG% (0.5158) from the team’s EFG% to measure how far above or below average they are. Multiply the difference by 7 to amplify its impact. Add 1 to center the term around 1 (a neutral score for an average team). Why Normalize Around 0.5158?: By subtracting the average, we ensure teams with league-average shooting get a neutral score (~1), while elite shooters get a boost and poor shooters get penalized. Why Multiply by 7?: The multiplier of 7 gives EFG% significant weight, reflecting its critical role in high-pressure games. This value was chosen to balance its influence with other factors and enhance the model’s predictive power. Unlike traditional rankings that might lean on raw win totals or RPI (which SOS partially overlaps), EFG% focuses on a team’s ability to maximize scoring efficiency—a key differentiator in March Madness.

Possession Differential (PD), calculated as rebound differential plus turnover differential, measuring a team’s ability to secure extra possessions. -13.8: The minimum PD observed in the dataset. 11: The maximum PD observed in the dataset. Subtract the minimum PD (-13.8) from the team’s PD (I3) to position it relative to the worst case. Divide by the full PD range (11 - (-13.8) = 24.8) to scale it to a 0–1 range. Multiply by 3 to amplify its impact, with a maximum value of 3 for the best PD. Why Scale to 0–1?: PD’s wide range (-13.8 to 11) needs normalization to make it comparable to other terms. Scaling to 0–1 ensures fairness across metrics. Why Multiply by 3?: The multiplier of 3 ensures PD has a meaningful but not overwhelming impact—a team with the best PD gets a 3x boost, while the worst gets near 0. This keeps it balanced with EFG% and SOS. Traditional rankings often overlook possession metrics, focusing instead on wins or RPI. PD highlights a team’s hidden strengths in game control, diverging from the “expert” reliance on surface-level stats.

Strength of Schedule (SOS), a measure of the difficulty of a team’s opponents. 0.508: The average SOS across teams. Subtract the average SOS (0.508) from the team’s SOS (D3) to measure deviation from the norm. Multiply the difference by 7 to amplify its effect. Add 1 to center the term around 1 for average SOS. Why SOS?: Teams that have faced tougher opponents are better battle-tested for the intensity of March Madness. SOS ensures we reward teams for challenging schedules rather than padding stats against weak foes. Why Normalize Around 0.508?: Centering around the average SOS ensures fairness—average schedules yield a neutral score (~1), while tough schedules get a boost. Why Multiply by 7?: Like EFG%, the 7 multiplier emphasizes SOS as a critical factor, reflecting its importance in preparing teams for tournament play. While SOS overlaps with RPI conceptually, we avoid RPI entirely to focus on a cleaner, more direct measure of schedule strength. This sidesteps the outdated baggage of traditional metrics and aligns with our goal to innovate.

Age Weighted by Minutes (AAWM), the average age of a team’s players, weighted by their playing time. 20.5: A reference age (likely the average or median AAWM). Divide the team’s AAWM by 20.5 to scale it relative to this benchmark. Why AAWM?: Experience matters in high-pressure situations. Older teams, especially those with veterans playing big minutes, often perform better under tournament stress. Why Divide by 20.5?: Scaling by a reference age keeps this term subtle—teams older than 20.5 get a slight boost (>1), while younger teams get a slight penalty (<1). It’s a fine-tuning factor, not a dominant one. Innovation Angle: Traditional rankings rarely factor in player age explicitly. AAWM adds a layer of nuance, rewarding experience without letting it overshadow the core performance metrics.

This Grok Score diverges from traditional rankings by:
Avoiding RPI: RPI is outdated and redundant with SOS. Instead, we use SOS directly, paired with advanced metrics like EFG% and PD, to focus on what drives tournament success.

Multiplicative Design: Multiplying the terms ensures a team must excel across shooting, possessions, schedule strength, and experience to score highly. This mirrors the holistic demands of basketball, unlike additive models that treat factors independently.

Normalization and Weighting: Each metric is normalized (around averages or ranges) and weighted (e.g., 7 for EFG% and SOS, 3 for PD) to balance their contributions. These weights were tuned to maximize predictive accuracy (e.g., R-squared of 0.72), grounding our innovation in data.

Focus on Advanced Metrics: EFG% and PD capture efficiency and control—undervalued by traditional rankings—while SOS and AAWM add context and experience, creating a more complete picture.

= (((EFG% - 0.5158) * 7 + 1) * (((PD - (-13.8)) / (11 - (-13.8))) * 3) * ((SOS - 0.508) * 7 + 1) * (AAWM / 20.5))

Components
EFG% Term: (EFG% - 0.5158) * 7 + 1
Measures shooting efficiency, normalized around the average (51.58%), scaled by 7.

PD Term: ((PD - (-13.8)) / (11 - (-13.8))) * 3
Scales possession control (-13.8 to 11) to 0–1, amplified by 3.

SOS Term: (SOS - 0.508) * 7 + 1
Normalizes schedule strength around the average (0.508), scaled by 7.

AAWM Term: AAWM / 20.5
Scales experience relative to a reference age (20.5).

Model 2 was also developed more with the assistance of Grok and to achieve a better R-squared value which it did, =.7287

## Model 3
I filled out a third bracket by just using everything I had learned during my research about the teams, and applying my human intution. This could be considered a control variable to see how well Grok does vs me the human, just extracting as much intuition from my work with Grok. Didn't stick to any sort of equation. 

## Results
Model 1 (Grok 1) 
Total Games - 38/63 (60.3%)
Sweet 16 - 7/16
Elite 8 - 4/8
Final 4 - 2/4
Final 2 - 1/2
Champ - 0/1

Model 2 (Grok 2) - 42/63 
Total Games - 42/63 (66.6%)
Sweet 16 - 12/16
Elite 8 - 4/8
Final 4 - 2/4
Final 2 - 0/2
Champ - 0/1
*Near Perfect East Bracket (14/15). Only missed the East Final (1 Duke v 2 Alabama).

Model 3 (Grok + Jeff) - 
Total Games - 38/63 (60.3%)
Sweet 16 - 8/16
Elite 8 - 4/8
Final 4 - 2/4
Final 2 - 1/2
Champ - 1/1 
* Correctly predicted Florida as National Champion.

In Summary, Model 2 was the best predictive model overall with a 66% accuracy, a solid start. It also predicted the East Region with almost perfect accuracy 14/15, only incorrectly predicting the East Region final 1 Duke v 2 Alabama. This could be in part however that there was only 1 upset in the entire region, 6 BYU over 3 Wisconsin, which we predicted. Model 3 (the human bracket), tied Model 1 (60.3%), however I accurately predicted the National Champion (Florida). This could indicate that when it comes to the final few games of the tournament, leaning away from the predicitve model and weighting human intution higher in these games may be sensible, as this is when the pressure is the highest and intangibles are most important. 


