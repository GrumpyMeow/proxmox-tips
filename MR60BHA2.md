

So i just received from Seeedstudio the [MR60BHA2](https://www.seeedstudio.com/MR60BHA2-60GHz-mmWave-Sensor-Breathing-and-Heartbeat-Module-p-5945.html) mmwave sensor for Breathing and Heartbeat detection.
It comes with pre-flashed with an ESPHome based firmware.

# Fork of original repository
I created [my own fork](https://github.com/grumpymeow/MR60BHA2_ESPHome_external_components) of the [original repository](https://github.com/limengdu/MR60BHA2_ESPHome_external_components).
This to resolve a few issues.

## Hard to upload firmware
At often times it was quite hard to upload a new firmware to the device over-the-air. Quite often no connection could be made from ESPHome to the device. Partly this is caused by the poor wireless connection the device had. 
I had to resort to using ESPTool and flash over-the-wire. I eventually added a `yield()` to `loop` function to give ensure the CPU would give more time to other processes. This resolved all timeout issues.
```
// main loop
void MR60BHA2Component::loop() {
  uint8_t byte;

  // Is there data on the serial port
  while (this->available()) {
    this->read_byte(&byte);
    this->rx_message_.push_back(byte);
    if (!this->validate_message_()) {
      this->rx_message_.clear();
    }
  }
  yield();       <= added this
}
```

## More frame types
At this time 3 time of frame types with data can be processed by the code, but the device itself transmits two additional frame types. In code I added these event-types. Although i didn't bother to implement the decoding as i saw no interesting information.
One of the frame-types transmitted the text `no targets!!!`. 
```
static const uint16_t UNKNOWN_TYPE_BUFFER = 0x0A13;    <== Added this
static const uint16_t UNKNOWN2_TYPE_BUFFER = 0x0100;    <== Added this
static const uint16_t BREATH_RATE_TYPE_BUFFER = 0x0A14;
static const uint16_t HEART_RATE_TYPE_BUFFER = 0x0A15;
static const uint16_t DISTANCE_TYPE_BUFFER = 0x0A16;
```
and
```
void MR60BHA2Component::process_frame_(uint16_t frame_id, uint16_t frame_type, const uint8_t *data, size_t length) {
  switch (frame_type) {
    case BREATH_RATE_TYPE_BUFFER:
      ...
      break;
    case HEART_RATE_TYPE_BUFFER:
      ...
      break;
    case DISTANCE_TYPE_BUFFER:
      ...
      break;
    case UNKNOWN_TYPE_BUFFER:
      break;
    case UNKNOWN2_TYPE_BUFFER:
      break;
    default:
      break;
  }
}
```

# No distance data
In the current version of the repository there is an error in the decoding of distance. 
```
    case DISTANCE_TYPE_BUFFER:
      if (data[0] != 0) {      <== changed
        if (this->distance_sensor_ != nullptr && length >= 8) {
          uint32_t current_distance_int = encode_uint32(data[7], data[6], data[5], data[4]);
          float distance_float;
          memcpy(&distance_float, &current_distance_int, sizeof(float));
          this->distance_sensor_->publish_state(distance_float);
        }
      }
```


# YAML-definition

I currently am using this Yaml-definition. In this definition i added template-sensors for heartrate and breathingrate which expose the minimum, maximum and average values.
For all the sensors i'd added the `timeout`-filter to make the sensor `unavailable` in Home Assistant when nothing is detected for a while. 
I also added the `clamp` filter to get rid of noise values.
```
substitutions:
  name: "slaapsensor"
  friendly_name: "Slaapsensor"
  timeout: 120s
  hrf_name: "Hartslagfrequentie"
  brf_name: "Ademfrequentie"

esphome:
  name: "${name}"
  friendly_name: "${friendly_name}"
  # project:
  #   name: "seeedstudio.mr60bha2_kit"
  #   version: "2.0"
  platformio_options:
    board_upload.maximum_size: 4194304
  min_version: "2024.3.2"

esp32:
  board: esp32-c6-devkitc-1
  variant: esp32c6
  flash_size: 4MB
  framework:
    type: esp-idf
    sdkconfig_options:
      CONFIG_ESPTOOLPY_FLASHSIZE_4MB: y

# Enable logging
logger:
  hardware_uart: USB_SERIAL_JTAG
  #level: VERY_VERBOSE
  level: DEBUG

api:
  reboot_timeout: 0s

ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: false
  enable_btm: true
  enable_rrm: true
  power_save_mode: none
  reboot_timeout: 120min
  ap:
    ssid: "Slaapsensor hotspot"
    password: !secret wifi_password
    ap_timeout: 5min

light:
  - platform: esp32_rmt_led_strip
    id: led_ring
    name: "Led"
    pin: GPIO1
    num_leds: 1
    rmt_channel: 0
    rgb_order: GRB
    chipset: ws2812

i2c:
 sda: GPIO22
 scl: GPIO23
 scan: true
 # frequency: 200kHz
 id: bus_a

uart:
  id: uart_bus
  baud_rate: 115200
  rx_pin: 17
  tx_pin: 16
  parity: NONE
  stop_bits: 1

seeed_mr60bha2:
  id: my_seeed_mr60bha2

sensor:
  - platform: bh1750
    name: "Omgevingslicht"
    address: 0x23
    update_interval: 60s
    filters:
    - timeout: ${timeout}
  - platform: seeed_mr60bha2
    breath_rate:
      id: brf_raw
      name: "${brf_name} ruw"
      unit_of_measurement: bpm    
      internal: false
      accuracy_decimals: 2
      filters:
      - clamp:
          min_value: 1
          max_value: 40
          ignore_out_of_range: true
      - timeout: ${timeout}    
      on_value:
        then:
          - component.update: brf_avg
          - component.update: brf_min
          - component.update: brf_max
    heart_rate:
      id: hrf_raw
      name: "${hrf_name} ruw"
      unit_of_measurement: bpm    
      internal: false
      accuracy_decimals: 2
      filters:
      - clamp:
          min_value: 40
          max_value: 130
          ignore_out_of_range: true
      - timeout: ${timeout}
      on_value:
        then:
          - component.update: hrf_avg
          - component.update: hrf_min
          - component.update: hrf_max
    distance:
      name: "Afstand"
      accuracy_decimals: 2
      filters:
      - exponential_moving_average:
          alpha: 0.1
          send_every: 1
      - timeout: ${timeout}

  # Hartslagfrequentie
  - platform: template
    id: hrf_avg
    name: "${hrf_name}"
    accuracy_decimals: 2
    unit_of_measurement: bpm    
    lambda: return id(hrf_raw).state;
    filters:
      - exponential_moving_average:
          alpha: 0.1
          send_every: 1
      - timeout: ${timeout}   
  - platform: template
    id: hrf_min
    name: "${hrf_name} min"
    accuracy_decimals: 2
    unit_of_measurement: bpm   
    lambda: return id(hrf_raw).state;
    filters:
      - min:
          window_size: 5
          send_every: 1
      - timeout: ${timeout}  
  - platform: template
    id: hrf_max
    name: "${hrf_name} max"
    accuracy_decimals: 2
    unit_of_measurement: bpm
    lambda: return id(hrf_raw).state;
    filters:
      - max:
          window_size: 5
          send_every: 1
      - timeout: ${timeout}  

  # Ademfrequentie
  - platform: template
    id: brf_avg
    name: "${brf_name}"
    accuracy_decimals: 2
    unit_of_measurement: bpm
    lambda: return id(brf_raw).state;
    filters:
      - exponential_moving_average:
          alpha: 0.1
          send_every: 1
      - timeout: ${timeout}   
  - platform: template
    id: brf_min  
    name: "${brf_name} min"
    accuracy_decimals: 2
    unit_of_measurement: bpm    
    lambda: return id(brf_raw).state;
    filters:
      - min:
          window_size: 5
          send_every: 1
      - timeout: ${timeout}  
  - platform: template
    id: brf_max
    name: "${brf_name} max"
    accuracy_decimals: 2
    unit_of_measurement: bpm
    lambda: return id(brf_raw).state;
    filters:
      - max:
          window_size: 5
          send_every: 1
      - timeout: ${timeout}  


external_components:
  - source:
      type: git
      url: https://github.com/grumpymeow/MR60BHA2_ESPHome_external_components
      ref: main
    components: [ seeed_mr60bha2 ]
    refresh: 0s
```

# Results

## 18-12-2024

### Distance detected object
The result of the measured distance looks accurate. It detected some tossing and turning of me during my sleep.
![image](https://github.com/user-attachments/assets/ce65fdb6-d4a6-48c8-84fe-dadbf923b26a)

### Light sensor
The result of the measured light looks accurate. At 6:45 a bedside lamp is turned on. From 8:40 and later it's daylight.
![image](https://github.com/user-attachments/assets/cc2c501f-ea02-439d-bf1f-bbe9da8f8bfa)


### Breathing rate
This is a graph of the raw sensor data. The variation is quite big and hopefully inaccuracy. This as otherwise i might have a severe case of slaap apnea. 
![image](https://github.com/user-attachments/assets/87ac53d9-36f6-45ee-911e-d9aea22a28b8)

I also do some filtering on the sensor data which results in this chart:
![image](https://github.com/user-attachments/assets/d0fe9463-e956-4fd7-be95-2454ff7e43db)

I also expose the min and max of the sensor data. 
![image](https://github.com/user-attachments/assets/1859197a-a593-43ce-ab32-ef5daba49b54)


### Heart rate
This is the chart of the raw and filtered sensor data. The brownish-line is of the raw data. Blue is of the filtered data.
![image](https://github.com/user-attachments/assets/74a245cd-bb11-4d0a-a899-cf0708e1c05a)

This is the chart of the min and max sensor data.
![image](https://github.com/user-attachments/assets/72adb0f6-7574-4c2c-a35b-60f2f0264a6b)

### Conclusion
My conclusion at this time is that i need to increase the average over a longer period. As the variation makes it impossible to visually see anything usefull.

Current filter settings:
```
      - exponential_moving_average:
          alpha: 0.1
          send_every: 1      (default=15, i might have used a weird value here)
          send_first_at: 1  <=default

      - max/min:
          window_size: 5
          send_every: 1
```

My new filter settings i'm gonna try:
```
    - quantile:
        window_size: 7        (default=7)
        send_every: 4
        send_first_at: 3
        quantile: .9

      - max/min:
          window_size: 20
          send_first_at: 1
          send_every: 1
```
I also chose to increase the timeout from `timeout: 120s` to `timeout: 600s` to get rid of some small gaps in the data. 

There seem to be a [few patterns](https://ouraring.com/blog/sleeping-heart-rate/) which can be seen during sleeping.  
It would be fun to eventually see these patterns arise.

Looking at the data i already suspect that there might not be enough of data to really get usefull insights. 
Maybe just like on the Oura-ring page, only three-point-patterns like hammock, slope, hill, etc. We will see. 

I noticed the device crashes on a regular basis which is recognizable by the short lighting of the led. This distorts the filters. So probably better to implement the filters in Home Assistant instead of on the device.
