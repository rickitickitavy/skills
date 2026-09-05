---
name: async-web-server
description: >-
  Applies ESP32 AsyncWebServer conventions: ESP32Async stack, route
  registration, and Wi-Fi STA rebind after disconnect. Use when adding or
  changing AsyncWebServer, AsyncTCP, ESPAsyncWebServer, HTTP on SoftAP/STA,
  web console firmware, or Wi-Fi reconnect that must keep the HTTP server
  reachable.
---

# AsyncWebServer

Use **ESP32Async/AsyncTCP** and **ESP32Async/ESPAsyncWebServer** on ESP32 Arduino / PlatformIO. Prefer this stack over blocking `WebServer` for device consoles.

Wire the server from outside the class (e.g. `main.cpp`). Do not add an ad-hoc `static` instance only so a Wi-Fi callback can reach `this`.

## STA rebind (hard rule)

ALWAYS rebind AsyncWebServer when WiFi reconnected in STA mode after disconnect because AsyncWebServer become inaccessable after WiFi disconnect.

- After STA drops, reconnect Wi-Fi, then when the station has an IP again call `server.end()` and `server.begin()`.
- Keep existing route handlers; rebind only the listen socket.
- First boot: start the server after Wi-Fi is up. Do not rebind that first `begin()`.
- Do not leave the console bound only to the pre-disconnect interface.

```cpp
void WebConsole::rebind() {
    server.end();
    delay(50);
    server.begin();
}
```
