I use my Google Home devices to control my smart-home via Home Assistant. This uses the Google Assistant integration in Home Assistant.

For this to work i need to allow the following IP-ranges through my firewall to my Home Assistant instance.

This are the IP-ranges:
* 66.102.0.0/20
* 66.249.80.0/20
* 74.125.0.0/16
* 108.177.0.0/17
* 192.178.0.0/15

I also had to add an IP-address from my ISP (Tmobile/Odido), but this might be from a mobile phone:
* 89.205.130.202

This are some of the IP-adresses i actually saw incomming traffic from:
* 66.102.9.73
* 66.102.9.74
* 66.249.81.133
* 66.249.93.14
* 66.249.93.2
* 74.125.208.72
* 74.125.208.73
* 74.125.208.74
* 74.125.211.7
* 108.177.64.4
* 108.177.64.5
* 108.177.64.6
* 108.177.76.2
* 192.178.13.200




* https://community.home-assistant.io/t/expose-home-assistant-for-google-ips-only-ipv4-only/184646
* https://md5calc.com/google/ip
* https://www.gstatic.com/ipranges/goog.json
