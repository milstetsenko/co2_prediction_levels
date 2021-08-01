
## Short Description

This project predict the CO2 levels in the future up until 2061, including the seasonal fluctuations based on whether it is winter or summer and also predicts the overal trends of the rising CO2 levels. It also depicts the threshold level of no turning back, given that the emission will not change.

## Suggested Model

 To get you started here is an example of what such a model might look like. This model is not particularly good and you should come up with something more accurate.
 
• Long-term trend: linear, 𝑐0 + 𝑐1𝑡

• Seasonal variation (every 3651⁄4 days): cosine, 𝑐 𝑐𝑜𝑠( 2𝑡 + 𝑐 ) 23* 365.25

• Noise: Gaussian with mean 0 and fixed standard deviation, 𝑐4

• The c variables are all unobserved parameters of the model.

Combining these three components gives the following likelihood function 𝑝(𝑥𝑡 |) = 𝑁 (𝑐0 +
2𝑡 2 𝑐1𝑡+𝑐2𝑐𝑜𝑠(365.25 +𝑐3),𝑐4)

where represents the set of all unobserved parameters. Since there are 3156 data, the full like comprises a product overall 3156 values, 𝑥𝑡 . To complete the model we would still need to define priors over all 5 model parameters.


## Improved Model
### Proposed Alternations to Example Model

The prior for the variance for the Gaussian. distribution is a Normal distribution, however, the uncertainty should also account for rapid, short term but low probability peaks and downs of 𝐶𝑂2 level increase that can be caused by natural disasters, fires, human developments. Therefore, a better prior should have heavier tails to be able to draw deviations - 𝐶𝑎𝑢𝑐h𝑦(0, 1)

The order of growth of the long term rate is assumed to be linear but based on the historical developments, after the industrial revolution, the production of 𝐶𝑂2 started growing much faster. Therefore, several models will be compared and the best fit will be found.

The cosine function is proposed, therefore, I would like to test the sine function and contrast it to the cosine since it offers the shift in the values. I have attempted other combinations of sines with cosines to account for a better skewness, but the models did not converge.

A linear model might not be a good fit to the model because of the reasons mentioned above. Therefore, I have built three different models with three different functions - linear, polynomial of the second order, and exponential - to compare the fit of the model to the data.

For each of the three models, I have selected appropriate priors of 𝑐1 and 𝑐2 - the parameters standing in front of x based on the impact of influence of the outcome - the exponential coefficient should be small, the second order coefficient for the 𝑡 can be slightly larger, and the linear coefficient for the t can be the widest. The exact values and the models can be found in the notebook.

The results of the comparison are shown in figure 3. AS one can notice, the polynomial model fits the data the best and aligns with the edges of the available data whereas the linear and exponential fail to accurate resemble the data. Therefore, the quadratic function and its model areusedinprediction:𝑐 +𝑐𝑡+𝑐𝑡2


### The Priors for the Long Term Model

After testing the different alternatives for the type of function that best matches the long term rate, I have arrived to priors for quadratic function with the hyperparameters of the priors.

• 𝑐0 ∼ Normal(310, 10). This prior is the minimum values of the measurement at the start time.The std is chosen as the average fluctuation size within one year.

• 𝑐1 ∼ Gamma(1, 1). Should be not less than 0, therefore, this is a gamma prior. This prior accounts for the intensity of the increase - the slope of the function, so the range is chosen accordingly. The standard deviation is wide because we are unaware of the slope

• 𝑐2 ∼ Gamma(0.2, 0.2). Should be not less than 0, therefore, this is a gamma prior. This is also a slope, and, because 𝑐2 account for the t squared and prior’s effect is larger, the values should be smaller.

• 𝑐4 (noise) ∼ Cauchy(0,1). As mentioned in the previous section, the noise should be less informed and Cauchy’s heavy tails are fit.

