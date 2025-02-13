Homework 2
================

``` r
knitr::opts_chunk$set(
        echo = TRUE,
        warning = FALSE,
  fig.width = 6,
  fig.asp = .6,
  out.width = "90%"
)

theme_set(theme_minimal() +theme(legend.position = "bottom"))

options(
  ggplot2.continuous.color = "viridis",
  ggplot2.continuous.fill = "viridis"
)

scale_color_discrete = scale_colour_viridis_d
scale_fill_discrete = scale_fill_viridis_d
```

``` r
SIR = function (t, state, parameters) {
  with(as.list(c(state, parameters)), {
    dS = -beta * S * (I/N); #rate of change of susceptible population
    dI = beta * S * (I/N) - gamma * I; #rate of change of infectious population
    
    dcumI = beta * S * (I/N) #computing the cumulative number of infections
    
    list(c(dS, dI, dcumI)) #returning the rate of change
  })
}
```

# Part 1: Simulate using the SIR model and check basic model outputs

Read the SIR function and code provided in the script. Then run it with
the following conditions and parameters. N = 1e5 I0 = 10 S0 = N-I0 beta
= 0.5 gamma = 0.3 times = seq(1, 100, by = 1)

``` r
N = 1e5 
I0 = 10 
S0 = N - I0 
state = c(S = S0, I = I0, cumI = I0) 
parameters = c(beta = 0.5, gamma = 0.3) 
times = seq(1, 100, by = 1) 
```

``` r
sim = ode (y = state, times = times, func = SIR, parms = parameters)
```

## Question 1

Based on the simulation, how many people are susceptible on Day 50? How
many people are susceptible at the end of simulation?

Based on the simulation, 45,209 people are susceptible on Day 50 and
32,431 are susceptible at the end of the simulation.

## Question 2

Based on the simulation, how many people are infectious on Day 50? How
many people are infectious at the end of simulation?

Based on the simulation, 7,164 people are infectious on Day 50 and 11
are infectious at the end of the simulation.

## Question 3

Based on the simulation, how many people have been infected on Day 50?
How many people have been infected by the end of simulation?

Based on the simulation, 54,790 people have been infected by Day 50 and
67,568 have been infected by the end of the simulation.

## Question 4

Plot the outputs I and S for each day during the simulated time period.

``` r
susceptible = ggplot(sim, aes(x = time, y = S, color = time)) +
  geom_point(alpha = 0.6) +
  geom_line() +
  labs(title = "Number of People Susceptible During the Simulated Time Period",
       x  = "Day", 
       y = "Number of People")

infectious = ggplot(sim, aes(x = time, y = I, color = time)) +
  geom_point(alpha = 0.6) +
  geom_line() +
  labs(titlw = "Number of People Infectious During the Simulated Time Period", 
       x = "Day",
       y = "Number of People")
```

``` r
susceptible + infectious
```

<img src="Homework-2_files/figure-gfm/unnamed-chunk-6-1.png" width="90%" />

# Part 2: Simple analyses of model outputs

Convert the simulated numbers (#S, \#I, \#cumI etc.) to fraction
relative to the population size\`

``` r
percent_sim = ode(y = state, times = times, func = SIR, parms = parameters)
percent_sim_df = as.data.frame(percent_sim)

percent_sim_df$S_frac = percent_sim_df$S / N
percent_sim_df$I_frac = percent_sim_df$I / N
percent_sim_df$cumI_frac = percent_sim_df$cumI / N
```

## Question 5

Based on the simulation, what is the population susceptibility (i.e. %S)
at the end of simulation? What percentage of the population are
infectious at the end of simulation? What percentage of the population
have been infected by the end of simulation?

Based on the simulation, the population susceptibility at the end of
simulation is 0.3243105 or 32.43%

Based on the simulation, the percentage of the population infectious at
the end of simulation is 0.0001171663 or 0.012%.

Based on the simulation, the percentage of the population infected by
the end of simulation is 0.6756895075 or 67.57%

## Question 6

Plot the outputs %I and %S for each day during the simulated time
period.

``` r
percent_susceptible = ggplot(percent_sim_df, aes(x = time, y = S_frac, color = time)) +
  geom_point(alpha = 0.6) +
  geom_line() +
  labs(title = "Fraction of Susceptible Population", 
       x = "Day",
       y = "Number of People")

percent_infectious = ggplot(percent_sim_df, aes(x = time, y = I_frac, color = time)) +
  geom_point(alpha = 0.6) +
  geom_line() +
  labs(title = "Fraction of Infected Population", 
       x = "Day",
       y = "Number of People")
```

``` r
percent_susceptible + percent_infectious
```

<img src="Homework-2_files/figure-gfm/unnamed-chunk-9-1.png" width="90%" />

# Part 3: Simple analyses of model outputs

Aggregate the simulated daily outputs to weekly outputs. See model code
in the script.

``` r
sim_df = as.data.frame(sim)
sim_df$week = ceiling(sim_df$time / 7)

weekly_data = sim_df |> 
  group_by(week) |> 
  summarize(
    S_weekly = mean(S),
    I_weekly = mean(I),
    cumI_weekly = mean(cumI)
  )
```

## Question 7

How many people are infected on Week 7?

There are 8748 people infected on Week 7.

## Question 8

Plot number of Susceptibles mid-week over time.

``` r
sim_df$midweek = ceiling(sim_df$time / 7) * (7 - 3)

midweek_data = sim_df |> 
  filter(time %in% midweek)
```

``` r
ggplot(midweek_data, aes(x = time, y = S)) +
  geom_point(alpha = 0.6) +
  geom_line() +
  labs(title = "Susceptible Mid-Week Over Time",
       x = "Week", 
       y = "Number of People Susceptible")
```

<img src="Homework-2_files/figure-gfm/unnamed-chunk-12-1.png" width="90%" />

## Question 9

Plot number of Infectious mid-week over time.

``` r
ggplot(midweek_data, aes(x = time, y = I)) +
  geom_point(alpha = 0.6) +
  geom_line() +
  labs(title = "Infectious Mid-Week Over Time",
       x = "Week", 
       y = "Number of People Infectious")
```

<img src="Homework-2_files/figure-gfm/unnamed-chunk-13-1.png" width="90%" />

## Question 10

Plot number of New Infectious each week over time.

``` r
sim_df$new_infectious = c(NA, diff(sim_df$I))
```

``` r
weekly_new_infectious = sim_df |> 
  group_by(week) |> 
  summarize(
    new_infectious_week = sum(new_infectious, na.rm = TRUE)
  )
```

``` r
ggplot(weekly_new_infectious, aes(x = week, y = new_infectious_week)) +
  geom_point(alpha = 0.6) +
  geom_line() +
  labs(title = "New Infectious Each Week Over Time",
       x = "Week", 
       y = "Number of People Newly Infectious")
```

<img src="Homework-2_files/figure-gfm/unnamed-chunk-16-1.png" width="90%" />
