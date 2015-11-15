# R语言中的泊松回归

泊松回归用来对计数型反应变量或者列联表进行回归分析，是在广义线性模型中除逻辑回归外最常用的回归分析模型。泊松回归假设反应变量服从泊松分布，反应变量的期望值的对数可以用线性回归分析（即连接函数为对数函数），泊松回归也被称为对数线性回归。

一、泊松分布

泊松分布由二项分布衍伸而来，适合描述某种事件在一定期间内发生的次数。比如，病人在一周内发病次数，工厂一月内劣质产品生产个数，排队论中一段时间内到达顾客的数量等。

泊松分布的概率质量函数为

P(X = k) = \frac{e^{-\lambda }\lambda ^{k}}{k!}
其中\lambda = E(X) = Var(X)

在R中，用dpois()计算该概率。比如，如果一周内买胡萝卜的兔子平均数量为3（\lambda = 3），那么一周内买胡萝卜的兔子数量为k(k=0,1,2,3,…,10)的概率分别为：

for (k in 0:10) {

    rabbit[k + 1] <- round(dpois(x = k, lambda = 3), 2)
}

rabbit


 [1] 0.05 0.15 0.22 0.22 0.17 0.10 0.05 0.02 0.01 0.00 0.00
 
在上面的例子中，其实还隐含了几个假设。只有在这四个假设条件下，用来计数的随机变量才服从泊松分布。（参考pmean)

1.一小段时间内事件发生的概率与该段时间的长短呈比例(proportional)

2.在一个极小时间段内事件发生两次以上的概率可以忽略(negligible)

3.无论时间先后，事件在一段给定时间内发生的概率稳定(stable)
4.非重叠时间段内事件发生的概率相互独立(independent)

二、泊松回归

当反应变量服从泊松分布时，用OLS线性回归得到的结果不佳，因为

1.记数变量>=0,而OLS不能保证这一点

2.反应变量和自变量之间的关系非线性

R中用glm()函数进行泊松回归，￼￼具体形式为：

glm(Y ~ X1 + X2, family = poisson(link = "log"), data = dataframe)

实际是以log()为连接函数的对数线性回归：
log(Y) =α +β X

下面以Princeton University提供的数据集The Children Ever Born Data为例。读者也可在以上网址找到其他根据统计分析类型（线性回归、逻辑回归、泊松回归、指数线性回归、生存分析）分类的其他数据集。

ceb <- read.table("http://data.princeton.edu/wws509/datasets/ceb.dat")
names(ceb)


 [1] "dur"  "res"  "educ" "mean" "var"  "n"    "y"
 
变量说明如下：

The cell number (1 to 71, cell 68 has no observations),

“dur” = marriage duration (1=0-4, 2=5-9, 3=10-14, 4=15-19, 5=20-24, 6=25-29),

“res” = residence (1=Suva, 2=Urban, 3=Rural),

“educ” = education (1=none, 2=lower primary, 3=upper

primary, 4=secondary+),

“mean” = mean number of children ever born (e.g. 0.50),

“var” = variance of children ever born (e.g. 1.14), and

“n” = number of women in the cell (e.g. 8),

“y” = number of children ever born.

给响应变量育子数做直方图，可以清楚看到其偏倚度。

hist(ceb$y, breaks = 50, xlab = "children ever born", main = "Distribution of CEB")
plot of chunk unnamed-chunk-1

下面对其做泊松回归，并输出结果。

