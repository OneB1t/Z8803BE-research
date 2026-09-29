# ZBT Z8803BE field notes: OpenWrt Wi-Fi 7 access point

**Updated:** 2026-09-29

**Scope:** one Zbtlink Z8803BE used as a bridged access point. These are
device-specific observations, not universal compatibility claims.

This report combines the sanitized scripts and configuration notes in the
[operator's router repository](https://github.com/davemessew/Z8803BE/tree/2cebce05fbd10b8cfe963aa2057f5b46627d3201)
with dated observations from that router. Evidence labels distinguish tested
behavior from source-code intent and pending validation. Private addresses,
wireless names, MAC addresses, serial numbers, credentials, raw logs, backups,
and firmware images are intentionally omitted.

## Test setup

- Board: Zbtlink Z8803BE (`zbtlink,zbt-z8803be`).
- Firmware: OpenWrt SNAPSHOT `r36402-820fbf4a9f`, Linux 6.18.52,
  `mediatek/filogic`.
- Wireless: MediaTek MT7996 with 2.4, 5, and 6 GHz radios configured as a
  three-link Wi-Fi 7 MLO access point.
- Network role: bridged access point; the upstream router provides DHCP and
  internet routing.

## 2.5 GbE copper SFP link

**Observed on this unit:** the SFP host interface is `eth2` (`gmac2`); `eth0`
is not the SFP interface. With host-side in-band autonegotiation enabled,
the router's SFP status reported `link_up`, and the operator separately
reported a green switch indication at 2.5 Gb/s, while `eth2` still had no
carrier. Configuring the host side as fixed 2500BASE-X, full duplex, with
autonegotiation disabled restored carrier and 2.5 Gb/s Ethernet. DHCP,
gateway, DNS, and internet access then worked through the link. A later
readback showed carrier up at 2500 Mb/s with no RX/TX errors.

The operator separately reported a green 2.5 Gb/s indication on the switch;
that does not identify the adapter or establish host-side carrier. The router
log identified the installed SFP module as `OEM SFP-2.5G-T-C`; the adapter's
retail make and exact hardware revision are not established here. The fixed
host-link setting is specific to this port/module combination. Reboot and
module-reinsertion behavior were not separately verified.

Implementation and setup notes: [SFP configuration and diagnosis](https://github.com/davemessew/Z8803BE/blob/b9afb9b2bc872df0f2d4c0f6cfa6067ce221bf71/README.md),
[sanitized network example](https://github.com/davemessew/Z8803BE/blob/b9afb9b2bc872df0f2d4c0f6cfa6067ce221bf71/config/network.example).

## MLO security and legacy clients

**Historical observation:** with `sae-compat` on the MLO setup, Apple N1
clients could authenticate, but hostapd then failed while adding the station
to the MT7996. The generated link configuration mixed WPA2 without PMF on
2.4/5 GHz with WPA3-SAE and required PMF on 6 GHz.

The documented workaround is a WPA3-SAE MLO network across the radios plus
separate WPA2 compatibility networks on 2.4 and 5 GHz. This is a report from
one setup, not a complete client-compatibility matrix or proof that every
client behaves the same way.

The configuration notes list 2.4 GHz channel 11/EHT40, 5 GHz channel
36/EHT80, and 6 GHz channel 37/EHT320 as the baseline; the boot-time channel
selector may choose different channels after its survey. In the tested
Swiss-regulatory configuration, increasing the 5 GHz UCI transmit-power
value did not override the kernel/firmware regulatory limit.

See the [wireless configuration notes at the recorded commit](https://github.com/davemessew/Z8803BE/commit/28b70b37fcf58bcfb2e5eb932501f0754d488c0f).

## Channel-selection helper

The helper scans the three radios sequentially and scores candidate
placements: 2.4 GHz channels 1/6/11, 5 GHz 80 MHz blocks 36/52/100/116/132,
and 6 GHz EHT320 primary channels 37/69. Active-MLO scans had returned stale
results, so the script stops Wi-Fi during the survey. This interrupts service
while it runs; use a wired management path where possible.

After applying the selected channels, the helper waits for three MLO link IDs
in `iw dev ap-mld0 info` and restores the previous channels if fewer appear.
That check confirms reported link IDs, not successful client traffic. The
updated check and a complete on-router channel-survey run were still pending
deployment in the recorded notes; the helper should not be described as
fully live-validated.

See the [channel-selection script](https://github.com/davemessew/Z8803BE/blob/2cebce05fbd10b8cfe963aa2057f5b46627d3201/zbt-channel-select)
and [pending-verification note](https://github.com/davemessew/Z8803BE/commit/7482d753a29f32b74f8c2ae797f8899bdc0d4242).

## 6 GHz transmit-power helper

The helper creates a temporary radio-2 BSS, applies a 14 dBm per-radio limit,
reads the value back, removes the temporary BSS, and checks that the MT7996
retained the limit. The reason for choosing 14 dBm is not established by the
available notes. The boot hotplug path applies it when the MLO interface is
added. A manual Wi-Fi reload can reset the radio power without producing that
add event, so the helper may need to be run again after a manual reload.

See the [power helper](https://github.com/davemessew/Z8803BE/blob/2cebce05fbd10b8cfe963aa2057f5b46627d3201/zbt-6g-txpower)
and [hotplug hook](https://github.com/davemessew/Z8803BE/blob/2cebce05fbd10b8cfe963aa2057f5b46627d3201/98-zbt-6g-txpower).

## NAPI and RPS CPU tuning

The helper discovers the MT7996 `napi/phy0-*` threads after each Wi-Fi
startup, assigns them round-robin to CPUs 1–3, and sets each MLO receive
queue's RPS mask to hexadecimal `e` (CPUs 1–3). Its stated intent is to leave
CPU 0 for Ethernet work and spread Wi-Fi receive processing. No before/after
throughput, latency, or CPU-load benchmark was recorded, so a performance
benefit has not been demonstrated.

See the [CPU-tuning helper](https://github.com/davemessew/Z8803BE/blob/2cebce05fbd10b8cfe963aa2057f5b46627d3201/zbt-cpu-tune).

## CPU frequency governor

**Observed and changed on the router:** the tested kernel used `userspace` as
its default CPU-frequency governor. No separate governor controller was found
in the inspected router setup. During a captured low-load interval the policy
remained at 1.5 GHz; which boot stage initially selected that frequency is
unknown. The available operating-point table starts at 800 MHz, which is the
configured lowest active OPP in that table, not proof of the silicon's
physical minimum.

The router now uses `schedutil`, selected by a small init service and
preserved through the listed sysupgrade paths. It changes the governor only;
the 800–1800 MHz policy bounds and thermal/fan settings are unchanged.
Reboot persistence and a sustained thermal benefit have not been tested.

A short passive frequency sample after the change lasted about eight seconds:
the counters added roughly 6.37 s at 800 MHz, 0.03 s at 1100 MHz, 0.03 s at
1500 MHz, and 1.58 s at 1800 MHz; the transition counter increased by about
580. This is a brief observation, not a workload benchmark or thermal test.
The exact source of the earlier 1.5 GHz selection remains unresolved; the
inspected U-Boot 2024.10-rc2 `ARM_B` fixed-clock source declaration alone
does not establish which stage wrote the active frequency.

See the [governor change and helper](https://github.com/davemessew/Z8803BE/commit/2cebce05fbd10b8cfe963aa2057f5b46627d3201).

## Fan and temperatures

**Single live snapshot, 2026-09-29:** CPU thermal-zone reading was 58.816 °C
(hwmon reading 58.853 °C); the SFP module sensor read 48.316 °C; MT7996
sensors read 58 °C, 57 °C, and 70 °C. The CPU fan cooling device reported
state 1 and PWM 80/255. No fan-RPM sensor was exposed by the checked hwmon
files, so this confirms the controller output, not measured fan speed.

A separate earlier module diagnostic reported module alarm and critical
thresholds of 90 °C and 95 °C, respectively, with neither alarm set at that
time. These are module-reported thresholds, not an independently verified
thermal rating.

The inspected running device tree tied fan cooling to the CPU thermal zone,
not the SFP sensor. Its reported trip points were 38/45/60 °C, with about
2 °C hysteresis and a 100 °C critical point; the fan PWM table was
0/80/112/144/176/216/255. A separate earlier sample showed a higher cooling
state around 58 °C. This is compatible with hysteresis after a prior CPU
excursion above 60 °C, but that crossing and the transition were not captured.
The samples do not establish a fan-control fault or a single cause of heat.
The public board DTS copy in this repository does not contain those thermal
nodes; recheck the running tree or matching firmware source before relying on
the exact table for another build.

The temperature snapshot is not evidence that `schedutil` reduced heat. A
longer, comparable load/temperature/fan observation would be needed for that
conclusion.

## Open verification items

- Recheck SFP host-link behavior after module replacement or firmware changes;
  this report does not establish compatibility with other modules.
- Verify SFP link behavior after a reboot and module reinsertion.
- Deploy and run the revised channel survey, then verify client traffic on all
  three MLO links.
- Check that the 6 GHz power limit is restored after a manual Wi-Fi reload.
- Benchmark NAPI/RPS settings before attributing a performance improvement.
- Verify `schedutil` persistence after reboot and compare temperature/fan
  behavior under a repeatable workload.
- Identify which boot component originally selected 1.5 GHz and confirm the
  provenance of the 800 MHz minimum OPP for the exact running build.

## Source revision

The scripts and sanitized configuration cited above are from
[`davemessew/Z8803BE` revision `2cebce05fbd10b8cfe963aa2057f5b46627d3201`](https://github.com/davemessew/Z8803BE/tree/2cebce05fbd10b8cfe963aa2057f5b46627d3201).
Live measurements are specific to the single device and dates stated in this
report.
