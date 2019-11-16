# IMDb-Movie-Ratings-Predictions-using-R

# Introduction
Over the years, the film industry has relied on ratings as a means of evaluating the merit of a movie. Ratings have become an increasingly significant factor in the overall success of a movie, as it encapsulates the quality of all aspects of a film. Ideally, the higher the rating, the better the movie. IMDb (Internet Movie Database), a comprehensive online movie database, allows its registered users to rate any film on a scale of 1 to 10. Although IMDb has grown in popularity, the current formula IMDb uses for calculating a movie’s rating is not disclosed to the public. <br/>

Since the IMDb rating of a movie has become a vital piece of information for viewers, is it important that an accurate calculation be made to represent the full scope of the movie. A large number of factors contribute to the high or low IMDb ratings of a movie. Some of these factors are able to be quantified or categorized, which allows for the construction of a model to attempt to predict the IMDb rating of a movie. The goal of this project is to construct such a statistical model in order to predict the IMDb ratings of twelve upcoming blockbuster movies. The twelve movies are the following: Mickey and the Bear, Noelle, Atlantics, Charlie’s Angels, Ford v Ferrari, The Good Liar, The Report, Waves, Dark Waters, 21 Bridges, Beautiful Day in the Neighbourhood, and Frozen 2. A variety of factors were examined such as film, cast and production characteristics from approximately 6,000 movies on IMDb. Each variable was thoroughly explored in order to identify a set of predictors that have a significant relationship with the IMDb rating of a movie. Once the significant predictors were identified, a statistical model was crafted with the aim of increasing its predictive power. To avoid overfitting the training data, K-fold Cross Validation was used in order to test its out-of-sample performance. The following report discusses the variable distributions and relationships, the methodology used to build the model, and the results of the final model. 

# Data Description

*Data Preprocessing*<br/>
The dataset used in this project was downloaded in .xlsx format containing approximately 6,000 movies from IMDb. For each movie, the dataset contained information regarding variables such as identifiers, film characteristics, cast characteristics and production characteristics. Arbitrary variables such as the IMDb ID and IMDb URL, were removed since they did not provide any significant information for predicting the IMDb rating. The variable budget was also removed since it contained many null values and would result in the deletion of a significant amount of observations if it was included in the analysis. Variables that contain strings of characters such as the Film Title and the Main Actor’s Name were also removed since they contain thousands of unique categories that would be computationally expensive to dummify. Furthermore, many of these categorical variables such as Main Actor 1 Known For can be captured in other numeric variables, such as Main Actor 1 Star Meter. Additional variables were modified from the existing features such as the creation of a Day of the Week variable from the Release Day feature. A similar process was used to create a Language Classification variable from the Main Spoken Language variable using the top six main spoken languages. This procedure was also used to re-classify the Main Production Country into the top six production countries. <br/>

*Understanding the Predictors*<br/>
The distribution of each predictor was studied and the relationship between the target and predictor variable was examined to understand the data points. The distribution helps to identify skewness in the variables, which is indicated by a particular concentration of data points in a certain range. When the distribution of a variable is skewed, it may not be able to accurately predict outside that range. The skewness of the distribution of a variable can also potentially lead to overfitting. To correct for skewness, certain predictors were converted into the logarithmic scale. However, this change did not have any major effect of the mean squared error of the model. Hence, they are not included in the explanation.<br/>

The list below has information regarding the distribution and statistics of the predictors included in the final model.<br/>
1.	Language Classification: Buckets of languages was created by taking the top 6 languages in the dataset based on the frequency and classified the rest into ‘Others’.<br/>
87% of the movies are in English. The median IMDb Rating in each language group is between 6.5-7.0 (refer Figure 2.1)<br/>
2.	Release Year: Left skewed. 42% of the movies were released after 2000 (refer Figure 2.2)<br/>
3.	Release Month: Distributed almost evenly. No skewness Most of the movies have a rating between 6 to 8 (refer Figure 2.3)<br/>
4.	Duration Minutes: Distributed almost normally. Most of the movies have a rating between 6 to 8 (refer Figure 2.4)<br/>
5.	Total Number of Spoken Languages: The distribution is right skewed. 90% of the movies have total number of spoken languages less than 2 (refer Figure 2.5)<br/>
6.	Main Actor Star 2 Meter: The distribution does not really give enough information to understand this predictor, but when we run this in our forward selection model, it comes out to be significant. Since there are 5000 data points, and this attribute covers a wide range of values, scatter plot does not really give good info.<br/>
7.	Main Actor Star 3 Meter: Similar explanation as Main Actor Star meter 2<br/>
8.	Total Number of Actors: Right skewed. 75% of the dataset has total #actors between 6 to 24. (refer to Figure 2.8)<br/>
9.	Total Number of Directors: Right skewed. ~95% of the movies have 1 director (refer to Figure 2.9)<br/>
10.	Total Number of Producers: Right skewed. 74% of the movies have less than 2 producers (refer to Figure 2.10)<br/>
11.	Total Number of Production Companies: Right skewed. ~80% of the movies have less than 3 production companies (refer to Figure 2.11)<br/>

