# Vehicle Electrical and Climate Control Systems Simulation
## Overview

This documentation provides an in-depth description of the **Vehicle Electrical and Climate Control Systems** model, which simulates the combined operation of a vehicle's climate control and electrical systems using **Simscape Electrical**, **Simulink**, and **Stateflow** in MATLAB. The model allows users to analyze how the climate control system impacts the overall vehicle electrical system, providing insights into energy consumption, load effects, and control dynamics.

## Climate Control System

The **Climate Control System** is responsible for managing the interior temperature of the vehicle, adjusting based on the user’s preferred settings and external conditions. The primary components and their functions are as follows:

1. **User Setpoint in Celsius**:
   - Users can specify their desired cabin temperature here. This block allows entry of a temperature value that serves as the target for the climate control system.

2. **External Temperature in Celsius**:
   - This block accepts the external (ambient) temperature value, simulating different weather conditions that the system will adapt to.

3. **Thermometer Display**:
   - Displays the temperature sensed behind the driver’s head. This temperature represents the climate the driver feels and reflects the effectiveness of the climate control adjustments.

## Stateflow® Controller

The **Stateflow® Controller** implements the supervisory control logic for the climate control system. The control chart, named **Temperature Control**, dictates the operation of the heater, air conditioner, blower, and air distribution systems. Key control elements include:

1. **Heater and Air Conditioner Control**:
   - **Heater Activation**: If the setpoint temperature is at least 0.5°C above the current internal vehicle temperature, the heater switches on. It remains active until the cabin temperature is within 0.5°C of the setpoint.
   - **Air Conditioner Activation**: If the setpoint temperature is at least 0.5°C below the current cabin temperature, the air conditioner turns on and stays active until the temperature is within 0.5°C of the setpoint.
   - **Dead Band**: A dead band of 0.5°C is implemented to prevent continuous toggling between heating and cooling, ensuring stable operation.

2. **Blower Control**:
   - The **Blower** output is proportional to the temperature difference between the setpoint and the current cabin temperature. A larger difference increases blower speed, allowing faster temperature adjustment. Once the cabin temperature is within 0.5°C of the setpoint, the blower turns off.

3. **Air Distribution and Recycling Air Control**:
   - **Air Distribution (AirDist)** and **Recycling Air (Recyc_Air)** are controlled by switches that trigger the Stateflow chart, managing airflow for various cabin zones.
   - **Defrost Mode**: To support window defrosting, the controller implements an internal transition within the AirDist and Recyc_Air states. When defrost is active, air recycling is turned off to ensure effective defogging.

## Heater and Air Conditioner Models

### Heater Model

The heater model simulates a **heat exchanger** using the following equation:

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

The model considers the **heater flap**, which adjusts its opening based on the difference between the setpoint and current cabin temperature. A larger temperature difference results in a wider flap opening, increasing the heating effect.

### Air Conditioner Model

The air conditioner (AC) model interfaces with the vehicle electrical system through the compressor load, which affects the engine load when the AC is active. The AC model calculates the final air temperature using the following equation:

\[
y \cdot (w \cdot T_{\text{comp}}) = \dot{m} \cdot (h_4 - h_1)
\]

where:
- \( y \) = Efficiency
- \( \dot{m} \) = Mass flow rate of air
- \( w \) = Engine speed
- \( T_{\text{comp}} \) = Compressor torque
- \( h_4, h_1 \) = Enthalpy at different stages of the process

The **bang-bang control** for the AC system uses the engine speed and compressor torque to manage the temperature of the air exiting the AC system, providing effective cooling as needed.

## Cabin Heat Transfer Model

The cabin temperature, as sensed by the driver, is affected by several factors:

1. **Vent Air Temperature**:
   - The model calculates the difference between the air temperature exiting the vents and the cabin’s current temperature, scaled by the fan speed (mass flow rate).

2. **External Air Temperature**:
   - Heat transfer between the outside and inside air is modeled based on their temperature difference, with a smaller mass flow rate factor accounting for radiative heating.

3. **Occupant Heat Contribution**:
   - The model adds approximately 100 W of heat energy per person in the cabin, simulating the impact of passengers on the cabin temperature.

The **Thermometer Display** block outputs the temperature reading of a sensor behind the driver's head, which is calculated based on these heat transfer dynamics. When the model starts, this temperature begins at the external temperature (e.g., 18°C) and moves toward the user setpoint as the system adjusts.

## Electrical System

The electrical system models the vehicle’s power generation and distribution, particularly at idle speed, where loading effects from the climate control system are notable. Key elements include:

1. **Alternator**:
   - Modeled as a synchronous machine with regulated field current, the alternator maintains the DC bus voltage necessary for the vehicle’s electrical needs. A PID controller ensures the alternator operates at the correct speed, regardless of varying loads.

2. **Rectifier Bridge**:
   - Converts the alternator’s three-phase AC output to DC. This DC power charges the vehicle battery and supplies the vehicle’s DC bus.

3. **Battery and DC Bus**:
   - The battery stabilizes the DC bus voltage, providing power to various vehicle systems, including the climate control fan, wipers, and radio.

4. **Fan and DC Bus Load**:
   - The climate control fan is connected to the DC bus. As the temperature difference between the setpoint and current cabin temperature decreases, the fan speed and load on the DC bus also decrease.

5. **Engine Speed Variation**:
   - The model allows for changes in engine speed to analyze its effect on DC bus voltage. Lower or higher engine speeds influence the alternator output, which in turn affects the DC bus voltage stability.

## Key Equations and Parameters

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
- **Fan Load Reduction**: Proportional to temperature difference between setpoint and current cabin temperature
- **Engine Speed Influence**: Allows simulation of varying engine load on the alternator and DC bus voltage
