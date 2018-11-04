---
title: What we talk about when we talk about Hedging
date: 2017-02-28T00:14:19-07:00
showDate: true
tags: ["Work"]
---

After a 4-hour meeting with Joel, I finally understood the basic ideas about how the so called hedging works. Although I'm not going to do a data scientist's work, however, in review of the automatic trading platform I'm about to build, it's supposed that I get a rough idea about the mechanisms of a hedge fund. So here are some of my memos. 

<!--more-->

## Short & Long

Suppose that you are a very clever guy, how can you make a profit after knowing the stock of Apple(AAPL) is going to fall, say 5%, in the next 3 days? That's about short.

The solution is easy. Suppose the price of AAPL is 100 dollars today and \$90 three days after. As a hedge fund, you can "borrow" 10 AAPL stocks today, which costs you \$1000. Then, you sell them, which results in a temporary 1000 dollars' profit. After 3 days, you buy 10 AAPL stocks from the market using 900 dollars. And "return" this 10 stocks to whom you borrowed the stocks. All in all, you make a profit of \$100. 

That's called a "SHORT". Using common sense, you can understand what is LONG, which is exact the opposite of SHORT.

## SPY & Risk Hedging

> To believe in something and not to live it, is dishonest. (Gandhi)

We believe in the fact that there are three things serve as the main determining factors for the price of a stock:

1. The performance and the set of subjective factors of the company, which is called $\alpha$;
2. The environment of the stock market;
3. The strength of association between the stock and the market, which is called $\beta$.

Then, suppose P(x) represents the price of stock x, we can say:

$$P(x) = \beta*P(SPY) + \alpha$$

SPY(or SPDR-ETFs, Standard & Poor's depository receipt exchange trade funds) is a special stock: It can be considered as a fund who makes a profit by investing 500 different companies in US. Therefore, the price of its stock can be considered as a rough indicator for stock market.

For $\beta$, it reveals the relationship between the particular stock and the stock market. If $\beta > 0$, we can say that they are positively related; otherwise, the correlation is negative. 

For $\alpha$, it indicates the rise and fall of this particular stock. If $\alpha < 0$, the price of stock is very likely to decrease, in this context, we should apply our "short" policy; otherwise, we buy "long".

After believing this, we are going to use SPY to fulfill the Risk Hedging. First , we suppose

$$P(AAPL) = 2P(SPY)$$

​There is no $\alpha$ here, so let's just buy in 100 Apple stock and in the mean time, sell 2*100 Spy stock. If the price of SPY is x per stock, then the price of AAPL is 2x. Because of (- 100\*2x) + (2\*100 \* x) = 0, we will neither win or lose. This is how hedging avoids the risk. So how can we make profits?

Then we suppose $$P(AAPL) = 2(SPY) - 5\%$$. We can see that $-5\% < 0$, which tells us to "short" (sell) AAPL.  

| stock | price                  | amount        |
| :---- | :--------------------- | :------------ |
| AAPL  | $\beta$ * x + $\alpha$ | 100           |
| SPY   | x                      | 100 * $\beta$ |

This is a use case when we know $\alpha$. The amount of apple stock we sell is 100. As a hedging operation, we buy in (100\*$\beta$)SPY stocks. In this case, the profit we make is 

$$-100*(\beta * x + \alpha) + 100 * \beta * x = -100 \alpha$$

From the cases above, we are able to have a general idea about how we use SPY in risk hedging.

## My Morning Jobs

So what am I supposed to do? In the morning, I'll receive a form(maybe excel from Bloomberg Terminal), something looks like:

| stock | $\beta$ | suggestion |
| :---: | :-----: | :--------: |
| AAPL  |   1.5   |   short    |
| TSLA  |   -2    |   short    |
| MSFT  |    3    |    long    |
|  GM   |   1.4   |    long    |

Our CRO Bloch will tell me something like, today we have \$1000 to buy stocks, the total prices of each stock are equal. Of course we are going to buy SPY for hedging, so let your machine do the automatic trading work. 

Take TSLA for an instance, the trading suggestion is "short". So we are going to sell \$x of its stock. In the mean time, we have to buy $\beta*x = -2x$ dollars of SPY for hedging, which actually means we should sell $2x$ dollars of SPY. Then we can fill in the form:

| stock | $\beta$ | suggestion | SPY(+buy/-sell) |
| :---: | :-----: | :--------: | :-------------: |
| AAPL  |   1.5   |   short    |      +1.5x      |
| TSLA  |   -2    |   short    |       -2x       |
| MSFT  |    3    |    long    |       -3x       |
|  GM   |   1.4   |    long    |      -1.4x      |

​In total, the amount of SPY we are going to buy/sell is $$1.5x - 2x - 3x - 1.4x = -4.9x$$. Because we have to buy \$x of each these 4 stocks. Our 1000 dollars this morning will be separated into $4.9 + 4 = 8.9$ parts. Therefore, we buy or sell (1/8.9 \* 1000) dollars of AAPL/TSLA/MSFT/GM. And we sell (4.9/8.9 \* 1000) dollars of SPY. 

Using TWS (Trade Workstation) API, I can find out the VWAP or mkt price of each stock, then the trading platform I build will calculate the number of each stock to be operated and do the correspond buy/sell jobs. 

## Portfolio  rebalance & 3-day  trading

The further step is about how to apply the methods to calculate number of each stock to be traded on a regular 3-day-basis.

Because we are going to do the trading on a three-day basis, it's natural to think of dividing the investment for one 3-day period into three equal parts and using each part to buy/sell stocks every day.

Nevertheless, there are occasions when we lose a lot of investment on the first day and therefore, we have to do correspond adjustment on the second and third day. For example, if we have \$999 in the beginning, the investment will be divided into three \$333 for each day. 

Unfortunately, the first \$333 we used to buy AMZN in day 1 became only 1 dollar after the first day and before the opening of the second day. Suppose we have rebalance triggers which will be trigged say, when the fortune we have become less than 25% or more 140% of the initial investment that day, one trigger will be trigged on the second day for rebalance. 

The so-called rebalance is: For now, the total investment is (1 + 666) dollars, what we want is to keep the share for the first-day investment be 1/3 of the total investment. Hence, we have to re-invest 667/3 dollars using the form in day-one to calculate how many stocks(same kind as in day-one) we are going to buy/sell in the second day. 

After that, we will calculate the difference between the new result and the result in the first day. The result of the difference is what we should do on the second trading day. e.g.

| day 1   | new result | difference          |
| ------- | ---------- | ------------------- |
| -1 AAPL | -3 AAPL    | short 2 AAPL stocks |

Of course, the investment we have for the form from the second day becomes 667/3 dollars.



