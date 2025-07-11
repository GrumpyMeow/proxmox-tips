For a while i've used Usteer and Dawn to enable devices to switch between Access-Points. Modern devices would roam using 802.11r and legacy devices would be kicked upon bad-connections. 

I was doing experimentation with `ubus` and noticed the ability to call `rrm_nr_list` to retrieve the list of WiFi-networks to be available for roaming. I think a static list of networks would be sufficient.
I have 4 access-points in my house with the same SSIDs for 2g and 5g bands. So devices should already be able to roam automatically. But wihout 802.11r a new handshake would probably need to be made.

```
ubus call hostapd.5g rrm_nr_get_own
ubus call hostapd.2g rrm_nr_get_own
```

So i created the following script:
```/root/rrm.sh
ssid="WiFiForMe"
ap_beneden=[\"f8:0d:a9:3a:19:ba\",\"$ssid\",\"f80da93a19baef5900008088090603028a00\"],[\"f8:0d:a9:3a:19:b9\",\"$ssid\",\"f80da93a19b9ef4900005101070603000100\"]
ap_keuken=[\"90:9f:22:0d:97:bf\",\"$ssid\",\"909f220d97bfef5900008040090603023a00\"],[\"90:9f:22:0d:97:be\",\"$ssid\",\"909f220d97beef4900005103070603000300\"]
ap_washok=[\"d4:1a:d1:18:87:01\",\"$ssid\",\"d41ad1188701ef5900008040090603023a00\"],[\"d4:1a:d1:18:87:00\",\"$ssid\",\"d41ad1188700ef4900005106070603000600\"]
ap_zolder=[\"f8:0d:a9:3a:01:01\",\"$ssid\",\"f80da93a0101ef5900008040090603023a00\"],[\"f8:0d:a9:3a:00:00\",\"$ssid\",\"f80da93a0000ef490000510d070603000d00\"]

# Remove current Access-Point from $list. This to not advertise itself.
list=[$ap_beneden,$ap_zolder,$ap_washok,$ap_keuken]

ubus call hostapd.5g rrm_nr_set '{"list": '$list' }'
ubus call hostapd.2g rrm_nr_set '{"list": '$list' }'
```
The script will register 5g and 2g of each Access-Point and assigns them both bands.

Check using:
```
ubus call hostapd.5g rrm_nr_list
ubus call hostapd.2g rrm_nr_list
```

more /var/run/hostapd-ph0.conf 


https://github.com/simonyiszk/openwrt-rrm-nr-distributor
