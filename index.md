---

title       : Machine Learning with R - II
author      : Ilan Man
job         : Strategy Operations  @ Squarespace
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : mathjax       # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}

----

## Agenda 
<space>

1. Logistic Regression
2. Principle Component Analysis
3. Clustering
4. Trees

----

## Objectives 
<space>

1. Understand some popular algorithms and techniques
2. Learn how to tune parameters
3. Practice R

----

## Logistic Regression
# Motivation
<space>


```r
x <- 1:10
log_ex <- data.frame(Y=c(rnorm(5,0,0.01),rnorm(5,5,0.01)),X=x)
ggplot(log_ex,aes(X,Y)) + geom_point(color='blue',size=3) + stat_smooth(method='lm',se=F,color='green',size=1)
```

![plot of chunk log_bad_fit](figure/log_bad_fit.png) 

----

## Logistic Regression
# Motivation
<space>


```r
library("MASS")
library(ggplot2)
data(menarche)
log_data <- data.frame(Y=menarche$Menarche/menarche$Total)
log_data$X <- menarche$Age

glm.out <- glm(cbind(Menarche, Total-Menarche) ~ Age,family=binomial(logit), data=menarche)
lm.out <- lm(Y~X, data=log_data)

log_data$fitted <- glm.out$fitted

data_points <- ggplot(log_data) + geom_point(aes(x=X,y=Y),color='blue',size=3)
line_points <- data_points + geom_abline(intercept = coef(lm.out)[1], slope = coef(lm.out)[2],color='green',size=1)
curve_points <- line_points + geom_line(aes(x=X,y=fitted),color='red',size=1) 
```

----

## Logistic Regression
# Notation
<space>

- type of regression to predict the probability of being in a class
  - typical to set threshold to 0.5
- assumes error terms are Binomially distributed
  - which generates 1's and 0's as the error term
- sigmoid or logistic function: $g(z) = \frac{1}{1+e^{-z}}$
  - interpret the output as $P(Y=1 | X)$
  - bounded by 0 and 1

----

## Logistic Regression
# Notation
<space>


```r
curve(1/(1+exp(-x)), from = -10, to = 10, ylab="P(Y=1|X)", col = 'red', lwd = 3.0)
abline(a=0.5, b=0, lty=2, col='blue', lwd = 3.0)
```

![plot of chunk log_curve](figure/log_curve.png) 

----

## Logistic Regression
# Find parameters
<space>

- The hypothesis function, $h_{\theta}(x)$, is P(Y=1|X)
- Linear Regression --> $h_{\theta}(x) = \theta x^{T}$
- Logistic Regression --> $h_{\theta}(x) = g(\theta x^{T})$ 
<br>
where $g(z) = \frac{1}{1+e^{-z}}$

----

## Logistic Regression
# Notation
<space>

- Re-arranging $Y = \frac{1}{1+e^{-\theta x^{T}}}$ yields $\log{\frac{Y}{1 - Y}} = \theta x^{T}$<br>
- "log odds"" are linear in X
- this is called the logit of theta
  - links X linearly with some function of Y

----

## Logistic Regression
# Find parameters
<space>

- So $h_{\theta}(x) = \frac{1}{1+e^{-\theta x^{T}}}$
- What is the cost function?
- Why can't we use the same cost function as for the linear hypothesis?
  - logistic residuals are Binomially distributed - not Normal
  - the regression function is not linear in X

----

## Logistic Regression
# Find parameters
<space>

- Define logistic cost function as:

$cost(h_{\theta}(x)):$<br>
&nbsp;&nbsp; $= -\log(x),$ &nbsp;&nbsp;&nbsp;  $y = 1$<br>
&nbsp;&nbsp; $= -\log(1-x),$ &nbsp;   $y = 0$

![plot of chunk cost_curves](figure/cost_curves1.png) ![plot of chunk cost_curves](figure/cost_curves2.png) 

----

## Logistic Regression
# Find parameters
<space>

- using statistics, it can be shown that<br>
$cost(h_{\theta}(x), y) = -y \log(h_{\theta}(x)) + (1-y) \log(1-h_{\theta}(x))$<br>

----

## Logistic Regression
# Find parameters
<space>

- using statistics, it can be shown that<br>
$cost(h_{\theta}(x), y) = -y \log(h_{\theta}(x)) + (1-y) \log(1-h_{\theta}(x))$<br>
- Logistic regression cost function is then<br>
$cost(h_{\theta}(x), y)  = \frac{1}{m} \sum_{i=1}^{m} -y \log(h_{\theta}(x)) + (1-y) \log(1-h_{\theta}(x))$

----

## Logistic Regression
# Find parameters
<space>

- using statistics, it can be shown that<br>
$cost(h_{\theta}(x), y) = -y \log(h_{\theta}(x)) + (1-y) \log(1-h_{\theta}(x))$<br>
- Logistic regression cost function is then<br>
$cost(h_{\theta}(x), y)  = \frac{1}{m} \sum_{i=1}^{m} -y \log(h_{\theta}(x)) + (1-y) \log(1-h_{\theta}(x))$
- Minimize the cost

----

## Logistic Regression
# Newton-Raphson Method
<space>

- efficient
- easier to calculate that gradient descent
- converges on *global* minimum

----

## Logistic Regression
# Newton-Raphson Method
<space>

- assume derivative of f(x_{0}) is zero and f^{``} is positive
- re-write $f(x)$ as it's Taylor expansion:<br>
$f(x) = f(x_{0}) + (x-x_{0})\frac{df}{dx} + \frac{1}{2}(x-x_{0})^{2}\frac{d^2f}{dx^2}$
- take the derivative w.r.t x and set = 0<br>
$0 = f′(w_{0}) + \frac{1}{2}f′′(w_{0})2(w_{1} − w_{0})$<br>
$w1 = w_{0} − \frac{f′(w_{0})}{￼f′′(w_{0})}$

----

## Logistic Regression
# Newton-Raphson Method
<space>

my.newton = function(f,f.prime,f.prime2,w0,tolerance=1e-3,max.iter=50) {
  w = w0
  old.f = f(w)
  iterations = 0
  made.changes = TRUE
  while(made.changes & (iterations < max.iter)) {
   iterations <- iterations +1
   made.changes <- FALSE
   new.w = w - f.prime(w)/f.prime2(w)
   new.f = f(new.w)
   relative.change = abs(new.f - old.f)/old.f -1
   made.changes = (relative.changes > tolerance)
   w = new.w
   old.f = new.f
  }
  if (made.changes) {
    warning("Newton’s method terminated before convergence")
  }
  return(list(minimum=w,value=f(w),deriv=f.prime(w),deriv2=f.prime2(w),
         iterations=iterations,converged=!made.changes))
}



----

## Logistic Regression
# Gradient descent
<space>

![plot of chunk grad_ex_plot](figure/grad_ex_plot.png) 

----

## Logistic Regression
# Gradient descent
<space>


```r
x <- cbind(1,x)  #Add ones to x  
theta<- c(0,0)  # initalize theta vector 
m <- nrow(x)  # Number of the observations 
grad_cost <- function(X,y,theta) return(sum(((X%*%theta)- y)^2))
```

