Peer Review Simulation for the Getting and Cleaning Data Project
========================================================

This is a bit off topic, but is data science.

I have not written any simulations before, but I thought the peer review process would be a good use case for a simulation.  The peer review simulation
takes in a range of possible responses to the evaluation questions and then simulates the peer review process determining a mean
score from the various possible combinations of responses. The combination of responses is controlled by an input parameter that defines the distribution of anticipated responses to the evaluation questions.

I have simplified the simulation process by making each response
independent, i.e. you might get a 0 for a github, but a 12 for having a CodeBook.  Also, each simulated project
is reviewed only 4 times.  

## Peer Review Simulation

The input to the simulation is a list which defines how many responses out of a 100 would be assigned to each possible grading option per review question.  I have used 100 for two reason:
* It allows me to "think" in percentages, i.e. 70% of the people would give me a 12 on this question 
* It also provides enough responses that sampling makes more sense. 
It does not have to be 100, you can use any number you want, just know that it builds a response vector that is then 
sampled for the simulation. 

The simulation.parm has the following structure:
```
        simulation.parm <- list(Q1=c(50,49,1),  
                                Q2=c(50,49,1),  
                                Q3=c(50,49,1),  
                                Q4=c(50,50),  
                                Q5=c(50,50))  
                               
        Q1=c(# of 12 responses, # of 6 responses, # of 0 responses)  
        Q2=c(# of 12 responses, # of 6 responses, # of 0 responses)  
        Q3=c(# of 12 responses, # of 6 responses, # of 0 responses)  
        Q4=c(# of 4 responses, # of 0 responses)  
        Q5=c(# of 1 responses, # of 0 responses)  
```

### Simulation Parameters

To test the sensitivity of the simulation I created 3 distinct simulations:
- Optimistic, most people would grade on the easy side
- Even Money, it is a coin flip on which grade they will give
- Pessimistic, most people are really hard graders


```r
### 
rm(list = ls())
### 

### The following paramaters are used to setup a response set to sample from.
### The following are used to

### An Optimistic view, 75% of the graders will give a 'high' grade and 24% a
### lower grade, and 1% a 0
optomistic.parm <- list(Q1 = c(60, 39, 1), Q2 = c(75, 24, 1), Q3 = c(75, 24, 
    1), Q4 = c(75, 25), Q5 = c(75, 25))

### An even money view, 50% of the graders will give a 'high' grade and 49% a
### lower grade, and 1% a 0
even.money.parm <- list(Q1 = c(50, 49, 1), Q2 = c(50, 49, 1), Q3 = c(40, 59, 
    1), Q4 = c(50, 50), Q5 = c(50, 50))

### A pesimitic view, 25% of the graders will give a 'high' grade and 74% a
### lower grade, and 1% a 0
pessimistic.parm <- list(Q1 = c(60, 39, 1), Q2 = c(25, 74, 1), Q3 = c(25, 74, 
    1), Q4 = c(25, 75), Q5 = c(25, 75))

```



### The Simulation Function:

The other inputs to the simulation are, number.of.samples and number.of.simulations.  These control the size of data used in the simulation 
and the number of simulations that will be run.  They default to 10000 and 1000, receptively.  The number.of.samples needs to be divisible by 4 
so 4 peers can be pulled for each review.   With number.of.samples = 10000, there will be 2500 peer reviews with 4 graders per review, with 1000 simulations, there are 2.5MM simulated evaluations (probably overkill.)


