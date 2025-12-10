# Stan for Bayesian Meta-Analysis




## Overview

This repository contains my Stan and R code for performing **Bayesian meta-analyses** (**hierarchical models**). Meta-analysis is a statistical method that combines results from multiple studies to estimate an overall effect. In a **random effects meta-analysis**, we assume that the true effect size may vary between studies due to differences in study populations or methods; this model accounts for such variability. A **network meta-analysis**, or **indirect treatment comparisons**, extends this approach to compare a network of multiple treatments across studies, allowing for both direct and indirect comparisons between treatments.

Currently, the repository includes code for a random effects model and a random effects network meta-analysis (indirect treatment comparisons). The random effects model uses data from a study on the effects of green tea on weight loss [^1], collected by [^2]. The network meta-analysis uses data on the effectiveness of different cognitive behavioral therapies for depression, as studied by [^3]. The data is formatted in the R package *dmetar* by [^4].




<br>

## Random Effects Model

We use a random effects meta-analysis to estimate the overall effect size across multiple studies. This approach accounts for variability in true effect sizes between studies, meaning that differences in study populations or methodologies may lead to heterogeneity. We only focus on the random effects model here, as the common effect model is just a special case of the random effects model when there is no heterogeneity (i.e., the between-study standard deviation $\tau$ is zero).

Suppose we have $N$ studies ($i = 1, 2, ..., N$), each reporting an observed effect $\hat{\theta}_i$ and its standard error $SE_i$. The model assumes:

$${\hat{\theta}_i} \sim Normal(\theta_i, \sigma_i^2)$$

where $\sigma_i = SE_i$ is the known standard error for study $i$.

<br>

The (underlying) true effect for each study, $\theta_i$, is modeled as:

$${\theta_i} \sim Normal(\mu, \tau^2)$$

, where $\mu$ represents the overall true effect across studies, and $\tau$ is the between-study standard deviation, capturing heterogeneity.

<br>

The priors for the parameters are:

$$\mu \sim Normal(0, 1)$$
$$\tau \sim HalfCauchy(0, 0.5)$$



<br>

We use data from a meta-analysis of green tea's effects on weight loss [^1], which includes mean differences in weight loss (in kg) between green tea and a control group, along with their standard errors, from 12 studies. The script [ma_02_fit_model.stan](./code/ma_02_fit_model.stan) fits the model and generates posterior samples for $\mu$ (the overall effect) and $\tau$ (heterogeneity), as well as study-specific effects $\theta_i$, using Markov Chain Monte Carlo (MCMC) methods.

These results can be visualized using a forest plot, which displays the estimated effects and their credible intervals for each study (see the script [ma_04_plot_forest.R](./code/ma_04_plot_forest.R)):

<p align="center">
<img src="./figures/forest_plot_ma_re.png" alt="Forest Plot Random Effects" width="75%">
</p>


<br>

A posterior predictive distribution of the mean difference, using 20 samples from the posterior distribution, can also be generated to visualize the uncertainty in the overall effect estimate (see the script [ma_05_plot_posterior_predictive.R](./code/ma_05_plot_posterior_predictive.R)):

<p align="center">
<img src="./figures/weight_loss_effect_re.png" alt="Posterior Predictive Plot Random Effects" width="75%">
</p>




<br>

## Network Meta-Analysis

To estimate the effects of multiple treatments, as compared to an overall baseline, we can use a network meta-analysis to synthesize evidence from a network of treatments across studies. This method is also called indirect treatment comparisons. The scripts for the following network meta-analysis are in [nma_02_fit_model.R](./code/nma_02_fit_model.R) and [nma_02_fit_model.stan](./code/nma_02_fit_model.stan).

Let's say we have $N$ studies ($i = 1, 2, ..., N$), each comparing two or more treatments from a set of $K$ treatments ($k = 1, 2, ..., K$). Each study compares treatment(s) $k$ to a study-specific baseline treatment $b_i$ (the baseline treatment may differ between studies) and observes a study-specific effect ${\hat \theta_{i, \space b_{i} k}}$ with its underlying true value $\theta_{i, \space b_{i} k}$.

