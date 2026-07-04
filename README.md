# Market-State-Fed-Sentiment-Lead-Lag-Analysis

## Overview
Uses a collection of macroeconomic variables to infer latent (hidden) macro states/regimes at given points in time using a Hidden Markov Model. Each state is then manually labelled using the mean values for each variable for each state (e.g. - a state with an upward yield curve, steady/target inflation, low volatility, high economic activity, tightening labour market would be labelled as 'steady expansion' or 'goldilocks'). Separately, FOMC minutes and Fed Chair speeches are ingested and classified by narrative theme using an LLM, producing a time series of Federal Reserve communication sentiment. Lead-lag analysis is then conducted to determine whether shifts in Fed communication tone precede macro regime transitions, and by how much. The sentiment pipeline is scoped to 2021 onwards due to the availability of structured digital Fed communication archives.

## Economic variables
**Volatility Index** (YFinance, VIX) - the most popular measure of the stock market's expected volatility on S&P 500 options. The most important variable, acting as a 'fear gauge', often being the first indicator to move after geopolitical tension or liquidity traps etc, certainly reacting before the data below.

**Initial Jobless Claims (US)** (FRED API, ICSA) - a weekly count of claims filed on the separation of an individual from an employer, for unemployment insurance. Rising count indicates weakening labour market, a declining economic health. NOTE - originally considered ISM services PMI, though data is not freely available, and is unrealised/sentimental in nature, where jobless claim count is a hard outcome.

**US Yearly CPI inflation** (FRED API, CPIAUCSL) - measures the inflation rate of the US basket of goods, obvious choice for determining market states (high inflation, tightening economy, falling risk appetite etc.)

**US 2year/10year spread** (FRED API, DGS2 and DGS10 difference) - the difference in yields of 2 and 10 year US treasuries. A positive difference would suggest steady future growth, with longer bonds costing more. A negative difference usually precedes a recession as markets predict falling long term interest rates. The spread is therefore more useful than the actual yields themselves in this case.

## Processing the variables
The volatility index is a daily metric so will require no processing, whereas, CPI and Jobless Claims are monthly and weekly measures respectively, so will need forward filling to daily frequency; this is done from release date not strictly at the start of each month/week to avoid lookahead bias (e.g. January CPI data is released mid Feb, so the January data only enters from this point onwards). The yield spread requires the daily collection of yields, so the difference can be calculated; forward filling not necessary here.

Then all data is normalised using a rolling z-score. For each day the mean and standard deviation of the previous 252 days (standard trading period, might change later) is taken, and the value for that day is calculated with (z = (x - μ) / σ) where z is the normalised value, x the raw value, μ the mean, and σ the stdev. This value tells you how many stdevs the value is from the mean. This means all data is normalised, where before jobless claims are in thousands, CPI is in percentage, VIX up to 80 etc, z-scoring removes this. Also, since the focus is on detecting regime changes, data must be contextualised - for example: 'high volatility' carries less value than 'volatility higher than the 252 day previous rolling average'. On consideration, CPI inflation will use a 1260 day window, as 252 days is insufficient - persistent high inflation over 2 years is plausible, using a small window would completely hide this signal over time, hence a 5 year window is probably better. Also, as CPI data is collected monthly, a longer window is sensible to match the lower frequency. The yield spread may pose a similar problem, but changing to a longer window would blunt the signal from an inversion, taking much longer to react so is not a good idea. Also, yield spread has no fixed target (like inflation has 2%), so what counts as 'normal' can change with rate cycles.

## Fitting the HMM
The data is arranged where each column is a variable, and each row a date. The K value is then determined - which tells the model how many states to create; might use BIC to select K, or just pick (4-5 should work fine). The model then returns the emission paramters (mean and variance for each state), and the probabilities of moving between states. Then run Viterbi  to get the most probable state for each data - giving a time series of market states. I ended up trying BIC, it gave too high a number, dividing actual economic states into indistinct states; due to this I chose 4 as the K value to compromise between statistical fit and economic interpretability.

## Determining the states
The model simply returns state 0 or 1 (or 2 or 3) for each data - obviously offering no insight into what each state actually is. For each state, I'll manually inspect the parameters for each variable (mean indicates what state, variance indicates the reliability - if high then the model lacks confidence), aswell as cross check known state changes in history. The results are below.

