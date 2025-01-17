# Assignment 2
_Jakob, Stine, Jan, Frida_ 

Code for assignment 2 in Advanced Cognitive Modeling, MSc Cognitive Science, Aarhus University.

------------
For this assignment, we are exploring the modeling of a strategy for playing the Matching Pennies Game. The overall purpose is to conduct a thorough prior robustness check and precision analysis on our proposed model using simulated data in order to ensure that the model is able to recover parameters meaningfully when adjusting priors and parameters appropriately.

## Analysis

### Modeling

__Strategy__:  
We have used FeedbackAgent defined in [assignment 1](https://github.com/CognitiveScienceAU/assignment1_fromverbal2formal-fridajakobjanstine) where the formalization (i.e., code) of the agent can also be found. 
This agent assumes that the opponent is a "human agent" who follows the standard human behavior of switching too often. It assumes that the bias for switching is 70%. As it plays as the matcher, its strategy is therefore that when it wins, 
It assumes that the opponent will be more likely to switch, and is therefore more likely to switch itself. Likewise, when it loses, it assumes the opponent is more likely to switch and is self more likely to stay. The strategy can be thought of as "win 70% switch, loose 70% stay". The assumed bias of switching 70% of the time was arbitrarily chosen. 

__Bayesian model__:  
The data we are using are simulated from these assumptions. We simulate a game between the FeedbackAgent described above and a completely random agent. They play for 10000 rounds. We then make two new columns, win_bias and lose_bias, respectively. win_bias has the value 1 (heads) if the Feedback Agent AND the random agent chose 0 (tails) in the present round, -1 if both agents chose 1 (heads) in the present round, and otherwise 0. This means that this column indicates pushing towards shifting when the two agents chose the same (meaning that Feedback Agent as the Matcher won). Likewise, lose_bias has value 1 when Feedback Agent chose 1 (heads) and random agent chose 0 (meaning that Feedback agent lost), -1 when Feedback agent chose 0 (tail) and random agents chose 1, and otherwise 0. 
We then lagged those two columns, so that one row of the data contain information of the bias given the previous round and the choice given this bias. 

The data as described above was used to model the following: 

choice ~ alpha + win_beta * win_bias + lose_beta * lose_bias

Here, _choice_ is the choice of the Feedback agent at the present round, alpha is a parameter modeling systematic noise, win_beta is the to-be estimated parameter for the win_bias, and lose_beta is the to-be estimated parameter for the lose_bias. Note, that due to the way the two columns were created, either one or the other will always be zero. If Feedback Agent won the previous round, then lose_bias is zero (because the agent didn't lose) and vice versa. 

### Priors and true values
Parameters to estimate:
- alpha: noise (since we added no noise in the model, we expect it to be close to zero)
- win_beta: true value is 0.7
- lose_beta: true value is 0.7

All parameters were modeled with normally distributed priors. For each parameter, we tested three different priors (varying mean) with the aim of performing some prior robustness checks. 

Prior means:
- alpha: -0.5, 0, 0.5 where 0 is the main prior, the other two are robustness checks
- win_beta: 0, 0.5, 1 where 0.5 is the main prior (which is equal to chance), the other two are robustness checks
- lose_beta: 0, 0.5, 1 where 0.5 is the main prior (which is equal to chance), the other two are robustness checks

Due to the relative complexity of the model we decided to not vary the standard deviations of the priors in order to keep the number of prior combinations at a managable level.

Prior sd:
- alpha: sd = 1 (large standard deviation to not constrain the estimate of the noise, even though we expect it to be low)
- betas: sd = 0.5 (smaller standard deviation to keep estimates of biases in a reasonable range)

### Parameter recovery 
When building our model, we use simulations to perform parameter recovery. By simulating data from known parameters we know the “true rates” in advance. By applying our model to the simulated data, we can run checks on the model to see how well it recovers (i.e., how accurate our estimates are) for different simulations. This could for example show us whether we need a different model (e.g. more simple) or more data (e.g. more trials). In this way, the process of parameter recovery will hopefully make it less likely that we draw invalid conclusions with our model. Parameter recovery is important for validating the inferences you make in a cognitive modeling analysis. It helps you understand whether your analysis can tell you anything about what is actually going on.

## Results and Discussion

### Prior robustness checks
We performed a prior robustness check of the mean values for the parameters alpha, win_beta and lose_beta. We kept the sd constant to reduce computation time. Ideally, we would also have performed robustness checks of the sds. The sd of the alpha (1) means the distribution is quite wide, so we argue it is unlikely that the priors has a too large impact on the parameter estimates. Likewise, the sd of the prior for the betas (0.5) is large enough that e.g. given a mean of 0.5 it allows for estimates of the bias to settle between 0 and 1, which is reasonable estimates. 

The prior robustness checks for the win_beta estimate can be seen in the two figures below. The win_beta estimate is the most relevant estimate to assess as this parameter reflects the Win-Shift-Lose-Stay bias defined in the data simulation and, hence, we only display sensitivity plots for this estimate. If recovered correctly (this is the case for all runs), the inverse logit of the lose_beta estimate represents 1-win_beta. As we have no noise parameter in any of the agents, this parameter is expected to be 0. All prior settings recover this parameter adequately as somewhere close to zero, so we have omitted plots for this as they are not too interesting.

![Prior robustness](prior_sensitivity.png "Prior robustness")
![Prior robustness](prior_sensitivity_lose.png "Prior robustness")

Complementary, the figure below shows prior-posterior update plots for the win_beta mean estimate. 
Each row correspond to a different lose_beta prior, and each panel/column correspond to win_beta mean prior. 

![Prior beta mean 0](plot0.png "PP-update for win beta mean = 0")
![Prior beta mean 0.5](plot05.png "PP-update for win beta mean = 0.5")
![Prior beta mean 1](plot1.png "PP-update for win beta mean = 1")

In general, the model seems well equipped to recover the true parameters given all of the combinations of parsed priors. Hence, the model is not overly sensistive to shifts in the priors and the defined set of priors all seem reasonable given the data. However, we deem that setting uninformed conservative beta priors with means of 0.5 and a sd of 0.5 makes most sense.

Summingly, based on these checks, we decided to use the following priors for the precision analysis: 
- alpha: mean = 0, sd = 1
- win_beta: mean = 0.5, sd = 0.5
- lose_beta: mean = 0.5, sd = 0.5  

Ideally, we would have included all different combinations of priors in the precision analysis, however, due to computation restraints we only use one prior for each parameter. Furthermore, given the low sensitivity of the model, it seems reasonable to only assess the effect of varying trials using a single set of priors.

### Precision analysis
To do a precision analysis, we simulated data varying on two conditions: rate of shifting (bias) and number of trials. We tested rates for shifting of 0.6, 0.65, 0.7, 0.75 and 0.8. For all these rates, we tested games of 100, 1000 and 10000. The range of number of trials was chosen to increase by orders of magnitude, and the rates were manually chosen to represent plausible real-life shifting biases given the assumption of natural variation in the population. A visualization of the parameter recovery under the conditions described above can be seen here: 
![Precision Analysis](plot_ntrials_check.png "Precision Analysis")
The results of the analysis can also be found in the table below.


| N simulated trials | Posterior 'Win Beta' Estimate Mean | Posterior 'Win Beta' Estimate SD | True mean rate | 
|---|---|---|---|
| 100 | 0.578 | 0.058 | 0.6 | 
| 1000 | 0.621 | 0.021 | 0.6 | 
| 10000 | 0.595 | 0.007 | 0.6 |
| 100 | 0.681 | 0.056 | 0.65 | 
| 1000 | 0.689 | 0.02 | 0.65 |
| 10000 | 0.646 | 0.007 | 0.65 |
| 100 | 0.712 | 0.056 | 0.7 |
| 1000 | 0.689 | 0.021 | 0.7 |
| 10000 | 0.703 | 0.007 | 0.7 |
| 100 | 0.656 | 0.057 | 0.75 |
| 1000 | 0.716 | 0.02 | 0.75 |
| 10000 | 0.761 | 0.006 | 0.75 |
| 100 | 0.732 | 0.058 | 0.8 |
| 1000 | 0.798 | 0.018 | 0.8 |
| 10000 | 0.801 | 0.006 | 0.8 |

In general, the results show that increasing the number of trials all the way to 10,000 continuously increases parameter recovery performance of the model; the beta estimates increasingly approaches the true bias rate and the model becomes increasingly more confident in its estimations (sd drops). We do not deem it necessary to define strict thesholds for identifying the acceptable number of trials but instead conclude that, given the visible improvements from 1000 trials to 10,000 trials, that 10,000 trials is the optimal number needed for sufficient parameter recovery. We do, however, acknowledge that in real life settings, it may be complicated and expensive to gather 10,000 trials and, therefore, also conclude that parameters are also recovered with accecptable precision using e.g. only 1000 trials providing the researcher is aware of the slight decrease in confidence.


## Conclusion
Overall, the analyses performed revealed that training the model using 10,000 simulated trials from an FeddbackAgent with a true Win-Shift-Loose-Stay bias of 0.7 and with the alpha prior defined as a Gaussian prior with a mean of 0 and sd = 1 and beta priors defined as Gaussian priors with means of 0.5 and sd of 0.5 adequately recovers the true parameter. The plot below display the prior-posterior update for the bias estimate (win_beta estimate) and shows that the model has confidently converged on the true bias:
![Prior-Posterior Main](pp_update_main.png "Prior-Posterior Main")

