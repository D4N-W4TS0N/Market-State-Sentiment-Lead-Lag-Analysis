# Macro-State-Sentiment-Lead
Investigates whether news sentiment changes lead latent macroeconomic state shifts, and by how much.

## Overview
Uses a collection of macroeconomic variables to infer latent (hidden) macro states/regimes at given points in time using a Hidden Markov Model. Each state is then manually labelled using the mean values for each variable for each state (e.g. - a state with an upward yield curve, steady/target inflation, low volatility, high economic activity would be labelled as 'steady expansion' or 'goldilocks'). Separately, historical news headlines are ingested and an LLM is used to classify the prevailing sentiment over time. Lead-lag analysis is then conducted to determine if the sentiment change preceded the corresponding regime shift, and by how much.

## Economic variables
**Volatility Index** (YFinance, VIX) - the most popular measure of the stock market's expected volatility on S&P 500 options. The most important variable, acting as a 'fear gauge', often being the first indicator to move after geopolitical tension or liquidity traps etc, certainly reacting before the data below.

**Initial Jobless Claims (US)** (FRED API, ICSA) - a weekly count of claims filed on the separation of an individual from an employer, for unemployment insurance. Rising count indicates weakening labour market, a declining economic health. NOTE - originally considered ISM services PMI, though data is not freely available, and is unrealised/sentimental in nature, where jobless claim count is a hard outcome.

**US Yearly CPI inflation** (FRED API, CPIAUCSL) - measures the inflation rate of the US basket of goods, obvious choice for determining market states (high inflation, tightening economy, falling risk apetite etc.)

**US 2year/10year spread** (FRED API, DGS2 and DGS10 difference) - the difference in yields of 2 and 10 year US treasuries. A positive difference would suggest steady future growth, with longer bonds costing more. A negative difference usually precedes a recession as markets predict falling long term interest rates. The spread is therefore more useful than the actual yields themselves in this case.

## Processing the variables
