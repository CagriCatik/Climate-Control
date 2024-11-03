# Automatic Climate Control System Simulation

## Overview

This documentation describes the simulation of an **automatic climate control system** for a vehicle using **Simulink** and **Stateflow** in MATLAB. The simulation enables setting desired internal temperatures, adjusting to external conditions, and dynamically managing the vehicle's climate system, which includes heater, air conditioner, blower, and air distribution mechanisms.

## Components of the System

The simulation model includes various components to emulate a real-world climate control system, each of which is described below:

1. **User Setpoint in Celsius**:
   - Allows users to specify the desired temperature inside the car. This is the target temperature that the climate control system aims to achieve.

2. **External Temperature in Celsius**:
   - This block allows users to input the external (ambient) temperature, simulating different outside conditions that the climate control system will respond to.

3. **Thermometer Display**:
   - Displays the temperature sensed behind the driver’s head, which represents the temperature the driver feels. This temperature reading reflects the real-time climate control adjustments in response to the setpoint and external conditions.

## Stateflow® Controller

The **Stateflow controller** handles the supervisory control logic of the climate control system. It coordinates the operation of the heater, air conditioner, blower, and air distribution mechanisms based on the difference between the setpoint and current cabin temperatures. Below are key elements of the control logic:

1. **Heater and Air Conditioner Control**:
   - **Heating Mode**: When the setpoint temperature is at least 0.5°C higher than the current car temperature, the heater activates. It remains on until the car's temperature is within 0.5°C of the setpoint.
   - **Cooling Mode**: When the setpoint temperature is at least 0.5°C lower than the current temperature, the air conditioner activates. It remains on until the car temperature is within 0.5°C of the setpoint.
   - **Dead Band**: A dead band of 0.5°C prevents the system from continuously toggling the heater or air conditioner, ensuring smoother operation.

2. **Blower Control**:
   - The **Blower** adjusts its speed based on the temperature difference between the setpoint and current cabin temperature. A larger difference results in a higher blower speed, accelerating the temperature adjustment process. When the cabin temperature reaches within 0.5°C of the setpoint, the blower turns off.

3. **Air Distribution and Recycling Air Control**:
   - **Air Distribution (AirDist)** and **Recycling Air (Recyc_Air)** are controlled by switches in Stateflow, which manage airflow within the cabin.
   - **Defrost Mode**: To enhance window defrosting, the system implements an internal transition that turns off air recycling when the defrost state is active.

## Heater and Air Conditioner Models

### Heater Model

The heater model uses a **heat exchange equation** to determine the temperature of the air exiting the heater:

\[
T_{\text{out}} = T_s - (T_s - T_{\text{in}}) e^{\left(\frac{-\pi \cdot D \cdot L \cdot h_c}{\dot{m} \cdot C_p}\right)}
\]

where:
- \( T_s \) = Radiator wall temperature (constant)
- \( D \) = Channel diameter (0.004 m)
- \( L \) = Radiator thickness (0.05 m)
- \( N \) = Number of channels (30,000)
- \( k \) = Thermal conductivity of air (0.026 W/mK)
- \( C_p \) = Specific heat of air (1007 J/kgK)
- **Heat Transfer Coefficient (\( h_c \))** = 23.8 W/m²K (based on laminar flow)

The heater's effectiveness increases with the temperature difference between the setpoint and the current interior temperature, just like the blower operation.

### Air Conditioner Model

The air conditioner model applies an **energy balance equation** based on compressor parameters:

\[
y \cdot (w \cdot T_{\text{comp}}) = \dot{m} \cdot (h_4 - h_1)
\]

where:
- \( y \) = Efficiency of the A/C system
- \( \dot{m} \) = Mass flow rate of air
- \( w \) = Engine speed
- \( T_{\text{comp}} \) = Compressor torque
- \( h_4, h_1 \) = Enthalpy at different points in the cycle

The **bang-bang control** for the A/C system uses engine speed and compressor torque to manage the temperature of air exiting the A/C system.

## Cabin Heat Transfer Model

The climate control system accounts for various factors affecting the temperature felt by the driver:

1. **Temperature of Air Exiting the Vents**:
   - The model calculates the difference between the temperature of the vent air and the current cabin temperature, then scales it by the fan speed proportion (mass flow rate).

2. **External Temperature Influence**:
   - The model simulates heat transfer between the external air and the cabin, based on the difference between the external and cabin temperatures and scaled by a small mass flow rate.

3. **Occupant Heat Contribution**:
   - For each person in the car, the model adds approximately 100 W of heat energy, simulating the impact of passengers on the cabin temperature.

The **Thermometer Display** block shows the temperature sensed behind the driver's head, which represents the perceived temperature of the cabin interior. By default, the temperature reading starts at the external temperature (18°C) and moves toward the setpoint temperature (9°C) as the simulation progresses.

## Key Simulation Equations and Parameters

### Heat Exchange Equation (Heater Model)

\[
T_{\text{out}} = T_s - (T_s - T_{\text{in}}) e^{\left(\frac{-\pi \cdot D \cdot L \cdot h_c}{\dot{m} \cdot C_p}\right)}
\]

### Energy Balance Equation (Air Conditioner Model)

\[
y \cdot (w \cdot T_{\text{comp}}) = \dot{m} \cdot (h_4 - h_1)
\]

### Parameters

- **Temperature Dead Band**: 0.5°C (prevents continuous toggling of heater/AC)
- **Occupant Heat Load**: 100 W per person
- **Vent Airflow Adjustment**: Proportional to the difference between setpoint and current cabin temperature
