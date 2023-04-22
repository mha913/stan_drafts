# ANCOVA in Stan

## Assumptions
```
1-The appropriate residuals are normally distributed.
2-The appropriate residuals are equally varied.
3-The relationship between the response variable and the covariate should be linear. Linearity can be explored using scatterplots- 
and residual plots should reveal no patterns.
4-In addition to the above assumptions, ANCOVA designs also assume that slopes of relationships between the response variable- 
and the covariate(s) are the same for each treatment level


```

### generate dat 

```r
 set.seed(123)
 n <- 10
 p <- 3
 A.eff <- c(40, -15, -20) #creates a vector called “A.eff” with three elements: 40, -15, and -201
 beta <- -0.45
 sigma <- 4
 B <- rnorm(n * p, 0, 15) # generates a vector called “B” with n * p elements that are normally distributed with a mean of 0 and a standard deviation of 15
 A <- gl(p, n, lab = paste("Group", LETTERS[1:3]))
 mm <- model.matrix(~A + B)
 data <- data.frame(A = A, B = B, Y = as.numeric(c(A.eff, beta) %*% t(mm)) + rnorm(n * p, 0, 4))
 data$B <- data$B + 20
 head(data)


```


### data look like
|   A	|  B 	| Y  	|     	
|---	|---	|---	|	
|   	Group A| 26.396963  	|   38.639924	|   	  	
|   Group C	|   23.799778	|  18.313157 	|   	 	
|  Group B 	|   38.119430	|  14.094222 	|   	   	


```r
scatterplot(Y ~ B | A, data = data) #create a scatter plot in R with Y as the dependent variable and B as the independent variable, conditioned on A. The vertical bar “|” separates the conditioning variable from the independent variable.
```

## stan code
```c++
data {
   int<lower=1> n;
   int<lower=1> nX;
   vector [n] y;
   matrix [n,nX] X;
   }
   parameters {
   vector[nX] beta;
   real<lower=0> sigma;
   }
   transformed parameters {
   vector[n] mu;
 
   mu = X*beta;
   }
   model {
   //Likelihood
   y~normal(mu,sigma);
   
   //Priors
   beta ~ normal(0,100);
   sigma~cauchy(0,5);
   }
   generated quantities {
   vector[n] log_lik;
   
   for (i in 1:n) {
   log_lik[i] = normal_lpdf(y[i] | mu[i], sigma); 
   }
   }
```

### refrence
```
https://agabrioblog.onrender.com/tutorial/ancova-stan/ancova-stan/
```
