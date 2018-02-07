# Basic Time-Series Analysis: The Cointegration and the VECM Model Explained




*This post is the third in a series explaining [Basic Time Series Analysis](http://blog.mindymallory.com/2018/01/basic-time-series-analysis-the-game/). Click the link to check out the first post which focused on stationarity versus non-stationarity, and to find a list of other topics covered. As a reminder, this post is intended to be a very applied example of how use certain tests and models in a time-sereis analysis, either to get someone started learning about time-series techniques or to provide a big-picture perspective to someone taking a formal time-series class where the stats are coming fast and furious. As in the first post, the code producing these examples is provided for those who want to follow along in R. If you aren't into R, just ignore the code blocks and the intuition will follow.* 

My colleague [\@TKuethe](https://twitter.com/TKuethe) showed me this awesome old paper by Michael Murray, [A Drunk and Her Dog: An Illusration of Cointegration and Error Correction](http://www.tandfonline.com/doi/abs/10.1080/00031305.1994.10476017).[^link] Then I found this extension by Aaron Smith and Robin Harrison, [A Drunk, Her Dog, and  Boyfiend: An Illusration of Multiple Cointegration and Error Correction](https://pdfs.semanticscholar.org/2ab1/0ac4252f50f9f0b7b6f22a1b5c7655c6eb40.pdf). The premise of both these papers is that if you follow a drunk out of the bar after a night of drinking, you will observe that their path looks much like a *random walk* - non-stationary. Also, if you have ever watched a dog freely exlporing, their path also looks much like a random walk. The dog will go this way and that, whever it's nose leads it. If however, the dog belongs to the drunk, and the drunk calls periodically for the dog, the dog will stay pretty close to the drunk. So the drunk and her dog go forth wandering aimlessly, together. 

This is exactly concept of cointegration, two (or more) series wandering aimlessly, together (at least never very far apart.)

In this blog post we cover cointeration and most common model for cointegrated time-series, the Vector Error Correction Model (VECM). The most intuitive cases (besides the drunk and her dog) are markets that are related by a production process, like the soybean complex. Soybean crushers buy soybeans and sell meal and oil. Economic theory would suggest the prices of those three commodities to maintain a relationship so that the profitability of soybean crushers trends around some modest number greater than zero. If there were persistant really high profits in the soybean crushing business, more soybean crushing capacity would be built, driving soybean prices higher and meal and oil prices lower. 

This can also come up in the stock market, especially if two companies are destined for similar fates. If you've ever read a trading book, trading this situation is sometimes called pairs trading. In commodities, it would be called spread trading. The idea is you look for two stocks (or commodity contracts) that are cointegrated (usually move together). Then you wait for a day when one goes up and one goes down. You bet that because they are cointegrated this deviation won't last long, so you sell the one whose price went up and buy the one whose price went down. Then, no matter if the prices go up or down generally, as long as the two prices come back in line, you can make money.  

[^link]: I linked the actual journal article, which is gated, but if you google the title you can find the pdf openly available. 

We will go through specific examples of statistical tests for cointegration and many of the things you might do to analyze a set of cointegrated series including: 

+ Johansen Cointegration Tests
+ Fitting and Interpreting a VECM Model
+ Granger Causality of Cointegrated Series
+ 

We will continue our examples using SPY (the S&P 500 exchange traded fund) and GS (Goldman Sachs) prices. 


```r
# If you are following along, uncomment the next lines and run once to install the required packages 
# install.packages('ggplot2')
# install.packages('xts')
# install.packages('quantmod')
# install.packages('broom')
# install.packages('tseries')
# install.packages("kableExtra")
# install.packages("knitr")
# install.packages("vars")

library(quantmod)
getSymbols(c('SPY', 'GS'))
```

```
## [1] "SPY" "GS"
```

```r
SPY                <- SPY$SPY.Adjusted
GS                 <- GS$GS.Adjusted
time_series           <- cbind(SPY, GS)
colnames(time_series) <- c('SPY', 'GS') 
```

# Vector Error Correction Model 

A VECM model adds a single term to the VAR in returns model that we learned in the last post. It adds an $\alpha \beta SPY_{t-1}$ and $\alpha \beta GS_{t-1}$ term to each equation. Using SPY and GS prices, it looks like this.  

\begin{align}
\Delta SPY_t &= \gamma^{spy}_0 + \alpha_1 (\beta_1 SPY_{t-1} + \beta_2 GS_{t-1}) + \gamma^{spy}_1 \Delta SPY_{t-1} + \gamma^{spy}_2 \Delta SPY_{t-2} + \gamma^{spy}_3 \Delta GS_{t-1} + \gamma^{spy}_4 \Delta GS_{t-2} + \nu_{spy} \\
\Delta GS_t  &= \gamma^{gs}_0  + \alpha_2 (\beta_1 SPY_{t-1} + \beta_2 GS_{t-1}) + \gamma^{gs}_1 \Delta SPY_{t-1}  + \gamma^{gs}_2 \Delta SPY_{t-2}  + \gamma^{gs}_3 \Delta GS_{t-1}  + \gamma^{gs}_4 \Delta GS_{t-2}  + \nu_{gs} 
\end{align}

The $\gamma$'s are regression coeficients on the lagged SPY and GS returns, just like the $\beta$'s from our VAR blog post. The $\alpha_1 (\beta_1 SPY_{t-1} + \beta_2 GS_{t-1})$, and $\alpha_2 (\beta_1 SPY_{t-1} + \beta_2 GS_{t-1})$ terms are called error correction terms. 

It's at about this point where you may start appreciating why people say you should take linear algebra if you want to do econometrics. This is really messy looking and it is hard to really understand what is going on, what the heck those error correction terms are, and how on earth they might be useful. 

Rewriting this in matrix form makes it much neater and aid understanding. If you've had linear algebra, make sure you can reproduce this next step; it will help your understanding of the working parts. If you haven't yet had linear algebra, consider signing up next semester! and try to understand the explanation of what the $\alpha$'s and $\beta$'s are.  

\begin{align}
\left[
\begin{array}
  SSPY_t \\
 GS_t 
\end{array} 
\right]

&= 

\left[
\begin{array}
  s\gamma^{spy}_0 \\
 \gamma^{gs}_0 
\end{array} 
\right]

+ 

\left[
\begin{array}
  s\alpha_1 & \alpha_2
\end{array} 
\right]

\left[
\begin{array}
  s\beta_1 \\
  \beta_2
\end{array} 
\right]

\left[
\begin{array}
  sSPY_{t-1} \\
  GS_{t-1}
\end{array} 
\right]

\end{align}
## The Johansen Test for Cointegration



# That's It! 

That covers the basics of what a VECM model is and how to analyze how a group of markets move around an equilibrium relationship. 

The final post in this series will summarize the series and provide a 'cook-book' like decision tree on how to decide which model is appropriate for your case. 

# References

Murray, M. P. (1994). A drunk and her dog: an illustration of cointegration and error correction. *The American Statistician*, 48(1), 37-39.




























