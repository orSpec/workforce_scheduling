# Workforce scheduling
A special case of the workforce scheduling problem is modelled and solved using Python. The following packages are used:

* [Python-MIP](https://www.python-mip.com/) for modelling the problem and solving it via the integrated [COIN-OR CBC](https://github.com/coin-or/Cbc) solver
* [pandas](https://pandas.pydata.org/) for data input and postprocessing the solution
* [plotly](https://plotly.com/python/) for visualization of the solution

## Problem description
A shop is open from Monday to Saturday. The opening hours are the same for each day. For each 30 minute slot during the opening hours there exists a demand for a specific number of employees. 

There exist the following sets:

<img src="https://user-images.githubusercontent.com/59450716/78698962-64ba1180-7903-11ea-8062-cdf95077df42.png" width="500">


In the model there are several parameters:

<img src="https://user-images.githubusercontent.com/59450716/78707005-50c8dc80-7910-11ea-94bd-215db877094b.png" width="550">


An employee can have a special qualification (e.g. management experience, safety training, ...).

<img src="https://user-images.githubusercontent.com/59450716/78701827-f461bf00-7907-11ea-8f32-7bd999497fc4.png" width="450">
 
 The demand for employees with this qualification for a slot with demand is given by λ.
 For each employee there can be requirements for the minimum and maximum working hours he/she has to work per week. Additionaly there is a global minimum for the length of a shift: If an employee works on a day the shift has to be equal or longer than δ. For each employee and day the length of the shift can't exceed a value of Ω hours.
The maximum working hours per week for an employee can be exceeded if Θ is greater than zero. In this case the employee does overtime. In contrast if Φ is greater than zero an employee can work less than the agreed minimum working hours per week.
The parameter γ denotes the availablity in hours of an employee on a given day. If set to 0 the employee is not available (e.g. due to sickness, vacation, etc.).

The problem needs these decision variables:
<img src="https://user-images.githubusercontent.com/59450716/78709527-16613e80-7914-11ea-9393-1a654746f5c4.png" width="650">


It is clear that at most one of the z variables per employee and day can be one. If the employee works on a day his/her shift can only start once.

The mathematical formulation of the problem is:

<img src="https://user-images.githubusercontent.com/59450716/78781764-1d825e00-79a1-11ea-9a57-9f17d0262bf0.png" width="700">

The objective function aims to minimize the sum of slots which have an employee assigned to. Constraint (1) ensures that the demand for each slot and day is met or exceeded. With Constraint (2) it is guaranteed that as mentioned above an employee's shift can only start once per day. This is the case if he/she works at all on this day. Constraint (3) connects the z with the x variables: If an employee works in slots s and not in slot s-1, then the start of the shift has to be in slot s. 

Constraints (4) - (9) are optional. By modifying the appropriate parameters in the input data which is an Excel file the user can choose to incorporate the constraints to the model or not.

Constraint (4) ensures that in each slot at least λ employees with special qualification are present. An employee can only work a maximum of Ω hours per day, see Constraint (5). The right hand side of the inequality is doubled because Ω is in hours and the slots are 30 min intervals. Constraint (6) ensures that if an employee works on a day the shift is at least δ hours long. The weekly working hours are taken into account by constraints (7) and (8): (7) makes sure that an employee can only exceed his maximum working hours per week by Θ hours (overtime). Notice that Θ can be 0. Similarly an employee has to work at least the agreed minimum working hours per week. It is possible to make minus hours in the amount of Φ hours (can be 0 too.)
Constraint (9) ensures that an employee can only work on a given day if he is available. Then the length of the shift can't exceed his availability in hours.

Finally, constraints (10) and (11) define the domain of the decision variables.
## Input data
Inside the excel file /data/InputData.xlsx all necessary information can be put into the following sheets:
* Employees:
	* the names of the employees
	* whether an employee has special qualification (1) or not (0)
	* their availability on a specific day, later referred to as 
		*max_employee_WorkingTime_per_Day*	
	* their minimum working hours per week (*min_hours_per_week*)
	* their maximum working hours per week (*max_hours_per_week*)
* Demand: the demand in personell for a 30 min time slot on a specific day
* Days: the days to plan and the corresponding dates
* Parameters (set *yes* if the constraint corresponding to the parameter is to be included in the model, else set *no*)
	* *min_workingTime_per_Day* corresponds to δ
	* *max_workingTime_per_Day* corresponds to Ω
	* *minusHours_per_Week* corresponds to Φ
	* *overtime_per_Week* corresponds to Θ
	* *demand_specialQualification_per_Slot* corresponds to λ
	
	For the last three parameters *max_hours_per_week*, *min_hours_per_week* and 		*max_employee_WorkingTime_per_Day* the corresponding values from the sheet *Employees* are taken as values. 
* Optimization_Parameters
	* *timeInSeconds*: maximum runtime in seconds (if left empty, there is no limit)
	* *mipGap*: maximum percentage deviation from the best node remaining in the branch and bound tree (if empty, default value of 1e-4 is used)

## Solution
If a feasible solution was found, one can generate the resulting schedule as a pandas DataFrame:
```python
plan = generateSchedule(x, z, nr_employees, nr_days, nr_slots, list_employees, days, slots, input_days)
``` 
The generated scheduled can be visualized using a plotly Gantt Chart:
```python
gantt = getGantt(plan)
```
Finally, with
```python
pio.renderers.default='browser'
gantt.show()
```
the Gantt chart is shown in the default browser.

One can even deploy the chart in a [website](https://orspec.github.io/workforce_scheduling/). Zoomed in on a single day it looks like this:

<a href="https://orspec.github.io/workforce_scheduling/">
<img src="https://user-images.githubusercontent.com/59450716/78926264-16407a80-7a9d-11ea-829d-fdaf4855b824.png" width="800">
</a>

You can see immediately that Mary and Jaden don't work on April 13th.



