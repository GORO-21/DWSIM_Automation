This file will tell how the code is executed and how to set up an environment in Python for DWSIM.

•	Importing .NET and DWSIM libraries

import clr  Imports the clr module, which allows python to interact with .NET assemblies (DWSIM is built on .NET assemblies).

import System.IO  Imports “System.IO” module from .NET, which provides tools for working with files and directories.

import System  Provides basic functionality like string handling and environment variables.

import  pythoncom  this provides support for COM (component Object model) services, needed for DWSIM’s .NET components 

from System.IO import Directory, Path, File  imports specific classes from System.IO to handle directory operations, file paths, and file operations. 

From System import String, Environment imports string and Environment classes from system to work with strings and environment variables (eg- to get desktop folder path)

•	Initialize COM and Setting DWSIM Path

pythoncom.CoInitialize() This ensures that python can properly interact with DWSIM’s .NET-based components. 

dwSimPath = r“path where DWSIM is installed”  sets a variable dwSimPath to the path where DWSIM is installed on the user computer. 

clr.AddReference( dwSimPath + ”CapeOpen.dll”)  Handles Cape-Open standards for process modelling.

clr.AddReference( dwSimPath + ”DWSIM.Automation.dll”)  Provides automation tool for controlling DWSIM.

clr.AddReference( dwSimPath + ”DWSIM.Interfaces.dll”) Defines interfaces for DWSIM objects

clr.AddReference( dwSimPath + ”DWSIM.GlobalSettings.dll”) Manages global settings for DWSIM

clr.AddReference( dwSimPath + ”DWSIM.SharedClasses.dll”) Contains shared classes used across DWSIM

clr.AddReference( dwSimPath + ”DWSIM.Thermodynamics.dll”) Handles thermodynamic calculation 

clr.AddReference( dwSimPath + ”DWSIM.UnitOperations.dll”) Defines unit operation like distillation column

clr.AddReference( dwSimPath + ”DWSIM.Inspector.dll”) Provides tools for inspecting simulation objects.

clr.AddReference( dwSimPath + ”System.Buffers.dll”) A .NET library for  efficient memory management

from DWSIM.Interfaces.Enums.GraphicObjects import ObjectType Defines type of objects in flowsheet (streams,unit operation)

from DWSIM.Thermodynamics import Streams, PropertyPackages  Handle material streams and thermodynamic models.

from DWSIM.UnitOperations import UnitOperations  Manages unit operation like distillation column 

from DWSIM.Automation import Automation3  Provides automation capabilities for running simulations

from DWSIM.GlobalSettings import Settings  Manages global simulation settings

Directory.SetCurrentDirectory(dwSimPath) Sets current working directory to DWSIM installation folder, ensuring that DWSIM can find its dll files.

•	Loading the FLowsheet

interf = Automation3() Creates an instance or example of the Automation3 class, which is used to automate DWSIM tasks like loading and running flow sheets.

fileNameToLoad = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop),r”simulation flow sheet path” or just try writing the path of sim file  Constructs a file path to a DWSIM flowsheet file by combining the user desktop path with relative path to the file.

sim = interf.LoadFlowsheet(fileNameToLoad) This loads flowsheet into the variable called sim using LoadFlowsheet method in Automation3 class. The sim variable represents the entire simulation.

•	Accessing the Distillation column and Feed stream 
SC = sim.GetObject(“SC”) Retrieves the object named “SC”(distillation column) from the flowsheet and assign it to variable SC.

SC = SC.GetAsObject() Converts the SC object to its underlying type (DWSIM specific object) to access its properties and method.
**Method which comes with SC.GetAsObject, when we access DWSIM specific object**
print(SC.GetDisplayName())  Prints display name of distillation column
print(SC.get_Calculated())  Result in Boolean, if calculation for distillation column has been successfully performed
print(SC.m_lightkey) Prints name of light key component
print(SC.m_heavykey) prints name of heavy key component
print(SC.m_lightkeymolarfrac) prints light key molar fraction
print(SC.m_heavykeymolarfrac) prints heavy key molar fraction
print(SC.m_refluxratio) prints reflux ratio
print(SC.m_condenserpressure) prints condenser pressure
print(SC.m_boilerpressure) prints reboiler pressure
print(SC.m_Rmin) prints minimum reflux ratio
print(SC.m_Nmin) prints minimum number of stages
print(SC.m_N) prints actual number of stages
print(SC.ofs) prints optimal number of stages
print(SC.L_) prints liquid flow rate in stripping section 
print(SC.L) prints liquid flow rate in rectifying section
print(SC.V_) prints vapour flow rate in stripping section
print(SC.V) prints vapour flow rate in rectifying section
print(SC.m_Qc) prints condenser heat duty
print(SC.m_Qb) prints reboiler heat duty

