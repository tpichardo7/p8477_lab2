SIR Simulations
================

# Ordinary Differential Equations (ODE)

**General Routine**:  
1. Code the ODE/model (or function here)  
2. Specify the initial conditions/parameters  
3. Call the “ode” function to solve the ODE/model and generate the
simulation  
4. Check/analyze model outputs

## Step 1: Code the ODE model (or function here)

``` r
myfunc = function(x, y, parms) {
  dy = x^2
  list(dy)
}
```

## Step 2: Specify initial conditions/parameters

``` r
xs = seq(0, 10, by = 0.2) #the 'time' step to integrate upon
state = c (y = 0);
```

## Step 3: Call the ode function to generate trhe simulation

``` r
out = ode (y = state, times = xs, func = myfunc, parms = NULL)
```
