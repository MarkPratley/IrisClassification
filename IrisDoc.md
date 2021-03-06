---
layout: page-fullwidth
title: "Iris Classification"
# subheadline: "Iris Classification Using Caret"
meta_teaser: "An exploration of simple classification using caret with Edgar Anderson's famous [Iris data set](http://archive.ics.uci.edu/ml/datasets/Iris)."
teaser: "An exploration of simple classification using caret with Edgar Anderson's famous Iris data set."
breadcrumb: true
header:
    title: ""
#    title: Iris Data
#    image: Iris-Data-Big.png
    image_fullwidth: Iris-Data-Big.png
    background-color: "#262930"
    caption: Iris Data Pairwise Comparison
#    caption_url: https://unsplash.com/
image:
    thumb:  Iris-Data.png
    homepage: Iris-Data-Big.png
    caption: Iris Data Pairwise Comparison
#    caption_url: "http://www.aisleone.net/"
categories:
    - projects
tags:
    - r
    - Iris Dataset
    - ggplot2
---

This is a quick exploration of simple classification using caret with Edgar Anderson's famous [Iris data set](http://archive.ics.uci.edu/ml/datasets/Iris).

The goal of the project is to utilise caret alongside a variety of machine learning algorithms to correctly classify irises into their species group.

### The Data

Anderson's Iris data set gives the measurements in centimeters of the 4 predictors, as well as the Species:

* Sepal Length
* Sepal Width
* Petal Length
* Petal Width
* Species
    * *iris setosa*
    * *iris versicolor* 
    * *iris virginica*


{% highlight r %}
summary(iris)
{% endhighlight %}



{% highlight text %}
##   Sepal.Length    Sepal.Width     Petal.Length    Petal.Width   
##  Min.   :4.300   Min.   :2.000   Min.   :1.000   Min.   :0.100  
##  1st Qu.:5.100   1st Qu.:2.800   1st Qu.:1.600   1st Qu.:0.300  
##  Median :5.800   Median :3.000   Median :4.350   Median :1.300  
##  Mean   :5.843   Mean   :3.057   Mean   :3.758   Mean   :1.199  
##  3rd Qu.:6.400   3rd Qu.:3.300   3rd Qu.:5.100   3rd Qu.:1.800  
##  Max.   :7.900   Max.   :4.400   Max.   :6.900   Max.   :2.500  
##        Species  
##  setosa    :50  
##  versicolor:50  
##  virginica :50  
##                 
##                 
## 
{% endhighlight %}

The iris data set was chosen partly as it requires no pre-processing or transformation, which is ideal for quickly looking at caret and classification, and also partly because it is a classic classification data set.

### Pairwise Correlation Matrix

First we will create a Pairwise Correlation Matrix to explore the variables and look for any relationships.


{% highlight r %}
ggpairs(iris,
        colour='Species',
        alpha=0.4,
        title = "Anderson's Iris Data -- 3 species")
{% endhighlight %}

![center](/figs/IrisDoc/unnamed-chunk-3-1.png)

These pairwise scatterplots show that there is strong correlation between Petal.Length and Petal.Width. And also correlation between Sepal.Length and Petal.Length, and also Sepal.Length with Petal.Width.

We can also see a degree of separation between *iris setosa* and the other Irises, but there is no clear separation between *iris versicolor* and *iris virginica*

### Comparing Sepal.Length and Sepal.Width


{% highlight r %}
ggplot(iris, aes(x=Sepal.Length, y=Sepal.Width, colour=Species)) + 
    geom_point() +
    ggtitle("Sepal.Length vs Sepal.Width")
{% endhighlight %}

![center](/figs/IrisDoc/unnamed-chunk-4-1.png)

Comparing Sepal.Width and Sepal.Length whilst colouring according to species shows good separation between *iris setosa* and the other iris species, but again only minor difference between *iris versicolor* and *iris virginica.*


### Comparing Petal.Length and Petal.Width


{% highlight r %}
ggplot(iris, aes(x=Petal.Width, y=Petal.Length, colour=Species)) + 
    geom_point() +
    ggtitle("Petal.Length vs Petal.Width")
{% endhighlight %}

![center](/figs/IrisDoc/unnamed-chunk-5-1.png)

From this graph, looking at the Petal.Width vs Petal.Length and colouring according to species, we can see both strong correlation and also pretty good separation betwen the 3 species, except for a few uncertain points between the blue/green groups.


## Models

We'll build some models and use them to classify species.



### Test/Training Data

First we need to split our data set in test and training sets.

