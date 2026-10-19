# Omnidirectional Wind Turbine

A small-scale wind turbine designed to generate power from wind arriving from any direction, without needing to turn to face it. This repository holds the data-logging code, the design model, and test results.

## Measurement system

Turbine output is measured with an [Adafruit INA219](https://www.adafruit.com/product/904) current/voltage sensor connected to an Arduino over I²C. The sketches read bus voltage, shunt voltage, current, and power twice per second, print them to the Serial Monitor (9600 baud), and store power readings in the Arduino's EEPROM so data can be collected away from a computer and read back later.

The INA219 is configured for the 16 V / 400 mA range for better voltage and current resolution at the turbine's output levels.

### Two versions of the sketch

| Sketch | Intended use | Logging behavior |
|---|---|---|
| `LisaTurbineCodeIndoors.ino` | Indoor testing (e.g. with a fan) | Writes a power reading to EEPROM on every loop for short, high-resolution runs |
| `LisaTurbineCodeOutdoors.ino` | Longer outdoor runs | Set up to store one averaged reading per 240 samples (about 2 minutes) so a run fits in EEPROM |

### On startup

1. Prints every stored EEPROM value (divided by 5) to Serial, so you can retrieve the previous run's data.
2. Clears the EEPROM.
3. Initializes the INA219 and begins measuring.

Connect the Arduino to a computer and open the Serial Monitor right after powering on to capture the previous run's data before it is erased on the next start.

## Repository contents

| File | Description |
|---|---|
| `LisaTurbineCodeIndoors.ino` | Data-logging sketch for indoor tests |
| `LisaTurbineCodeOutdoors.ino` | Data-logging sketch for outdoor tests |
| `Omnidirectional Wind Turbine Model.rtf` | Description of the turbine design/model |
| `Omnidirectional Wind Turbine Indoor Data.pdf` | Results from indoor testing |

## Hardware

- Arduino-compatible board
- Adafruit INA219 current/voltage sensor breakout
- Small generator and omnidirectional rotor assembly (see the model document)
- Load resistor or load circuit across the turbine output

## Software setup

1. Install the [Arduino IDE](https://www.arduino.cc/en/software).
2. Install the **Adafruit INA219** library (Sketch → Include Library → Manage Libraries). The EEPROM and Wire libraries ship with the IDE.
3. Wire the INA219 to the Arduino's SDA/SCL pins, power it, and place it in series with the turbine's output.
4. Open the indoor or outdoor sketch, select your board and port, and upload.
5. Open the Serial Monitor at 9600 baud.

## Serial output

Each reading prints:

```
Bus Voltage:  ... V
Shunt Voltage: ... mV
Load Voltage:  ... V
Current:       ... mA
Power:         ... mW
Calculated Power: ... mW
```

## Known limitations

- EEPROM stores one byte per reading, so values are truncated to the 0–255 range before storage and are scaled by 5 when read back.
- In the outdoor sketch, the `timer` and `cumulative_power` counters are declared but not yet incremented in `loop()`, so the averaged EEPROM write does not trigger. Adding `timer++` and `cumulative_power += calculated_power;` inside `loop()` completes the intended behavior.
- The INA219 is initialized twice in `setup()`; this is harmless but redundant.

## Results

Indoor test results are in `Omnidirectional Wind Turbine Indoor Data.pdf`.
