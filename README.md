# Macro-State-Sentiment-Lead
Investigates whether news sentiment changes lead latent macroeconomic state shifts, and by how much.

## Overview
Uses a collection of macroeconomic variables to infer latent (hidden) macro states/regimes at given points in time using a Hidden Markov Model. Each state is then manually labelled using the mean values for each variable for each state (e.g. - a state with an upward yield curve, steady/target inflation, low volatility, high economic activity would be labelled as 'steady expansion' or 'goldilocks'). Separately, historical news headlines are ingested and an LLM is used to classify the prevailing sentiment over time. Lead-lag analysis is then conducted to determine if the sentiment change preceded the corresponding regime shift, and by how much.

## Economic variables
**Volatility Index (YFinance, VIX)** - the most popular measure of the stock market's expected volatility on S&P 500 options. The most important variable, acting as a 'fear gauge', often being the first indicator to move after geopolitical tension or liquidity traps etc, certainly moving before the data below.

**ISM Services PMI** - measures forecast economic activity of the US services sector through supply chain surveys. Disctint from GDP which measures past output, this measures future sentiment. Services sector preferred here due to larger share of US economy (~70%). NOTE - a relatively young index, restricting the project to 1997 onwards)

**US Yearly CPI inflation** - measures the inflation rate of the US basket of goods, obvious choice for determining market states.

**US 2year/10year spread** - the difference in yields of 2 and 10 year US treasuries. A positive difference would suggest a calmer macro climate, with longer bonds costing more. A negative difference usually precedes a recession as markets predict falling long term interest rates. The spread is therefore more useful than the actual yields themselves in this case.

## Processing the variables
