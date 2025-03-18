# March-Madness-Model
Statistical Analysis project to create a near perfect bracket. I'm utilizing xAI Grok 3 beta, Microsoft Excel 365, Python and Power BI to attempt to create a (near) perfect bracket. Each year I plan to iterate on a our composite "Grok Score" to improve the performance of the predictive model for the NCAA Men's Basketball Tournament. The foundational Key Indicators (KPIs) I'm focusing on to start are; Possession Differential (PD), Effective Field Goal Percentage (EFG%), Strength of Schedule (SOS) and Average Age Weighted by Minutes (AAWM). 

## Possession Differential (PD)
How many more possessions do you get than your opponent on average? Every possession matters in a single elimination tournament basketball. Scores on average are lower in this competition format, two teams facing elimination become more risk averse, every possession counts. To get an approximation of the possession battle (with fouls/ refs constant), we're simply using a team's season average Rebound Differential (RD) and their season average Turnover Differential (TD). RD + TD = PD. 

## Effective Field Goal Percentage (EFG%)
EFG weights three pointers more, and is a measure of shooting efficiency. EFG % = (FGM + 0.5 x 3PM) / FGA. Note: Next year we'll include Free Throws Made (FTM) to get an even more accurate indicator of shooting efficiency. This also would account for points scored off fouls drawn, which is not currently accounted for in this model. 

## Strength of Schedule (SOS)
SOS is included in RPI, which is the primary metric for determind Rank. We want our model to be correlated to Rank (RPI), but not perfectly, otherwise we would never predict any upsets. So in this model we'll just focus simply on SOS. For our purposes, SOS = (Opponent's Winning % / # of Games ). We want to use this to be able to discount those teams to some degree that achieve a high rating in PD and EFG%, because of a lack of competition. 

## Average Age Weighted by Minutes (AAWM)
AAWM is a proxy for experience and maturity essentially. The NIL era has ushered in an older generation of college players, which has increased the disparity between ages on the court, in some cases you have 18 year olds playing against 23 or 24 years olds who still have eligibility. This is a crucial factor, especially in terms of motivation (someone playing maybe the last game of their career vs someone just trying not to get hurt before the NBA draft). There is a counter intutive argument however that the older players in NCAA aren't as good, otherwise they would be playing pro already. In any case, we believe it is still a factor, albeit not as significant as PD, EFG or SOS. 

