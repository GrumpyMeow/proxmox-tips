For a while i've used Usteer and Dawn to enable devices to switch between Access-Points. Modern devices would roam using 802.11k and 802.11r and legacy devices would be kicked upon bad-connections. 

I was doing experimentation with `ubus` and noticed the ability to call `rrm_nr_list` to retrieve the list of WiFi-networks to be available for roaming. I think a static list of networks would be sufficient.
I have 4 access-points in my house with the same SSID for 2g and 5g bands. So devices should already be able to roam automatically. But wihout 802.11r a new handshake would probably need to be made.

I have configured the radio's to align with the names of the bands: `2g` and `5g` instead of (phy0-ap0, phy1-ap0).

Check if 802.11k and 802.11v is enabled on your access-points, by running:
```
ubus call hostapd.5g get_status
ubus call hostapd.2g get_status
```

You can enable with:
```
ubus call hostapd.5g bss_mgmt_enable '{ "neighbor_report": true, "beacon_report": true, "link_measurements": true, "bss_transition": true }'
```

```
{
        "status": "ENABLED",
        "bssid": "90:9f:22:0d:97:bf",
        "ssid": "WiFiForMe",
        "freq": 5320,
        "channel": 64,
        "op_class": 128,
        "beacon_interval": 100,
        "bss_color": 8,
        "phy": "phy1",
        "rrm": {
                "neighbor_report_tx": 13
        },
        "wnm": {
                "bss_transition_query_rx": 0,
                "bss_transition_request_tx": 0,
                "bss_transition_response_rx": 0
        },
        "airtime": {
                "time": 6528591,
                "time_busy": 454027,
                "utilization": 18
        },
        "dfs": {
                "cac_seconds": 0,
                "cac_active": false,
                "cac_seconds_left": 0
        }
}

```


For all your access-points you need to retrieve some info of the interfaces by running:
```
ubus call hostapd.5g rrm_nr_get_own
ubus call hostapd.2g rrm_nr_get_own
```



Using the info i retrieved from my Access-Points i created the following script:
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

```
more /var/run/hostapd-ph0.conf
```

In Startup: `sleep 10 && /root/rrm.sh &`


https://github.com/simonyiszk/openwrt-rrm-nr-distributor


```
root@AP-Keuken:~# hostapd_cli -i 5g show_neighbor
d4:1a:d1:18:87:00 ssid=57694669466f724d65 nr=d41ad1188700ef4900005106070603000600
d4:1a:d1:18:87:01 ssid=57694669466f724d65 nr=d41ad1188701ef5900008040090603023a00
f8:0d:a9:3a:00:00 ssid=57694669466f724d65 nr=f80da93a0000ef490000510d070603000d00
f8:0d:a9:3a:01:01 ssid=57694669466f724d65 nr=f80da93a0101ef5900008040090603023a00
f8:0d:a9:3a:19:b9 ssid=57694669466f724d65 nr=f80da93a19b9ef4900005101070603000100
f8:0d:a9:3a:19:ba ssid=57694669466f724d65 nr=f80da93a19baef5900008088090603028a00
90:9f:22:0d:97:bf ssid=57694669466f724d65 nr=909f220d97bfef5900008040090603023a00 stat
```
