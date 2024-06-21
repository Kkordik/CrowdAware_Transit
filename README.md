# CrowdAware_Transit

CrowdAware Transit is an UPPER Hackathon project, presented with [this .pdf](https://github.com/Kkordik/CrowdAware_Transit/blob/main/Upper_Presentation.pdf). The idea of the project is to share expected fullness of the bus next to arrival time. It would allow people that require a seat, big families and other passengers to plan their trips with comfort.

Here are examples of current EMT app UI and improved one:

![Current](https://i.ibb.co/80VkwF6/i-Phone-14-Problem-1.png) 

vs  

![Improved](https://i.ibb.co/fvLSrL6/i-Phone-14-Solution-1.png)


## Global steps explained:

What we want is to estimate how many passengers will exit and enter the bus at specific time on stops between bus and passenger's stop (2nd and 3rd bus stops):

![Bus Example](https://i.ibb.co/FgQ6TK7/UPPER-hack-Page-3-drawio-2.png)


Here is how it can be done (consider that data has info only about enters, so it may be overcomplicated if we have collected data exits as well):

![Global Scheme](https://i.ibb.co/h7W58mh/UPPER-hack-Page-2-drawio.png)

<br/><br/>

### Predict destination and exit time for each trip

**NOTE**: This step is done to historical data to generete statistical information about exits.

This is done by simple algorithm, that founds round trips (trips with returns) and considers station(bus stop) where passenger entered to return as station where passenger exited while going there.
In the [notebook](https://github.com/Kkordik/CrowdAware_Transit/blob/main/UPPER_Hackathon.ipynb) that I created at hackathon, this is done like that:

1. For each bus line create one unit vector, that represents average direction of the line. (where unit vector*(-1) is the same line but opposite direction). This is done by sum of vectors between stops of the line and division by vector length.
2. Taking this unit vector of trip _n_ and _n+1_ of passenger x we calculate angle between these vectors.
3. If angle is bigger than pi/3, it means that it is most likely returning trip.
4. If time between two trips is less than 1h and angle < pi/3, it is most likely just line changing and still trip in one direction (not returning).
5. In case if passenger returned on other line (for example equivalent line) we search closest stations: for the station where passenger entered while going there we search closest station at the line on which passenger returned and reversed action. 
6. And then having these pairs we can can predict exit time of each trip.
7. boom, we have complete (or almost complete) historical data about enter and exit time and location.

<br/><br/>

### Forecasting

This is simple, here we forecast for each bus stop and each bus line how many passengers will enter or exit in the future based on historical (+real-time, if we have it) data. 

We don't create any wheel by making such algorithm from 0, but rather we use foundation model for time-series forecasting. I've chosen lag-llama model as it lets us fine-tune it.

So the workflow would be:
1. Prepare the data, make features for the lag-lama model (for example nuber of passengers that entered in period of 20 min for one month), **NOTE**: of course we do computation for exits and enters separately
2. Just use the model and get results (in the [notebook](https://github.com/Kkordik/CrowdAware_Transit/blob/main/UPPER_Hackathon.ipynb) I had no time to finish this part, but it is easy you can try make it)

<br/><br/>

### How number of passengers in a bus is calculated

Well, we just get real-time data on how many passengers entered on each stop and minusing previously forecasted data on how mahy passengers would exit on each stop (and also we can edit it for example based on anomalies in real-time enters data) and the result is how many passengers are in the bus right now,
for estimating it for future stops (between bus and passenger) we do forecast for how many passengers will enter on each stop and minusing the same as earlier but for other stops and times.

**Boom** thats it, later other data like weather or some big events can be added to the process for better accuracy but for MVP it is enough.

Feel free to use my code and idea. Would appreceate if you refer me or took me on a job :)
