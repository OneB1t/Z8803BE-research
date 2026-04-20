# Z8803BE-research

Z8803BE Chinese router 10GB/s SFP+ + WIFI 7 19000Mbps

## Fan Control
Script location:
/usr/sbin/fan_ctl.sh

## OpenWRT Version
BusyBox v1.33.2 (2023-11-06 12:29:45 UTC) built-in shell (ash)

OpenWrt 21.02-SNAPSHOT, r16881-4a1d8ef

## Tunnel Back Home (backdoor?)
![Tunnel Configuration](https://github.com/user-attachments/assets/1c4311ae-ff8d-4e39-8707-dfa0f73372eb)

## De-Chinese / Package Fix
To fix package installation issues, modify the packages list to use `http` instead of `https`, then run:

opkg install libustream-mbedtls ca-bundle ca-certificates

After installation, revert the package list back to `https`.

10GB/s SFP+ working as expected:

![10GB/s connected](https://github.com/user-attachments/assets/49e15b29-a444-427c-a3e4-8b551ece5f46)

MLD 5+6GHZ working:

![MLD 5+6GB WIFI 7](https://github.com/user-attachments/assets/276dead0-fbbe-4824-b00d-084c6a5882db)

FAN:

![FAN](https://github.com/user-attachments/assets/d3fd11f5-92f6-4e60-a13e-9f54717eb7e9)

PCB:
![PCB](https://github.com/user-attachments/assets/8cba53e7-41d2-4045-a785-bf4a80374f89)

CPU load:
there is constant 2.00 kernel load (probably some kernel calls)
![LOAD](https://github.com/user-attachments/assets/9783358a-e96e-491a-97ba-c6fee083b99d)


EDIT: there is now new firmware from ZBT
<img width="1740" height="646" alt="image" src="https://github.com/user-attachments/assets/6108a03d-51c5-4545-8066-873c05707180" />


## CUSTOM BUILD:
there is now custom builded firmware 	Zbtlink Z8803BE - OpenWrt 25.12-SNAPSHOT r32815-fce4731b5c / LuCI openwrt-25.12 branch 26.108.61903~4c5b5ef:
https://github.com/pttuan/openwrt/commits/zbt8803_25.12/
<img width="1231" height="826" alt="image" src="https://github.com/user-attachments/assets/63463f68-6a30-4a4b-9dd0-560157f9cc57" />

320mhz working
<img width="688" height="172" alt="image" src="https://github.com/user-attachments/assets/5199932b-5292-4b7a-92a3-0bed6fed15d3" />

10GB SFP+ working
<img width="652" height="344" alt="image" src="https://github.com/user-attachments/assets/a014b288-ecf1-4f13-9d4b-79a8aff8144f" />


MLO config example /etc/config/wireless
```
config wifi-device 'radio0'
  option type 'mac80211'
  option path 'soc/11300000.pcie/pci0000:00/0000:00:00.0/0000:01:00.0'
  option radio '0'
  option band '2g'
  option channel 'auto'
  option htmode 'EHT40'
  option cell_density '0'
  option country 'CZ'

config wifi-iface 'default_radio0'
  option device 'radio0'
  option network 'lan'
  option mode 'ap'
  option ssid 'OpenWrt'
  option encryption 'sae-mixed'
  option key '12345678'
  option ocv '0'

config wifi-device 'radio1'
  option type 'mac80211'
  option path 'soc/11300000.pcie/pci0000:00/0000:00:00.0/0000:01:00.0'
  option radio '1'
  option band '5g'
  option channel 'auto'
  option htmode 'EHT160'
  option country 'CZ'
  option cell_density '0'
  option rnr '1'
  option txpower '20'

config wifi-device 'radio2'
  option type 'mac80211'
  option path 'soc/11300000.pcie/pci0000:00/0000:00:00.0/0000:01:00.0'
  option radio '2'
  option band '6g'
  option channel 'auto'
  option htmode 'EHT320'
  option country 'CZ'
  option cell_density '0'
    option txpower '20'
  option rnr '1'

config wifi-iface 'mlo_ap'
  option device 'radio1 radio2'
  option network 'lan'
  option mode 'ap'
  option ssid 'MLO WIFI'
  option encryption 'sae'
  option key '12345678'
  option ocv '0'
  option ieee80211w '2'
  option mlo '1'
```


