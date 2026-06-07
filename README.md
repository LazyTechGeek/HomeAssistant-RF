# RF in Home Assistant

[Insert description]

## Watch the video here:
▶️ [RF in Home Assistant](YouTube-Link)

## 🛒 Parts List

### RF & ESP32 only
- [ESP32 Dev Board x1 (38-pin - requires both 3.3V and 5V pins)](https://www.amazon.co.uk/dp/B0CNYM28CK) - pack of 3  
- [RF Transmitter](AMAZON LINK GOES HERE)
- [Female to Female Jumper Cables](https://www.amazon.co.uk/dp/B01EV70C78) (ELEGOO 120pcs kit - includes M-F, M-M, F-F)

### RF, IR PCB Option
- [ESP32 Dev Board x1 (38-pin - requires both 3.3V and 5V pins)](https://www.amazon.co.uk/dp/B0CNYM28CK) - pack of 3  
- [ELEGOO 32 Pcs Double Sided PCB Board Kit](https://www.amazon.co.uk/dp/B0734XYJPM) - use the 5cm x 7cm board from the kit
- [2N2222 NPN Transistor x1](https://www.amazon.co.uk/dp/B0CPBR1FGB)
- 22Ω Resistors x2 ⬎
- 1kΩ Resistor x1   [AUKENIEN 1250pcs 25 Values Resistor Assortment Kit](https://www.amazon.co.uk/dp/B09MMBPSH9)
- IR Transmitter LEDs x4 (5mm 940nm) ⬎
- VS1838B IR Receiver x1               [100PCS 50 Pairs 5mm Infrared Diode LED 940nm](YOUR_AMAZON_LINK)

## 🖨️ 3D Print Files

- [IR Hub Base](https://raw.githubusercontent.com/LazyTechGeek/HomeAssistant-Infrared/main/assets/LINK TO PART GOES HERE)
- [IR Hub Lid](https://raw.githubusercontent.com/LazyTechGeek/HomeAssistant-Infrared/main/assets/LINK TO PART GOES HERE)


## CC1101 Pinout A & B Wiring Diagrams

> [!NOTE]
> Two common CC1101 variants exist with different pin layouts. Identify your module using the images below and follow the corresponding wiring diagram.

<table>
<tr>
<td width="60%">

<img src="assets/CC1101_PinoutA_Wiring_Diagram.png" width="550">

</td>
<td width="40%" valign="top">

#### ESP32 GPIO Mapping

| Function | ESP32 Pin |
|----------|-----------|
| GND | GND |
| VCC | 3.3V |
| MOSI | GPIO23 |
| SCK | GPIO18 |
| MISO | GPIO19 |
| GDO2 | GPIO15 |
| GDO0 | GPIO4 |
| CSN | GPIO5 |

</td>
</tr>
</table>

<table>
<tr>
<td width="60%">

<img src="assets/CC1101_PinoutB_Wiring_Diagram.png" width="550">

</td>
<td width="40%" valign="top">

#### ESP32 GPIO Mapping

| Function | ESP32 Pin |
|----------|-----------|
| GND | GND |
| VCC | 3.3V |
| GDO0 | GPIO4 |
| CSN | GPIO5 |
| SCK | GPIO18 |
| MOSI | GPIO23 |
| MISO | GPIO19 |
| GDO2 | GPIO15 |

</td>
</tr>
</table>

## ESPHome YAML
## RF Only (Breadboard)
```yaml
substitutions:
  ############################################################
  # 1. DEVICE SETTINGS - change these for your own device
  ############################################################

  device_name: your-device-name
  friendly_name: Your Device Name

  api_encryption_key: "YOUR_API_ENCRYPTION_KEY"
  ota_password: "YOUR_OTA_PASSWORD"
  ap_ssid: "IR RF Hub Fallback Hotspot"
  ap_password: "YOUR_FALLBACK_PASSWORD"

  ############################################################
  # 2. SET GPIO PINS - change these to match your wiring / PCB
  ############################################################

  # SPI pins used by the CC1101 RF module
  spi_clk_pin: GPIO18
  spi_mosi_pin: GPIO23
  spi_miso_pin: GPIO19

  # CC1101 RF module pins
  cc1101_cs_pin: GPIO5
  remote_receiver_rf_pin: GPIO15
  remote_transmitter_rf_pin: GPIO4

  ############################################################
  # 3. ADVANCED RF SETTINGS - usually leave these alone
  ############################################################
  cc1101_frequency: "433.92MHz"
  cc1101_output_power: "10"
  cc1101_symbol_rate: "5000"
  cc1101_filter_bandwidth: "200kHz"

  rf_carrier_duty_percent: "100%"
  rf_receiver_tolerance: "50%"
  rf_receiver_filter: "200us"
  rf_receiver_idle: "40ms"

 ############################################################
 # End of substitutions
 # Most users should only need to edit the device settings,
 # GPIO pins, and the buttons later in this file.
 ############################################################

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}

esp32:
  board: esp32dev
  framework:
    type: arduino

logger:

api:
  encryption:
    key: ${api_encryption_key}

ota:
  - platform: esphome
    password: ${ota_password}

web_server:
  port: 80

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: ${ap_ssid}
    password: ${ap_password}

captive_portal:

globals:
  - id: learned_code
    type: std::vector<int32_t>
    restore_value: no
  - id: learning_mode
    type: bool
    initial_value: 'false'
  - id: signal_learned
    type: bool
    initial_value: 'false'

spi:
  clk_pin: ${spi_clk_pin}
  mosi_pin: ${spi_mosi_pin}
  miso_pin: ${spi_miso_pin}
  
cc1101:
  id: cc1101_module
  cs_pin: ${cc1101_cs_pin}
  frequency: ${cc1101_frequency}
  output_power: ${cc1101_output_power}
  modulation_type: ASK/OOK
  symbol_rate: ${cc1101_symbol_rate}
  filter_bandwidth: ${cc1101_filter_bandwidth}

remote_receiver:
  - id: rf_receiver
    pin: ${remote_receiver_rf_pin}
    tolerance: ${rf_receiver_tolerance}
    filter: ${rf_receiver_filter}
    dump:
      - raw
    idle: ${rf_receiver_idle}
    on_raw:
      then:
        - lambda: |-
            std::string code_str = "";
            for (int i = 0; i < x.size(); i++) {
              if (i > 0) code_str += ", ";
              code_str += std::to_string(x[i]);
            }
            ESP_LOGI("raw_code", "Pulses: %d", x.size());
            ESP_LOGI("raw_code", "Copy this line:");
            ESP_LOGI("raw_code", "[%s]", code_str.c_str());
            if (id(learning_mode)) {
              id(learned_code).clear();
              for (int32_t val : x) {
                id(learned_code).push_back(val);
              }
              id(signal_learned) = true;
              id(learning_mode) = false;
              std::string preview = "";
              int count = 0;
              for (int32_t val : id(learned_code)) {
                if (count > 0) preview += ", ";
                preview += std::to_string(val);
                count++;
                if (count >= 20) {
                  preview += "... (" + std::to_string(id(learned_code).size()) + " total)";
                  break;
                }
              }
              id(last_signal).publish_state(preview.c_str());
              id(learn_status).publish_state("Signal learned! " + std::to_string(id(learned_code).size()) + " pulses.");
            }

remote_transmitter:
  - id: rf_transmitter
    pin: ${remote_transmitter_rf_pin}
    carrier_duty_percent: ${rf_carrier_duty_percent}
    on_transmit:
      then:
        - cc1101.begin_tx: cc1101_module
    on_complete:
      then:
        - cc1101.begin_rx: cc1101_module

switch:
  - platform: template
    name: "Learning Mode"
    id: learning_mode_switch
    icon: "mdi:school"
    lambda: 'return id(learning_mode);'
    turn_on_action:
      - lambda: |-
          id(learning_mode) = true;
          id(learn_status).publish_state("Waiting for signal... Press remote now!");
      - logger.log: "Learning mode ON - press your remote"
    turn_off_action:
      - lambda: |-
          id(learning_mode) = false;
          id(learn_status).publish_state("Learning cancelled");
      - logger.log: "Learning mode OFF"

button:
  ##############################################################
  # BUILT-IN BUTTONS - do not remove these
  ##############################################################

  - platform: restart
    name: "Restart"

  - platform: template
    name: "Learn Signal"
    icon: "mdi:record-rec"
    on_press:
      - switch.turn_on: learning_mode_switch


  - platform: template
    name: "Replay Learned Signal"
    icon: "mdi:replay"
    on_press:
      - lambda: |-
          if (!id(signal_learned) || id(learned_code).empty()) {
            ESP_LOGW("replay", "No signal learned yet!");
            id(learn_status).publish_state("No signal to replay - learn one first!");
            return;
          }
          ESP_LOGI("replay", "Replaying %d pulses...", id(learned_code).size());
          id(learn_status).publish_state("Transmitting...");
      - remote_transmitter.transmit_raw:
          transmitter_id: rf_transmitter
          carrier_frequency: 0Hz
          code: !lambda 'return id(learned_code);'
      - lambda: |-
          id(learn_status).publish_state("Signal transmitted!");
          ESP_LOGI("replay", "Replay complete");


  - platform: template
    name: "Replay x3"
    icon: "mdi:replay"
    on_press:
      - lambda: |-
          if (!id(signal_learned) || id(learned_code).empty()) {
            ESP_LOGW("replay", "No signal learned yet!");
            return;
          }
      - repeat:
          count: 3
          then:
            - remote_transmitter.transmit_raw:
                transmitter_id: rf_transmitter
                carrier_frequency: 0Hz
                code: !lambda 'return id(learned_code);'
            - delay: 50ms


  - platform: template
    name: "Clear Learned Signal"
    icon: "mdi:delete"
    on_press:
      - lambda: |-
          id(learned_code).clear();
          id(signal_learned) = false;
          id(last_signal).publish_state("(empty)");
          id(learn_status).publish_state("Signal cleared");
          ESP_LOGI("learn", "Learned signal cleared");

  # ADD YOUR OWN RF BUTTONS BELOW - see README: https://github.com/LazyTechGeek/HomeAssistant-RF


text_sensor:
  ##############################################################
  # BUILT-IN TEXT SENSORS - do not remove these
  ##############################################################

  - platform: template
    name: "Last Received Signal"
    id: last_signal
    icon: "mdi:signal"

  - platform: template
    name: "Learn Status"
    id: learn_status
    icon: "mdi:information"

  # ADD YOUR OWN TEXT SENSORS BELOW

sensor:
  ##############################################################
  # BUILT-IN SENSOR - do not remove these
  ##############################################################

  - platform: template
    name: "Learned Signal Length"
    id: signal_length
    icon: "mdi:counter"
    unit_of_measurement: "pulses"
    lambda: 'return id(learned_code).size();'
    update_interval: 5s
  
  # ADD YOUR OWN SENSORS BELOW

binary_sensor:

  ##############################################################
  # BUILT-IN BINARY SENSOR - do not remove these
  ##############################################################

  - platform: template
    name: "Signal Learned"
    icon: "mdi:check-circle"
    lambda: 'return id(signal_learned);'

  # ADD YOUR OWN BINARY SENSOR SENSORS BELOW
```
## Full PCB (IR, RF, Light & Temp)
```YAML

substitutions:
  ############################################################
  # 1. DEVICE SETTINGS - change these for your own device
  ############################################################

  device_name: your-device-name
  friendly_name: Your Device Name

  api_encryption_key: "YOUR_API_ENCRYPTION_KEY"
  ota_password: "YOUR_OTA_PASSWORD"
  ap_ssid: "IR RF Hub Fallback Hotspot"
  ap_password: "YOUR_FALLBACK_PASSWORD"

  ############################################################
  # 2. SET GPIO PINS - change these to match your wiring / PCB
  ############################################################

  # SPI pins used by the CC1101 RF module
  spi_clk_pin: GPIO18
  spi_mosi_pin: GPIO23
  spi_miso_pin: GPIO19

  # CC1101 RF module pins
  cc1101_cs_pin: GPIO5
  remote_receiver_rf_pin: GPIO15
  remote_transmitter_rf_pin: GPIO4

  # IR module pins
  remote_receiver_ir_pin: GPIO27
  remote_transmitter_ir_pin: GPIO26

  # I2C pins used by BH1750 and BME280/BMP280 sensors
  i2c_sda_pin: GPIO21
  i2c_scl_pin: GPIO22

  ############################################################
  # 3. ADVANCED RF SETTINGS - usually leave these alone
  ############################################################
  cc1101_frequency: "433.92MHz"
  cc1101_output_power: "10"
  cc1101_symbol_rate: "5000"
  cc1101_filter_bandwidth: "200kHz"

  rf_carrier_duty_percent: "100%"
  rf_receiver_tolerance: "50%"
  rf_receiver_filter: "200us"
  rf_receiver_idle: "40ms"

  ############################################################
  # 4. ADVANCED IR SETTINGS - usually leave these alone
  ############################################################
  ir_carrier_duty_percent: "50%"
  ir_receiver_filter: "50us"
  ir_receiver_idle: "4ms"
  ir_receiver_input: "true"
  ir_receiver_pullup: "true"
  ir_receiver_inverted: "true"

############################################################
# End of substitutions
# Most users should only need to edit the device settings,
# GPIO pins, and the buttons later in this file.
############################################################

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}

esp32:
  board: esp32dev
  framework:
    type: arduino

logger:

api:
  encryption:
    key: ${api_encryption_key}

ota:
  - platform: esphome
    password: ${ota_password}

web_server:
  port: 80

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: ${ap_ssid}
    password: ${ap_password}

captive_portal:

globals:
  - id: learned_code
    type: std::vector<int32_t>
    restore_value: no
  - id: learning_mode
    type: bool
    initial_value: 'false'
  - id: signal_learned
    type: bool
    initial_value: 'false'

spi:
  clk_pin: ${spi_clk_pin}
  mosi_pin: ${spi_mosi_pin}
  miso_pin: ${spi_miso_pin}
  
cc1101:
  id: cc1101_module
  cs_pin: ${cc1101_cs_pin}
  frequency: ${cc1101_frequency}
  output_power: ${cc1101_output_power}
  modulation_type: ASK/OOK
  symbol_rate: ${cc1101_symbol_rate}
  filter_bandwidth: ${cc1101_filter_bandwidth}

i2c:
  # Shared I2C bus for BH1750 (light) and BME280/BMP280 (temp/pressure/humidity) 
  sda: ${i2c_sda_pin}
  scl: ${i2c_scl_pin}
  scan: true
  id: bus_a

remote_receiver:
  - id: rf_receiver
    pin: ${remote_receiver_rf_pin}
    tolerance: ${rf_receiver_tolerance}
    filter: ${rf_receiver_filter}
    dump:
      - raw
    idle: ${rf_receiver_idle}
    on_raw:
      then:
        - lambda: |-
            std::string code_str = "";
            for (int i = 0; i < x.size(); i++) {
              if (i > 0) code_str += ", ";
              code_str += std::to_string(x[i]);
            }
            ESP_LOGI("raw_code", "Pulses: %d", x.size());
            ESP_LOGI("raw_code", "Copy this line:");
            ESP_LOGI("raw_code", "[%s]", code_str.c_str());
            if (id(learning_mode)) {
              id(learned_code).clear();
              for (int32_t val : x) {
                id(learned_code).push_back(val);
              }
              id(signal_learned) = true;
              id(learning_mode) = false;
              std::string preview = "";
              int count = 0;
              for (int32_t val : id(learned_code)) {
                if (count > 0) preview += ", ";
                preview += std::to_string(val);
                count++;
                if (count >= 20) {
                  preview += "... (" + std::to_string(id(learned_code).size()) + " total)";
                  break;
                }
              }
              id(last_signal).publish_state(preview.c_str());
              id(learn_status).publish_state("Signal learned! " + std::to_string(id(learned_code).size()) + " pulses.");
            }

  - id: ir_receiver
    pin:
      number: ${remote_receiver_ir_pin}
      inverted: ${ir_receiver_inverted}
      mode:
        input: ${ir_receiver_input}
        pullup: ${ir_receiver_pullup}
    dump:
      - samsung
      - nec
      - raw
    filter: ${ir_receiver_filter}
    idle: ${ir_receiver_idle}

remote_transmitter:
  - id: rf_transmitter
    pin: ${remote_transmitter_rf_pin}
    carrier_duty_percent: ${rf_carrier_duty_percent}
    on_transmit:
      then:
        - cc1101.begin_tx: cc1101_module
    on_complete:
      then:
        - cc1101.begin_rx: cc1101_module

  - id: ir_transmitter
    pin: ${remote_transmitter_ir_pin}
    carrier_duty_percent: ${ir_carrier_duty_percent}

switch:
  - platform: template
    name: "Learning Mode"
    id: learning_mode_switch
    icon: "mdi:school"
    lambda: 'return id(learning_mode);'
    turn_on_action:
      - lambda: |-
          id(learning_mode) = true;
          id(learn_status).publish_state("Waiting for signal... Press remote now!");
      - logger.log: "Learning mode ON - press your remote"
    turn_off_action:
      - lambda: |-
          id(learning_mode) = false;
          id(learn_status).publish_state("Learning cancelled");
      - logger.log: "Learning mode OFF"

button:
  ##############################################################
  # BUILT-IN BUTTONS - do not remove these
  ##############################################################

  - platform: restart
    name: "Restart"

  - platform: template
    name: "Replay Learned Signal"
    icon: "mdi:replay"
    on_press:
      - lambda: |-
          if (!id(signal_learned) || id(learned_code).empty()) {
            ESP_LOGW("replay", "No signal learned yet!");
            id(learn_status).publish_state("No signal to replay - learn one first!");
            return;
          }
          ESP_LOGI("replay", "Replaying %d pulses...", id(learned_code).size());
          id(learn_status).publish_state("Transmitting...");
      - remote_transmitter.transmit_raw:
          transmitter_id: rf_transmitter
          carrier_frequency: 0Hz
          code: !lambda 'return id(learned_code);'
      - lambda: |-
          id(learn_status).publish_state("Signal transmitted!");
          ESP_LOGI("replay", "Replay complete");


  - platform: template
    name: "Replay x3"
    icon: "mdi:replay"
    on_press:
      - lambda: |-
          if (!id(signal_learned) || id(learned_code).empty()) {
            ESP_LOGW("replay", "No signal learned yet!");
            return;
          }
      - repeat:
          count: 3
          then:
            - remote_transmitter.transmit_raw:
                transmitter_id: rf_transmitter
                carrier_frequency: 0Hz
                code: !lambda 'return id(learned_code);'
            - delay: 50ms

  - platform: template
    name: "Clear Learned Signal"
    icon: "mdi:delete"
    on_press:
      - lambda: |-
          id(learned_code).clear();
          id(signal_learned) = false;
          id(last_signal).publish_state("(empty)");
          id(learn_status).publish_state("Signal cleared");
          ESP_LOGI("learn", "Learned signal cleared");

  # ADD YOUR OWN RF BUTTONS BELOW - see README: https://github.com/LazyTechGeek/HomeAssistant-RF

text_sensor:
  ##############################################################
  # BUILT-IN TEXT SENSORS - do not remove these
  ##############################################################

  - platform: template
    name: "Last Received Signal"
    id: last_signal
    icon: "mdi:signal"

  - platform: template
    name: "Learn Status"
    id: learn_status
    icon: "mdi:information"

  # ADD YOUR OWN TEXT SENSORS BELOW

sensor:
  ##############################################################
  # BUILT-IN SENSOR - do not remove these
  ##############################################################

  - platform: template
    name: "Learned Signal Length"
    id: signal_length
    icon: "mdi:counter"
    unit_of_measurement: "pulses"
    lambda: 'return id(learned_code).size();'
    update_interval: 5s

  ######################
  # LIGHT SENSOR: BH1750
  ######################

  - platform: bh1750
    name: "BH1750 Illuminance"
    address: 0x23
    update_interval: 60s

  ###############################################################
  # TEMP SENSOR: Use ONE only - comment out the other
  ###############################################################

  # OPTION 1: BME280 - includes humidity (recommended)
  - platform: bme280_i2c
    address: 0x76   # or 0x77 depending on SDO
    temperature:
      name: "BME280 Temperature"
    pressure:
      name: "BME280 Pressure"
    humidity:
      name: "BME280 Humidity"

  # OPTION 2: BMP280 - no humidity (comment out OPTION 1 above first)
  # - platform: bmp280_i2c
  #   address: 0x76   # or 0x77 depending on SDO
  #   temperature:
  #     name: "BMP280 Temperature"
  #   pressure:
  #     name: "BMP280 Pressure"

  # ADD YOUR OWN SENSORS BELOW

  
binary_sensor:
  ##############################################################
  # BUILT-IN BINARY SENSOR - do not remove these
  ##############################################################

  - platform: template
    name: "Signal Learned"
    icon: "mdi:check-circle"
    lambda: 'return id(signal_learned);'

  # ADD YOUR OWN BINARY SENSOR SENSORS BELOW
```
    lambda: 'return id(signal_learned);'
```