<img width="616" height="180" alt="image" src="https://github.com/user-attachments/assets/45d72e11-5f57-4958-b4b9-b98b1491a6f2" />

**State 0 is recession/stress** - higher than average job losses, elevated volatility, steep yield curve, neutral/falling inflation, evidently prevalent in 2001, 2007-9.

**State 1 is inflation/tightening** - tight labour market, inverted yield curve, very high volatility, very high inflation, evidently prevalent in 2021-22.

**State 2 is late cycle/complacency** - healthy labour market, strongly inverted yield curve, falling/calm volatility, unchanging inflation, evidently prevalent in 2004 and 2023 and 2026 so far.

**State 3 is is early recovery/expansion** - strong labour market, steep upward curve, calming volatility, disinflationary, evidently prevalent in 2002-3, 2010, 2024-25.

Some concerns with the models output - 2024/2025 partially measured as crisis/recession when it was not; suggests state 1 may not be hard recession  but general stress. Also, state 3 does not distinguish genuine growth from post-crisis stabilisation. May signal the need for credit spreads as a variable to differentiate them.

## Fed language classification
I scrape the html links from the fomc calendar website, then extract the date and text content from each link. To keep token usage down, I'll pass only the text content after which the actual fed discussion starts, skipping the first section. This is then passed to the LLM, which is told to classify each fates fed discussion into one of the 4 narratives. The LLM is told to focus on relevant economic discussion, not the opening boilerplate, hence searching for phrases like 'members agreed', 'policy committee action'. The output of this is 4 scores for each date adding to 1 together, for each narrative on a given date:

<img width="1422" height="336" alt="image" src="https://github.com/user-attachments/assets/392f340a-e628-4d5f-9400-d4f256216d3b" />

The output appears plausible; recession/stress dominates discussions in early 2021, before shifting towards inflation/tightening as spending post covid hiked inflationary pressure.

## Lead-Lag analysis on Fed sentiment vs market state
The analysis will start by creating a list of dates for each regime shift detected by the HMM, ignoring those where there were multiple shifts within 2 weeks. For each of these dates, the last 2 fed meeting are observed, and the overall sentiment is noted. If the sentiment over the 2 discussions changed, the time period between the last meeting and the HMM detected shift is noted.

This approach only detected 2 qualifying transitions in fed sentiment before a regime shift, likely becuase inflation tightening sentiment dominated throughout the entire time period. Instead of looking at when the fed sentiment shifted completely, I'll try looking instead at when sentiment begun to strengthen - for example when a certain theme began increasing - not when it finally took over.

<img width="1316" height="684" alt="image" src="https://github.com/user-attachments/assets/0755c1ef-b911-4619-a27b-bfa9cbbb05d5" />

NOTE: transition_date refers to the macro state change, first_rise_date refers to when Fed sentiment first begins to change.
Across the observed period, Fed sentiment BEGINNING to change generally preceded the HMM detected regime change by a number of months (averaging 215 days). The shortest lead was 75 days in the early 2023 shift to recession/stress after the SVB crash. The longest lead was after the Trump tariffs in 2025, where Fed sentiment change preceded the observed shift in market state by 302 days, reflecting the very gradual transmission of inflation expectations into real macro data. (No causal claim can be made here though, as the market shift came after the Hormuz crisis, so it cannot be determined that the fed shift and the macro shift are related here).

## Limitations
The data used to detect market states is inherently backward looking in nature, measuring actual economic outcomes, whereas the Fed discussions are forward looking - focusing on where members think the economy is going (not where it is). As a result, the wide gap between Fed language sentiment shifting, and the corresponding shift in all 4 economic variables to qualify a regime change is expected. It is also worth noting that the macro states are infered latent themes based on each states average characteristics, NOT objectively defined regimes. 14 transitions over the period of 43 fed disucssions is much too small a dataset for any real inference, a longer time series is likely needed.

If this relationship proves robust over longer samples, changes in Fed communication could provide an earlier, qualitative indication of changing macroeconomic states, than realised macroeconomic data alone.