** Automation of Distillation for a range of flow rates**
Feed = sim.GetObject(“Feed”) Retrieves the object named “Feed” (input stream to the column) from the flowsheet and assign it to the variable feed.

Feed = Feed.GetAsObject() Converts the Feed objects to its underlying type to access its properties
** Properties which can be accessed **
print(Feed.GetTemperature()) Prints temperature
print(Feed.GetPressure()) prints pressure
print(Feed.GetMassFlow()) prints mass flow rate of feed stream

•	Simulating with Varying Feed Mass Flow
from tabulate import tabulate imports the tabulate lib to display in a formatted table.

Feed_mass_flow = float(input(“Enter the initial mass flow rate:”)) prompts the user to input initial mass flow rate for the feed stream (kg/s) and converts string to float and stores value in variable Feed_mass_flow.

Settings.SolverMode = 0 sets DWSIM’s solver mode to default solver (0), ensuring standard calculation methods are used. This determines how DWSIM handles the calculation for solving the flow sheet in automation.

Results = [] Creates an empty list called result to store simulation results

** Code for getting, increasing mass flow rate by 1kg/s each time** 

# Iterate 10 times, increasing the mass flow rate by 1 kg/s each time

for i in range(10):
    # Set the new mass flow rate of the inlet stream
    Feed.SetMassFlow(Feed_mass_flow)
    
    # Calculate the flowsheet
    errors = interf.CalculateFlowsheet2(sim)
    
    # Get the new flash spec and pressure
    Mass_Flow_Feed = Feed.GetMassFlow()
    Refluxratio = SC.m_refluxratio
    Actual_Number_of_Stages = SC.m_N
    Optimal_Feed_Stage = SC.m_Nmin
    Stripping_Liquid = SC.L_
    Rectify_Liquid = SC.L
    Stripping_Vapor = SC.V_
    Rectify_Vapor = SC.V
    Condenser_Duty = SC.m_Qc
    Reboiler_Duty = SC.m_Qb

    # Append the results to the list
    results.append([Feed_mass_flow, Refluxratio, Actual_Number_of_Stages, Optimal_Feed_Stage, Stripping_Liquid, Stripping_Vapor, Rectify_Liquid, Rectify_Vapor, Condenser_Duty, Reboiler_Duty])

    # Increase the mass flow rate by 1 kg/s for the next iteration
    Feed_mass_flow += 1.0

# Define the headers for the table
headers = ["Feed mass flow", "Reflux Ratio", "Actual Number of Stages", "Optimal Feed Stage", "Stripping Liquid","Stripping Vapor", "Rectifying Liquid", "Rectifying Vapor", "Condenser Duty", "Reboiler Duty"]

# Print the results in a table format
print(tabulate(results, headers=headers, tablefmt="grid"))

**Note- here in results, in results.append at index[0], Feed mass flow is used in your code check for both “Feed mass flow” and Mass_Flow_Feed
If you want to  run values such that you want next 10 itteration for mass flow, in comparision to simulation you are running. Then go for, Mass_Flow_Feed.
And if you want to start code from random mass flow value, input the initial value and the code will go for next 10 iteration. 

•	Plotting Feed Mass flow results

import matplotlib.pyplot  as plt  imports matplotlib for creating plots

feed_mass_flow_rates = [row[0] for row in results]
reflux_ratios = [row[1] for row in results]
actual_stages = [row[2] for row in results]
optimal_feed_stages = [row[3] for row in results]
stripping_liquid = [row[4] for row in results]
stripping_vapor = [row[5] for row in results]
rectify_liquid = [row[6] for row in results]
rectify_vapor = [row[7] for row in results]
condenser_duty = [row[8] for row in results]
reboiler_duty = [row[9] for row in results] 