fit <- glm(y ~ educ + res + dur, offset = log(n), family = poisson(), data = ceb)
summary(fit)



 Call:
 
 glm(formula = y ~ educ + res + dur, family = poisson(),
 data = ceb, 
     offset = log(n))
 
 Deviance Residuals: 
    Min      1Q  Median      3Q     Max  
 -2.291  -0.665   0.076   0.661   3.679  
 
 Coefficients:
             Estimate Std. Error z value Pr(>|z|)    
 (Intercept)   0.0570     0.0480    1.19     0.24    
 educnone     -0.0231     0.0227   -1.02     0.31    
 educsec+     -0.3327     0.0539   -6.17  6.7e-10 ***
 educupper    -0.1247     0.0300   -4.16  3.2e-05 ***
 resSuva      -0.1512     0.0283   -5.34  9.4e-08 ***
 resurban     -0.0390     0.0246   -1.58     0.11    
 dur10-14      1.3705     0.0511   26.83  < 2e-16 ***
 dur15-19      1.6142     0.0512   31.52  < 2e-16 ***
 dur20-24      1.7855     0.0512   34.86  < 2e-16 ***
 dur25-29      1.9768     0.0500   39.50  < 2e-16 ***
 dur5-9        0.9977     0.0528   18.91  < 2e-16 ***
 ---
 Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
 
 (Dispersion parameter for poisson family taken to be 1)
 
     Null deviance: 3731.525  on 69  degrees of freedom
 Residual deviance:   70.653  on 59  degrees of freedom AIC: Inf
 
 Number of Fisher Scoring iterations: 4
为了更好地解释模型参数，将其指数化。

exp(coef(fit))


 (Intercept)    educnone    educsec+   educupper     resSuva    resurban 
      1.0586      0.9772      0.7170      0.8827      0.8597      0.9618 
    dur10-14    dur15-19    dur20-24    dur25-29      dur5-9 
      3.9374      5.0240      5.9625      7.2196      2.7119
可见随着婚龄的增长，期望的育子数将相应增长；教育程度越高，期望育子数越低；农村预期育子数比城市高等。

泊松回归中需要注意过度离势问题。泊松分布中均值与方差相等，当观测到的响应变量实际分布不满足这一点时，泊松回归可能会出现这样的问题。这个问题一般原因是缺少解释变量。我们可以用qcc包对泊松模型检验过度离势。

require(qcc)
qcc.overdispersion.test(ceb$y, type = "poisson")


                    
 Overdispersion test Obs.Var/Theor.Var Statistic p-value
        poisson data               323     22284       0
p值为0，果然该数据存在过度离势的情况，可以用类泊松模型对数据进行分析。

fit2 <- glm(y ~ educ + res + dur, offset = log(n), family = quasipoisson(), data = ceb)
summary(fit2)


 
 Call:
 glm(formula = y ~ educ + res + dur, family = quasipoisson(), 
     data = ceb, offset = log(n))
 
 Deviance Residuals: 
    Min      1Q  Median      3Q     Max  
 -2.291  -0.665   0.076   0.661   3.679  
 
 Coefficients:
             Estimate Std. Error t value Pr(>|t|)    
 (Intercept)   0.0570     0.0529    1.08  0.28596    
 educnone     -0.0231     0.0249   -0.93  0.35854    
 educsec+     -0.3327     0.0593   -5.61  5.7e-07 ***
 educupper    -0.1247     0.0330   -3.78  0.00037 ***
 resSuva      -0.1512     0.0312   -4.85  9.4e-06 ***
 resurban     -0.0390     0.0271   -1.44  0.15597    
 dur10-14      1.3705     0.0562   24.37  < 2e-16 ***
 dur15-19      1.6142     0.0564   28.64  < 2e-16 ***
 dur20-24      1.7855     0.0564   31.66  < 2e-16 ***
 dur25-29      1.9768     0.0551   35.88  < 2e-16 ***
 dur5-9        0.9977     0.0581   17.18  < 2e-16 ***
 ---
 Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
 
 (Dispersion parameter for quasipoisson family taken to be 1.212)
 
 Null deviance: 3731.525  on 69  degrees of freedom
 
 Residual deviance:   70.653  on 59  degrees of freedom
 AIC: NA
 
 Number of Fisher Scoring iterations: 4
 
比较以上两个模型参数结果，发现参数估计值一致，而t值/p值不同。在过度离势的情况下，应采用类泊松结果的t值/p值检验自变量的显著程度。