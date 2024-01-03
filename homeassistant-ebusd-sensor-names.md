When using EBusD via MQTT, i got a lot of cryptic-named entities in Home Assistant.
In the following table i provided more descriptive entity names.


| Entity id                             | Entity name     | Icon           |
|---------------------------------------|------------------|----------------|
| binary_sensor.ebusd_bai_hwcdemand_yesno | Warm-water vraag | mdi:hand-water |
| sensor.ebusd_bai_flame                  | CV brander       | mdi:fire |
| sensor.ebusd_bai_gasvalveasicfeedback   | CV gas-klep terugkoppeling |   |
| sensor.ebusd_bai_gasvalve3uc            | CV gas-klep activeer 3 signaal | mdi:ray-start  |
| sensor.ebusd_bai_gasvalve               | CV gas-klep actief | mdi:valve |
| binary_sensor.ebusd_bai_fluegasvalve_onoff | CV rookgasklep actief | mdi:valve |
| sensor.ebusd_bai_status01_pumpstate | CV pomp actief | mdi:pump |
| binary_sensor.ebusd_bai_extwp_onoff  | CV externe verwarming pomp actief | mdi:pump |
| sensor.ebusd_bai_templimiter | | |
| sensor.ebusd_bai_statenumber | | |
| sensor.ebusd_bai_status | | |
| sensor.ebusd_bai_pumphours_hoursum2 | CV uren pomp actief | mdi:pump |
| sensor.ebusd_bai_setmode_disablehc | CV verwarming uitgeschakeld | mdi:radiator-off | 
| sensor.ebusd_bai_status02_hwcmode | Warm-water circuit geactiveerd | mdi:hand-water |
| sensor.ebusd_bai_averageignitiontime | CV gemiddelde ontbrandingstijd | mdi:fire |
| sensor.ebusd_bai_blocktimehcmax_minutes0 | CV maximale wachttijd | mdi:timer-sand |
| binary_sensor.ebusd_bai_externalfaultmessage_onoff | CV extern fout bericht | |
| binary_sensor.ebusd_bai_extwp_onoff | CV externe verwarming pomp actief | mdi:pump |
| sensor.ebusd_bai_hcpumpmode | CV verwarming pomp modus | mdi:pump |
| sensor.ebusd_bai_hwcwaterflowmax_uin100 | Warm-water circuit maximale doorstroming | |
| number.ebusd_bai_partloadhckw_power | Deellast verwarming | |
| number.ebusd_bai_partloadhwckw_power | Deellast warm-water | |
| sensor.ebusd_bai_positionvalveset | CV gas-klep positie | mdi:valve-open |
| sensor.ebusd_bai_remainingboilerblocktime_minutes0 | Resterende CV wachttijd | |
| sensor.ebusd_bai_targetfanspeed | CV gewenste ventilator snelheid | mdi:fan |
| sensor.ebusd_bai_targetfanspeedoutput | CV gerealiseerde ventilator doel snelheid | mdi:fan |
| sensor.ebusd_bai_waterpressure_press | Verwarming waterdruk | |
| sensor.ebusd_bai_wp_onoff | Waterpomp CV |
