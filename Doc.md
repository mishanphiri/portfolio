Artifcial Neural Networks
================
Mishan Phiri
2023-12-07

## Exploratory Data Analysis

Ecologists often capture and release animals and record various physical
measurements as well as species classification. Determining species
require some expertise, and ecologists wish to automate the
classification process using only the measurements taken.  
Firstly let us take a look at the data available to us.

<table>

| Variable | Description                                                                |
|----------|----------------------------------------------------------------------------|
| `SpecA`  | One hot encoded indicating Species A                                       |
| `SpecB`  | One hot encoded indicating Species B                                       |
| `SpecC`  | One hot encoded indicating Species C                                       |
| `Wing`   | Length in metres of primary wing feather from tip to wrist it attaches to. |
| `Weight` | Body weight in kilograms.                                                  |

<table>

The data is already cleaned and standardised, therefore we skip this
step and instead visualise the data to seek out any patterns.

``` r
#Combine Hot encoded data for easy EDA
Spec = c(rep('SpecA', 49), rep('SpecB', 50), rep('SpecC', 49)) 
eda.data = cbind(Spec, Data[,c(4,5)])
eda.data$Spec = as.factor(Spec)
#Plot wings vs Weight
qplot(x = Wing, y =Weight,
      data = eda.data, color = Spec,
      main = 'Species Weight and Wing Span')
```

![Hawk species wing length and body
weight](Doc_files/figure-gfm/eda-1.png) There is a clear distinction
between Species A measurements and the other species. Species B and C
have a slight overlap and more similar values at their bounds, but over
all the Species are clearly separable.

## Building a Neural Network.

### Activation Functions.

For simplicity’s sake we will consider a ‘vanilla’ neural network with a
single hidden layer. We will use a softmax activation function in on the
output layer:  

![A^L = exp(\textbf{Z})\cdot(exp(\textbf{Z})^\prime\textbf{1})](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;A%5EL%20%3D%20exp%28%5Ctextbf%7BZ%7D%29%5Ccdot%28exp%28%5Ctextbf%7BZ%7D%29%5E%5Cprime%5Ctextbf%7B1%7D%29 "A^L = exp(\textbf{Z})\cdot(exp(\textbf{Z})^\prime\textbf{1})")

The softmax activation function was chosen as it behaves as a
distribution and the output can be treated as probabilities,which is
more suitable for classification type problems. The rectified linear
units (ReLU) defined as
![max(0,z) \in \mathbb{R}^+](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;max%280%2Cz%29%20%5Cin%20%5Cmathbb%7BR%7D%5E%2B "max(0,z) \in \mathbb{R}^+")
is the activation function used on the single hidden layer. We use the
matrix notation of the activation functions as it computes faster than
evaluating each observation individually.  
Firstly we build a function called `soft_max` that evaluates the matrix
of inputs and returns a transpose of the class probabilities and a
wrapper function for the ReLU activation function called `sig1`.

``` r
soft_max = function(z){
  #z is input in layer L
  n = ncol(z)
  one = matrix(1,q,1) #A matrix of ones to sum the denominator
  den = 1/(c(t(exp(z))%*%one)) #suming the rows and inverting the variables
  d = diag(den)
  out= exp(z)%*%d 
  #return class probabilities
  return(out)
}

sig1  = function(x){
  #ReLU Function applied on Matrix X
  pmax(x,0)
}
```

### Neural Network

Secondly we build a function called `neural_net` that takes in
arguments:

<table>

| Argument                                                                                                          | Description                                                                                                                                              |
|-------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| X                                                                                                                 | the design matrix                                                                                                                                        |
| Y                                                                                                                 | the response matrix                                                                                                                                      |
| theta (![\theta](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Ctheta "\theta")) | a vector of starting parameter values                                                                                                                    |
| m                                                                                                                 | the number of nodes on the missing layer                                                                                                                 |
| nu (![\nu](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cnu "\nu"))             | the regularization parameter ![\nu \geq 0](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cnu%20%5Cgeq%200 "\nu \geq 0") |
| pred                                                                                                              | a boolean operator specifying whether we are making a prediction.                                                                                        |