{**Note- when table is created, n columns so, row[n], represents the n index each row.
Example, in feed_mass_flow_rates = [row[0] for row in results], here index 0 that is column 1 constitutes to feed mass flow rates. 
reflux_ratios = [row[1] for row in results], here index 1 that is column 2 constitues to reflux ratio……..and so on}

fig, axs = plt.subplots(2, 2, figsize=(10, 8))                                    (1)

axs[0, 0].plot(feed_mass_flow_rates, stripping_liquid, label='Stripping Liquid')
axs[0, 0].set_xlabel('Feed mass flow rate')
axs[0, 0].set_ylabel('Stripping Liquid')
axs[0, 0].legend()
axs[0, 0].grid()                                        (2)

axs[0, 1].plot(feed_mass_flow_rates, rectify_liquid, label='Rectifying Liquid')
axs[0, 1].set_xlabel('Feed mass flow rate')
axs[0, 1].set_ylabel('Rectifying Liquid')
axs[0, 1].legend()
axs[0, 1].grid()

axs[1, 0].plot(feed_mass_flow_rates, stripping_vapor, label='Stripping Vapor')
axs[1, 0].set_xlabel('Feed mass flow rate')
axs[1, 0].set_ylabel('Stripping Vapor')
axs[1, 0].legend()
axs[1, 0].grid()

axs[1, 1].plot(feed_mass_flow_rates, rectify_vapor, label='Rectifying Vapor')
axs[1, 1].set_xlabel('Feed mass flow rate')
axs[1, 1].set_ylabel('Rectifying Vapor')
axs[1, 1].legend()
axs[1, 1].grid()

plt.tight_layout()
plt.show()

{Note: (1) Here, creates a grid of sub-plots 2x2 grid, with a figure size of 10x8 inches.}
{Note: (2) Here, plots stripping liquid vs feed mass flow rate in the top-left subplot, with labels, legend (explains what the graph line shows), and a grid}
{Note: fig, axs = plt.subplots(2, 2, figsize=(10, 8)) , here grid indicates that there will be 2 rows and 2 columns, that is each row can have 2 figure}
               # Plotting the condenser and reboiler duties
fig, axs = plt.subplots(1, 2, figsize=(10, 4))

axs[0].plot(feed_mass_flow_rates, condenser_duty, label='Condenser Duty')
axs[0].set_xlabel('Feed mass flow rate')
axs[0].set_ylabel('Condenser Duty')
axs[0].legend()
axs[0].grid()

axs[1].plot(feed_mass_flow_rates, reboiler_duty, label='Reboiler Duty')
axs[1].set_xlabel('Feed mass flow rate')
axs[1].set_ylabel('Reboiler Duty')
axs[1].legend()
axs[1].grid()

plt.tight_layout()
plt.show()

{Note: fig, axs = plt.subplots(1, 2, figsize=(10, 4)) , so here the grid will have 1 row and 2 column so each row can accumulate 2 figure. That is the reason why axs[0] and axs[1] is used}

•	Simulating with Varying Reflux Ratio
import matplotlib.pyplot as plt
	
# Set the initial mass flow rate
Feed_mass_flow = float(input("Enter the initial mass flow rate: "))

# Set the initial reflux ratio
Reflux_ratio = float(input("Enter the initial reflux ratio: "))

# Set the iterative step
iterative_step = float(input("Enter the iterative step: "))

# Set the number of loops
num_loops = int(input("Enter the number of loops: "))

{Note: Prompts user to input the initial data }

Settings.SolverMode = 0  sets solver to default solver again

# Create empty lists to store the results
reflux_ratios = []
actual_stages = []
stripping_liquid = []
stripping_vapor = []
rectifying_liquid = []
rectifying_vapor = []
condenser_duty = []
reboiler_duty = []

{Note: Creates empty list to store results for each parameter during the reflux ratio simulations}

