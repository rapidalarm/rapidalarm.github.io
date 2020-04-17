# Illinois RapidAlarm - A sensor and alarm module for pressure-cycled ventilators

*Developed at the [Grainger College of Engineering](https://grainger.illinois.edu/) at the University of Illinois at Urbana-Champaign with support from Carle Health and Creative Thermal Solutions, Inc.*

![](pictures/rapid_alarm_photo.png)

<iframe style="display: block;margin: 3em auto 3em auto; width: 75%;" height="315" src="https://www.youtube.com/embed/8bSyTYTYtEM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The Illinois RapidAlarm is a sensor and alarm module for use with pressure-cycled ventilators such as the [Illinois RapidVent](https://rapidvent.grainger.illinois.edu/). The module connects to a ventilator circuit and monitors the pressure delivered to the patient airway. It creates an audible alarm when it detects that the system is not operating normally and displays information about pressure and breathing rate. The Illinois RapidAlarm is designed to be rapidly produced from readily available parts. The electronic kit can be assembled by hand or by machine using through-hole or surface-mount parts. This website contains hardware designs, software code, and documentation to allow anyone to build their own Illinois RapidAlarm or to adapt the design to their own needs. All resources are provided with open licenses (see [legal disclaimer](legal.md)).

## Overview

![](pictures/rapid_alarm_setup.png)

The Illinois RapidAlarm monitors pressure in the patient airway using an electronic pressure sensor, which connects to a ventilator circuit using a tube. The pressure signal is processed by an ATmega328 microcontroller (identical to the one found in the popular Arduino hobbyist kit), which calculates ventilation parameters and monitors for alarm conditions. A seven-segment display shows three measurements: high pressure, low pressure, and respiratory rate. A buzzer creates an audible alarm when the ventilator stops cycling or when the pressure or respiratory rate is outside normal operating parameters. The system can be powered by any 5 volt power supply, such as a USB phone charger or AC wall-mount adapter. The user controls the system using three buttons.

### Features
- Connects to a pressure-cycled ventilator using standard tubing adapters.
- Powered by any 5 volt source. Available with barrel connections for standard wall-mount power adapters or micro-USB connections for USB phone chargers or battery packs.
- Audible alarm when the ventilator stops cycling, e.g. due to obstruction or disconnect.
- Audible alarm when pressure or respiratory rate fall outside their normal range.
- Alarm thresholds can be configured using buttons on the device.
- Display shows PIP (high pressure), PEEP (low pressure), and respiratory rate.
- Can be assembled by hand or by machine using inexpensive and readily available parts.


### Limitations
- The Illinois RapidAlarm is configured for and tested with the [Illinois RapidVent](https://rapidvent.grainger.illinois.edu/) emergency ventilator. It is likely to work with other pressure-cycled ventilators but may require additional configuration and testing. It may not work with ventilators that are not pressure-cycled.
- The Illinois RapidAlarm may be powered by a battery pack for portable use, but it does not monitor battery charge level and does not provide any warning or alarm when the battery is low. 
- Alarm settings are reset to default values when power is interrupted.
- The alarm may falsely trigger when the pressure settings of the ventilator are adjusted.
- While the alarm will trigger as soon as an alarm condition is detected, the values shown on the display are averaged over several breaths and may take up to 30 seconds to respond to large changes in pressure or respiratory rate. They may be inaccurate if breathing is irregular, shortly after alarm settings are changed, or during and shortly after an alarm is triggered.

## Resources
The [Illinois RapidAlarm repository on Github](https://github.com/rapidalarm/rapidalarm) contains the resources required to produce the device:

- Printed circuit board design files, schematics, and bill of materials
- Enclosure design files for 3D printing
- Software source code to run on the microcontroller
- Instructions for producing and assembling the device
- User documentation

## Specifications

### General characteristics
| Illinois RapidAlarm |     |
| ------------------- | --- |
| Patient population | Patients using pressure-cycled ventilators |
| Compatible ventilators | [Illinois RapidVent](https://rapidvent.grainger.illinois.edu/) and similar pressure-cycled ventilators |
| Patient interface | Connects via 1/16” (1.6mm) inner diameter tube to tee fitting of breathing circuit |
| Environment of care | Stationary or portable (battery pack required) |
| Sterility | Not sterile |
| Size | 82 mm x 48 mm x 15.5 mm (enclosure) |
| Power required | 5 V DC, 100 mA (e.g. USB power) |
| Power connector | Barrel jack or micro-USB variants available |

### Displayed measurements

| Metric | Display range | Display resolution |
| ---- | ---- | ---- |
| PIP (High pressure) | 0-99 cm H2O | 1 cm H2O |
| PEEP (Low pressure) | 0-99 cm H2O | 1 cm H2O |
| Respiratory rate | 0-99 breaths/min | 1 breath/min |

### Alarm conditions

| Condition | Default setting | Adjustable range |
| --------- | --------------- | ---------------- |
| Non-cycling | 10 sec | 5–30 sec |
| Low pressure | 2 cm H2O | 1–20 cm H2O |
| High pressure | 40 cm H2O | 30–90 cm H2O |
| Low respiratory rate | 6 breaths/min | 5–15 breaths/min |
| High respiratory rate | 35 breaths/min | 15–60 breaths/min |


## Contact
Please contact the Illinois RapidAlarm team lead, Professor Andrew Singer (acsinger@illinois.edu), with any inquiries.

2020 Copyright University of Illinois Board of Trustees.  This work is licensed under CC BY 4.0 license: https://creativecommons.org/licenses/by/4.0/
