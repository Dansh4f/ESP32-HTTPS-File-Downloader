# ESP32 High-Throughput HTTPS Download to SPIFFS

Overview
--------
This project demonstrates an ESP32 (Arduino framework) HTTPS client that downloads a file over TLS and writes it into SPIFFS while measuring throughput. It provides a straightforward example using `WiFiClientSecure` + `HTTPClient`, streaming the response to SPIFFS and printing measured download/write speeds on the serial monitor.

Key features
------------
- HTTPS download using `WiFiClientSecure` + `HTTPClient`.
- Streaming write to SPIFFS with throughput measurement.
- Simple WiFi connection helper.
- Focus on practical tuning knobs to maximize sustained throughput.
- Clear serial output to validate performance and behavior.

Project layout
--------------
- `src/main.cpp` — boots the device, mounts SPIFFS, connects to WiFi, and invokes the HTTPS download routine.
- `lib/core/WiFiManager.hpp` — WiFi connection helper (adds AP entries and blocks until connected).
- `lib/core/HttpsClient.hpp` — HTTPS download and SPIFFS write logic; measures download and write speeds and prints them to Serial.
- `lib/core/SPIFFS.hpp` — helper to read back the saved file.

How it works (implementation summary)
------------------------------------
- WiFi: `WiFiMulti` is used to manage WiFi networks. `wifimanager()` adds configured AP credentials and waits until the device is connected.
- SPIFFS: Mounted at startup using `SPIFFS.begin(true)` (formats if needed).
- HTTPS download: the client uses `WiFiClientSecure` and `HTTPClient`. For convenience in examples `client.setInsecure()` is used to simplify TLS setup (see Security note below).
- Buffered I/O: a large network buffer is allocated (8 KB by default) and used to read from the HTTP stream and write to SPIFFS in the same block sizes, keeping RAM and call overhead balanced.
- Throughput measurement: the code uses timestamps (micros()) around download and write operations to compute and print average download and write speeds to Serial.

Configuration
-------------
- Download URL: Edit the URL constant in `lib/core/HttpsClient.hpp` (`const char* url`) to change the target for download. For microbenchmarks use a large binary (e.g. `https://speed.hetzner.de/100MB.bin`) to measure sustained throughput.
- Output filename: Edit the `filename` setting in `lib/core/HttpsClient.hpp` (default is `"/file.txt"`).
- WiFi credentials: Edit `lib/core/WiFiManager.hpp` to set `ssid` and `password`.
- Buffer size: Default read/write buffer is 8192 bytes. Increasing it may improve throughput (watch RAM usage).

Build and flash (PlatformIO)
----------------------------
Open a terminal at the project root and run:

# Build
pio run

# Upload / Flash
pio run -t upload

# Open serial monitor (115200 baud)
pio device monitor -b 115200

How to run the test
-------------------
1. Set WiFi credentials in `lib/core/WiFiManager.hpp`.
2. Edit `lib/core/HttpsClient.hpp` and update `const char* url` to a file suitable for throughput testing (large binary for sustained measurements).
3. Build and upload the firmware.
4. Open the serial monitor and reset the ESP32 to start the download routine.
5. The serial output shows connection, file size, SPIFFS stats, and measured average download/write speeds.

Example serial output (illustrative)
-----------------------------------
The firmware prints human-readable progress and a final throughput summary. Example lines:

Phase 1 completed! you are connected to : MyWiFiAP
IP address 192.168.1.100  WIFI RSSI -45 dBm

fetching url: https://speed.hetzner.de/100MB.bin
HTTP begin..
HTTP GET code: 200
file size: 104857600 byte
SPIFFS total space: 1310720 bytes
used space: 0 bytes
free space: 1310720 bytes
writing to SPIFFS...
file written to spiffs
file size written: 104857600 bytes
average download speed: 350.12 KB/s
avg write speed: 780.45 KB/s

Notes:
- Reported speeds vary with network, AP, router, server, and device flash configuration. Use large files hosted on high-bandwidth servers and run multiple trials for reliable averages.
- The code prints both download and write speeds so you can identify whether network or flash is the limiting factor.

Performance and tuning guidance
------------------------------
To maximize sustained throughput:
- Use a high-bandwidth host for the test (large binary on a fast server).
- Keep the ESP32 close to the access point and avoid congested WiFi channels (2.4 GHz interference).
- Increase the read/write buffer size carefully (e.g., 16 KB) if RAM allows.
- Tune flash parameters and partition settings in `platformio.ini` (flash frequency and mode) — faster flash modes can improve SPIFFS write throughput.
- Consider using LittleFS (if supported by your core) for improved wear-leveling and possibly better performance.
- If TLS validation is required, prefer certificate pinning or proper root CA verification in production (instead of setInsecure()), even though it may add a small CPU/TLS overhead.

Security note
-------------
For development convenience the example uses `client.setInsecure()` to skip certificate validation. For production use, validate server certificates or implement certificate pinning to preserve TLS integrity.

Error handling and robustness
-----------------------------
- The code checks HTTP result codes and prints error messages on failure.
- Suggested improvements include adding a connection timeout and retry logic with exponential backoff, and making configuration values runtime-configurable (serial menu or small web UI).

Files changed / created by this README
-------------------------------------
- `README.md` — project documentation and usage instructions.

Acknowledgements
----------------
This README is derived from the implementation in `src/main.cpp` and `lib/core/*.hpp`. Feedback and contributions are welcome — open an issue or PR on the repository.