----

## Logistic Regression
# Gradient descent
<space>


```r
gradDescent<-function(X,y,theta,iterations,alpha){
  m <- length(y)
  grad <- rep(0,length(theta))
  cost.df <- data.frame(cost=0,theta=0)
  
  for (i in 1:iterations){
    h <- X%*%theta
    grad <-  (t(X)%*%(h - y))/m
    theta <- theta - alpha * grad
    cost.df <- rbind(cost.df,c(grad_cost(X,y,theta),theta))    
  }  
  
  return(list(theta,cost.df))
}
```

----

## Logistic Regression
# Gradient descent
<space>


```r
## initialize X, y and theta
X1<-matrix(ncol=1,nrow=nrow(df),cbind(1,df$X))
Y1<-matrix(ncol=1,nrow=nrow(df),df$Y)

init_theta<-as.matrix(c(0))
grad_cost(X1,Y1,init_theta)
```

```
[1] 5371
```

```r
iterations = 100
alpha = 0.1
results <- gradDescent(X1,Y1,init_theta,iterations,alpha)
```

----

## Logistic Regression
# Gradient descent
<space>

![plot of chunk grad_curve](figure/grad_curve.png) 

----

## Logistic Regression
# Gradient descent
<space>


```r
grad_cost(X1,Y1,theta[[1]])
```

```
[1] 360.9
```

```r
## Make some predictions
intercept <- df[df$X==0,]$Y
pred <- function (x) return(intercept+c(x)%*%theta)
new_points <- c(0.1,0.5,0.8,1.1)
new_preds <- data.frame(X=new_points,Y=sapply(new_points,pred))
```

----

## Logistic Regression
# Gradient descent
<space>


```r
ggplot(data=df,aes(x=X,y=Y))+geom_point(size=2)
```

![plot of chunk new_point](figure/new_point1.png) 

```r
ggplot(data=df,aes(x=X,y=Y))+geom_point()+geom_point(data=new_preds,aes(x=X,y=Y,color='red'),size=3)+scale_colour_discrete(guide = FALSE)
```

![plot of chunk new_point](figure/new_point2.png) 

----

## Regression example
# Gradient descent - summary
<space>

- minimization algorithm
- approximation, non-closed form solution
- good for large number of examples
- hard to select the right $\alpha$
- traditional looping is slow - optimization algorithms are used in practice

----

## Logistic Regression
# Summary
<space>

- very popular classification algorithm
- based on Binomial error terms, i.e. 1's and 0's

----

## Principle Component Analysis
# Motivation
<space>

- used widely in modern data analysis
- not well understood
- intuition: reduce data into only relevant dimensions
- the goal of PCA is to compute the most meaningful was to re-express noisy data, revealing the hidden structure

----

## Principle Component Analysis
# Concepts
<space>

- first big assumption: linearity
- $PX=Y$
  - $X$ is original dataset, $P$ is a transformation of $X$ into $Y$
- how do we choose $P$?
  - reduce noise
  - maximize variance

----

## Principle Component Analysis
# Concepts
<space>

- covariance matrix
     - $C = X*X^{T}$

- restated goals are
  - minimize covariance and maximize variance
  - the optimizal $C$ is a diagonal matrix, off diagonals are = 0
  
----

## Principle Component Analysis
# Concepts
<space>

- summary of assumptions
  - linearity (non-linear is a kernel PCA)
  - largest variance indicates most signal, low variance = noise
  - orthogonal components - makes the linear algebra easier
  - assumes data is normally distributed, otherwise PCA might not diagonalize matrix
    - can use ICA
    - but most data is normal and PCA is robust to slight deviance from normality

----

## Principle Component Analysis
# Eigenwhat?
<space>

- $Ax = \lambdax$
  - $\lambda$ is an eigenvalue of $A$ and $x$ is an eigenvector of $A$
- $Ax - \lambdaIx = 0$
- $(A - \lambdaI)x = 0$
- $\det(A - \lambdaI)$ = 0


----

## Principle Component Analysis
# Eigenwhat?
<space>