There were 3 predictors that were removed after the forward and backward subset selection method was performed on the initial linear model. The below list captures information on their distribution:<br/>
1.	Main Actor 1 Star Meter: Similar explanation as for Main Actor 2 Star Meter (refer to Figure 2.12)<br/>
2.	Total Number of Production Countries: The distribution is right skewed. 74% of the movies were produced in 1 country (refer to Figure 2.13)<br/>
3.	Total Number of Genres: 60% of the movies have 3 genres (refer to Figure 2.14)<br/>

*Dummified Variables*<br/>
The following information describes each dummified variable in the dataset: <br/>
1.	Genres: There are 24 genres in total, and a movie can fall into multiple genres (which gives an indication that interaction variables can be used). 54% of the movies fall in the genre Drama, followed by 35% in Comedy. The other 3 dominant categories are Crime (19%), Romance (18%) and Thriller (15%). (refer to Figure 2.15)<br/>
2.	Main Actor/Director Female Variables: 23% of the movies have main actor 1 as female, while 42% and 17% of the movies have main actors 2 and 3 as females respectively. Just 4% of the movies have a female director. (refer to Figure 2.16)<br/>

# Model Selection
*Variable Analysis*<br/>
After pre-processing the data and assessing the distribution of each variable, a simple linear model containing the IMDb rating and remaining independent variables (excluding dummified variables) was created. The purpose of this model was to identify different model issues that were present in the dataset such as collinearity, heteroskedasticity, outliers, and non-linearity. The final predictive model was created after addressing these issues. <br/>

Using the simple linear model, a Correlation Heat Map was created, and the Variance Inflation Factors (VIF) were tested to measure the correlation and collinearity between variables. The results showed that no collinearity exists between any variables since the absolute correlation coefficients between each variable was below 0.8 (Figure 3.1) and the VIF scores of all predictors were below 4 (Figure 3.2). In order to detect heteroskedasticity numerically, the Non-Constant Variance Test was used on each variable. Variables that returned a p-value less than 0.05 on the ncvTest were noted (Figure 3.3). These heteroskedastic predictors were corrected to obtain the true linear significance of the predictor. To detect outliers in the dataset numerically, a Bonferroni outlier test was performed. The 10 outliers that had a Bonferroni p-value less than 0.05 were reported and then removed from the dataset (Figure 3.4). In order to determine the non-linear predictors, residual plots were built for each predictor and the model as a whole (Figure 3.5). The results obtained from the residual plots revealed that the model and certain predictors cannot be treated a linear since their p-values were less than 0.05 (Figure 3.6). These predictors were flagged in order to further explore their polynomial degrees in a later stage of model selection. <br/>

*Model Building*<br/>
Once all possible model issues were identified and corrected, forward and backward subset selection was used to select the optimal combination of predictors that results in the highest adjusted r-squared (Figure 3.7 and Figure 3.8). The 11 predictors that were identified by these selection tests were included in the final statistical model. Of the selected predictors, those that were identified as non-linear from the residual plots’ tests were treated as polynomial functions to avoid underfitting the data. The polynomial degree of each non-linear predictor that resulted in the best out-of-sample performance (lowest MSE) of the model using K-Fold Cross-Validation with 20 folds was used. Using spline functions for non-linear predictors did not improve the MSE of the model significantly and were avoided to reduce the possibility of overfitting the data. To increase the predictive power of the model, all dummified variables which increased the out-of-sample performance were included. Additionally, interaction variables between predictors that reduced the MSE of the model were included.<br/>

