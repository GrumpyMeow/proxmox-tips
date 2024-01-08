Page 44/45: https://www.vaillant.nl/downloads/handleidingen-na-10-december-2014/installatiehandleiding-ecotec-plus-0020116691-05-272298.pdf

My Vaillant EcoTec CV publishes it's "statenumber" on the ebus. By translating the numbers into a textual description, the functioning of the CV is much more clear.

Home Assistant "Template helper sensor" to translate "ebusd bai statenumber" into a description:
```text
{{
   {
     '0':'Verwarming geen warmtevraag',
     '1':'CV-bedrijf ventilatorstart',
     '2':'CV-bedrijf pompvoorloop',
     '3':'CV-bedrijf ontsteking',
     '4':'CV-bedrijf brander aan',
     '5':'CV-bedrijf pomp-/ventilatornaloop',
     '6':'CV-bedrijf ventilatornaloop',
     '7':'CV-bedrijf pompnaloop',
     '8':'CV-bedrijf restwachttijd',
     '9':'Meetprogramma',
     '10':'Warmwatervraag door stromingssensor',
     '11':'Warmwaterbedrijf ventilatorstart',
     '13':'Warmwaterbedrijf ontsteking',
     '14':'Warmwaterbedrijf brander aan',
     '15':'Warmwaterbedrijf pomp-/ventilatornaloop',
     '16':'Warmwaterbedrijf ventilatornaloop',
     '17':'Warmwaterbedrijf pompnaloop',
     '30':'Kamerthermostaat (RT) blokkeert CV vraag',
     '31':'Zomermodus actief of geen warmtevraag door eBus-thermostaat',
     '32':'Wachttijd wegens afwijking ventilatortoerental',
     '34':'Vorstbeveiligingsfunctie actief',
     '76':'Installatiedruk te gering. Water bijvullen'
    }.get(states('sensor.ebusd_bai_statenumber') | default('') )
}}
```

The above states can also be viewed using the "live monitor" functionality on the display of the CV.

In Home Assistant i see this in the logbook:
![image](https://github.com/GrumpyMeow/proxmox-tips/assets/12073499/fc6c5bf8-ad4e-4ee2-8465-6f5b2c5136b5)
