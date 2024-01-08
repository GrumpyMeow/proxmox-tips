When using EBusD via MQTT, i got a lot of cryptic-named entities in Home Assistant.
In the following table i provided more descriptive entity names for the Diagnose-fields.


| D-code | Default sensor Entity id     | Given entity name     | Icon           | Remark |
|--------|------------------------------|------------------|----------------|-------|
|D.000| ebusd_bai_partloadhckw_power         | CV-deellast (D.000) | | |
|D.001| ebusd_bai_wppostruntime_minutes0     | Nalooptijd interne pomp voor CV-bedrijf (D.001) | | |
|D.002| ebusd_bai_blocktimehcmax_minutes0    | Maximum branderwachttijd verwarming (D.002) | | |
|D.003| ebusd_bai_hwctemp_temp               | Warmwater temperatuur gemeten (D.003) | | |
|D.004| ebusd_bai_storagetemp_temp           | Meetwaarde van de warmwatersensor (D.004) | | |
|D.005| ebusd_bai_flowtempdesired_temp       | Gewenste aanvoertemperatuur (D.005) | | |
|D.006| ebusd_bai_hwctempdesired_temp        | Gewenste waarde warmwatertemperatuur (D.006) | | |
|D.007| ebusd_bai_storagetempdesired_temp    | Gewenste waarde warmestarttemperatuur (D.007) | | |
|D.008| # niets | | |
|D.009| ebusd_bai_extflowtempdesiredmin_temp | Gewenste waarde van externe eBUS thermostaat (D.009) | | |
|D.010| ebusd_bai_wp_onoff                    | Status interne pomp (D.010) | | |
|D.011| ebusd_bai_extwp_onoff                 | Status externe CV-pomp (D.011) | mdi:pump | |
|D.012| ebusd_bai_storageloadpump_percent0    | Status boilerlaadpomp (D.012) | | |
|D.013| ebusd_bai_cirpump_onoff               | Status warmwater - circulatiepomp (D.013) | | |
|D.013| ebusd_bai_statuscirpump               | Status warmwater - circulatiepomp (D.013) | | |
|D.014| ebusd_bai_pumppowerdesired            | Pomp kracht ingesteld (D.014) | | |
|D.015| ebusd_bai_pumppower                   | Pomp kracht werkelijke waarde (D.015) | | |
|D.016| DCRoomthermostat                      | Kamerthermostaat 24V DC (D.016) | | |
|D.017| ebusd_bai_returnregulation_onoff      | Retour temperatuurregeling verwarming (D.017) | | |
|D.018| ebusd_bai_hcpumpmode                  | Instelling van de pompmodus (D.018) | mdi:pump | |
|D.019| ebusd_bai_secondpumpmode              | Modus van de 2-traps pomp  (D.019) | | |
|D.020| ebusd_bai_hwctempmax_temp             | Max. instelwaarde voor gewenste boilerwaarde (D.020) | | |
|D.021| 
|D.022| ebusd_bai_hwcdemand_yesno             | Vraag warm water (D.022) | mdi:hand-water | |
|D.023| ebusd_bai_heatingswitch_onoff         | Verwarming geactiveerd (D.023) | | |
|D.024| 
|D.025| ebusd_bai_storagereleaseclock_yesno   | Warmwaterbereiding vrijgegeven door eBus-thermostaat (D.025) | | |
|D.026|                                       | Aansturing hulprelais  | | |
|D.027| ebusd_bai_accessoriesone              | Omschakeling relais 1 (D.027)  | | Decode error |
|D.028| ebusd_bai_accessoriestwo              | Omschakeling relais 2 (D.028)  | | Decode error |
|D.029| 
|D.030| 
|D.031| 
|D.032| 
|D.033| ebusd_bai_targetfanspeed                | Gewenste waarde ventilatortoerental (D.033) | mdi:fan | |
|D.034| ebusd_bai_fanspeed                      | Actuele waarde ventilatortoerental (D.034) | | |
|D.035| ebusd_bai_positionvalveset              | Stand van de driewegklep (D.035) | mdi:valve-open | |
|D.036| ebusd_bai_hwcwaterflow_uin100           | Warmwaterdebiet (D.036) | | |
|D.037| 
|D.038| 
|D.039|                                         | Zonne-inlooptemperatuur (D.039)  | | |
|D.040| ebusd_bai_flowtemp_temp                 | Aanvoertemperatuur (D.040) | | |
|D.041| ebusd_bai_returntemp_temp               | Retourtemperatuur (D.041) | | |
|D.042| 
|D.043| 
|D.044| ebusd_bai_ionisationvoltagelevel        | Gedigitaliseerde ionisatiewaarde (D.044) | | |
|D.045| 
|D.046|                                         | Soort pomp 0 (D.046) | | |
|D.047| ebusd_bai_outdoorstempsensor_temp       | Buitentemperatuur (D.047) | | |
|D.048|
|D.049|
|D.050| ebusd_bai_fanspeedoffsetmin             | Offset voor minimaal toerental (D.050) | | |
|D.051| ebusd_bai_fanspeedoffsetmax             | Offset voor maximaal toerental (D.051) | | |
|D.052|
|D.053|
|D.054|
|D.055|
|D.056|
|D.057|
|D.058| ebusd_bai_solpostheat                  | Activering naverwarming via zonneenergie voor combiproduct (D.058) | | |
|D.059|
|D.060| ebusd_bai_deactivationstemplimiter     | Aantal uitschakelingen door temperatuurbegrenzer (D.060) | | |
|D.061| ebusd_bai_deactivationsifc             | Aantal storingen branderautomaat (D.061) | | |
|D.062|
|D.063|
|D.064| ebusd_bai_averageignitiontime          | Gemiddelde ontstekingstijd (D.064) | mdi:fire | |
|D.065| ebusd_bai_maxignitiontime              | Maximale ontstekingstijd (D.065) | | |
|D.066|
|D.067| ebusd_bai_remainingboilerblocktime_minutes0   |  Resterende branderwachttijd (D.067) | | |
|D.068| ebusd_bai_counterstartattempts1_temp0    | Mislukte ontstekingen bij 1e poging (D.068) | | |
|D.069| ebusd_bai_counterstartattempts2_temp0    | Mislukte ontstekingen bij 2e poging (D.069) | | |
|D.070| ebusd_bai_valvemode                      | Instellen stand driewegklep (D.070) | | |
|D.071| ebusd_bai_flowsethcmax_temp              | Gewenste waarde max. aanvoertemperatuur verwarming (D.071) | | |
|D.072| ebusd_bai_hwcpostruntime                 | Nalooptijd interne pomp na boilerlading (D.072) | | |
|D.073| ebusd_bai_warmstartoffset_temp           | Gewenste warme start offset (D.073) | | |
|D.074| ebusd_bai_apclegioprotection             | Legionellabeveiligingsfunctie actoSTOR (D.074) | | Geen actoSTOR aanwzg  |
|D.075| ebusd_bai_storageloadtimemax_minutes0    | Max. laadtijd voor warmwaterboiler zonder eigen regeling (D.075)  | | |
|D.076|                                   	   | Toestelidentificatie (D.076) | | |
|D.077| ebusd_bai_partloadhwckw_power            | Begrenzing van het boilerlaadvermogen in kW (D.077) | | Decode error |
|     | ebusd_bai_modulationtempdesired			   | Modulatie temperatuur gevraagd  | |  ebusd_bai_partloadhwckw_power = % van partloadhcKW |
|D.078| ebusd_bai_flowsethwcmax_temp             | Begrenzing van de boilerlaadtemperatuur in °C (D.078)  | | |
|D.079|
|D.080| ebusd_bai_hchours_hoursum2               | Bedrijfsuren verwarming (D.080)  | | |
|D.081| ebusd_bai_hwchours_hoursum2              | Bedrijfsuren warmwaterbereiding (D.081)  | | |
|D.082| ebusd_bai_hcstarts                       | Aantal branderstarts in CV-bedrijf (D.082)  | | |
|D.083| ebusd_bai_hwcstarts                      | Aantal branderstarts in warmwaterbedrijf (D.083)  | | |
|D.084| ebusd_bai_hourstillservice_hoursum2      | Onderhoudsindicatie: aantal uren tot de volgende onderhoudsbeurt (D.084)  | | |
|D.085|
|D.086|
|D.087|
|D.088| ebusd_bai_specialadj                     | Inschakelvertraging voor warmwatertapherkenning via vleugelwiel (D.088)  | | # werkt niet |
|D.089|
|D.090| ebusd_bai_ebusheatcontrol_yesno       | Status digitale thermostaat (D.090) | | |
|D.091| thermostaat_statusdcf_dcfstate        | Status DCF bij aangesloten buitentemperatuurvoeler (D.091) | | |
|D.092| ebusd_bai_apccomstatus                  | actoSTOR moduleherkenning (D.092) | | |
|D.093| ebusd_bai_dsnoffset                      | Instelling toestelidentificatie (D.093) | | |
|D.094|                                   | Foutcode historie verwijderen (D.094) | | |
|D.095|                                   | Softwareversie PeBUS-componenten Printplaat (D.095) | | |
|D.096|                                   | Fabrieksinstelling Reset (D.096) | | |
|D.097| ebusd_bai_password                | Activering installateurniveau Code (D.097) | | werkt niet |
|D.098|                                   | Waarde van de codeerweerstanden (D.098) |