Given the observed effect sizes ${\hat \theta_{i, \space b_{i} k}}$ and their standard errors $SE_{i, \space b_{i} k}$ from all pairwise comparisons across studies, we estimate the overall true effects between treatments and a common baseline $\theta_{b k}$ :

$${\hat \theta_{i, \space b_{i} k}} \sim Normal(\theta_{i, \space b_{i} k}, \space \sigma_{i, \space b_{i} k}^2 )$$

<br>

$$
\theta_{i, \space b_{i} k} \sim
\begin{cases}
Normal(\theta_{b k}, \space \tau^2), & \text{for} \space b_i = b \\
Normal(\theta_{b k} - \theta_{b b_{i}}, \space \tau^2), & \text{for} \space b_i \neq b
\end{cases}
$$

<br>

$$\space \sigma_{i, \space b_{i} k}  = SE_{i, \space b_{i} k} $$


<br>

with the following priors:

$$\theta_{bk} \sim Normal(0, 10^2)$$

$$\tau \sim HalfCauchy(0, 0.5)$$




<br>

In the demonstration below, we use data from a network meta-analysis of different cognitive behavioral therapy (CBT) formats for treating depression [^3]. There are 182 studies in which 181 of them have a pairwise comparison between two treatments, and only one study has pairwise comparisons between all three treatments. This gives us a total of 184 pairwise comparisons across all studies (see [nma_01_load_data.R](./code/nma_01_load_data.R) for data description). 

There are 7 treatments in total, including the baseline treatment *Care As Usual*:

- Care As Usual (baseline)
- Group
- Guided Self-Help
- Individual
- Telephone
- Unguided Self-Help
- Waitlist


<br>

The following network produced by [nma_05_plot_network.R](./code/nma_05_plot_network.R) shows the pairwise comparisons between treatments across all studies. The thickness of the edge represents the count of pairwise comparisons between treatments.

<p align="center">
<img src="./figures/network.png" alt="Network of Pairwise Comparisons" width="50%">
<p>




<br>

After fitting the model, we obtain the following estimates of the true effects for each treatment compared to *Care As Usual*, along with their 95% credible intervals:

<p align="center">
<img src="./figures/forest_plot_nma.png" alt="Treatment Effects" width="80%">
<p>


<br>

The trace plots of the posterior samples are shown below, indicating good mixing and convergence of the Markov Chain Monte Carlo (MCMC) chains:

<p align="center">
<img src="./figures/trace_plot.png" alt="Trace Plot" width="60%">
<p>




<br>

Once we have the posterior samples for the true effects of each treatment as compared to the baseline, we can obtain the mean difference between any two treatments, along with their credible intervals, in our network. Specifically, while the estimated $\theta_{b k}$, ($b = 1$, and $k = 2, 3, ..., 6$) gives us only 6 effects, we can derive the mean effects between all pairs of treatments using the following relationship:

$$\theta_{k_1 k_2} = \theta_{b k_2} - \theta_{b k_1},$$

whether $k_1$ or $k_2$ is the baseline treatment or not. See the last part of [nma_03_analyze.R](./code/nma_03_analyze.R) for details.


<br>

The table below shows the mean effects between all pairs of treatments:

<p align="center">
<img src="./figures/table_network_effects.png" alt="Table of Network Effects" width="100%">
<p>




<br>

## References
[^1]: Jurgens TM, Whelan AM, Killian L, Doucette S, Kirk S, Foy E. Green tea for weight loss and weight maintenance in overweight or obese adults. *Cochrane Database of Systematic Reviews 2012, Issue 12*.

[^2]: Grant, R., & Di Tanna, G. L. (2025). *Bayesian meta-analysis: a practical introduction*. CRC Press.

[^3]: Cuijpers, P., Noma, H., Karyotaki, E., Cipriani, A., & Furukawa, T. A. (2019). Effectiveness and acceptability of cognitive behavior therapy delivery formats in adults with depression: a network meta-analysis. *JAMA psychiatry, 76*(7), 700-707.

[^4]: Harrer, M., Cuijpers, P., Furukawa, T.A., & Ebert, D.D. (2021). *Doing Meta-Analysis with R: A Hands-On Guide*. Boca Raton, FL and London: Chapman & Hall/CRC Press. ISBN 978-0-367-61007-4.