The function createDataPartition can be used to create a [stratified random sample](http://stattrek.com/statistics/dictionary.aspx?definition=stratified_sampling) of the data.


{% highlight r %}
inTrainingSet <- createDataPartition(iris$Species, p = .50, list = FALSE)
irisTrain <- iris[ inTrainingSet,]
irisTest  <- iris[ -inTrainingSet,]
{% endhighlight %}

### [Random Forest](https://en.wikipedia.org/wiki/Random_forest)

We now can use caret with a standard random forest method to create a model for clasifying iris species.


{% highlight text %}
## Loading required package: randomForest
{% endhighlight %}

Let's check our model by using it on the unseen test data to predict the species based on the other variables.


{% highlight r %}
pred.rf <- predict(m.rf, irisTest)
{% endhighlight %}

Now we can view a [confusion matrix](https://en.wikipedia.org/wiki/Confusion_matrix) to see how well our random forest classified the data.


{% highlight r %}
cm.rf <- confusionMatrix(pred.rf, irisTest$Species)
kable(cm.rf$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         24|         1|
|virginica  |      0|          1|        24|

This is pretty good, classifying all 25/25 of the *iris setosa* data correctly, only misclassifying 1/25 *iris versicolor* as *iris virginica*, and misclassifying 1/25 *iris virginica* as *iris versicolor*



### [C5.0](https://en.wikipedia.org/wiki/C4.5_algorithm)

Now we will try classifying with the C5.0 algorithm.


{% highlight r %}
m.C50 <- train(Species ~ ., data=irisTrain, method="C5.0" )
{% endhighlight %}



{% highlight text %}
## Loading required package: C50
{% endhighlight %}


{% highlight r %}
pred.C50 <- predict(m.C50, irisTest)
cm.C50 <- confusionMatrix(pred.C50, irisTest$Species)
kable(cm.C50$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     23|          0|         0|
|versicolor |      2|         24|         1|
|virginica  |      0|          1|        24|


### [Naive Bayes](https://en.wikipedia.org/wiki/Naive_Bayes_classifier)

<BR>

{% highlight r %}
m.nb <- train(Species~., data=irisTrain, method="nb")

pred.nb <- predict(m.nb, irisTest)
cm.nb <- confusionMatrix(pred.nb, irisTest$Species)
kable(cm.nb$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         24|         2|
|virginica  |      0|          1|        23|


### Naive Bayes with [k-fold Cross Validation](http://en.wikipedia.org/wiki/Cross-validation_(statistics\))

Adding k-fold (in this case 10-fold) cross validation to create our naive bayes model.

{% highlight r %}
train_control <- trainControl(method="cv", number=10)

m.nbkf <- train(Species~., data=irisTrain, trControl=train_control, method="nb")

pred.nbkf <- predict(m.nbkf, irisTest)
cm.nbkf <- confusionMatrix(pred.nbkf, irisTest$Species)
kable(cm.nbkf$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         24|         2|
|virginica  |      0|          1|        23|


### [k-nearest neighbours](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm)

<BR>

{% highlight r %}
m.knn <- train(Species~., data=irisTrain, method="knn")

pred.knn <- predict(m.knn, irisTest)
cm.knn <- confusionMatrix(pred.knn, irisTest$Species)
kable(cm.knn$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         25|         2|
|virginica  |      0|          0|        23|


### [Neural Network](https://en.wikipedia.org/wiki/Artificial_neural_network)

<BR>

{% highlight r %}
m.nnet <- train(Species~.,data=irisTrain,method="nnet", trace=FALSE)
{% endhighlight %}



{% highlight text %}
## Loading required package: nnet
{% endhighlight %}



{% highlight r %}
pred.nnet <- predict(m.nnet, irisTest)
cm.nnet <- confusionMatrix(pred.nnet, irisTest$Species)
kable(cm.nnet$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         24|         0|
|virginica  |      0|          1|        25|


### Neural Network with [Bootstrapping](https://www.wikiwand.com/en/Bootstrapping_(statistics\))

<BR>

{% highlight r %}
tc <- trainControl(method="boot",number=25)

m.nnet.bs <- train(Species~.,data=irisTrain,method="nnet",trControl=tc, trace=FALSE)

pred.nnet.bs <- predict(m.nnet.bs, irisTest)
cm.nnet.bs <- confusionMatrix(pred.nnet.bs, irisTest$Species)
kable(cm.nnet.bs$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         24|         0|
|virginica  |      0|          1|        25|


### Random Forest with [Leave One Out Cross Validation](https://www.wikiwand.com/en/Bootstrapping_(statistics\))


{% highlight r %}
tc <- trainControl(method="LOOCV", number=25)

m.rf.bs <- train(Species~.,data=irisTrain,method="rf",trControl=tc, trace=FALSE)

pred.rf.bs <- predict(m.rf.bs, irisTest)
cm.rf.bs <- confusionMatrix(pred.rf.bs, irisTest$Species)
kable(cm.rf.bs$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         24|         1|
|virginica  |      0|          1|        24|


### Elastic Net [glmnet](https://cran.r-project.org/web/packages/glmnet/vignettes/glmnet_beta.html)


{% highlight r %}
m.glmnet <- train(Species~., 
                  data=irisTrain, 
                  method="AdaBoost.M1")
{% endhighlight %}



{% highlight text %}
## Loading required package: adabag
{% endhighlight %}


{% highlight r %}
pred.glmnet <- predict(m.glmnet, irisTest)
cm.glmnet <- confusionMatrix(pred.glmnet, irisTest$Species)
kable(cm.glmnet$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         24|         1|
|virginica  |      0|          1|        24|


### Using Pre-Processing (including [PCA]) with Neural Net


{% highlight r %}
m.nnet.pp <- train(Species~.,
                data=irisTrain,
                method="nnet",
                preProcess=c("BoxCox", "center", "scale", "pca"),
                trace=FALSE)

pred.nnet.pp <- predict(m.nnet.pp, irisTest)
cm.nnet.pp <- confusionMatrix(pred.nnet.pp, irisTest$Species)
kable(cm.nnet.pp$table)
{% endhighlight %}



|           | setosa| versicolor| virginica|
|:----------|------:|----------:|---------:|
|setosa     |     25|          0|         0|
|versicolor |      0|         23|         4|
|virginica  |      0|          2|        21|


## Results

Having used a variety of different models to classify the iris data set, we can now attempt to compare their performance.
Comparing confusion matrices isn't entirelty straight-forward as models perform differently in different areas, some optimising for specificity, some for sensitivity, or for some other metric.
I have chosen to use [Matthews Correlation Coefficient](https://en.wikipedia.org/wiki/Matthews_correlation_coefficient) which gives a single number in the range -1 to +1, and is a correlation coefficient between the observed and predicted binary classifications.


{% highlight r %}
results <- results %>% arrange(desc(MCC))
kable(results)
{% endhighlight %}



|Name                  |       MCC|
|:---------------------|---------:|
|Neural Net            | 0.9802614|
|Neural Net w Boot     | 0.9802614|
|KNN                   | 0.9610256|
|Random Forest         | 0.9600000|
|Random Forest w LOOCV | 0.9600000|
|Random Forest w LOOCV | 0.9600000|
|Naive Bayes           | 0.9402508|
|Naive Bayes k-fold    | 0.9402508|
|C5.0                  | 0.9209829|
|Neural Net PP         | 0.8809402|



{% highlight r %}
ggplot(results, aes(x=Name, y=MCC, fill=Name)) + 
geom_bar(stat="identity") +
ggtitle("Comparison of Classification Methods") + 
theme(axis.text.x=element_blank(),
      axis.title.x=element_blank())
{% endhighlight %}

![center](/figs/IrisDoc/unnamed-chunk-28-1.png)

From the table and graph it is possible to see that all the models performed similarly, in the range 0.88 - 0.98. But the Neural Network performed the best MCC=0.98, followed by KNN with MCC=0.96.

Another point of interest is that the worst model was also the neural network, but with pre-processed data using BoxCox, centering, scaling and PCA which implies that innapropriate pre-processing can be worse than none.

<!-- ggplot(results, aes(x=Name, y=MCC, fill=Name)) +  -->
<!--     geom_bar(stat="identity") + -->
<!--     theme(axis.text.x  = element_text(angle=90, vjust=0.5, size=16)) + -->
<!--     ggtitle("Comparison of Classification Methods") + +     theme( -->
<!--             axis.line=element_blank(), -->
<!--             axis.text.x=element_blank(), -->
<!--             axis.text.y=element_blank(), -->
<!--             axis.ticks=element_blank(), -->
<!--             axis.title.x=element_blank(), -->
<!--             axis.title.y=element_blank(), -->
<!--             legend.position="none", -->
<!--             panel.background=element_blank(), -->
<!--             panel.border=element_blank(), -->
<!--             panel.grid.major=element_blank(), -->
<!--             panel.grid.minor=element_blank(), -->
<!--             plot.background=element_blank() -->
<!--            ) -->