# Results
*Model Significance*<br/>
After training the final model on the training data set, an F-statistic of 41.68 was obtained. The p-value of the F-statistic was <0.01 which indicates that the model is highly significant at predicting the IMDb rating of a movie. The p-value of a model indicates the probability that the relationship between the model and the target variable (IMDb rating) is purely random. Therefore, obtaining a small p-value of the F-statistic implies that the model is significant at predicting IMDb ratings of a movie. 
The adjusted r-squared value indicates the percentage of the variance in the IMDb ratings that can be explained by changes in the predictors using a linear regression. The final model has an adjusted r-squared value of 0.365 which implies that the model is fairly robust and is not trying to overfit to the training data. An adjusted r-squared value of 0.365 implies that the model captured the variability of IMDb ratings for 36.5% of the movies in the training dataset. To see the final model’s summary statistics for interpretation purposes, please refer to Figure 4.1. <br/>


*Predictor Significance*<br/>
The final model included over 39 variables for predicting the IMDb rating of the twelve blockbuster movies. It was found that certain predictors have a more significant relationship with the IMDb ratings of movies than others. The significance of the predictors in the model is indicated by a low p-value (<0.01). The most significant predictors of the final model, which had a p-value <0.01 were the following:<br/>

*Model Performance* <br/>
After testing the significance of the final model, the out-of-sample performance of the model was tested. A K-Fold Cross-Validation Test with 20 folds was used on the model to understand how the model performs with new data. The Leave-One-Out Cross-Validation Test and Validation Set-Test was not performed because it would be too computationally expensive due to the data size constraints. Furthermore, K-Fold Cross-Validation provides the right balance of accuracy and computational power required. A low value of k=20 was chosen to ensure that the model’s performance was tested thoroughly. <br/>

The mean squared error obtained from the K-Fold Cross-Validation Test was 0.627. This implies that the theoretical average error of the model is 0.791. Therefore, on average, the model will deviate from the true IMDb rating of a new movie by ±0.79 ratings points. 
The true test of the final model comes from its ability to predict the IMDb ratings of twelve upcoming blockbuster movies. The relevant data was collected for each predictor included in the model for seven of the twelve movies from the IMDb website. The predictions obtained from the model for the twelve movies was plotted with their current IMDb ratings (as of November 6, 2019) to observe the variation in the IMDb ratings per movie. Some movies did not have a rating on IMDb as of November 6, 2019 and therefore the ratings could not be plotted nor compared. Please note, the IMDb ratings of the movies can vary over time.<br/>

From the seven movies whose IMDb rating was available on IMDb, the errors between the actual IMDb ratings and the predicted IMDb ratings were all within the predicted average error range of ±0.79 ratings points. The model performed exceptionally well when predicting the IMDb ratings of the movies Waves and The Report as the error was only 0.03 and 0.15 rating points respectively. For the complete list of all twelve predicted IMDb rating points refer to Figure 4.4 in the appendix. <br/>


# Conclusion
In conclusion, to build a model that predicts the IMDb ratings of twelve upcoming blockbuster movies, the following steps were performed:<br/>
1.	Data Pre-Processing <br/>
2.	Predictor Significance Testing <br/>
3.	Testing for Heteroskedasticity, Collinearity, Outlier Testing and Non-Linearity of Predictor Testing<br/>
4.	Forward/Backward Subset Selection<br/>
5.	Selecting Polynomial Degrees for Non-Linear Predictors<br/>
6.	Adding Interaction Variables and Dummified Variables<br/>
7.	Performing K-Fold Cross-Validation Testing<br/>
8.	Predicting on Test Dataset<br/><br/>
The next steps in this project would be to add data that has greater diversity in the training stages to improve the model. For example, adding data that contains predictors of greater relevance to IMDB ratings could possibly help lower the mean square error of the model. Predictors like budget, certification, and number of premiers would have been key predictors to include in the model. However, the lack of consistency in the data available for these predictors made it challenging to include them in the final model.
In conclusion, with the given data the final model is able to predict the IMDb ratings of the twelve upcoming blockbuster movies with a high-level of accuracy and is fairly robust to outliers. The final model address both issues of overfitting and underfitting while simultaneously capturing the full essence of the dataset accurately. <br/>