<table>

``` r
neural_net = function(X,Y,theta,m, nu, pred = FALSE){
  #Dimension Variables
  N = dim(X)[1]
  p = dim(X)[2]
  q = dim(Y)[2]
  
  #Populate Relevant matrices, b are bias vectors and W are weight matrices
  index = 1:(p*m)
  W1    = matrix(theta[index],p,m)
  index = max(index) + 1:(m*q)
  W2    = matrix(theta[index],m,q)
  index = max(index) + 1:m
  b1    = matrix(theta[index],m,1)
  index = max(index) + 1:q
  b2    = matrix(theta[index],q,1)
  # A vector of ones for creating bias matrices
  ones = matrix(1,1,N)
  
  # Evaluate the updating equation in matrix form
 
  A0 = t(X) #Input layer
  A1 = sig1(t(W1)%*%A0+b1%*%ones) #Hidden layer
  A2 = soft_max(t(W2)%*%A1+b2%*%ones) #Output layer
  pi_hat = t(A2) #estimated probabilities
  if(!pred){ #Run if not predicting: Calculating Error
   error = rep(0,N)
     whA = which(Y[,1]==1)
     whB = which(Y[,2]==1)
     whC = which(Y[,3]==1)
     error[whA] = -log(pi_hat[whA,1])
     error[whB] = -log(pi_hat[whB,2])
     error[whC] = -log(pi_hat[whC,3])
    
     E1 = sum(error)/N #Cost Function
     E2 = E1 + nu*(sum(W1^2)+sum(W2^2))/(2*N) # Penealised cost function
     # Return a list of relevant objects: estimated probabilities, The error and penalised error
    return(list(out = pi_hat, E1 = E1, E2 = E2))
  }else #When predicting only return estimated errors
    {return(pi_hat)}
}
```

Since we start with random parameters of the network, therefore we need
to find the ‘best’ approximation of the relationship between the
predictors and the response. In the context of a classification problem,
the ‘best’ parameters are taken to be the set that maximises the
probability of
![\hat{y}=y](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Chat%7By%7D%3Dy "\hat{y}=y").
Under such construction we can minimise the cross-entropy error:

![- \frac{1}{N}\sum\_{i=1}^N \left(y\_{iA}log(\hat{y}\_{iA}) + y\_{iB} log(\hat{y}\_{iB}) + y\_{iC}log(\hat{y}\_{iC}) \right)](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;-%20%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bi%3D1%7D%5EN%20%5Cleft%28y_%7BiA%7Dlog%28%5Chat%7By%7D_%7BiA%7D%29%20%2B%20y_%7BiB%7D%20log%28%5Chat%7By%7D_%7BiB%7D%29%20%2B%20y_%7BiC%7Dlog%28%5Chat%7By%7D_%7BiC%7D%29%20%5Cright%29 "- \frac{1}{N}\sum_{i=1}^N \left(y_{iA}log(\hat{y}_{iA}) + y_{iB} log(\hat{y}_{iB}) + y_{iC}log(\hat{y}_{iC}) \right)")

### Validation and Regularisation.

A crucial component of training a neural network is the test set. We use
it to determine whether the model has captured the data’s underlying
pattern, and ensure generalisation. One way to prevent overfiting the
model to the training set is to penalise the model for being overly
complex through regularisation. This essentially constrains the
optimization problem and the objective function can be rewritten as:

