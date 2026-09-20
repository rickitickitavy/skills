---
name: wifi
description: >-
  Applies ESP32 Arduino Wi-Fi STA and SoftAP rules: clean STA-to-AP
  switch, recovery AP after a failed join, and SoftAP start order. Use when
  changing WiFiController, SoftAP, STA begin/reconnect, MODE AP/STA, or
  fallback after AUTH_EXPIRE / join timeout.
---

# Wi-Fi (STA and AP)

## STA → AP (hard rule)

Never tear down a station that is still connecting by erasing its config.

`WiFi.disconnect(true, true)` on Arduino 3 means `wifioff` + **erase AP/STA config**. While STA is connecting that fails with `ESP_ERR_WIFI_STATE` (`sta is connecting, cannot set config`). SoftAP then does not start and the device AP is invisible.

Stop STA first, then start AP:

```cpp
WiFi.disconnect(false, false, 2000);
delay(200);
WiFi.mode(WIFI_OFF);
delay(400);
WiFi.mode(WIFI_AP);
```

Do not call `softAP` / `softAPConfig` / `esp_wifi_set_config` until mode is off or AP-only.

## Recovery AP after failed STA

If MODE is STA and join fails (`AUTH_EXPIRE`, timeout, `AUTH_LEAVE`):

- Bring up a **recovery SoftAP** with a **known** name and password (project defaults, e.g. `z2m-gateway` / `00000000`).
- Do **not** advertise the router SSID/password as the SoftAP. One stored BSSID field used for STA join would name the AP `HOME` (or whatever the router is), so the user cannot find the device.

`AUTH_EXPIRE` means the router rejected the handshake (PSK / WPA3 / cut-off). Still fall back to the recovery AP.

`NO_AP_FOUND` (reason 201) means the station scan did not see the SSID. ESP32-C6 is **2.4 GHz only**. Before `WiFi.begin`, set country channels 1–13, `WIFI_ALL_CHANNEL_SCAN`, scan (including hidden), log what was seen, then join the **strongest RSSI** BSSID of that SSID (mesh/repeaters list the same name many times; last-match picks the worst). A 5 GHz-only SSID will never appear.

## SoftAP start

- One start path: `softAPConfig` then `softAP`. Do not rewrite the radio afterward (`esp_wifi_set_config`, bandwidth, IPv6 toggle) — that logs `AP start`/`AP stop` and leaves a broken handshake.
- Set 802.11b/g/n on the interface **before** `softAP` if the chip defaults to 11ax.
- If AP is already up, do not rebind it in a loop during a client join.
- SoftAP TX: `WIFI_POWER_8_5dBm`. STA TX: `WIFI_POWER_21dBm` (enum max) plus `esp_wifi_set_max_tx_power(84)` and `WIFI_PS_NONE`.
- Do **not** force STA to 11b/g/n-only on ESP32-C6. That starves a Wi-Fi 6 router and shows RSSI like −80 dBm at a few metres. Keep SoftAP on 11b/g/n.
- After `Zigbee.begin()` in STA, call `applyStaRadio()` again, then start or rebind AsyncWebServer. Zigbee on the same RF resets sleep/TX and leaves STA “connected” with a dead netif (ping and HTTP fail). Prefer Wi-Fi in coexist (`ESP_COEX_PREFER_WIFI`). Call `esp_coex_wifi_i154_enable()` **before** `Zigbee.begin()`. Pick a Zigbee channel that overlaps the STA Wi-Fi 20 MHz (`zigbee = wifiChannel + 10`, clamp 11–26). SoftAP stays 11b/g/n. A single C6 cannot match a dual-SoC gateway. RSSI is **receive** level; MQTT to another subnet (e.g. device `192.168.4.x`, broker `192.168.1.1`) will still time out even with a good link.
- Mark `staWasConnected` when the boot join succeeds so `update()` can see a later drop and rebind the console.
- Software reboot (`ESP.restart()`, USB, settings save) leaves the C6 RF wedged: STA fails until a power cycle. Before restart: `disconnect(false, false, 2000)`, `WIFI_OFF`, delay. On warm boot: `WIFI_OFF` and wait longer (~800 ms) before STA/AP. Never restart while STA is still connecting.
