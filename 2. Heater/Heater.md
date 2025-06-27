Automating Input Stream Manipulations
You can modify the inlet streams parameter or properties to simulate different feed conditions affecting the heaters performance.
To calculate the heat duty if the input temperature is changed.
CODE:

Print(dir(Heater))  to print all the properties involved in Unit Operation
import clr
import System
import pythoncom
import os
from System.IO import Directory, Path
from System import String, Environment
from DWSIM.Interfaces.Enums.GraphicObjects import ObjectType
from DWSIM.Thermodynamics import Streams, PropertyPackages
from DWSIM.UnitOperations import UnitOperations
from DWSIM.Automation import Automation3
from DWSIM.GlobalSettings import Settings
from tabulate import tabulate
# Execution:
Python loads the required lib which are imported to the environment. This lib are important for setting up of the automation environment for DWSIM. 
.clr enables the module interaction with .NET assemblies 
Pythoncom prepares for COM interaction

pythoncom.CoInitialize()
# This is necessary because DWSIM automation interface (Automation3()) relies on COM for inter-process communication.
dwSimPath = r"C:/Users/VCI RG/AppData/Local/DWSIM/"
dlls = [
    "CapeOpen.dll",
    "DWSIM.Automation.dll",
    "DWSIM.Interfaces.dll",
    "DWSIM.GlobalSettings.dll",
    "DWSIM.SharedClasses.dll",
    "DWSIM.Thermodynamics.dll",
    "DWSIM.UnitOperations.dll",
    "DWSIM.Inspector.dll",
    "System.Buffers.dll"
]
for dll in dlls:
    try:
        clr.AddReference(os.path.join(dwSimPath, dll))
    except Exception as e:
        print(f"Error loading {dll}: {e}")
        raise
Directory.SetCurrentDirectory(dwSimPath) # sets working directory to DWSIM path, ensuring DWSIM can locate the files which are required for the automation
interf = Automation3()
fileNameToLoad = Path.Combine(r"C:\Users\VCI RG\OneDrive\Desktop\DWSIM simulation\6. Heater\Heater.dwxml")
sim = interf.LoadFlowsheet(fileNameToLoad)
if sim is None:
    raise Exception(f"Failed to load flowsheet: {fileNameToLoad}")
Settings.SolverMode = 0 # sets the solver to its default mode, ensuring standard calculation behaviour
def get_dwsObject(sim, name):
    obj = sim.GetObject(name) # The above function is defined such that it can be used afterwards to access the objects (heater,material stream) in the simulation file. 
    if obj is None:
        raise Exception(f"Object '{name}' not found in flowsheet.")
    obj_instance = obj.GetAsObject()
    if obj_instance is None:
        raise Exception(f"Failed to get object '{name}'. Verify object type.")
    return obj_instance
try:
    Heater = get_dwsObject(sim, "HT-1")
    Inlet_Stream = get_dwsObject(sim, "FEED")
    outlet_stream = get_dwsObject(sim, "3")
    heat_q = Heater.get_HeatDuty()
except Exception as e:
    print(f"Error accessing object: {e}")
    raise
# The above code is executed, to access the different object in simulation file.
The script calls get_dwsObject to retrieve: 
•	HT-1: The heater unit operation.
•	FEED: The inlet material stream.
•	3: The outlet material stream.
Target_temperature = float(input("Enter the desired temperature for feed (K): "))
initial_Temp = Inlet_Stream.GetTemperature()
Inlet_Temperature = initial_Temp
Outlet_initial_Temperature = outlet_stream.GetTemperature()
Heater_Outlet_Temperature = Outlet_initial_Temperature
initial_heatq = Heater.get_HeatDuty()
Heat_Duty = initial_heatq
Iteration = 20
Step_size = (Target_temperature - initial_Temp) / Iteration


Results = []
headers = ["Iteration", "Inlet_Temperature (K)", "Heater_Outlet_Temperature (K)", "Heat_Duty (kW)"]
for i in range(Iteration):
    Inlet_Temperature = initial_Temp + (i * Step_size) # here, i is an integer from 0 to i-1. And step size is already defined
    Inlet_Stream.SetTemperature(Inlet_Temperature)
    errors = interf.CalculateFlowsheet2(sim) # recalculates the entire flowsheet, propagating the temperature change through the heater to update the outlet stream and heat duty. 
    if errors:
        print(f"Simulation errors in iteration {i+1}: {errors}")
        break
    try:
        Heater_Outlet_Temperature = outlet_stream.GetTemperature()
        Heat_Duty =grupo Heater.get_HeatDuty()
    except Exception as e:
        print(f"Error retrieving data in iteration {i+1}: {e}")
        break
    Results.append([i+1, Inlet_Temperature, Heater_Outlet_Temperature, Heat_Duty])
    print(tabulate(Results, headers=headers, tablefmt="grid"))

# To make a graph for inlet temperature vs Heat duty
fig, axs = plt.subplots(1, 1, figsize=(10, 8))
inlet_temps = [row[1] for row in Results]  # Extract Inlet_Temperature from Results
heat_duties = [row[3] for row in Results]  # Extract Heat_Duty from Results
axs.plot(inlet_temps, heat_duties, label="Heat Duty vs Inlet Temperature", marker='o')
axs.set_xlabel("Inlet Temperature (K)")
axs.set_ylabel("Heat Duty (kW)")
axs.set_title("Heater Heat Duty as a Function of Inlet Temperature")
axs.legend()
axs.grid(True)
plt.show()
