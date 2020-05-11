# How It Works

The Illinois RapidAlarm is an electronic sensor module that attaches to a pressure-cycled ventilator and measures pressure in the patient airway. It displays information about pressure and respiration rate and creates an alarm when the ventilator stops working correctly. This page explains how the RapidAlarm’s measurement and alarm algorithms work. These algorithms are implemented in `algorithms.h` in the RapidAlarm firmware.

[gimmick: math]()

## Pressure-cycled ventilation

TODO: Ventilator diagram

Pressure-cycled ventilators, like the Illinois RapidVent, aid breathing by delivering pressurized air to a patient’s airway. These ventilators are powered by pressurized gas and operate using a set of mechanical valves; there are no electronics required.

During inhalation, high-pressure gas flows from the ventilator to the patient’s lungs. As the lungs inflate, the pressure in the airway steadily increases. Once the airway pressure has reached a specified level, known as the peak inspiratory pressure (PIP), the modulator on the ventilator is triggered automatically and exhalation begins. The PIP can be adjusted by the caregiver using a knob.

Once PIP is reached, the modulator disconnects the patient airway from the high-pressure gas and allows air from the lungs to exit the ventilator. During exhalation, the pressure in the airway drops steadily, but does not fall to atmospheric pressure. Instead, once it drops below a specified positive end-expiratory pressure (PEEP), the modulator switches to inhalation and the airway is again connected to the high-pressure gas.

TODO: Ideal waveform

Pressure-cycled ventilators produce characteristic pressure waveforms, as shown above. The airway pressure rises from PEEP to PIP during inspiration, then drops from PIP to PEEP during exhalation. The Illinois RapidAlarm uses an electronic pressure sensor to monitor this pressure signal. Signal processing algorithms detect when the ventilator malfunctions and calculate the PIP, PEEP, and respiratory rate.

## Pressure tracking

A simple breath tracking algorithm could look for maxima and minima in the pressure signal. However, real signals do not always increase and decrease monotonically like the waveform above. A more realistic signal, measured with the Illinois RapidVent, is shown below. The tracking algorithm must be robust against small pressure variations and must have low computational requirements so that it can run on inexpensive, low-power microcontrollers.

TODO: Realistic waveform

The Illinois RapidAlarm uses a pair of nonlinear recursive filters to track the envelope of the pressure signal. Each envelope tracker performs a moving average, but gives more weight to changes in one direction than another. The high-pressure envelope increases quickly but decreases slowly, so it follows the top of the pressure signal, while the low-pressure envelope decreases quickly and increases slowly, following the bottom of the signal.

TODO: Envelope figure