<img width="557" alt="Screen Shot 2021-08-01 at 7 12 31 PM" src="https://user-images.githubusercontent.com/65287937/127777896-b045c549-5f55-4221-9d91-e70eeb348e86.png">

## Choosing the Short Term Fluctuations Model

The same procedure goes for the short term fluctuations. I am building two models in Stan with sine and cosine functions. The cosine model did not converge and did not have enough independent samples. The sine model performed much better. The model converged. However, the c3 parameter had only 600 independent observations. Further investigations on the independence of the samples were conducted and the results were independent (see Figure 4 and 5). Therefore, I proceed with the model, considering this hesitation.


Based on the models outcomes, the model selected is 𝑐 𝑠𝑖𝑛( 2𝑡 + 𝑐 ) 34 * 365.25



### The Priors for the Short Term Model

After testing two different alternatives for the type of function that best matches the short term rate, I have arrived to priors for sine function with the hyperparameters of the priors.

• 𝑐3 ∼ Normal(3,1).Thiscoefficientaccountsfortheamplitudeofthefunction.Byplotting the short term fluctuation without the upward trend, the approximate fluctuation seems to be around 3. Sigma is added for the variance.

• 𝑐5 (noise) ∼ Cauchy(0,1). As mentioned in the previous section, the noise should be less informed and Cauchy’s heavy tails are fit.

<img width="386" alt="Screen Shot 2021-08-01 at 7 11 24 PM" src="https://user-images.githubusercontent.com/65287937/127777869-5d3db905-3fbb-426b-8d2b-e497ca0f6618.png">

## Model Directed Graph
Based on the results of experimenting in the previous section, the final model is ready. The graph includes both the short term and the long term fluctuations, as well as all the priors and the hyperparameters. The model is:

𝑝(𝑥𝑡|)=𝑁(𝑐0 +𝑐1𝑡+𝑐2𝑡 +𝑐3𝑠𝑖𝑛(365.25 +𝑐5),𝑐4)


The directed graph shows the unknown and known quantities: as we can see, only the data itself is observed and the parameters of the model are obtained by using statistical instruments

<img width="840" alt="Screen Shot 2021-08-01 at 7 10 07 PM" src="https://user-images.githubusercontent.com/65287937/127777828-1ddb3452-77a6-4edd-8c93-e4f89590a3e2.png">

## Modeling the Future

Below, in figure 5, you can see the predicted model based on existing data. The horizontal black line showcases a critical level of 450 ppm predicts that the Earth will surpass this level in 2036 if I am orienting on the upper bound. The level of the 𝐶𝑂2 in the atmosphere in 2061 will be approximately 534 with the confidence intervals of [530.31, 539.51].

This prediction is yet another sign that strong actions should be taken before the irreversible changes of the climate begin.If the trend was to continue, which it will without significant interventions based on the model, we will pass the threshold very soon and the negative consequences of our actions will be put on many future generations. These results can be used as evidence to the public policy hearings, conventions, and summits created to deal with climate change and hopefully will be a push to policy makers to prevent this from happening.


<img width="792" alt="Screen Shot 2021-08-01 at 7 09 03 PM" src="https://user-images.githubusercontent.com/65287937/127777809-7901e697-f21f-4583-838d-480a40de7b59.png">


## Conclusions


### Shortcomings of the Model
#### Uncertainty for the future prediction

Unfortunately, I was not able to create a wider uncertainty for the future predictions as the timestamp deviates further in Stan. Because of MCMC methods it uses, the model treats all the data points likewise and does not differentiate the past datapoints from the future datapoints. Therefore, it does not increase uncertainty in the future and the credible intervals aer not widening as we are deviating into the future.

#### Sine Prior

The sine part of the model accounting for the seasonal fluctuations is a small concern. Because Stan was struggling to converge the model perfectly, one has to make certain assumptions about the reliability of the results and therefore, the predictions of the seasonal trends should be considered with more caution. And the model indeed shows much less fluctuations in the future than it showed in empirical data.