```r
## PEER REVIEW SIMULATION
peer.review.simulation <- function(sim.parm, number.of.samples = 10000, number.of.simulations = 1000) {
    final <- list()
    
    if ((number.of.samples%%4) != 0) 
        stop("number.of.samples has to be evenly divisable by 4")
    
    Q1.resp.set <- c(rep(12, sim.parm[["Q1"]][1]), rep(6, sim.parm[["Q1"]][2]), 
        rep(0, sim.parm[["Q1"]][3]))
    Q2.resp.set <- c(rep(12, sim.parm[["Q2"]][1]), rep(6, sim.parm[["Q2"]][2]), 
        rep(0, sim.parm[["Q2"]][3]))
    Q3.resp.set <- c(rep(12, sim.parm[["Q3"]][1]), rep(6, sim.parm[["Q3"]][2]), 
        rep(0, sim.parm[["Q3"]][3]))
    Q4.resp.set <- c(rep(4, sim.parm[["Q4"]][1]), rep(0, sim.parm[["Q4"]][2]))
    Q5.resp.set <- c(rep(1, sim.parm[["Q5"]][1]), rep(0, sim.parm[["Q5"]][2]))
    
    ## RUN SIMULATIONS
    for (s in 1:number.of.simulations) {
        
        
        ### SAMPLE number.of.samples FROM THE POSSIBLE RESPONSES TO THE QUESTIONS
        sample.12.1 <- sample(Q1.resp.set, number.of.samples, replace = TRUE)
        sample.12.2 <- sample(Q2.resp.set, number.of.samples, replace = TRUE)
        sample.12.3 <- sample(Q3.resp.set, number.of.samples, replace = TRUE)
        sample.4.4 <- sample(Q4.resp.set, number.of.samples, replace = TRUE)
        sample.1.5 <- sample(Q5.resp.set, number.of.samples, replace = TRUE)
        
        ### RUN SIMULATED PEER ASSESSMENTS
        iteration <- 1
        scores <- rep(0, number.of.samples/4)
        
        ## SKIP 4 ACROSS THE SAMPLES TO DRAW 4 PEERS
        for (x in seq(4, number.of.samples, 4)) {
            
            ## TOTAL SCORE IS THE SUM OF ALL GRADES, FROM ALL THE PEERS (SIMULATED THAT
            ## IS)
            total.score <- (sample.12.1[x] + sample.12.2[x] + sample.12.3[x] + 
                sample.4.4[x]) + (sample.12.1[x - 1] + sample.12.2[x - 1] + 
                sample.12.3[x - 1] + sample.4.4[x - 1]) + (sample.12.1[x - 2] + 
                sample.12.2[x - 2] + sample.12.3[x - 2] + sample.4.4[x - 2]) + 
                (sample.12.1[x - 3] + sample.12.2[x - 3] + sample.12.3[x - 3] + 
                  sample.4.4[x - 3])
            
            ## DIVIDE BY 4 TO GET THE FINAL SCORE.
            scores[iteration] <- total.score/4
            
            ## INCREMENT THE ITERATION COUNTER
            iteration <- iteration + 1
        }
        
        ## SAVE THE QUANTILES, MEAN AND ALL THE RAW SCORES FOR EACH ASSESSMENT
        final[[s]] <- list(summary = c(quantile(scores, prob = c(0, 0.25, 0.5, 
            0.75, 0.9, 0.95, 1)), average.score = mean(scores)), all.scores = scores)
        
    }
    
    ## RETURN A LIST CONTAINING THE QUANTILE AND MEAN, AND ALL THE RAW DATA.
    return(final)
    
}  # END OF FUNCTION
```


## Run the Simulations

The following runs the three separate simulations


```r

# set.seed(1234)
set.seed(5574)

## AN OPTIMISTIC SIMULATION
optimistic.FINAL <- peer.review.simulation(optomistic.parm)

## EVEN MONEY SIMULTATION
even.money.FINAL <- peer.review.simulation(even.money.parm)

## PESSIMITIC SIMULATON
pessimistic.FINAL <- peer.review.simulation(pessimistic.parm)
```



## Processing the Results

The output of the simulation that contains the summary stats for each simulation and the raw data 
for each simulation.  


```r
suppressMessages(require(Hmisc, quietly = TRUE))

## CONVERT THE FINAL RESULTS FROM A LIST TO 2 SEPARATE DATAFRAMES

O.FINAL.summary <- data.frame(do.call("rbind", lapply(optimistic.FINAL, function(x) return(x$summary))))
O.FINAL.all.scores <- data.frame(do.call("rbind", lapply(optimistic.FINAL, function(x) return(x$all.scores))))

E.FINAL.summary <- data.frame(do.call("rbind", lapply(even.money.FINAL, function(x) return(x$summary))))
E.FINAL.all.scores <- data.frame(do.call("rbind", lapply(even.money.FINAL, function(x) return(x$all.scores))))

P.FINAL.summary <- data.frame(do.call("rbind", lapply(pessimistic.FINAL, function(x) return(x$summary))))
P.FINAL.all.scores <- data.frame(do.call("rbind", lapply(pessimistic.FINAL, 
    function(x) return(x$all.scores))))


## DESCRIBE THE RESULTS OF A SIMULATION THAT RAN 1000 SIMULATIONS OF 2500
## SIMULATED PEER ASSESSMENTS (WOW THATS ALOT OF SAMPLES)
O.mean <- as.numeric(describe(O.FINAL.summary)$average.score$counts["Mean"])
E.mean <- as.numeric(describe(E.FINAL.summary)$average.score$counts["Mean"])
P.mean <- as.numeric(describe(P.FINAL.summary)$average.score$counts["Mean"])
```


## Simulation Variability

The following plot shows the variability in the simulation, i.e. the range of scores that are produced for each simulation.