| Other        | Name       | Icon     |
|---------------|-----|------|
| sensor.ebusd_bai_flame                  | CV brander       | mdi:fire |
| sensor.ebusd_bai_gasvalveasicfeedback   | CV gas-klep terugkoppeling |   |
| sensor.ebusd_bai_gasvalve3uc            | CV gas-klep activeer 3 signaal | mdi:ray-start  |
| sensor.ebusd_bai_gasvalve               | CV gas-klep actief | mdi:valve |
| binary_sensor.ebusd_bai_fluegasvalve_onoff | CV rookgasklep actief | mdi:valve |
| sensor.ebusd_bai_status01_pumpstate | CV pomp actief | mdi:pump |
| sensor.ebusd_bai_templimiter | | |
| sensor.ebusd_bai_statenumber | | |
| sensor.ebusd_bai_status | | |
| sensor.ebusd_bai_pumphours_hoursum2 | CV uren pomp actief | mdi:pump |
| sensor.ebusd_bai_setmode_disablehc | CV verwarming uitgeschakeld | mdi:radiator-off | 
| sensor.ebusd_bai_status02_hwcmode | Warm-water circuit geactiveerd | mdi:hand-water |
| sensor.ebusd_bai_blocktimehcmax_minutes0 | CV maximale wachttijd | mdi:timer-sand |
| binary_sensor.ebusd_bai_externalfaultmessage_onoff | CV extern fout bericht | |
| sensor.ebusd_bai_hwcwaterflowmax_uin100 | Warm-water circuit maximale doorstroming | |
| sensor.ebusd_bai_remainingboilerblocktime_minutes0 | Resterende CV wachttijd | |
| sensor.ebusd_bai_targetfanspeedoutput | CV gerealiseerde ventilator doel snelheid | mdi:fan |
| sensor.ebusd_bai_waterpressure_press | Verwarming waterdruk | |


![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/9c1e8cf2-09b0-408d-9ce2-bf953a4720eb)
