[README (4).md](https://github.com/user-attachments/files/33834783/README.4.md)
# Smart Plant Watering System (ESP32, Wokwi)

An automatic plant watering controller built on an ESP32 and simulated in [Wokwi](https://wokwi.com). It reads soil moisture and tank level, switches a pump through a relay using hysteresis control, and protects itself with a run-time limit and an empty-tank lockout. Status is shown on an OLED display and logged over Serial as CSV.

**Live simulation:** [Open in Wokwi](YOUR-WOKWI-LINK-HERE)

![Circuit](images/circuit.png)

## Features

- **Hysteresis control:** the pump turns on below 30% moisture and off at 60%, so it doesn't flicker around a single threshold.
- **Safety timeout:** if the pump runs for more than 10 seconds without the soil reaching the stop level, it shuts off and raises a fault.
- **Empty-tank lockout:** the pump is blocked when the tank level is below 10%.
- **Audible alert:** the buzzer beeps during a fault or an empty tank.
- **Manual override:** a push button runs the pump for 5 seconds. During a fault, it clears the fault instead.
- **OLED status screen:** moisture, tank level, air temperature and humidity, pump state and system status.
- **CSV logging:** one line per second over Serial, ready for plotting.
- **Non-blocking code:** all timing uses `millis()`, so the button and safety checks stay responsive.

## Hardware (simulated)

| Part | Wokwi element | Purpose |
|---|---|---|
| ESP32 DevKit | ESP32 | Main controller |
| Potentiometer | Soil moisture (simulated) | Analog moisture reading |
| Potentiometer | Tank level (simulated) | Analog tank level reading |
| DHT22 | Temperature and humidity sensor | Air readings |
| Relay module | 1-channel relay | Switches the pump |
| LED + 220 Ω resistor | Pump indicator | Stands in for the pump |
| Buzzer | Passive buzzer | Alerts |
| Push button | Pushbutton | Manual watering and fault reset |
| SSD1306 OLED | 128x64 I2C display | Status screen |

> Wokwi has no soil moisture sensor, so a potentiometer stands in for it. On real hardware, replace it with a capacitive soil moisture sensor on the same ADC pin and recalibrate the mapping.

## Pin connections

| Component | Component pin | ESP32 pin |
|---|---|---|
| Soil moisture (pot) | SIG | GPIO 34 |
| Tank level (pot) | SIG | GPIO 35 |
| DHT22 | DATA | GPIO 15 |
| Relay module | IN | GPIO 26 |
| Buzzer | Positive (+) | GPIO 27 |
| Push button | One leg | GPIO 25 (other leg to GND, `INPUT_PULLUP`) |
| OLED SSD1306 | SDA | GPIO 21 |
| OLED SSD1306 | SCL | GPIO 22 |

Both potentiometers, the DHT22 and the OLED are powered from 3V3 and GND. The relay module is powered from VIN (5V) and GND. The pump LED is connected from the relay's NO output through a 220 Ω resistor to GND, with COM on 3V3.

## How it works

```
read sensors -> decide -> act -> display -> log
```

1. **Read:** moisture and tank level are mapped from the 0-4095 ADC range to 0-100%.
2. **Decide:** hysteresis thresholds start and stop the pump; safety checks can override it.
3. **Act:** the relay switches the pump, and the buzzer sounds during alerts.
4. **Display and log:** the OLED refreshes every 500 ms and a CSV line is printed every second.

### Settings

| Setting | Value |
|---|---|
| Start watering below | 30% moisture |
| Stop watering at | 60% moisture |
| Tank empty below | 10% |
| Auto-run timeout | 10 s |
| Manual run time | 5 s |

All values are constants at the top of `main.ino`.

## Libraries

- Adafruit GFX Library
- Adafruit SSD1306
- DHT sensor library

## Running it

1. Open the [Wokwi project](YOUR-WOKWI-LINK-HERE) and press play, or create a new ESP32 project and copy in `main.ino`, `diagram.json` and `libraries.txt`.
2. Click a potentiometer to change its value while the simulation runs.
3. Open the Serial Monitor to see the CSV output.

## Test cases

| # | Test | Action | Expected result |
|---|---|---|---|
| 1 | Auto watering | Set moisture below 30% | Pump turns on, OLED shows `ON (AUTO)` |
| 2 | Hysteresis | Raise moisture to about 45% | Pump stays on |
| 3 | Stop level | Raise moisture to 60% or above | Pump turns off |
| 4 | Timeout fault | Keep moisture low for more than 10 s | Pump stops, `! PUMP FAULT`, buzzer beeps |
| 5 | Fault reset | Press the button | Fault clears and the buzzer stops |
| 6 | Empty tank | Set tank below 10% | `! TANK EMPTY`, buzzer beeps, pump blocked |
| 7 | Manual watering | Moisture high, tank above 10%, press button | Pump runs for 5 s |
| 8 | Manual blocked | Tank below 10%, press button | Pump does not start |

![Normal operation](images/normal.png)
![Tank empty alert](images/tank-empty.png)

## Serial output format

```
time_ms,moisture,tank,temp_c,humidity,pump
1000,45,78,24.0,40,0
```

The last column is `1` while the pump is running.

## Possible improvements

- A web dashboard hosted on the ESP32 with a "Water now" button.
- Cloud logging over MQTT or ThingSpeak.
- A Python script to plot the logged CSV data.
- Real hardware version with a capacitive soil moisture sensor and a water pump driven through the relay.

## What I learned

- Hysteresis control and why a single threshold causes chatter.
- Non-blocking timing with `millis()`.
- Designing safety conditions for an actuator.
- Driving a passive buzzer with `tone()`.
- Simulating an embedded system in Wokwi.

## License

MIT
