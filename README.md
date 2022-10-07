
#Shell-AI Hackaton Submission Explanation<br/>
Link to the Competition: https://www.hackerearth.com/en-us/challenges/competitive/shellai-hackathon-2022/

-Sebastian Bott, Alexander Heelemann-<br/>
##Introduction:<br/>
Our Team consists of two people. Both of us are studying Information Technology and Economics at the University FOM Bonn, Germany. We’re working in the field of Data Science and joined this competition to compete, learn, enhance our portfolio, and potentially collaborate with other data science enthusiasts around the world. This text file contains the explanation of our approach, as well as further thoughts and possible alterations, which couldn’t be done due to time constraints. Take care and enjoy.<br/>
<br/>
##The Problem:<br/>
The problem we were given by this competition focuses on the topic EV charging station optimization. The goal is to satisfy the exponentially rising demand of available charging locations, due to same exponentially rising popularity in electrical vehicles, for the upcoming years (here 2019 and 2020). The data we were given consists of two csv files, each containing different data. The file demand history holds the demand-history of each of the 4096 demand points, in the time range 2010-2018 and the coordinates of the demand points. The second file contains the existing charging infrastructure, the number of total parking spots for each of the 100 parking locations, as well as their respective coordinates.<br/>
Evaluation metric of the optimization problem are split into 3 parts. The first one (and in terms of accelerating the EV – popularity, the most important) is the customer satisfaction cost. The main part here is to satisfy the forecasted demand at each demand location with  charging spots as close as possible. The second one focuses on forecasting future demand based on past data. This one is indirectly connected to the customer satisfaction, since if the forecasted demand is way less than the actual demand, customers will have downsides in terms of range anxiety and time consumption. The third one focuses on cost minimization while building the infrastructure. Here we want to build as much as needed, but as less as possible. Also, there are two possible chargers one can build, a slow charger with an output of 200 KW, and a fast charger with 400 KW, but also 1.5 higher building costs. On these goals and metrics, we get constraints to fulfill. More on them in our approach explanation. <br/>
<br/>
##Approach:<br/>
###Divide and conquer:<br/>
Since we’re dealing with a multidimensional problem, with different goals, metrics, and constraints, we started to divide all of them in smaller problems to optimize. <br/>
The first big separation is to work on 2019 and 2020 optimization on their own. The algorithm will be the same but we’re looking to optimize each year of their own. Reason for that is, in the real world, we can use the data we get from 2019 to optimize the forecast what needs to be build for 2020. Our algorithm can be used for suggestions many years in the future, but in general, it should be used to exactly optimize the building for the upcoming year.<br/>
The second separation splits the optimization functions, to find the optimal algorithm for each of their own. First, we worked on the forecast of the demand for 2019 and 2020. Second, we worked on the optimal charging station building. Third we worked on the assignment from each demand point to each supply point to get the overall cost of possible customer satisfaction. All of them have their respective constraints, which we’ll add to the algorithm. After that we put everything together and get the final algorithm and solution.<br/>
<br/>
###Forecasting:<br/>
The data we got for forecasting is sparse and visually has a linear rising demand in the year 2013 to 2018. Since EV’s are exponentially gaining popularity since the late 20s, we can’t just use the data we get from the history file, but also need add a factor, which we get from market research, to better pinpoint the expected demand. For this competition we can use data of sold EV’s in the year 2019 and 2020. For real world adaptation of our algorithm, one could use market research data on predicted EV-sales for the years to come. Those would include upcoming new EV-types, government incentives and price adjustments for example. To optimize this approach, one could find the optimal factor based on EV-market research in comparison to actual demand for each year.<br/>
Due to time constraints, we used a 2-degree polynomial regression algorithm to include potentially exponential rise of demand, which wouldn’t be included using linear regression. Scikit Learn provides a model which is easy to implement. We also added a factor to the predicted demands, based on sales in the year 2019 and 2020. The factor for 2019 is 1.1 and for 2020 1.3. Further research could optimize these factors.<br/>
Side node, there were demand points with 0 demand in the dataset. These could be an error in the csv file. Since we can’t talk to the people who own the dataset, we kept them as they are. We also tried to fill them with average data of the demand points around them, which decreased our online score, hence we left them untouched for this competition. Possible explanation for them could be, they are inaccessible areas where demand can’t arise like lakes, or closed factory areas.<br/>
<br/>
###Charging Station Building:<br/>
To build the charging stations, we need different information, to optimize the result. The first one is the distance from the charging station to the demand points. The second one is the expected demand for each demand point. We already have the second one, because of the forecast we did one step prior. The first one will be an extra data frame we create with a short algorithm in our ipynb file. There, one cell produces the output ‘df_distance_demand’supply’ which includes the distance from each supply point to each of the 4096 demand points. With that two information’s we can write an optimization function which builds as much charging station as needed, at the locations that need them the most, while choosing between Fast and Slow Charger. Constraints here are that the Number of Charging stations build, must be consistent or more than the year before, in short: We don’t want to tear down stations. The second constraint is that the number of charging stations must be a full integer, we can’t build halve a station (duh). The third one is that the number of total charging stations per parking space can’t exceed the maximum number of parking spots.<br/>
With these constraints in mind, we designed a linear optimization algorithm, using the pulp library as backbone. Pulp is an open-source library which takes linear functions and variables as an input and  uses different optimizer algorithms like CPLEX or COIN to maximize or minimize a target function, depending on the use case. For this problem we use pulp minimize to reduce the overall costs of building charging stations. The code for this section is below the Headline “Algorithm for Optimization Demand and Supply points”<br/>
<br/>
Xe initialize 100 variables, 1 for each of the supply points. We divide them in y and z, so we can look for both fast and slow charger. With cat=’Integer’, the constraint of having only full integers will automatically be satisfied.<br/>
<br/>
The next line is our Optimization goal. We want to minimize the costs, which are a result of this function. In general, this is just a translation of the 3. Cost function, cost of infrastructure.<br/>
<br/>
Next upcoming are the constraints, they speak for their own. <br/>
To optimize the charger, build location we need to add the forecasted demand for each demand point with the distance to the supply points. In earlier iterations we implemented this also in our charging station building optimization algorithm. In the final iteration we already put them together with the demand assignment.<br/>
<br/>
###Demand assignment:<br/>
The demand assignment was a bit more complex. Here we want to satisfy as much demand as possible on the closest supply point. Since supply points can be full (in terms on demand satisfied), a demand point might be split on multiple supply points. To have every combination possible in our optimization function we need the cross product of demand points (n=4096) and supply points (n=100), which leads to 409600 possible combinations. With this step we enable the algorithm to try out every possible solution, including splitting up demands on multiple supply stations. It increases the computational time and load drastically, since we’re dealing with 406900 Variables to optimize, just for the demand points, but enables us to find the optimal solution.<br/> 
<br/>
Here we initialize our variable dictionary x with the 406900 variables. We get them from the data frame we previously created, containing all combinations with their respective range. <br/>
<br/>
The next code contains the optimization function which is just a translation of the Cost function number 1, cost of customer dissatisfaction.<br/>
<br/>
The following constraints are speaking for themselves. <br/>
<br/>
###Final Solution:<br/>
With each of the problems tackled, we can put them all together to get our result. We start off with a Prediction of the demand in the following year and follow up with the supply station building and demand assignment. Pulp enables us to put the two different optimization problems together to find the total minimum of costs with all the constraints in mind. We’re certain our algorithm finds the optimal minima for a given demand prediction, hence the only wildcard here is the prediction for the next year. It’s a wildcard because we won’t be able to predict the actual demand with just the past data. Many factors like the ones mentioned at the start and many more alter the demand for a given point.<br/>
The rest of the code is used to extract the values, as well as transform and prepare the output.<br/>
<br/>
##Final thoughts<br/>
We were able to optimize charging station placement and (hopefully) correctly predict the demand for the upcoming years. Still there are some parts, which we wanted to mention at the end. <br/>
First, we think that the demand satisfied by a slow charger and a fast charger shouldn’t be the only metric in terms of customer satisfaction. It’s also important how much time the customer spends at the station. If we build 3 Fast Charger which satisfy a demand of 1200 kw, there’s a chance that customer will be unhappy, because they might need to wait since only 3 vehicles can charge simultaneously. With the current battery technology, the peak charging speed of 400 will only be reached between a specific percentage of battery capacity, so the time spent at charging station won’t (in general) be halve as long if we use a fast charger instead of a slow charger. For specific locations, multiple slow chargers instead of a couple fast charger could increase customer satisfaction. To help the placement, one would need to track time spent charging, time spent waiting for the charger, and time spent finding different charging locations. On the other hand, if we build a slow charger at a parking station, just because the demand is not enough to build a fast charger, we gain more costs of infrastructure in the future. So essentially, it can be an investment to place a fast charger in locations, where the demand rises slowly, so we don’t need to build one slow charger every time the demand has risen above a certain threshold. An opinion here: even with a rising cost factor of 1.5 for building a fast charger, it would still be the best solution to build, since charging your car faster will increase the societies view on elective vehicles which results in more people deciding to join the EV-owner club and in conclusion, increases ROI due to more people using the chargers.(Tesla went really well with that tactic)<br/>
The history data could be optimized by including more EV-Focused data. Like numbers of EV bought, numbers of people living at the location, numbers of EV’s sold etc. This would enable one to predict demand more accurately. <br/>
<br/>
That’s all from us.<br/>
Cheers and take care<br/>
Alexander Heelemann and Sebastian Bott<br/>
<br/>
Python Libraries Used:<br/>
    Pandas<br/>
    Numpy<br/>
    Scikit Learn<br/>
    Pulp<br/>
    Matplotlib<br/>