$\[A=\left[{\begin{array}{cc}5 & 2 \\2 & 5\\\end{array}\right ]\]$

A = matrix(c(5,2,2,5),nrow=2)
I = diag(nrow(A))
|A - L*I| = 0
det(c(5-l,2,2,5-l))
(5-l)*(5-l) - 4 = 0
25 - 10l + l^2 - 4 = 0
l^2 - 10l + 21 = 0
roots <- Re(polyroot(c(21,-10,1)))
```

----

## Principle Component Analysis
# Eigenwhat?
<space>

- when lambda = -3
Ax = 3x
5x1 + 2x2 = 3x1
2x1 + 5x2 = 3x2
x1=-x2
- one eigenvector = [1 -1]

----

## Principle Component Analysis
# Eigenwhat?
<space>

- when lambda = 7
5x1 + 2x2 = 7x1
2x2 + 5x2 = 7x2
x1 = x2
- another eigenvector = [1 1]

----

## Principle Component Analysis
# Eigenwhat?
<space>

A%*%c(1,-1) == 3 * as.matrix(c(1,-1))
A%*%c(1,1) == 7 * as.matrix(c(1,1))
roots

----

## Principle Component Analysis
# Eigenwhat?
<space>

- check
m <- matrix(c(1,-1,1,1),ncol=2)
m <- m/sqrt(norm(m))
as.matrix(m%*%diag(roots)%*%t(m))
- lambda is a diagonal matrix, with 0 off diagonals

----

## Principle Component Analysis
# Motivation
<space>

PX = Y

CY = (1/(n-1))*YYt
=PX(PX)t
=PXXtPt
=PAPt
# P is a matrix with columns that are eigenvectors
# A is a diagonalized matrix of eigenvalues (by linear algebra) and symmetric
A = EDEt

----

## Principle Component Analysis
# Motivation
<space>

# each row of P should be an eigenvector of A
P=Et
# also note that Pt = P-1 (linear algebra)
A = PtDP
CY = PPtDPPt
= (1/(n-1))*D
# D is a diagonal matrix, depending on how we choose P
# therefore CY is diagonalized

----

## Principle Component Analysis
# Example
<space>


```r
data <- read.csv('tennis_data_2013.csv')
```

```
## Warning: cannot open file 'tennis_data_2013.csv': No such file or
## directory
```

```
## Error: cannot open the connection
```

```r
data$Player1 <- as.character(data$Player1)
```

```
## Error: replacement has 0 rows, data has 6497
```

```r
data$Player2 <- as.character(data$Player2)
```

```
## Error: replacement has 0 rows, data has 6497
```

```r
tennis <- data
m <- length(data)

for (i in 10:m){
  tennis[,i] <- ifelse(is.na(data[,i]),0,data[,i])
}

str(tennis)
```

```
## 'data.frame':	6497 obs. of  13 variables:
##  $ type                : Factor w/ 2 levels "red","white": 1 1 1 1 1 1 1 1 1 1 ...
##  $ fixed.acidity       : num  7.4 7.8 7.8 11.2 7.4 7.4 7.9 7.3 7.8 7.5 ...
##  $ volatile.acidity    : num  0.7 0.88 0.76 0.28 0.7 0.66 0.6 0.65 0.58 0.5 ...
##  $ citric.acid         : num  0 0 0.04 0.56 0 0 0.06 0 0.02 0.36 ...
##  $ residual.sugar      : num  1.9 2.6 2.3 1.9 1.9 1.8 1.6 1.2 2 6.1 ...
##  $ chlorides           : num  0.076 0.098 0.092 0.075 0.076 0.075 0.069 0.065 0.073 0.071 ...
##  $ free.sulfur.dioxide : num  11 25 15 17 11 13 15 15 9 17 ...
##  $ total.sulfur.dioxide: num  34 67 54 60 34 40 59 21 18 102 ...
##  $ density             : num  0.998 0.997 0.997 0.998 0.998 ...
##  $ pH                  : num  3.51 3.2 3.26 3.16 3.51 3.51 3.3 3.39 3.36 3.35 ...
##  $ sulphates           : num  0.56 0.68 0.65 0.58 0.56 0.56 0.46 0.47 0.57 0.8 ...
##  $ alcohol             : num  9.4 9.8 9.8 9.8 9.4 9.4 9.4 10 9.5 10.5 ...
##  $ quality             : int  5 5 5 6 5 5 5 7 7 5 ...
```

```r
features <- tennis[,10:m]

head(features)
```

```
##     pH sulphates alcohol quality
## 1 3.51      0.56     9.4       5
## 2 3.20      0.68     9.8       5
## 3 3.26      0.65     9.8       5
## 4 3.16      0.58     9.8       6
## 5 3.51      0.56     9.4       5
## 6 3.51      0.56     9.4       5
```

```r
str(features)
```

```
## 'data.frame':	6497 obs. of  4 variables:
##  $ pH       : num  3.51 3.2 3.26 3.16 3.51 3.51 3.3 3.39 3.36 3.35 ...
##  $ sulphates: num  0.56 0.68 0.65 0.58 0.56 0.56 0.46 0.47 0.57 0.8 ...
##  $ alcohol  : num  9.4 9.8 9.8 9.8 9.4 9.4 9.4 10 9.5 10.5 ...
##  $ quality  : int  5 5 5 6 5 5 5 7 7 5 ...
```

```r
dim(features)
```

```
## [1] 6497    4
```

----

## Principle Component Analysis
# Example
<space>


```r
scaled_features <- as.matrix(scale(features))
Cx <- cov(scaled_features)
eigenvalues <- eigen(Cx)$values
eigenvectors <- eigen(Cx)$vectors
PC <- scaled_features %*% eigenvectors
```

----

## Principle Component Analysis
# Example
<space>


```r
Cy <- cov(PC)
sum(round(diag(Cy) - eigenvalues,5))
```

```
## [1] 0
```

```r
sum(round(Cy[upper.tri(Cy)],5)) ## off diagonals are 0 since PC's are orthogonal
```

```
## [1] 0
```

----

## Principle Component Analysis
# Example
<space>


```r
var_explained <- round(eigenvalues/sum(eigenvalues) * 100, digits = 2)
cum_var_explained <- round(cumsum(eigenvalues)/sum(eigenvalues) * 100, digits = 2)

var_explained <- as.data.frame(var_explained)
names(var_explained) <- "variance_explained"
var_explained$PC <- as.numeric(rownames(var_explained))
var_explained <- cbind(var_explained,cum_var_explained)

library(ggplot2)
ggplot(var_explained) +
  geom_bar(aes(x=PC,y=variance_explained),stat='identity') +
  geom_line(aes(x=PC,y=cum_var_explained))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

----

## Principle Component Analysis
# Example
<space>


```r
pca.df <- prcomp(scaled_features)
eigenvalues == (pca.df$sdev)^2
```

```
## [1] FALSE FALSE FALSE FALSE
```

```r
eigenvectors[,1] == pca.df$rotation[,1]
```

```
##        pH sulphates   alcohol   quality 
##     FALSE     FALSE     FALSE     FALSE
```

```r
sum((eigenvectors[,1])^2)
```

```
## [1] 1
```

----

## Principle Component Analysis
# Example
<space>


```r
rows <- nrow(tennis)
pca.plot <- as.data.frame(pca.df$x[,1:2])
pca.plot$gender <- data$Gender
ggplot(data=pca.plot,aes(x=PC1,y=PC2,color=gender)) + geom_point()
```

```
## Error: object 'gender' not found
```

----

## Principle Component Analysis
# Example
<space>

- how accurate is the first PC at dividing the dataset?
gen <- ifelse(pca.df$x[,1] > abs(mean(pca.df$x[,1]))*2,"F","M")
sum(diag(table(gen,as.character(data$Gender))))/rows

----

## Principle Component Analysis
# Summary
<space>

----

## Clustering
# Motivation
<space>

- separate data into meaningful or useful groups
  - capture natural structure of the data
  - starting point for further analysis
- cluster for utility
  - summarizing data for less expensive computation
  - data compression

----

## Clustering
# Types of Clusters
<space>

- data that looks more like other data in that cluster than outside
- each data point is more similar to the prototype (centeroid) of the cluster than the prototype of other clusters
- where the density is highest, that is a cluster

----

## Clustering
# Typical clustering problem
<space>

![plot of chunk cluster_plot_example](figure/cluster_plot_example.png) 

----

## Clustering
# Density based cluster
<space>

<img src="http://upload.wikimedia.org/wikipedia/commons/0/05/DBSCAN-density-data.svg" density_based />

----

## Clustering
# Kmeans algorithm
<space>

- Select K points as initial centroids 
- Do
  - Form K clusters by assigning each point to its closest centroid
  - Recompute the centroid of each cluster 
- Until centroids do not change, or change very minimally, i.e. <1%

----

## Clustering
# Kmeans algorithm
<space>

- Use similarity measures (Euclidean or cosine) depending on the data
- Minimize the squared distance of each point to closest centroid
$SSE(k) = \sum_{i=1}^{m}\sum_{j=1}^{n} (x_{ij} - \bar{x}_{kj})$

----

## Clustering
# Kmeans - notes
<space>

- Choose initial K randomly 
  - can lead to poor centroids - local minimuum
  - Run kmeans multiple times
- Reduce the total SSE by increasing K
- Increase the cluster with largest SSE
- Decrease K and minimize SSE
- Split up a cluster into other clusters
  - The centroid that is split will increase total SSE the least

----

## Clustering
# Kmeans
<space>

- Bisecting K means
  - Split points into 2 clusters
    - Take cluster with largest SSE - split that into two clusters
  - Rerun bisecting K mean on resulting clusters
  - Stop when you have K clusters
- Less susceptible to initialization problems

----

## Clustering
# Kmean fails
<space>

![different_density](C:/Users/Ilan%20Man/Desktop/Personal/RPres_ML_2/figure/different_density.png)

----

## Clustering
# Kmean fails
<space>

![different_size_clusters](C:/Users/Ilan%20Man/Desktop/Personal/RPres_ML_2/figure/different_size_clusters.png)

----

## Clustering
# Kmean fails
<space>

![non-globular](C:/Users/Ilan%20Man/Desktop/Personal/RPres_ML_2/figure/non-globular.png)

----

## Clustering
# Kmeans
<space>


```r
wine <- read.csv('http://archive.ics.uci.edu/ml/machine-learning-databases/wine/wine.data')
names(wine) <- c("class",'Alcohol','Malic','Ash','Alcalinity','Magnesium','Total_phenols',
                 'Flavanoids','NFphenols','Proanthocyanins','Color','Hue','Diluted','Proline')
str(wine)
```

```
## 'data.frame':	177 obs. of  14 variables:
##  $ class          : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Alcohol        : num  13.2 13.2 14.4 13.2 14.2 ...
##  $ Malic          : num  1.78 2.36 1.95 2.59 1.76 1.87 2.15 1.64 1.35 2.16 ...
##  $ Ash            : num  2.14 2.67 2.5 2.87 2.45 2.45 2.61 2.17 2.27 2.3 ...
##  $ Alcalinity     : num  11.2 18.6 16.8 21 15.2 14.6 17.6 14 16 18 ...
##  $ Magnesium      : int  100 101 113 118 112 96 121 97 98 105 ...
##  $ Total_phenols  : num  2.65 2.8 3.85 2.8 3.27 2.5 2.6 2.8 2.98 2.95 ...
##  $ Flavanoids     : num  2.76 3.24 3.49 2.69 3.39 2.52 2.51 2.98 3.15 3.32 ...
##  $ NFphenols      : num  0.26 0.3 0.24 0.39 0.34 0.3 0.31 0.29 0.22 0.22 ...
##  $ Proanthocyanins: num  1.28 2.81 2.18 1.82 1.97 1.98 1.25 1.98 1.85 2.38 ...
##  $ Color          : num  4.38 5.68 7.8 4.32 6.75 5.25 5.05 5.2 7.22 5.75 ...
##  $ Hue            : num  1.05 1.03 0.86 1.04 1.05 1.02 1.06 1.08 1.01 1.25 ...
##  $ Diluted        : num  3.4 3.17 3.45 2.93 2.85 3.58 3.58 2.85 3.55 3.17 ...
##  $ Proline        : int  1050 1185 1480 735 1450 1290 1295 1045 1045 1510 ...
```

- set.seed() to make sure results are reproducible
- add nstart to the function call so that it attempts multiple configurations, selecting the best
- use a screeplot to select optimal K

----

## Clustering
# Kmeans
<space>


```r
s.wine <- scale(wine[,-1])
best_k <- 0
num_k <- 20
for (i in 1:num_k){
  best_k[i] <- sum(kmeans(s.wine,centers=i)$withinss)
  }

barplot(best_k, xlab = "Number of clusters",
        names.arg = 1:num_k,
        ylab="Within groups sum of squares",
        main="Scree Plot for Wine dataset")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 

----

## Clustering
# Kmeans animation
<space>

install.packages('animation')
library(animation)

oopt = ani.options(interval = 1)
ani_ex = rbind(matrix(rnorm(100, sd = 0.3), ncol = 2), 
          matrix(rnorm(100, sd = 0.3), 
          ncol = 2))
colnames(ani_ex) = c("x", "y")

kmeans.an = function(
  x = cbind(X1 = runif(50), X2 = runif(50)), centers = 4, hints = c('Move centers!', 'Find cluster?'),
  pch = 1:5, col = 1:5
) {
  x = as.matrix(x)
  ocluster = sample(centers, nrow(x), replace = TRUE)
  if (length(centers) == 1) centers = x[sample(nrow(x), centers), ] else
    centers = as.matrix(centers)
  numcent = nrow(centers)
  dst = matrix(nrow = nrow(x), ncol = numcent)
  j = 1
  pch = rep(pch, length = numcent)
  col = rep(col, length = numcent)
  
  for (j in 1:ani.options('nmax')) {
    dev.hold()
    plot(x, pch = pch[ocluster], col = col[ocluster], panel.first = grid())
    mtext(hints[1], 4)
    points(centers, pch = pch[1:numcent], cex = 3, lwd = 2, col = col[1:numcent])
    ani.pause()
    for (i in 1:numcent) {
      dst[, i] = sqrt(apply((t(t(x) - unlist(centers[i, ])))^2, 1, sum))
    }
    ncluster = apply(dst, 1, which.min)
    plot(x, type = 'n')
    mtext(hints[2], 4)
    grid()
    ocenters = centers
    for (i in 1:numcent) {
      xx = subset(x, ncluster == i)
      polygon(xx[chull(xx), ], density = 10, col = col[i], lty = 2)
      points(xx, pch = pch[i], col = col[i])
      centers[i, ] = apply(xx, 2, mean)
    }
    points(ocenters, cex = 3, col = col[1:numcent], pch = pch[1:numcent], lwd = 2)
    ani.pause()
    if (all(ncluster == ocluster)) break
    ocluster = ncluster
  }
  invisible(list(cluster = ncluster, centers = centers))
}

kmeans.an(ani_ex, centers = 5, hints = c("Move centers","Cluster found?"))

----

## Clustering
# K-medoid
<space>

- multiple distance metrics
- robust medioids
- computationally expensive
- cluster center is one of the points itself

----

## Clustering
# K-medoid
<space>

- cluster each point based on the closest center
- replace each center by the medioid of points in its cluster

----

## Clustering
# K-medoid
<space>

- Selecting the optimal number of clusters
- For each point p, first find the average distance between p and all other points in the same cluster, $A$
- Then find the average distance between p and all points in the nearest cluster, $B$
- The silhouette coefficient for p is $\frac{A - B}{\max(A,B)}$
  - Values close to 1 mean point clearly belongs to that cluster
  - Values close to 0 mean points might belong in another cluster

----

## Clustering
# K-medoid
<space>


```r
library(cluster)

pam.best <- as.numeric()
for (i in 2:20){
  pam.best[i] <- pam(s.wine, k=i)$silinfo$avg.width
}
best_k <- which.max(pam.best)
best_k
```

```
## [1] 3
```

----

## Clustering
# K-medoid
<space>


```r
clusplot(pam(s.wine,best_k), main="K-medoids with K = 3",sub=NULL)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

----

## Clustering
# DBSCAN
<space>

- A cluster is a dense region of points separated by low-density regions
- Group objects into one cluster if they are connected to one another by densely populated area
- Used when the clusters are irregular or intertwined, and when noise and outliers are present

----

## Clustering
# Terminology
<space>

- Core points are located inside a cluster
- Border points are on the borders between two clusters
- Neighborhood of p are all points within some radius of p, Eps

----

## Clustering
# Terminology
<space>

- Core points are located inside a cluster
- Border points are on the borders between two clusters
- Neighborhood of p are all points within some radius of p, Eps
![density](/Users/ilanman/Desktop/Data/RPres_ML_2/figure/density_structure.png)

----

## Clustering
# Terminology
<space>

- Core points are located inside a cluster
- Border points are on the borders between two clusters
- Neighborhood of p are all points within some radius of p, Eps
- High density region has at least Minpts within Eps of point p
- Noise points are not within Eps of border or core points

----

## Clustering
# Terminology
<space>

- Core points are located inside a cluster
- Border points are on the borders between two clusters
- Neighborhood of p are all points within some radius of p, Eps
- High density region has at least Minpts within Eps of point p
- Noise points are not within Eps of border or core points
- If p is density connected to q, they are part of the same cluster, if not, then they are not
- If p is not density connected to any other point, its considered noise

----

## Clustering
# DBSCAN
<space>

![density_win](/Users/ilanman/Desktop/Data/RPres_ML_2/figure/density_ex_win.png)

----

## Clustering
# DBSCAN
<space>


```r
x <- c(2,2,8,5,7,6,1,4)
y <- c(10,5,4,8,5,4,2,9)
cluster <- data.frame(X=c(x,2*x,3*x),Y=c(y,-2*x,1/4*y))
plot(cluster)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

----

## Clustering
# DBSCAN
<space>


```r
library(fpc)
cluster_DBSCAN<-dbscan(cluster, eps=3, MinPts=2, method="hybrid")
plot(cluster_DBSCAN, cluster, main="Clustering using DBSCAN algorithm (eps=3, MinPts=3)")
```

![plot of chunk dbscan_ex](figure/dbscan_ex.png) 

----

## Clustering
# Summary
<space>

- Unsupervised learning
- Not a perfect science - lots of interpretation
- Hard to define "correct" clustering
- Many types of algorithms

----

## Trees
# Motivation
<space>

- representation of decisions made in order to classify or predict
![overview](/Users/ilanman/Desktop/Data/RPres_ML_2/figure/tree_example.png)

----

## Trees
# Structure
<space>

![structure](/Users/ilanman/Desktop/Data/RPres_ML_2/figure/tree_structure.png)

----

## Trees
# Structure
<space>

- recursive partitioning -> "divide and conquer"
- going down, choose feature that is most *predictive* of target class
  - split the data according to feature
  - continue...

----

## Trees
# Structure
<space>

until...
- all examples at a node are in same class
- no more features left to distinguish (prone to overfitting)
- tree has grown to some prespecified limit (prune)

----

## Trees
# Algorithms
<space>

- ID3
  - original, popular, DT implementation
- C4.5
  - like ID3 +
  - handles continuous cases
  - imputing missing values
  - weighing costs
  - pruning post creation
- C5.0
  - like C4.5 + 
  - faster, less memory usage
  - boosting

----

## Trees
# Selecting features
<space>

- How does tree decide how to select feature?
  - purity of resulting split
- __Entropy__: amount of information contained in a random variable
  - For a feature with N classes:
    - 0 = purely homogenous
    - $\log_{2}(N)$ = completely mixed

----

## Trees
# Entropy
<space>

$Entropy(S) = \sum_{i=1}^{c} -p_{i}\log_{2}(p_{i})$
  - where $S$ is a dataset
  - $c$ is the number of levels in that data
  - $p_{i}$ is the proportion of values in that level

----

## Trees
# Entropy - example
<space>

What is the entropy of a fair, 6 sided die?


```r
entropy <- function(probs){
  ent <- 0
  for(i in probs){
    ent_temp <- -i*log2(i)
    ent <- ent + ent_temp
  }
  return(ent)
}
```

----

## Trees
# Entropy - example
<space>


```r
fair <- rep(1/6,6)
entropy(fair)
```

```
## [1] 2.585
```

```r
log2(6)
```

```
## [1] 2.585
```

----

## Trees
# Entropy - example
<space>

What is the entropy of a biased, 6 sided die?
- $P(X=1) = P(X=2) = P(X=3) = 1/9$
- $P(X=4) = P(X=5) = P(X=6) = 2/9$


```r
biased <- c(rep(1/9,3),rep(2/9,3))
entropy(biased)
```

```
[1] 2.503
```

----

## Trees
# Entropy - example
<space>


```r
more_biased <- c(rep(1/18,3),rep(5/18,3))
entropy(more_biased)
```

```
[1] 2.235
```

```r
most_biased <- c(rep(1/100,5),rep(95/100,1))
entropy(most_biased)
```

```
[1] 0.4025
```

----

## Trees
# Entropy - example
<space>


```r
curve(-x*log2(x)-(1 - x)*log2(1 - x), col =" red", xlab = "x", ylab = "Entropy", 
      lwd = 4, main='Entropy of a coin toss')
```

![plot of chunk entropy_curve](figure/entropy_curve.png) 

----

## Trees
# Entropy
<space>

- C5.0 uses the change in entropy to determine the change in purity
- InfoGain = Entropy (pre split) - Entropy (post split)
  - Entropy (pre split) = current Entropy
  - Entropy (post split) is trickier
    - need to consider Entropy of each possible split
  - $E(post) = \sum_{i=1}^{n}w_{i}Entropy(P_{i})$

- Notes:
  - The more a feature splits the data in obvious ways, the less informative it is, entropy is lower
  - The more a feature splits the data - in general - the higher the entropy and hence information gained by splitting at that feature

----

## Trees
# Example
<space>


```r
voting_data <- read.csv('http://archive.ics.uci.edu/ml/machine-learning-databases/voting-records/house-votes-84.data')
names(voting_data) <- c('party','handicapped-infants','water-project-cost-sharing',
                        'adoption-of-the-budget-resolution','physician-fee-freeze',
                        'el-salvador-aid','religious-groups-in-schools',
                        'anti-satellite-test-ban','aid-to-nicaraguan-contras',
                        'mx-missile','immigration','synfuels-corporation-cutback',
                        'education-spending','superfund-right-to-sue','crime',
                        'duty-free-exports','export-administration-act-south-africa')
```

----

## Trees
# Example
<space>


```r
prop.table(table(voting_data[,1]))
```

```

  democrat republican 
    0.6152     0.3848 
```

```r
n <- nrow(voting_data)
train_ind <- sample(n,2/3*n)
voting_train <- voting_data[train_ind,]
voting_test <- voting_data[-train_ind,]
```

----

## Trees
# Example
<space>

<img src="/Users/ilanman/Desktop/Data/RPres_ML_2/figure/real_tree_example.png" height="500px" width="500px" />


----

## Trees
# Example
<space>


```

 
   Cell Contents
|-------------------------|
|                       N |
|         N / Table Total |
|-------------------------|

 
Total Observations in Table:  145 

 
             | predicted class 
actual class |   democrat | republican |  Row Total | 
-------------|------------|------------|------------|
    democrat |         94 |          1 |         95 | 
             |      0.648 |      0.007 |            | 
-------------|------------|------------|------------|
  republican |          3 |         47 |         50 | 
             |      0.021 |      0.324 |            | 
-------------|------------|------------|------------|
Column Total |         97 |         48 |        145 | 
-------------|------------|------------|------------|

 
```

----

## Trees
# Example
<space>


```r
# most important variables
head(C5imp(tree_model))
```

```
##                                   Overall
## physician-fee-freeze                97.92
## synfuels-corporation-cutback        42.21
## mx-missile                          10.73
## adoption-of-the-budget-resolution    9.69
## handicapped-infants                  0.00
## water-project-cost-sharing           0.00
```

----

## Trees
# Example
<space>


```r
# in-sample error rate
summary(tree_model)
```

```
## 
## Call:
## C5.0.default(x = voting_train[, -1], y = voting_train[, 1], trials = 1)
## 
## 
## C5.0 [Release 2.07 GPL Edition]  	Mon Aug 18 09:15:15 2014
## -------------------------------
## 
## Class specified by attribute `outcome'
## 
## Read 289 cases (17 attributes) from undefined.data
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (163.4/2.1)
## physician-fee-freeze = y:
## :...synfuels-corporation-cutback in {?,n}: republican (100.2/2.3)
##     synfuels-corporation-cutback = y:
##     :...mx-missile = ?: republican (0)
##         mx-missile = y: democrat (2.5)
##         mx-missile = n:
##         :...adoption-of-the-budget-resolution in {?,n}: republican (17.7/3)
##             adoption-of-the-budget-resolution = y: democrat (5.2/2.2)
## 
## 
## Evaluation on training data (289 cases):
## 
## 	    Decision Tree   
## 	  ----------------  
## 	  Size      Errors  
## 
## 	     5   10( 3.5%)   <<
## 
## 
## 	   (a)   (b)    <-classified as
## 	  ----  ----
## 	   167     5    (a): class democrat
## 	     5   112    (b): class republican
## 
## 
## 	Attribute usage:
## 
## 	 97.92%	physician-fee-freeze
## 	 42.21%	synfuels-corporation-cutback
## 	 10.73%	mx-missile
## 	  9.69%	adoption-of-the-budget-resolution
## 
## 
## Time: 0.0 secs
```

----

## Trees
# Boosting
<space>

- by combining a number of weak performing learners create a team that is much stronger than any one of the learners alone.
- this is where C5.0 improves on C4.5

----

## Trees
# Example - Boosting
<space>


```r
boosted_tree_model <- C5.0(voting_train[,-1],voting_train[,1], trials=25)
boosted_tennis_predict <- predict(boosted_tree_model,voting_test[,-1])

boosted_conf <- CrossTable(voting_test[,1], boosted_tennis_predict, prop.chisq = FALSE,
                           prop.c = FALSE, prop.r = FALSE, 
                           dnn = c("actual class", "predicted class"))
```

```
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## |         N / Table Total |
## |-------------------------|
## 
##  
## Total Observations in Table:  145 
## 
##  
##              | predicted class 
## actual class |   democrat | republican |  Row Total | 
## -------------|------------|------------|------------|
##     democrat |         94 |          1 |         95 | 
##              |      0.648 |      0.007 |            | 
## -------------|------------|------------|------------|
##   republican |          2 |         48 |         50 | 
##              |      0.014 |      0.331 |            | 
## -------------|------------|------------|------------|
## Column Total |         96 |         49 |        145 | 
## -------------|------------|------------|------------|
## 
## 
```

----

## Trees
# Example - Boosting
<space>


```r
# in-sample error rate
summary(boosted_tree_model)
```

```
## 
## Call:
## C5.0.default(x = voting_train[, -1], y = voting_train[, 1], trials = 25)
## 
## 
## C5.0 [Release 2.07 GPL Edition]  	Mon Aug 18 09:15:16 2014
## -------------------------------
## 
## Class specified by attribute `outcome'
## 
## Read 289 cases (17 attributes) from undefined.data
## 
## -----  Trial 0:  -----
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (163.4/2.1)
## physician-fee-freeze = y:
## :...synfuels-corporation-cutback in {?,n}: republican (100.2/2.3)
##     synfuels-corporation-cutback = y:
##     :...mx-missile = ?: republican (0)
##         mx-missile = y: democrat (2.5)
##         mx-missile = n:
##         :...adoption-of-the-budget-resolution in {?,n}: republican (17.7/3)
##             adoption-of-the-budget-resolution = y: democrat (5.2/2.2)
## 
## -----  Trial 1:  -----
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (137.2/15.1)
## physician-fee-freeze = y:
## :...immigration in {?,y}: republican (66.8/1.9)
##     immigration = n:
##     :...adoption-of-the-budget-resolution in {?,n}: republican (65.6/24.5)
##         adoption-of-the-budget-resolution = y: democrat (19.4/2.6)
## 
## -----  Trial 2:  -----
## 
## Decision tree:
## 
## synfuels-corporation-cutback = ?: republican (0)
## synfuels-corporation-cutback = y: democrat (131.6/32.4)
## synfuels-corporation-cutback = n:
## :...adoption-of-the-budget-resolution in {?,n}: republican (70.6/3.3)
##     adoption-of-the-budget-resolution = y: democrat (86.8/22.5)
## 
## -----  Trial 3:  -----
## 
## Decision tree:
## 
## physician-fee-freeze = n: democrat (107/18.4)
## physician-fee-freeze in {?,y}: republican (182/55.4)
## 
## -----  Trial 4:  -----
## 
## Decision tree:
## 
## crime in {?,n}: democrat (64.9/6.5)
## crime = y:
## :...synfuels-corporation-cutback in {?,n}: republican (111.2/32.7)
##     synfuels-corporation-cutback = y:
##     :...export-administration-act-south-africa in {?,n}: democrat (64/10.5)
##         export-administration-act-south-africa = y: republican (49/20.5)
## 
## -----  Trial 5:  -----
## 
## Decision tree:
## 
## el-salvador-aid in {?,n}: democrat (89.2/10.1)
## el-salvador-aid = y:
## :...anti-satellite-test-ban in {?,y}: republican (52.2/10.2)
##     anti-satellite-test-ban = n:
##     :...adoption-of-the-budget-resolution in {?,y}: democrat (54.7/7.1)
##         adoption-of-the-budget-resolution = n:
##         :...immigration = n: democrat (71.4/32)
##             immigration in {?,y}: republican (21.5/0.7)
## 
## -----  Trial 6:  -----
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (109.4/15.9)
## physician-fee-freeze = y:
## :...synfuels-corporation-cutback in {?,n}: republican (90.5/16.1)
##     synfuels-corporation-cutback = y: democrat (89.1/41.6)
## 
## -----  Trial 7:  -----
## 
## Decision tree:
## 
## physician-fee-freeze = n: democrat (95.6/17.6)
## physician-fee-freeze in {?,y}: republican (193.4/59.4)
## 
## -----  Trial 8:  -----
## 
## Decision tree:
## 
## adoption-of-the-budget-resolution = ?: democrat (0)
## adoption-of-the-budget-resolution = y:
## :...physician-fee-freeze in {?,n}: democrat (63/4.3)
## :   physician-fee-freeze = y:
## :   :...anti-satellite-test-ban in {?,n}: democrat (55.7/8.6)
## :       anti-satellite-test-ban = y: republican (24.1/2.5)
## adoption-of-the-budget-resolution = n:
## :...mx-missile = ?: republican (0)
##     mx-missile = y: democrat (18.4/3.3)
##     mx-missile = n:
##     :...synfuels-corporation-cutback in {?,n}: republican (48.7/2.8)
##         synfuels-corporation-cutback = y:
##         :...superfund-right-to-sue = n: democrat (14.5/3.2)
##             superfund-right-to-sue in {?,y}: republican (64.6/20.9)
## 
## -----  Trial 9:  -----
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (86.3/17.1)
## physician-fee-freeze = y:
## :...export-administration-act-south-africa = n: democrat (78.1/31.3)
##     export-administration-act-south-africa in {?,y}: republican (124.5/35.7)
## 
## -----  Trial 10:  -----
## 
## Decision tree:
## 
## education-spending in {?,n}: democrat (134.7/40.3)
## education-spending = y:
## :...physician-fee-freeze = ?: republican (0)
##     physician-fee-freeze = n: democrat (22.4/7.4)
##     physician-fee-freeze = y:
##     :...anti-satellite-test-ban in {?,y}: republican (21.4)
##         anti-satellite-test-ban = n:
##         :...adoption-of-the-budget-resolution in {?,n}: republican (85.4/19.1)
##             adoption-of-the-budget-resolution = y: democrat (25.2/7.9)
## 
## -----  Trial 11:  -----
## 
## Decision tree:
## 
## physician-fee-freeze = ?: republican (0)
## physician-fee-freeze = n: democrat (71.1/18.5)
## physician-fee-freeze = y:
## :...synfuels-corporation-cutback in {?,n}: republican (111.2/18.2)
##     synfuels-corporation-cutback = y:
##     :...mx-missile in {?,y}: democrat (14/1.3)
##         mx-missile = n:
##         :...anti-satellite-test-ban in {?,y}: republican (8.2)
##             anti-satellite-test-ban = n:
##             :...adoption-of-the-budget-resolution = n: republican (68.1/29.9)
##                 adoption-of-the-budget-resolution in {?,y}: democrat (16.5/1.6)
## 
## -----  Trial 12:  -----
## 
## Decision tree:
## 
## synfuels-corporation-cutback in {?,y}: democrat (138.2/42.4)
## synfuels-corporation-cutback = n:
## :...adoption-of-the-budget-resolution in {?,n}: republican (72.7/6.2)
##     adoption-of-the-budget-resolution = y: democrat (78.1/31)
## 
## -----  Trial 13:  -----
## 
## Decision tree:
## 
## immigration = ?: republican (0)
## immigration = y:
## :...physician-fee-freeze = n: democrat (22.1/1.8)
## :   physician-fee-freeze in {?,y}: republican (83.1/8.8)
## immigration = n:
## :...adoption-of-the-budget-resolution in {?,y}: democrat (54.1/9.5)
##     adoption-of-the-budget-resolution = n:
##     :...synfuels-corporation-cutback in {?,n}: republican (45.7/6.8)
##         synfuels-corporation-cutback = y:
##         :...export-administration-act-south-africa in {?,
##             :                                          n}: democrat (62.9/17.1)
##             export-administration-act-south-africa = y: republican (21/6.5)
## 
## -----  Trial 14:  -----
## 
## Decision tree:
## 
## physician-fee-freeze = ?: republican (0)
## physician-fee-freeze = n: democrat (76.2/17.6)
## physician-fee-freeze = y:
## :...export-administration-act-south-africa in {?,y}: republican (113.7/30.5)
##     export-administration-act-south-africa = n:
##     :...mx-missile in {?,y}: democrat (8.6/0.6)
##         mx-missile = n:
##         :...superfund-right-to-sue in {?,n}: democrat (12/1.6)
##             superfund-right-to-sue = y: republican (78.5/32.7)
## 
## -----  Trial 15:  -----
## 
## Decision tree:
## 
## crime in {?,n}: democrat (33.2/5.8)
## crime = y:
## :...anti-satellite-test-ban = ?: democrat (0)
##     anti-satellite-test-ban = y: republican (63.9/12.9)
##     anti-satellite-test-ban = n:
##     :...religious-groups-in-schools = ?: democrat (0)
##         religious-groups-in-schools = n: republican (11.4/0.2)
##         religious-groups-in-schools = y:
##         :...physician-fee-freeze in {?,n}: democrat (22.3/1.3)
##             physician-fee-freeze = y:
##             :...adoption-of-the-budget-resolution = n: republican (93.2/41.1)
##                 adoption-of-the-budget-resolution in {?,
##                                                       y}: democrat (65/12)
## 
## -----  Trial 16:  -----
## 
## Decision tree:
## 
## el-salvador-aid in {?,n}: democrat (61.7/10.4)
## el-salvador-aid = y:
## :...anti-satellite-test-ban = ?: democrat (0)
##     anti-satellite-test-ban = y: republican (49.1/10.1)
##     anti-satellite-test-ban = n:
##     :...crime = ?: democrat (0)
##         crime = n: republican (5.8/0.1)
##         crime = y:
##         :...religious-groups-in-schools = ?: democrat (0)
##             religious-groups-in-schools = n: republican (7.1/0.1)
##             religious-groups-in-schools = y:
##             :...physician-fee-freeze in {?,n}: democrat (18.7/1.1)
##                 physician-fee-freeze = y:
##                 :...duty-free-exports in {?,y}: democrat (18.9/2.2)
##                     duty-free-exports = n: [S1]
## 
## SubTree [S1]
## 
## export-administration-act-south-africa in {?,n}: democrat (80.4/26.2)
## export-administration-act-south-africa = y: republican (47.3/17.1)
## 
## -----  Trial 17:  -----
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (88.7/17)
## physician-fee-freeze = y:
## :...synfuels-corporation-cutback in {?,n}: republican (95/18.8)
##     synfuels-corporation-cutback = y: democrat (105.3/44.4)
## 
## -----  Trial 18:  -----
## 
## Decision tree:
## 
## physician-fee-freeze = ?: republican (0)
## physician-fee-freeze = n: democrat (78.9/18.7)
## physician-fee-freeze = y:
## :...adoption-of-the-budget-resolution in {?,n}: republican (134.2/31.3)
##     adoption-of-the-budget-resolution = y: democrat (75.9/33.7)
## 
## -----  Trial 19:  -----
## 
## Decision tree:
## 
## physician-fee-freeze = ?: republican (0)
## physician-fee-freeze = n: democrat (68.5/18.7)
## physician-fee-freeze = y:
## :...immigration in {?,y}: republican (93/14.7)
##     immigration = n:
##     :...mx-missile = ?: republican (0)
##         mx-missile = y: democrat (16.6/2.4)
##         mx-missile = n:
##         :...anti-satellite-test-ban in {?,y}: republican (11.6)
##             anti-satellite-test-ban = n:
##             :...adoption-of-the-budget-resolution in {?,
##                 :                                     n}: republican (86.6/34.4)
##                 adoption-of-the-budget-resolution = y: democrat (12.6/0.8)
## 
## -----  Trial 20:  -----
## 
## Decision tree:
## 
## synfuels-corporation-cutback in {?,n}: republican (132.7/36.2)
## synfuels-corporation-cutback = y:
## :...education-spending in {?,n}: democrat (66.2/11.1)
##     education-spending = y:
##     :...export-administration-act-south-africa in {?,n}: democrat (56.9/18.4)
##         export-administration-act-south-africa = y: republican (33.2/8)
## 
## -----  Trial 21:  -----
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (97.5/19.4)
## physician-fee-freeze = y:
## :...anti-satellite-test-ban in {?,y}: republican (42.5/3.4)
##     anti-satellite-test-ban = n:
##     :...adoption-of-the-budget-resolution in {?,y}: democrat (50.1/13)
##         adoption-of-the-budget-resolution = n:
##         :...immigration in {?,y}: republican (17.8)
##             immigration = n:
##             :...synfuels-corporation-cutback in {?,n}: republican (10.3)
##                 synfuels-corporation-cutback = y: democrat (70.7/30.8)
## 
## -----  Trial 22:  -----
## 
## Decision tree:
## 
## crime = ?: republican (0)
## crime = n: democrat (44.5/4.8)
## crime = y:
## :...physician-fee-freeze = ?: republican (0)
##     physician-fee-freeze = n: democrat (54.9/20.1)
##     physician-fee-freeze = y:
##     :...anti-satellite-test-ban in {?,y}: republican (31.9)
##         anti-satellite-test-ban = n:
##         :...religious-groups-in-schools in {?,n}: republican (8.3)
##             religious-groups-in-schools = y:
##             :...duty-free-exports in {?,n}: republican (135.1/52.3)
##                 duty-free-exports = y: democrat (13.2/1.8)
## 
## -----  Trial 23:  -----
## 
## Decision tree:
## 
## physician-fee-freeze in {?,n}: democrat (59.4/1.5)
## physician-fee-freeze = y:
## :...religious-groups-in-schools in {?,n}: republican (18.1/1.5)
##     religious-groups-in-schools = y:
##     :...education-spending = ?: republican (0)
##         education-spending = n: democrat (76.9/26.3)
##         education-spending = y:
##         :...export-administration-act-south-africa in {?,
##             :                                          y}: republican (51.4/6.2)
##             export-administration-act-south-africa = n:
##             :...superfund-right-to-sue in {?,n}: democrat (12.8/0.4)
##                 superfund-right-to-sue = y: republican (68.5/29.2)
## 
## -----  Trial 24:  -----
## 
## Decision tree:
## 
## physician-fee-freeze = ?: republican (0)
## physician-fee-freeze = n: democrat (49.7)
## physician-fee-freeze = y:
## :...water-project-cost-sharing in {?,n}: republican (61.5/3.5)
##     water-project-cost-sharing = y:
##     :...el-salvador-aid = ?: republican (0)
##         el-salvador-aid = n: democrat (10.5)
##         el-salvador-aid = y:
##         :...anti-satellite-test-ban in {?,y}: republican (13.2)
##             anti-satellite-test-ban = n:
##             :...adoption-of-the-budget-resolution in {?,
##                 :                                     y}: democrat (53.7/9.8)
##                 adoption-of-the-budget-resolution = n:
##                 :...superfund-right-to-sue = n: democrat (10.7/0.1)
##                     superfund-right-to-sue in {?,y}: republican (85.6/16.2)
## 
## 
## Evaluation on training data (289 cases):
## 
## Trial	    Decision Tree   
## -----	  ----------------  
## 	  Size      Errors  
## 
##    0	     5   10( 3.5%)
##    1	     4   13( 4.5%)
##    2	     3   38(13.1%)
##    3	     2   14( 4.8%)
##    4	     4   53(18.3%)
##    5	     5   70(24.2%)
##    6	     3   21( 7.3%)
##    7	     2   14( 4.8%)
##    8	     7   22( 7.6%)
##    9	     3   45(15.6%)
##   10	     5   24( 8.3%)
##   11	     6    6( 2.1%)
##   12	     3   38(13.1%)
##   13	     6   17( 5.9%)
##   14	     5   10( 3.5%)
##   15	     6   47(16.3%)
##   16	     8   64(22.1%)
##   17	     3   21( 7.3%)
##   18	     3   22( 7.6%)
##   19	     6    9( 3.1%)
##   20	     4  107(37.0%)
##   21	     6   17( 5.9%)
##   22	     6   13( 4.5%)
##   23	     6   21( 7.3%)
##   24	     7    7( 2.4%)
## boost	          5( 1.7%)   <<
## 
## 
## 	   (a)   (b)    <-classified as
## 	  ----  ----
## 	   170     2    (a): class democrat
## 	     3   114    (b): class republican
## 
## 
## 	Attribute usage:
## 
## 	 98.62%	adoption-of-the-budget-resolution
## 	 98.27%	immigration
## 	 97.92%	physician-fee-freeze
## 	 97.23%	el-salvador-aid
## 	 96.54%	synfuels-corporation-cutback
## 	 95.50%	crime
## 	 92.04%	education-spending
## 	 64.36%	anti-satellite-test-ban
## 	 49.83%	religious-groups-in-schools
## 	 47.06%	export-administration-act-south-africa
## 	 45.33%	mx-missile
## 	 41.18%	water-project-cost-sharing
## 	 30.45%	duty-free-exports
## 	 30.10%	superfund-right-to-sue
## 
## 
## Time: 0.0 secs
```

----

## Trees
# Error Cost
<space>

- still getting too many false positives (predict republican but actually democrat)
- introduce higher cost to getting this wrong


```r
error_cost <- matrix(c(0,1,2,0),nrow=2)
cost_model <- C5.0(voting_train[,-1],voting_train[,1], trials=1, costs = error_cost)
```

```
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
```

```r
cost_predict <- predict(cost_model, newdata=voting_test[,-1])
conf <- CrossTable(voting_test[,1], cost_predict, prop.chisq = FALSE,
                   prop.c = FALSE, prop.r = FALSE,
                   dnn = c("actual class", "predicted class"))
```

```
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## |         N / Table Total |
## |-------------------------|
## 
##  
## Total Observations in Table:  145 
## 
##  
##              | predicted class 
## actual class |   democrat | republican |  Row Total | 
## -------------|------------|------------|------------|
##     democrat |         86 |          9 |         95 | 
##              |      0.593 |      0.062 |            | 
## -------------|------------|------------|------------|
##   republican |          1 |         49 |         50 | 
##              |      0.007 |      0.338 |            | 
## -------------|------------|------------|------------|
## Column Total |         87 |         58 |        145 | 
## -------------|------------|------------|------------|
## 
## 
```

----

## Trees
# Error Cost
<space>


```
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
## 
## Warning: 
## no dimnames were given for the cost matrix; the factor levels will be used
```

![plot of chunk plot_boost_acc](figure/plot_boost_acc.png) 

----

## Trees
# Pros and Cons
<space>

- trees are non-parametric, rule based classification or regression method
- simple to understand and interpret
- little data preparation
- works well with small or large number of features
<br>
- easy to overfit
- biased towards splits on features with large number of levels
- usually finds local optimum
- difficult concepts are hard to learn
- avoid pre-pruning
- hard to know optimal length of tree without growing it there first

----

## Summary
# ML - Part II
<space>

- Logistic regression
- Math behind PCA
- 3 types of clusters
- Trees and improvements

----

## Resources
<space>

- [Machine Learning with R](http://www.packtpub.com/machine-learning-with-r/book)
- [Machine Learning for Hackers](http://shop.oreilly.com/product/0636920018483.do)
- [Elements of Statistical Learning](http://web.stanford.edu/~hastie/local.ftp/Springer/OLD/ESLII_print4.pdf)

----