# Iterate for the specified number of loops
for i in range(num_loops):
    # Set the new reflux ratio
    SC.m_refluxratio = Reflux_ratio
    
    # Calculate the flowsheet
    errors = interf.CalculateFlowsheet2(sim)
    
    # Get the new flash spec and pressure
    Mass_Flow_Feed = Feed.GetMassFlow()
    Refluxratio = SC.m_refluxratio
    Actual_Number_of_Stages = SC.m_N
    Optimal_Feed_Stage = SC.m_Nmin
    Stripping_Liquid = SC.L_
    Rectify_Liquid = SC.L
    Stripping_Vapor = SC.V_
    Rectify_Vapor = SC.V
    Condenser_Duty = SC.m_Qc
    Reboiler_Duty = SC.m_Qb

{Note: Retrieves the same set of results as in the feed mass flow simulation}

# Append the results to the lists
    reflux_ratios.append(Reflux_ratio)
    actual_stages.append(Actual_Number_of_Stages)
    stripping_liquid.append(Stripping_Liquid)
    stripping_vapor.append(Stripping_Vapor)
    rectifying_liquid.append(Rectify_Liquid)
    rectifying_vapor.append(Rectify_Vapor)
    condenser_duty.append(Condenser_Duty)
    reboiler_duty.append(Reboiler_Duty)

{Note: Append the result to their respective list}

   # Increase the reflux ratio by the iterative step for the next iteration
    Reflux_ratio += iterative_step 

# Create subplots for all the plots
fig, axs = plt.subplots(3, 2, figsize=(10, 10))

# Plot reflux ratio vs. actual number of stages
axs[0, 0].plot(reflux_ratios, actual_stages)
axs[0, 0].set_xlabel('Reflux Ratio')
axs[0, 0].set_ylabel('Actual Number of Stages')
axs[0, 0].set_title('Reflux Ratio vs. Actual Number of Stages')
axs[0, 0].grid()

# Plot reflux ratio vs. stripping liquid
axs[0, 1].plot(reflux_ratios, stripping_liquid)
axs[0, 1].set_xlabel('Reflux Ratio')
axs[0, 1].set_ylabel('Stripping Liquid')
axs[0, 1].set_title('Reflux Ratio vs. Stripping Liquid')
axs[0, 1].grid()

# Plot reflux ratio vs. stripping vapor
axs[1, 0].plot(reflux_ratios, stripping_vapor)
axs[1, 0].set_xlabel('Reflux Ratio')
axs[1, 0].set_ylabel('Stripping Vapor')
axs[1, 0].set_title('Reflux Ratio vs. Stripping Vapor')
axs[1, 0].grid()

# Plot reflux ratio vs. rectifying liquid
axs[1, 1].plot(reflux_ratios, rectifying_liquid)
axs[1, 1].set_xlabel('Reflux Ratio')
axs[1, 1].set_ylabel('Rectifying Liquid')
axs[1, 1].set_title('Reflux Ratio vs. Rectifying Liquid')
axs[1, 1].grid()

# Plot reflux ratio vs. rectifying vapor
axs[2, 0].plot(reflux_ratios, rectifying_vapor)
axs[2, 0].set_xlabel('Reflux Ratio')
axs[2, 0].set_ylabel('Rectifying Vapor')
axs[2, 0].set_title('Reflux Ratio vs. Rectifying Vapor')
axs[2, 0].grid()

# Plot reflux ratio vs. condenser duty
axs[2, 1].plot(reflux_ratios, condenser_duty)
axs[2, 1].set_xlabel('Reflux Ratio')
axs[2, 1].set_ylabel('Condenser Duty')
axs[2, 1].set_title('Reflux Ratio vs. Condenser Duty')
axs[2, 1].grid()

# Adjust the spacing between subplots
plt.tight_layout()

# Show the plots
plt.show()

{Note: refer for the explanation above}

•	Saving the file: 
fileNameToSave = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Desktop), r"D:\08 Linked In\03 DWSim Automation\04 Automation of Column\00 Modified_flowsheet.dwxmz")

interf.SaveFlowsheet(sim, fileNameToSave, True)

{Note: Here the modified version of the original sim file is saved by name, Modified_flowsheet }