![- \frac{1}{N}\sum\_{i=1}^N \left(y\_{iA}log(\hat{y}\_{iA}) + y\_{iB} log(\hat{y}\_{iB}) + y\_{iC}log(\hat{y}\_{iC}) \right) + \frac{\nu}{2N}\sum{W^2}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;-%20%5Cfrac%7B1%7D%7BN%7D%5Csum_%7Bi%3D1%7D%5EN%20%5Cleft%28y_%7BiA%7Dlog%28%5Chat%7By%7D_%7BiA%7D%29%20%2B%20y_%7BiB%7D%20log%28%5Chat%7By%7D_%7BiB%7D%29%20%2B%20y_%7BiC%7Dlog%28%5Chat%7By%7D_%7BiC%7D%29%20%5Cright%29%20%2B%20%5Cfrac%7B%5Cnu%7D%7B2N%7D%5Csum%7BW%5E2%7D "- \frac{1}{N}\sum_{i=1}^N \left(y_{iA}log(\hat{y}_{iA}) + y_{iB} log(\hat{y}_{iB}) + y_{iC}log(\hat{y}_{iC}) \right) + \frac{\nu}{2N}\sum{W^2}")

It is the cross entropy error plus a second term; the sum of the squared
weights scaled by a factor of
![\frac{\nu}{2N}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cfrac%7B%5Cnu%7D%7B2N%7D "\frac{\nu}{2N}").
Half the data is sampled using a seed of 2023, and used as our training
set. Since
![\nu](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cnu "\nu")
is a hyper parameter, we aim to asses how it affects out of sample error
at different levels.

<figure>
<img src="Doc_files/figure-gfm/select%20nu-1.png"
alt="The validation error plotted against different levels of regularization" />
<figcaption aria-hidden="true">The validation error plotted against
different levels of regularization</figcaption>
</figure>

Therefore we optimise the objective function using newtons method over
different values of
![\nu](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cnu "\nu").
The figure above shows the plot of the test error and the range of
![\nu](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cnu "\nu").
The out of sample error is minimised at
![\nu = 0.026](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cnu%20%3D%200.026 "\nu = 0.026"),
past this value, the network is overly complex and begins returning
gibberish. A regularization parameter close to 0, such as this case
indicates a model that is complex enough to capture the underlying
pattern.

### Regularization and Parameters

![](Doc_files/figure-gfm/parameters-1.png)<!-- -->

We used
![L_2](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;L_2 "L_2")
regularization, the effect on the individual weights is to shrink the
weights in magnitude, in opposed to setting less important weights to
zero. The Figure above shows how the weights and biases have been
affected by the addition of the
![\nu](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cnu "\nu").

### Prediction Map and closing remarks

In order to create a prediction map, we create all possible combinations
of body weight and wing length and predict the possible species. The
result is the figure below with a 98% prediction accuracy.

``` r
#Plotting Response curves

cols = c('palegreen', 'lightblue', 'orchid2')
cols2 = c('red', 'green', 'blue')

plot(y.coords~ x.coords, pch = 16, col = cols[y1],
     main = 'Response Map', 
     xlab = "Wing", ylab = "Weight")
points(X[,2]~ X[,1],pch = 20, cex = 1, col = cols2[eda.data$Spec])

legend(0.15, 1.5 ,legend = c("SpecA", 'SpecC', 'SpecB'),
       col = c('palegreen', 'lightblue', 'orchid2'),
       title = "Predicted",
       pch=15, cex=0.5,
       box.lty=0)
legend(0.18, 1.5 ,legend = c("SpecA", 'SpecC', 'SpecB'),
       col = c('red', 'green', 'blue'),
       title = "Actual",
       pch=16, cex=0.5,
       box.lty=0)
```

![](Doc_files/figure-gfm/response%20heatmap-1.png)<!-- -->

Neural networks have proven to be a sufficient tool to predict the
species classes. Few hidden layers and nodes were needed as the data is
separable. A more complex dataset would need more parameters. Often
Neural networks make use of a sort of stochastic gradient decent, where
we evaluate the cost function with respect to the model parameters. This
is refereed to as back propagation, as we work backwards from the cost
function and calculate the gradient w.r.t the parameters in each layer
consecutively.
