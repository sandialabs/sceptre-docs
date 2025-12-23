# End-Process Simulations

End-process simulations are a critical part of SCEPTRE. They simulate the physical part of the system being emulated. End-process simulations consist of two pieces: 1) a [solver](glossary.md#terminology) which is the physical process simulation itself and 2) a [provider](glossary.md#terminology) which is a service responsible for processing data exchange (both input and output) between the solver and the SCEPTRE field devices. Various tools for end-process simulation have been integrated into SCEPTRE and include: 
+ PowerWorld
+ RTDS
+ OpenDSS
+ PyPower
+ Matlab Simulink 
+ GenericPython

Implementation of new end-process simulations into SCEPTRE involve connecting many pieces of information to ensure end-to-end data flow. Below, outlines how to create a generic python simulation that integrates with SCEPTRE. Details on the other simulations will be released soon. 

## Generic Python Provider
The `GenericPython` provider was developed for quick and easy development of discrete event simulations that do not require extensive system specific modeling tools like power systems. 

To create a new generic python solver, a simple python script needs to be developed representing the desired physics. The script must define a class `Simulation` which inherits from the SCEPTRE `GenericPython` solver base class `BaseSimulation` and implements the `__int__` method which defines data input/output that make up the simulation `state` and the `update_state` method which defines the dynamics of the simulation per time step. 

The names of the variables defined in the simulation `state` must be of the format `device.field` and the chosen naming convention must align with the downstream SCEPTRE infrastructure used in the scenario file configuration. Furthermore, variables in the state must be categorized as either inputs (independent variables of the simulation that can be changed to have effect of the simulation state) or outputs (dependent variables of the simulation) and their data type must be defined as either analog (integer and float) or binary (boolean). 

Below is an example of a generic python solver for a simple heater/cooler system controlled by a thermostat. Note the physics in this example are drastically simplified for example purposes. The simulation has two inputs: 
+ The temperature the thermostat is set to `temperature_setpoint.value`
+ The state of the thermostat being on/off `on.status`. 

The simulation has three outputs:
+ The current temperature of the room `temperature.value`
+ The difference in temperature between the room and thermostat value `temperature_diff.value`
+ The indicator of if the room is at the setpoint of the thermostat `at_temp.status`

On each timestep of the simulation, the outputs of the simulation update:
+ If the thermostat is on and the difference between the room temperature and thermostat setpoint is negative (room temperature is below thermostat setting) then the heater heats up the room by 1 degree. 
+ If the thermostat is on and the difference between the room temperature and thermostat setpoint is positive (the temperature is above thermostat setting) then the cooler cools down the room by 1 degree. 
+ Update the difference between the room temperature and the thermostat setpoint.
+ Calculate if the room temperature has reached the thermostat setpoint. 

```
from pybennu.providers.power.solvers.generic_python.base_simulation import BaseSimulation

class Simulation(BaseSimulation):
    """Simulation class for demonstration purposes."""

    def __init__(self):
        dt = 1
        super().__init__(dt)

        """In the simulator, inputs are inputs to the simulation (outputs from a fd) and vice versa"""
        self.state = {
            'input': {
                'analog': {
                    'temperature_setpoint.value': 100.0
                },
                'binary': {
                    'on.status': True
                }
            },
            'output': {
                'analog': {
                    'temperature.value': 25.0,
                    'temperature_diff.value': 75.0
                },
                'binary': {
                    'at_temp.status': False
                }
            }
        }
        self.check_state()

    def update_state(self):
        # Simulate changes in outputs based on inputs
        #if on, change heat to setpoint, otherwise, do nothing
        if self.state['input']['binary']['on.status']:
            if self.state['output']['analog']['temperature_diff.value'] > 0:
                self.state['output']['analog']['temperature.value'] -= 1
            elif self.state['output']['analog']['temperature_diff.value'] < 0:
                self.state['output']['analog']['temperature.value'] += 1
        self.state['output']['analog']['temperature_diff.value'] = self.state['output']['analog']['temperature.value'] - self.state['input']['analog']['temperature_setpoint.value']
        self.state['output']['binary']['at_temp.status'] = (self.state['output']['analog']['temperature_diff.value'] == 0)
``` 

Once the solver is implemented, it is time to integrate with the SCEPTRE scenario config file and the [SCEPTRE app](08-sceptre-user-app.md) specifically. The provider device in the scenario configuration file for a python simulation must include the metadata fields `publish_endpoint`, `simulator: GenericPython`, `simulation_file` which points to the location of the above mentioned simulation, and `type:provider`. Below is an example provider configuration. 

```yaml
- hostname: provider
  metadata:
    publish_endpoint: udp://*;239.0.0.1:40000
    simulator: GenericPython
    simulation_file: <path_to>/simulation.py
    type: provider
```

Finally, the SCEPTRE field device configuration must be added to read/write data from the simulation. The most important part of this step is to ensure that the naming convention defined in the associated `infrastructure` maps to the variable naming convention used in the implemented solver. 

Below is an example for the heater/cooler solver. This field device configuration uses the `generic` infrastucture. This infrastructure expects any analog variables in the solver to be of the form `<variable name>.value` and any binary variables to be of the form `<variable name>.status`. This configuration also maps inputs/outputs of the solver to registers in the field device. Note that "inputs" to the solver are variables that the field device can both read and write to, while "outputs" of the solver can only be read by the field device.
```yaml
- hostname: rtu-1
  metadata:
    modbus:
    - name: temperature
      type: analog-read
    - name: temperature_diff
      type: analog-read
    - name: at_temp
      type: binary-read
    - name: temperature-setpoint
      type: analog-read-write
    - name: on
      type: binary-read-write
    infrastructure: generic
    provider: provider
    type: fd-server
```
