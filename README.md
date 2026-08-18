**Finger Heart Rate / SpO2 Monitor — nRF7002-DK + ESP32-S3 + MAX30101**

Measures heart rate (BPM) and blood oxygen saturation (SpO2) from a fingertip using a MAX30101 optical sensor, read by an nRF7002-DK (nRF5340), with a separate ESP32-S3 board used purely as a clean power source for the sensor.

Why two boards?

The MAX30101's LED driver pulls short current spikes on every sample (tens of mA). The nRF7002-DK's onboard 3.3V rail is shared with the rest of the DK's logic, so those spikes sag the rail enough to corrupt readings. The fix: give the sensor its own dedicated 3.3V rail from the ESP32-S3 board's onboard regulator (rated for 500 mA–1 A vs. the DK's shared, more limited rail), and keep the nRF doing only I2C + logic on a separate power domain.
Board	Role
ESP32-S3	Power only. Boots, blinks once, deep-sleeps forever. Its onboard 3.3V regulator stays live regardless, feeding the sensor.
nRF7002-DK (nRF5340)	All I2C, sensor configuration, FIFO reading, and the HR/SpO2 algorithm.
MAX30101	The sensor itself — LEDs + photodiode + ADC + FIFO.

Wiring

Power domain (ESP32-S3 → sensor):
ESP32-S3 3V3 → MAX30101 VIN (/ VLED if separate on your breakout)
10 µF + 0.1 µF decoupling caps at the MAX30101 VIN pin (critical — this is what actually stops LED-pulse sag)
ESP32-S3 GND → common ground

Signal domain (MAX30101 → nRF7002-DK):
1. MAX30101 SDA → nRF7002-DK P1.02
2. MAX30101 SCL → nRF7002-DK P1.03
3. MAX30101 INT → not wired (firmware polls the FIFO instead of using the interrupt pin)
4. nRF7002-DK GND → common ground

Do not tie the ESP32-S3's 3V3 and the nRF board's 3V3 together — only share ground between the two boards. Most red MAX30101 breakouts have onboard SDA/SCL pull-ups referenced to their own VCC (fed from the ESP32-S3 here), so no external pull-up resistors are normally needed.


Connections

MAX30101 to nRF7002-DK
MAX30101     nRF7002-DK
SDA	         P1.02
SCL	         P1.03
GND	         Common GND
VCC	         ESP32-S3 3.3 V
The MAX30101 is powered separately from the ESP32-S3 while the nRF5340 communicates with it through I²C.


How It Works

The MAX30101 contains:
Red LED
Infrared (IR) LED
Photodetector
The LEDs illuminate the fingertip and the photodetector measures the returning light.
Blood absorbs RED and IR light differently depending on the oxygenation of hemoglobin.
The resulting PPG signals are processed by the nRF5340.


Finger Measurement

For measurement:
1. Place the fingertip directly over the MAX30101 sensor.
2. Ensure the sensor has consistent contact with the skin.
3. Keep the finger relatively still.
4. Avoid excessive pressure.
5. Avoid strong external light reaching the sensor.
6. Wait for the PPG signal to stabilize.


Serial Output

The processed results are displayed through the serial terminal.

Example:

Heart Rate : 76.4 BPM
SpO2       : 97.8 %
IR average : 84231
The system continuously acquires sensor data and periodically calculates new measurements.


Software:

The project is developed using:
VS Code
nRF Connect for VS Code
nRF Connect SDK
C


Project Structure

BioPulse-F/

│

├── src/

│   └── main.c

│

├── CMakeLists.txt

├── prj.conf

└── README.md


Future Development

A wrist-based version, BioPulse-W, can be developed using the same MAX30101 and nRF5340 architecture.
The wrist implementation would require additional optimization of:
. PPG filtering
. Motion/noise handling
. Sensor placement
. Signal quality detection
. SpO₂ calibration


Disclaimer
BioPulse-F is developed for educational, research, and embedded-systems prototyping purposes.
It is not a medical device and should not be used for medical diagnosis, treatment, or clinical decision-making.