I am plotting the density function for each simulation raw scores for each peer review.  If the defaults for the simulation are selected, 10000 samples and 1000 simulations there will be 2.5MM peer reviews.  


```r

number.of.peer.reviews <- ncol(P.FINAL.all.scores)
number.of.sim <- nrow(P.FINAL.all.scores)

plot(density(do.call("rbind", (O.FINAL.all.scores[1, 1:number.of.peer.reviews]))), 
    ylim = c(0, 0.17), xlim = c(10, 41), lwd = 4, col = "#7FC97F", main = "Variablity in the Simulation")

for (x in 2:number.of.sim) {
    lines(density(do.call("rbind", ((O.FINAL.all.scores[x, 1:number.of.peer.reviews])))), 
        col = "#7FC97F0f")
}

lines(density(do.call("rbind", ((E.FINAL.all.scores[1, 1:number.of.peer.reviews])))), 
    col = "#BEAED4", lwd = 4)

for (x in 2:number.of.sim) {
    lines(density(do.call("rbind", ((E.FINAL.all.scores[x, 1:number.of.peer.reviews])))), 
        col = "#BEAED40f")
}

lines(density(do.call("rbind", ((P.FINAL.all.scores[1, 1:number.of.peer.reviews])))), 
    col = "#FDC086", lwd = 4)

for (x in 2:number.of.sim) {
    lines(density(do.call("rbind", ((P.FINAL.all.scores[x, 1:number.of.peer.reviews])))), 
        col = "#FDC0860f")
}

legend("topleft", legend = c(paste("Optimistic Mean =", O.mean), paste("Even Money Mean =", 
    E.mean), paste("Pessimistic Mean =", P.mean)), col = c("#7FC97F", "#BEAED4", 
    "#FDC086"), lwd = 4)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 



## So what is my Grade

So what do I think my grade will be?

Q1: I am pretty confident in my tidyData file and am optimistic that most folks either understand what is included
in the file or are scoring on the easy side.

Q2: I have a github with a run_analysis.R, README.md and CodeBook.md, and pretty sure most folks will understand my code...

Q3: The CodeBook is pretty subjective as there was limited guidance on how to grade this, so while I am pretty sure I 
covered the important parts, I think there is the possibility some will find fault.

Q4: The README file, like the CodeBook this is subjective, and in my case I did not document one of the files in the README.md and anticipate 
some will mark me down for the omission.

Q5: I think it will be apparent to everyone that I completed the assignment.


```
        MY.Peer.Review.parm <- list(Q1=c(80,19,1),  
                                Q2=c(90,9,1),  
                                Q3=c(75,24,1),  
                                Q4=c(65,35),  
                                Q5=c(99,1))  
```


```r

MY.Peer.Review.parm <- list(Q1 = c(80, 19, 1), Q2 = c(90, 9, 1), Q3 = c(75, 
    24, 1), Q4 = c(65, 35), Q5 = c(99, 1))

## SIMULATION OF MY PEER REVIEW:
My.FINAL <- peer.review.simulation(MY.Peer.Review.parm)

## CONVERT THE FINAL RESULTS FROM A LIST TO A DATAFRAME
My.FINAL.summary <- data.frame(do.call("rbind", lapply(My.FINAL, function(x) return(x$summary))))
My.FINAL.all.scores <- data.frame(do.call("rbind", lapply(My.FINAL, function(x) return(x$all.scores))))

My.mean <- as.numeric(describe(My.FINAL.summary)$average.score$counts["Mean"])

### PLOT PARMS
number.of.peer.reviews <- ncol(P.FINAL.all.scores)
number.of.sim <- nrow(P.FINAL.all.scores)


## PLOT THE DENSITY OF THE SIMULATION RUNS
plot(density(do.call("rbind", (My.FINAL.all.scores[1]))), ylim = c(0, 0.19), 
    xlim = c(10, 41), lwd = 4, col = "#7FC97F", main = "My Grade Simulation Results")

for (x in 2:number.of.sim) {
    lines(density(do.call("rbind", ((My.FINAL.all.scores[x, 1:number.of.peer.reviews])))), 
        col = "#7FC97F0f")
}

legend("topleft", legend = c(paste("My Grade Mean =", My.mean)), col = c("#7FC97F", 
    "#BEAED4", "#FDC086"), lwd = 4)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 




## Observations and Conclusion

So I am looking at 35, which is about what I was anticipating without running the simulation.

I have not written a simulation like this before, so I'd be interested in hearing from anyone
that has and would like to critique the work.  

Give it a try and see your if your estimate holds up.

Stephen...