Let \\( p[t] \\) be the discrete-time pressure signal from the sensor. The envelopes are given by
$$ v_{high}[t]=\begin{cases}
(\alpha_A v_{high}[t-1]+(1-\alpha_A )p[t], & if p[t]≥v_{high}[t]\\
\alpha_R v_{high}[t-1]+(1-\alpha_R )p[t],if p[t]<v_{high}[t] \end{cases} $$
$$ v_{low}[t]=\begin{cases}(\alpha_A v_{low}[t-1]+(1-\alpha_A )p[t], & if p[t]≤v_{low}[t]\\
\alpha_R v_{low}[t-1]+(1-\alpha_R )p[t],if p[t]>v_{low}[t] \end{cases} $$

The main advantage of the recursive envelope tracker is that it does not need to store any past values of the pressure signal in memory. If instead we wanted to measure the maximum of the signal over the last two seconds, we would need to store about 200 past measurements. With the recursive tracker, more recent measurements are automatically weighted more strongly; each sample’s contribution to the average decays exponentially over time. The rate of decay is controlled by the \\(\alpha\\) coefficients. 

When the tracker is responding rapidly to a change in signal level (an increase for the high-pressure envelope or a decrease for the low-pressure envelope), it is said to be in attack mode. The attack coefficient α_A is small (around 0.5 in our implementation at 100 samples/sec), so that the tracker quickly forgets past estimates. When the tracker is responding slowly (to a decrease in pressure for the high-pressure envelope or an increase for the low-pressure envelope), it is said to be in release mode. The release coefficient α_R is large (around 0.999 in our implementation) so that the envelope decays slowly.

This slow decay also allows the tracker to ignore small fluctuations in pressure, which is helpful when counting breath cycles. The rate of decay should be slow enough that the high and low envelopes stay far apart during a normal breath cycle. If the tracker is too slow, however, it could miss breaths when the pressure settings are adjusted or, worse, might take too long to trigger an alarm when the ventilator stops working. The choice of α_R is discussed further below.

## Measurements

The Illinois RapidAlarm shows three measurements: PIP, PEEP*, and respiratory rate. All three of these metrics are tracked by detecting inhalation and exhalation cycles from the pressure envelopes.

TODO: Breath tracking diagram

The two envelope trackers both store the most recent pressure samples that triggered their attack mode. These points are shown on the diagram above. During each inhalation cycle, there are several attack-mode samples in a row for the high-pressure envelope. During exhalation, there are several attack-mode samples in a row for the low-pressure envelope. The system tracks breath cycles by looking for low-pressure attack events that follow high-pressure attack events and vice versa. A low-pressure attack event causes the system to switch from inhalation to exhalation mode, and a high-pressure attack event causes it to switch from exhalation to inhalation mode.

When a mode switch occurs, the previous attack value is used to update the corresponding PIP or PEEP metric. That is, when a low-pressure attack event occurs, the PIP display is updated with the most recent high-pressure attack value. When a high-pressure attack event occurs, the PEEP display is updated with the most recent low-pressure attack value. Both PIP and PEEP are recursively smoothed over time to remove small fluctuations:

$$PIP[n]=\alpha_S PIP[n-1]+(1-\alpha_S)(last high pressure peak)$$
$$PEEP[n]=\alpha_S PEEP[n-1]+(1-\alpha_S)(last low pressure peak)$$
where \\(\alpha_S\\) is a smoothing coefficient (we use 0.5) and the time index n counts breath cycles (not samples).

TODO: PIP/PEEP diagram

The system also keeps track of the time elapsed between these mode-switch events. A complete breath cycle is measured between high-pressure peaks. Thus, the respiratory rate is given by
$$ RR[n]=\alpha_S RR[n-1]+(1-\alpha_S)\frac{60\times sample rate}{samples between peak n-1 and peak n} $$

\* Strictly speaking, the RapidAlarm tracks the minimum pressure reached during the breathing cycle. During mandatory breathing, when the ventilator initiates all breaths, this is the same as the PEEP level at which the valve triggers. However, when the ventilator is operating in assisted-breathing mode, the patient’s spontaneous breaths cause the pressure to fall below the valve’s trigger point. In this case, the displayed pressure is lower than the ventilator’s PEEP setting.

## Alarm conditions

The Illinois RapidAlarm triggers alarms in several conditions that indicate the ventilator is not working properly:
1. Pressure is too high (user-configurable threshold)
2. Pressure is too low (user-configurable threshold)
3. Respiratory rate is too high (user-configurable threshold)
4. Respiratory rate is too low (user-configurable threshold)
5. Non-cycling (user-configurable time)

The first four alarm conditions require little explanation. The high- and low-pressure alarms trigger immediately if the pressure threshold is crossed. The high- and low-respiratory-rate alarms are triggered by the calculated respiratory rate. 

The non-cycling alarm is slightly more complex. It must detect when the breathing cycle has stopped, which can happen in several ways, as illustrated below. Thus, this alarm can be triggered in several ways.

TODO: different non-cycling conditions

First, the alarm triggers if too much time has passed since the last attack event. For example, if the pressure drops to PEEP and remains constant, as shown in ????, there will be no attack events in the high-pressure envelope tracker, so it will trigger the alarm. If, however, the pressure fluctuates slightly over time, the tracking algorithm will still detect frequent peaks, as shown in ????.

To handle this case, the alarm will also trigger if the high-pressure envelope and low-pressure envelope are too close together. In the Illinois RapidVent and similar pressure-cycled ventilators, the ratio between PIP and PEEP is a constant determined by the mechanical design of the device. Thus, an alarm is triggered if \\(v_{high} [t]/v_{low} [t]\\) drops too far below that nominal value. It also triggers if the difference \\(v_{high} [t] – v_{low} [t]\\) is too small.

The alarm-triggering conditions are summarized in the table below:
| Alarm |	Condition |
| ----- | --------- |
| Low pressure | p[t]<low pressure threshold |
| High pressure | p[t]>high pressure threshold |
| Low rate | RR[n]<low RR threshold |
| High rate | RR[n]>high RR threshold |
| Noncycling | Time since last high pressure attack > alarm time 
Time since last low pressure attack > alarm time
$$ v_{high}[t] / v_{low}[t]  < alarm ratio $$
$$ v_{high}[t]– v_{low}[t]< alarm ratio $$|

## Calculating the release coefficient

The rate of decay of the envelope tracker depends on the release coefficient, alpha_R. The filter should adapt quickly enough to keep up with changes in pressure settings and to react to non-cycling conditions, but not so quickly that it registers false breaths or triggers a false alarm during normal breathing.

In our implementation, we have set alpha_R so that, if the pressure were to fall suddenly from PIP to PEEP and remain constant at PEEP, the noncycling alarm’s high-to-low-envelope ratio condition would be triggered at around the same time as its time-since-last-high-pressure-attack condition for the default alarm setting. In this scenario, the high-pressure envelope decays as
$$ v_{high} [t]=PEEP+\alpha_R^{t-t_0} (PIP-PEEP), $$
where \\(t_0\\) is the time at which the pressure drop occurs. The alarm will therefore be triggered when
$$ v_{high} [t]=alarm ratio \times PEEP, $$
or 
$$ alarm ratio=1+\alpha_R^{t-t_0} (\frac{PIP}{PEEP}-1). $$
Setting the elapsed time to the default alarm time and solving for \\(\alpha_R\\), we have
$$\alpha_R=\left(\frac{alarm ratio-1}{nominal ratio-1}\right)^{1/(alarm time \times sample rate)}.$$
