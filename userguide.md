# User Guide

## Hardware Setup
- Attaching the Ventmon to the bed/pole/whatever
- Connecting the tube to the ventilator
- Connecting to a power supply
- Turning on the device
- Initial setup

## Display
The Illinois Ventmon features a display that cycles through three metrics: high pressure, low pressure, and respiratory rate. The diagram below shows a typical pressure waveform for a pressure-cycled ventilator. The ventilator fills the lungs until the pressure reaches PIP, which triggers a valve to open for exhalation. Another valve triggers inhalation when the pressure has dropped below PEEP.

![Pressure cycle figure](pictures/pressure_diagram.png)

**High pressure (HP)**: the maximum pressure in a breath cycle in cm H2O, which should correspond to the peak inspiratory pressure (PIP) setting of the ventilator.

**Low pressure (LP)**: the minimum pressure in a breath cycle in cm H2O, which corresponds to the positive end-expiratory pressure (PEEP) during mandatory breathing. In assisted breathing, the minimum pressure may not correspond to PEEP.

**Respiratory rate (rr)**: the number of complete breath cycles per minute, calculated from the time between the last several breaths.

**WARNING**: The displayed parameters are averaged over several breath cycles and may take up to 30 seconds to reflect large changes in ventilator settings.

## Alarm conditions
The Illinois Ventmon generates an audible alarm if it detects that the ventilator is not operating normally. The alarm conditions are summarized in the following table:

Condition |	Default setting |	Adjustable range
--------- | --------------- | ---------------
Non-cycling | 15 sec | 5-30 sec
Low pressure | 5 cm H2O | ???
High pressure | 40 cm H2O | ???
Low respiratory rate | 5 breaths/min | ???
High respiratory rate | 30 breaths/min | ???

**Non-cycling**: Triggers if the pressure has not changed in more than the specified number of seconds. This indicates that the ventilator has stopped working.

**Low pressure**: Triggers if the pressure falls below the specified threshold. Pressure-cycled ventilators are designed to maintain positive pressure at all times, so a drop to atmospheric pressure could indicate a disconnection.

**High pressure**: Triggers if the pressure exceeds the specified threshold. Pressure-cycled ventilators should never exceed the user-specified PIP value, so a high pressure reading could indicate a mechanical failure.

**Low respiratory rate**: Triggers if the breathing rate is too slow. This could indicate that the ventilator has not been adjusted correctly.

**High respiratory rate**: Triggers if the breathing rate is too fast. This could occur if the ventilator has not been adjusted correctly or if the tidal volume is too low.

**WARNING**: The alarm can sometimes trigger incorrectly shortly after the pressure settings of the ventilator have changed. These false alarms typically occur within 30 seconds of adjustments.

## User interface
- How to silence/reset alarm
- How to enable/disable alarm
- How to adjust alarm thresholds
