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


