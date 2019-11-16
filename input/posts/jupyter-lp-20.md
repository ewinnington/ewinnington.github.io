Title: Applications in LP and MILP with C# and OR-Tools inside Jupyter
Published: 16/11/2019 23:30
Tags: [CSharp, LinearProgramming, Mixed Integer linear programming, Google OR-Tools, Dotnet try, Jupyter notebook] 
---

Thanks to the integration of C# into [Jupyter notebooks](https://jupyter.org/) with the [kernel from Donet try](https://github.com/dotnet/try) and support from the [MyBinder.org](https://mybinder.org/) hosting, it's easy to share with you runnable workbooks to illustrate how to use the [Google OR-Tools](https://developers.google.com/optimization) to solve the [linear](https://en.wikipedia.org/wiki/Linear_programming) (LP) and [mixed-integer linear problems](https://en.wikipedia.org/wiki/Integer_programming) (MILP) . 

# Applications of linear programming 

## LP Problem : Wire production 

A plant makes aluminium and copper wires. Each Kg of aluminium wire requires 10 kWh of electricity and $\frac{1}{2}$ hour of labour. Each Kg of Copper wire requires 4 kWh of electricity and $1$ hour of labour. Electricity is limited to 450 kWh/day, labour is limited to 42.5 hours/day at a cost of 11 € an hour, Electricity cost is 20 € / MWh, Aluminium cost is 1.8 €/Kg, Copper cost is 5.4 €/Kg. Total weight delivered to the plant daily is limited to 56 Kg. Aluminium wire sales price is 45 €/Kg, Copper wire sales price is 50 €/Kg. 

What should be produced to maximise profit and what is the maximum profit? 

[Launch the binder](https://mybinder.org/v2/gh/ewinnington/noteb/master?filepath=Lp_WireProduction.ipynb)  [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ewinnington/noteb/master?filepath=Lp_WireProduction.ipynb)

```CSharp
Solver solver = Solver.CreateSolver("LinearProgramming", "CLP_LINEAR_PROGRAMMING");

//Setting up the variables and constants of the problem 
Variable Al = solver.MakeNumVar(0.0, double.PositiveInfinity, "Al");
Variable Cu = solver.MakeNumVar(0.0, double.PositiveInfinity, "Cu");
double El_Price = 20.0; double El_Max = 450; 
double La_Price = 11.0; double La_Max = 42.5; 
double Weight_Max = 56.0; 
double Al_Purchase = 1.8 ; double Al_Sale = 45.0;
double Cu_Purchase = 5.4; double Cu_Sale = 50.0;

// Maximize revenue 
Objective objective = solver.Objective();
objective.SetCoefficient(Al, Al_Sale-Al_Purchase-(La_Price * 1/2.0)-(El_Price * 10/1000.0));
objective.SetCoefficient(Cu, Cu_Sale-Cu_Purchase-(La_Price * 1)-(El_Price * 4/1000.0));
objective.SetMaximization();

// Electricity usage limit
Constraint c0 = solver.MakeConstraint(0, El_Max);
c0.SetCoefficient(Al, 10);
c0.SetCoefficient(Cu, 4);

// Labour usage limit
Constraint c1 = solver.MakeConstraint(0, La_Max);
c1.SetCoefficient(Al, 1/2.0);
c1.SetCoefficient(Cu, 1);

// Weight limit
Constraint c2 = solver.MakeConstraint(0, Weight_Max);
c2.SetCoefficient(Al, 1);
c2.SetCoefficient(Cu, 1);

SolveAndPrint(solver);
```

## MILP Problem : Knapsack 

The [knapsack problem](https://en.wikipedia.org/wiki/Knapsack_problem) defines a bag that was a maximal weight of $W$, we can take items from a set of items each with a weight of $w_i$ and a value of $v_i$. Typically, the problem is defined with an $x_i \in \{0,1\}$ variable set to either 0 or 1 knapsack where each item is either taken or not. 

Here we are going to allow fractional parts of the items to be taken so that we can solve it as a linear problem (also known as a linear relaxation), allowing $x_i \in [0,1]$. 

[Launch the binder](https://mybinder.org/v2/gh/ewinnington/noteb/master?filepath=Knapsack_Lp_Milp.ipynb)  [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/ewinnington/noteb/master?filepath=Knapsack_Lp_Milp.ipynb)

```CSharp
//Array of items, weights and totals
int nItems = 10; 
double maxWeight = 220; 
double[] weights = {31, 27, 12, 39,  2, 69, 66, 29, 45, 58};
double[] values =  {24, 27, 26, 15, 19, 33, 30, 28, 65, 42};

Solver milp_solver = Solver.CreateSolver("MILP", "CBC_MIXED_INTEGER_PROGRAMMING");

Variable[] Items = milp_solver.MakeBoolVarArray(10, "Items");

// Maximize revenue 
Objective objective = milp_solver.Objective();
for(int i = 0; i < nItems; i++) objective.SetCoefficient(Items[i], values[i]);
objective.SetMaximization();

// Weight limit
Constraint c0 = milp_solver.MakeConstraint(0, maxWeight);
for(int i = 0; i < nItems; i++) c0.SetCoefficient(Items[i], weights[i]);

SolveAndPrint(milp_solver, nItems, weights);
```

There are many ways of solving the knapsack problem, using LP and MILP solvers as seen here or using Dynamic Programming. The Google OR-Tools have a specific solver for multi-dimensional knapsack problems, including one which uses Dynamic Programming.