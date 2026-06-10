# RF in Home Assistant

[Insert description]

## Watch the video here:
▶️ [RF in Home Assistant](YouTube-Link)

## 🛒 Parts List

> As an Amazon Associate I earn from qualifying purchases at no extra cost to you.

### RF & ESP32 ONLY

- [ESP32 Dev Board x1 (38-pin, requires both 3.3V and 5V pins)](https://www.amazon.co.uk/dp/B0CNYM28CK?tag=lazytechgeekshop-21) *(pack of 3)*
- [Female-to-Female Jumper Wires](https://www.amazon.co.uk/dp/B0BLZC3SQ2?tag=lazytechgeekshop-21) *(40-piece kit, 10cm)*
- **CC1101 433MHz RF Module (Choose One)**
  - [Green PCB Version (Pinout A)](https://www.amazon.co.uk/dp/B09WKGYFPV?tag=lazytechgeekshop-21)
  - [Blue PCB Version (Pinout B)](https://www.amazon.co.uk/dp/B0DQL117RW?tag=lazytechgeekshop-21)

### RF, IR PCB Option
- [Carrier PCB](https://www.pcbway.com/project/shareproject/IR_RF_Hub_c6b0235f.html)
- [ESP32 Dev Board x1 (38-pin, requires both 3.3V and 5V pins)](https://www.amazon.co.uk/dp/B0CNYM28CK?tag=lazytechgeekshop-21) *(pack of 3)*
- **CC1101 433MHz RF Module (Choose One)**
  - [Green PCB Version (Pinout A)](https://www.amazon.co.uk/dp/B09WKGYFPV?tag=lazytechgeekshop-21)
  - [Blue PCB Version (Pinout B)](https://www.amazon.co.uk/dp/B0DQL117RW?tag=lazytechgeekshop-21)
- [2N2222 NPN Transistor x1](https://www.amazon.co.uk/dp/B0CPBR1FGB?tag=lazytechgeekshop-21)
- 22Ω Resistors x2
- 1kΩ Resistor x1
  - [Resistor Assortment Kit](https://www.amazon.co.uk/dp/B09MMBPSH9?tag=lazytechgeekshop-21)
- IR Transmitter LEDs x4 (5mm, 940nm)
- VS1838B IR Receiver x1
  - [IR LED & Receiver Kit](https://www.amazon.co.uk/dp/B09NN5VXHX?tag=lazytechgeekshop-21)

## 🖨️ 3D Print Files
- [IR RF Hub Enclosure Base](https://raw.githubusercontent.com/LazyTechGeek/HomeAssistant-RF/main/assets/ir_rf_hub_enclosure_base.3mf)
- [IR RF Hub Enclosure Led](https://raw.githubusercontent.com/LazyTechGeek/HomeAssistant-RF/main/assets/ir_rf_hub_enclosure_lid.3mf)
- [IR LED Guide](https://raw.githubusercontent.com/LazyTechGeek/HomeAssistant-RF/main/assets/ir_led_guide.3mf)
- [IR Receiver Guide](https://raw.githubusercontent.com/LazyTechGeek/HomeAssistant-RF/main/assets/ir_receiver_guide.3mf)

## 🧩 PCB Files
- [Carrier PCB V1.0](https://raw.githubusercontent.com/LazyTechGeek/HomeAssistant-RF/main/assets/carrier_pcb_v1.0_gerbers.zip)
## PCB
<table align="center">
<tr>
<td>
<img src="assets/carrier_pcb_ir_rf_hub_pcb_top_render.png" width="300">
</td>
<td>
<img src="assets/carrier_pcb_ir_rf_hub_pcb_assembled.png" width="300">
</td>
</tr>
</table>

## 3D Printed Enclosure

<table align="center">
<tr>
<td>
<img src="assets/carrier_pcb_ir_rf_hub_pcb_enclosure_assembled.png" width="300">
</td>
<td>
<img src="assets/carrier_pcb_ir_rf_hub_pcb_enclosure_closed.png" width="300">
</td>
</tr>
</table>

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
  ############################################################

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}

esp32:
  board: esp32dev
  framework:
    type: esp-idf

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
    idle: ${rf_receiver_idle}
    dump: all

remote_transmitter:
  - id: rf_transmitter
    pin: ${remote_transmitter_rf_pin}
    carrier_duty_percent: ${rf_carrier_duty_percent}
    non_blocking: true
    on_transmit:
      then:
        - cc1101.begin_tx: cc1101_module
    on_complete:
      then:
        - cc1101.begin_rx: cc1101_module

button:
  ##############################################################
  # BUILT-IN BUTTONS - do not remove these
  ##############################################################

  - platform: restart
    name: "Restart"

  # ADD YOUR OWN RF BUTTONS BELOW - see README: https://github.com/LazyTechGeek/HomeAssistant-RF
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
  ir_receiver_idle: "10ms"
  ir_receiver_input: "true"
  ir_receiver_pullup: "true"
  ir_receiver_inverted: "true"
  ir_receiver_buffer_size: "10kb"
  ir_receiver_tolerance: "25%"

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
    type: esp-idf

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
    idle: ${rf_receiver_idle}
    dump: all   # see README for full protocol list and how to filter
    # https://github.com/LazyTechGeek/HomeAssistant-RF

  - id: ir_receiver
    pin:
      number: ${remote_receiver_ir_pin}
      inverted: ${ir_receiver_inverted}
      mode:
        input: ${ir_receiver_input}
        pullup: ${ir_receiver_pullup}
    filter: ${ir_receiver_filter}
    idle: ${ir_receiver_idle}
    buffer_size: ${ir_receiver_buffer_size}
    tolerance: ${ir_receiver_tolerance}    
    dump: all    # see README for full protocol list and how to filter
    # https://github.com/LazyTechGeek/HomeAssistant-IR

remote_transmitter:
  - id: rf_transmitter
    pin: ${remote_transmitter_rf_pin}
    carrier_duty_percent: ${rf_carrier_duty_percent}
    non_blocking: true
    on_transmit:
      then:
        - cc1101.begin_tx: cc1101_module
    on_complete:
      then:
        - cc1101.begin_rx: cc1101_module

  - id: ir_transmitter
    pin: ${remote_transmitter_ir_pin}
    carrier_duty_percent: ${ir_carrier_duty_percent}
    non_blocking: true

button:
  ##############################################################
  # BUILT-IN BUTTONS - do not remove these
  ##############################################################

  - platform: restart
    name: "Restart"

  # ADD YOUR OWN RF BUTTONS BELOW - see README: https://github.com/LazyTechGeek/HomeAssistant-RF

sensor:
  ##########################################
  # LIGHT SENSOR: BH1750 - remove if unused
  ##########################################

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
```
