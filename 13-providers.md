# End-Process Simulations

End-process simulations are a critical part of SCEPTRE. They simulate the physical part of the system being emulated. End-process simulations consist of two pieces: 1) a [solver](glossary.md#terminology) which is the physical process simulation itself and 2) a [provider](glossary.md#terminology) which is a service responsible for processing data exchange (both input and output) between the solver and the SCEPTRE field devices. Various tools for end-process simulation have been integrated into SCEPTRE and include:
+ PowerWorld
+ RTDS
+ OPALRT
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

```python
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

## Real-time simulators (RTDS and OPALRT)

There are two "real-time simulators" currently supported by pybennu:
- `OPALRT`, providing integration with the [OPAL-RT](https://www.opal-rt.com/) simulator.
- `RTDS`, providing integration with the [RTDS](https://www.rtds.com/) simulator.


### OPALRT

The OPALRT provide reads data from the OPAL-RT simulator using C37.118 (PMU) protocol (utilizing a [fork](https://github.com/sandialabs/sceptre-bennu/tree/main/src/pybennu/pybennu/pypmu) of [pypmu](https://github.com/iicsys/pypmu)) or Modbus/TCP and publishes to SCEPTRE field devices. Writes to the OPAL-RT are performed using Modbus/TCP.

Code for this provider was based on work done on [RTDS provider](#rtds).

#### Note on boolean tags in OPC

In OPC, there will be 3 tags for a boolean point named `G1CB1`:
- `G1CB1_binary_output_closed`
- `G1CB1_binary_output_closed_opset`
- `G1CB1_binary_output_closed_optype`

To write to a point: set `G1CB1_binary_output_closed` to "1" in Quick Client in TOPServer in OPC

This will write `G1CB1.closed` to SCEPTRE.

To read status of a point from OPC, read `G1CB1_binary_output_closed_opset`.
This is because DNP3 can't have values that are written and read, apparently.
So, `G1CB1_binary_output_closed` is "write", `G1CB1_binary_output_closed_opset` is "read".
The quality of `G1CB1_binary_output_closed` will show up as "Bad" in QuickClient, since it has no value.


#### OPALRT Configuration

Configuration done typically done via metadata in the SCEPTRE app. All of the fields from the YAML file in the [example below](#example-opalrt-yaml-configuration) are should be specified in the `metadata` for the `power-provider` host in the `sceptre` app.

```yaml
apiVersion: phenix.sandia.gov/v2
kind: Scenario
metadata:
  name: opalrt-example
  annotations:
    topology: opalrt-example
spec:
  apps:
    - name: sceptre
      assetDir: "/phenix/topologies/opalrt-example"
      hosts:
      - hostname: power-provider
        metadata:
        server_endpoint: tcp://172.16.1.2:5555
          publish_endpoint: udp://*;239.0.0.1:40000
          simulator: OPALRT
          type: provider
          debug: false
          elastic:
            enabled: true
            host: "http://192.0.2.11:9200"
          ...
```

#### Example OPALRT YAML configuration

Example of a YAML configuration for the OPALRT pybennu provider that was auto-generated by the sceptre phenix app.

```yaml
csv:
  enabled: false
  file_path: /root/opalrt_data/
  max_files: 3
  rows_per_file: 50000
data_dir: /root/opalrt/
debug: true
elastic:
  enabled: true
  host: http://192.0.2.122:9200
  index_basename: opalrt-gridna-clean-baseline-test
  num_threads: 3
pmu:
  bind_ip: 172.16.1.2
  enabled: true
  pmus:
  - ip: 192.0.2.111
    label: '1'
    name: PMU1
    pdc_id: 2
    port: 4730
    protocol: udp
  - ip: 192.0.2.111
    label: '2'
    name: PMU2
    pdc_id: 6
    port: 4731
    protocol: udp
provider-pmu-names: PMU1, PMU2
provider_hostname: power-provider
provider_ip: 172.16.1.2
publish_endpoint: udp://*;239.0.0.1:40000
publish_rate: 0.1
rack_ip: 192.0.2.111
sceptre_experiment: gf-test
sceptre_scenario: griDFACE
sceptre_topology: griDFACE
server_endpoint: tcp://172.16.1.2:5555
simulator: OPALRT
type: provider
```


### RTDS

SCEPTRE Provider for the Real-Time Dynamic Simulator (RTDS).

#### RTDS Protocols

Data is read via C37.118 (PMU) protocol from Phasor Measurement Unit (PMU) interface on the RTDS GTNET card, utilizing a [fork](https://github.com/sandialabs/sceptre-bennu/tree/main/src/pybennu/pybennu/pypmu) of the [pypmu](https://github.com/iicsys/pypmu) library. Data is written via GTNET-SKT protocol to the RTDS GTNETx2 card.
See docstring in [pybennu/gtnet_skt.py](https://github.com/sandialabs/sceptre-bennu/blob/main/src/pybennu/pybennu/gtnet_skt.py) for details on the GTNET-SKT protocol.


#### RTDS CSV files

Data read from the PMUs is saved to CSV files in the directory specified by the
`csv.file_path` configuration option in the app metadata or YAML config, if `csv.enabled` is true.

CSV header example:
```
rtds_time,sceptre_time,freq,dfreq,VA_real,VA_angle,VB_real,VB_angle,VC_real,VC_angle,IA_real,IA_angle,IB_real,IB_angle,IC_real,IC_angle,NA_real,NA_angle,NA_real,NA_angle
```

CSV filename example: `PMU1_BUS4-1_25-04-2022_23-49-22.csv`

#### RTDS Elasticsearch

Data read from the PMUs is exported to an Elasticsearch server if `elastic.enabled` is true. See the [Elasticsearch section](#elasticsearch) for details.

Index name: `<basename>-<YYYY.MM.DD>` (e.g. `rtds-clean-2023.06.08`)

### RTDS Tips and Tricks

NOTE: in the bennu VM, rtds.py is located in dist-packages.
Just in case you need to modify it on the fly e.g. via an inject.
`/usr/lib/python3/dist-packages/pybennu/providers/power/solvers/rtds.py`

If observing C37.118 traffic in Wireshark, configure manual decode for
each PMU port and set the protocol to "SYNCHROPHASOR" (synphasor).
Wireshark -> "Analyze..." -> "Decode As..."

#### RTDS Configuration

Configuration done typically done via metadata in the SCEPTRE app. All of the fields from the YAML file in the [example below](#example-rtds-yaml-configuration) are should be specified in the `metadata` for the `power-provider` host in the `sceptre` app. The `sceptre` app generates a YAML file in `/etc/sceptre/rtds_config.yaml` in the `power-provider` VM.

```yaml
apiVersion: phenix.sandia.gov/v2
kind: Scenario
metadata:
  name: rtds-example
  annotations:
    topology: rtds-example
spec:
  apps:
    - name: sceptre
      assetDir: "/phenix/topologies/rtds-example"
      hosts:
      - hostname: power-provider
        metadata:
        server_endpoint: tcp://172.16.1.2:5555
          publish_endpoint: udp://*;239.0.0.1:40000
          simulator: RTDS
          type: provider
          debug: false
          elastic:
            enabled: true
            host: "http://192.0.2.11:9200"
          ...
```

#### Example RTDS YAML configuration

```yaml
csv:
  enabled: true
  file_path: /root/rtds_data/
  max_files: 3
  rows_per_file: 50000
data_dir: /root/rtds_data/
debug: false
elastic:
  enabled: true
  host: http://192.0.2.121:9200
  index_basename: rtds-clean
  num_threads: 3
gtnet_skt:
  enabled: true
  ip: 192.0.2.52
  port: 7000
  protocol: udp
  tags:
  - initial_value: 1
    name: G1CB1.closed
    type: int
  - initial_value: 1
    name: CBL5.closed
    type: int
  - initial_value: 0
    name: DL5shed.value
    type: float
  - initial_value: 0
    name: PsetL5.value
    type: float
  - initial_value: 0
    name: QsetL5.value
    type: float
  tcp_retry_delay: 1
  udp_write_rate: 30
pmu:
  enabled: true
  pmus:
  - ip: 192.0.2.51
    label: BUS4
    name: PMU1
    pdc_id: 41
    port: 4714
    protocol: tcp
  - ip: 192.0.2.51
    label: BUS9
    name: PMU3
    pdc_id: 91
    port: 4719
    protocol: tcp
  retry_attempts: 3
  retry_delay: 5
provider_hostname: power-provider
provider_ip: 172.16.1.2
publish_rate: 0.03
rack_ip: 192.0.2.122
sceptre_experiment: rtds_test
sceptre_scenario: wscc-rtds
sceptre_topology: wscc-rtds
simulator: RTDS
type: provider
```

### Elasticsearch

The OPALRT and RTDS providers can directly export data read from the OPALRT or RTDS to an Elasticsearch server, as a form of "ground truth". This functionality is enabled if the `elastic` field is present in the YAML config, `elastic.enabled: true`, and `elastic.host` is specified.

Index name pattern is `<basename>-<YYYY.MM.DD>` (e.g. `example-index-2026.01.23` for data points generated on Jan 23, 2026). The basename is configured in the YAML config via `elastic.index_basename`.

Example configuration:

```yaml
elastic:
  enabled: true
  host: http://192.0.2.122:9200
  index_basename: example-index
  num_threads: 3
```

Elasticsearch functionality in pybennu is implemented in [pybennu/elastic.py](https://github.com/sandialabs/sceptre-bennu/blob/main/src/pybennu/pybennu/elastic.py).


#### Index Mapping

The Elasticsearch fields roughly follow the [Elastic Common Schema (ECS)](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html), extended with multiple custom field sets (e.g. `pmu`, `measurement`, `sceptre`). The values in the `type` column are Elasticsearch [mapping types](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html).

| field                    | type          | example                   | description |
| ------------------------ | ------------- | ------------------------- | ----------- |
| @timestamp               | date          | 2022-04-20:11:22:33.000   | Timestamp from SCEPTRE. This should match the value of sceptre_time. |
| rtds_time                | date          | 2022-04-20:11:22:33.000   | Timestamp from RTDS. |
| sceptre_time             | date          | 2022-04-20:11:22:33.000   | Timestamp from SCEPTRE provider (the `power-provider` VM in the emulation). |
| time_drift               | double        | 433.3                     | The difference in milliseconds in times between the RTDS and SCEPTRE (the "drift" between the two systems). This value will always be positive, even if the RTDS is ahead of SCEPTRE. This is calculated as abs(sceptre_time - rtds_time) * 1000. |
| event.ingested           | date          | 2022-04-20:11:22:33.000   | Timestamp of when the data was ingested into Elasticsearch. |
| ecs.version              | keyword       | 8.11.0                    | [Elastic Common Schema (ECS)](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html) version this schema adheres to. |
| agent.type               | keyword       | sceptre-provider          | Type of system running the provider. |
| agent.version            | keyword       | unknown                   | Version of the provider. |
| observer.hostname        | keyword       | power-provider            | Network name of system running the provider, as configured in the OS. |
| observer.ip              | ip            | 172.16.0.2                | IP address of the provider VM, typically set by the `sceptre` phenix app. |
| observer.name            | keyword       | power-provider            | Hostname of the provider, typically set by the `sceptre` phenix app. |
| observer.geo.timezone    | keyword       | America/Denver            | Timezone of system running the provider. |
| network.protocol         | keyword       | `c37.118`                      | Network protocol used to retrieve the data. Currently, this will be either `modbus` or `c37.118`. |
| network.transport        | keyword       | tcp                       | Transport layer (Layer 4 of OSI model) protocol used to retrieve the data. Currently, this is usually `tcp`, but it could be udp if UDP is used for C37.118 or GTNET-SKT. |
| pmu.name                 | keyword       | PMU1                      | Name of the PMU. |
| pmu.label                | keyword       | BUS4-1                    | Label for the PMU. |
| pmu.ip                   | ip            | 172.24.9.51               | IP address of the PMU. |
| pmu.port                 | integer       | 4714                      | TCP port of the PMU. |
| pmu.id                   | long          | 41                        | PDC ID of the PMU. |
| measurement.stream       | byte          | 1                         | Stream ID of this measurement from the PMU. |
| measurement.status       | keyword       | ok                        | Status of this measurement from the PMU. |
| measurement.time         | double        | 1686089097.13333          | Absolute time of when the measurement occurred. This timestamp can be used as a sequence number, and will match across all PMUs from the same RTDS case. It should match `rtds_time`, after being converted to a date. |
| measurement.frequency    | double        | 60.06                     | Nominal system frequency. |
| measurement.dfreq        | double        | 8.835189510136843e-05     | Rate of change of frequency (ROCOF). |
| measurement.channel      | keyword       | PHASOR CH 1:VA            | Channel name of this measurement from the PMU. |
| measurement.phasor.id    | byte          | 0                         | ID of the phasor. For example, if there are 4 phasors, then the ID of the first phasor will be 0. |
| measurement.phasor.real  | double        | 132786.5                  | Phase magnitude? |
| measurement.phasor.angle | double        | -1.5519471168518066       | Phase angle? |
| modbus.register | integer | 1 | Modbus Register ID |
| modbus.name | keyword | `cb_battery_voltage` | Modbus Register name |
| modbus.register_type | keyword | input | Modbus Register type, one of `holding`, `coil`, `discrete`, or `input`. |
| modbus.data_type | keyword | float32 | Modbus Register data type, one of `float32` or `bool`. |
| modbus.unit_type | keyword | V | Human-readable unit for the measurement for the Modbus register. This will be appended after the value when humanized. |
| modbus.description | keyword | Battery Voltage | Human-readable description of the register's purpose. |
| modbus.sceptre_tag | keyword | `ng1_bus_power.value` | The corresponding SCEPTRE tag, e.g. 'reg1.value' for a register named 'reg1'. If unset, the value for this field is automatically populated using the 'name' field. |
| modbus.raw_value | keyword | 600.0 | Value as returned from modbus read, converted to a string. |
| modbus.human_value | keyword | `379.9391174316406V` | Human-readable version of the value read with the unit type appended. |
| modbus.ip | ip | 192.0.2.1 | IP address of the Modbus server the value was read from (usually this is the OPAL-RT simulator). |
| modbus.port | integer | 502 | Port of the Modbus server the value was read from (usually this is the OPAL-RT simulator). |
| groundtruth.description | keyword | Ground truth description | Description of the ground truth data. |
| groundtruth.tag | keyword | `groundtruth_tag` | Tag associated with the ground truth data. |
| groundtruth.type | keyword | `float` | Type of the ground truth data (e.g., `float`, `bool`, `int`). |
| groundtruth.value | keyword | `42.0` | Value of the ground truth data stored as a keyword. Kibana doesn't allow dynamic typing without using runtime fields. This is a minor hack to store the value as both keyword type in `.value`, and as it's actual type in a separate field, e.g. `.float` field will have the value for floating point values. |
| groundtruth.float | double | 42.0 | Floating-point value of the ground truth data. |
| groundtruth.bool | boolean | true | Boolean value of the ground truth data. |
| groundtruth.int | integer | 42 | Integer value of the ground truth data. |
| sceptre.experiment | keyword | `experiment1` | Name of the experiment associated with the SCEPTRE data. |
| sceptre.topology | keyword | `example-topology` | Topology of the SCEPTRE experiment. |
| sceptre.scenario | keyword | `example-scenario` | Scenario of the SCEPTRE experiment. |
| sceptre.provider | keyword | `OPALRT` | Provider associated with the SCEPTRE experiment, this will be either `OPALRT` or `RTDS`. |
| sceptre.server_endpoint | keyword | `tcp://172.16.1.2:5555` | TCP server endpoint for the SCEPTRE provider. |
| sceptre.publish_endpoint | keyword | `udp://*;239.0.0.1:40000` | UDP publish endpoint for the SCEPTRE provider. |
| node | keyword | `node1` | Node associated with the SCEPTRE data. |

### YAML settings

The dynamic simulators (RTDS and OPALRT) use a new method of configuration using YAML files instead of the old-style INI file. This is due to their complex configuration requirements that exceeded what a INI file can reasonably accomodate, and also to add input validation and documentation using [pydantic](https://docs.pydantic.dev/latest/concepts/pydantic_settings/).

#### Documentation
Valid settings are documented in [src/pybennu/settings_schema/](https://github.com/sandialabs/sceptre-bennu/tree/main/src/pybennu/settings_schema)
- Human-readable docs: [settings_schema/pybennu_settings_schema.html](https://github.com/sandialabs/sceptre-bennu/blob/main/src/pybennu/settings_schema/pybennu_settings_schema.html)
- JSON schema: [settings_schema/pybennu_settings_schema.json](https://github.com/sandialabs/sceptre-bennu/blob/main/src/pybennu/settings_schema/pybennu_settings_schema.json)

These settings are loaded from a YAML file that's (presently) pointed to from the legacy `config.ini`, via the `"config-file = <path>"` setting. This is auto-generated by the `sceptre` app (sceptre.py), so most users shouldn't have to worry about this.

For the RTDS, this will typically be `/etc/sceptre/rtds_config.yaml`.
For the OPALRT, this will typically be `/etc/sceptre/opalrt_config.yaml`.

## Environment variables
- Environment variables
- Values read from a `.env` file in the working directory
- Values from YAML file (or whatever is used to initialize)

Values from the YAML file can be overridden by environment variables.
The environment variables start with `bennu_`.
Nested variables are separated with two underscore characters (`__`), e.g. `bennu_elastic__enabled`
Empty environment variables are ignored.

For example:

```shell
export bennu_publish_rate=2.5
pybennu-power-solver -e pybennu -c /etc/sceptre/config.ini -d restart
```

Some other examples:

```shell
export bennu_debug=true
export bennu_elastic__enabled=false
```
