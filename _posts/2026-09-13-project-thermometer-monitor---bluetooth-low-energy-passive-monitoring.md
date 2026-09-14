---
layout: post
title: "Project Thermometer Monitor - Bluetooth Low Energy Passive Monitoring"
date: 2026-09-13 15:31:25 +0000
tags: ["Asynchronous.", "Bluetooth", "Bluetooth Low Energy (BLE)", "Docker", "Grafana", "Linux", "Python", "SQLite"]
blogger_orig_link: https://lexsysko.blogspot.com/2026/09/project-thermometer-monitor-bluetooth.html
---

Thermometer Monitor [is a Python-based asynchronous Bluetooth Low Energy (BLE) environmental monitoring app and telemetry logger](https://github.com/lexsysko/ThermometerMonitor). It continuously listens for BLE advertising packets from smart thermometers, filters out duplicate readings, and saves the data to a local SQLite database.  

It constantly listens for BLE advertising packets in passive mode from smart thermometers and hygrometers (like the Xiaomi Mijia / LYWSD03MMC running custom ATC or PVVX firmware), decodes the sensor data, removes duplicate readings, and saves everything to a local SQLite database.

It supports cross-platform operation (Linux, macOS, Windows) out of the box via Bleak, with an automatic Linux raw HCI socket fallback for passive mode in legacy adapters or specialized container setups. If passive mode isn’t available, like on macOS, it switches to active listening mode. In active mode, the listener device requests extra responses from the target device’s advertising packets, such as details like the device name. These additional responses drain the BLE device’s battery more, so it’s best to use passive scanning whenever possible.

It comes with ready-to-use Docker Compose setups and a Grafana dashboard integration for real-time visualization.  
  
This project is a continuation of other projects focused on monitoring BLE devices:  
- [Моніторинг температури та сповіщення про критичні значення.](https://lexxai.blogspot.com/2024/12/python.html)  
- [Скрипт для показу значень температури та вологості Bluetooth BLE термометра Xiaomi](https://lexxai.blogspot.com/2022/11/bluetooth-ble-xiaomi.html)

  
Grafana Example View:

[![](/assets/images/blog/b811936c4039b2b5-0e868a2106a58a51.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh0EFJPDns5BYSbfLnubjmuHXd__Oj1LHcSyHrMSy0-5ovcrZPYI6l-IWUCP3bktXpzSF14cxDHNA9O8WwwSg829lB1GtHmxePdtamOkHCj6V5RSAG8vnQ6f_ZBLPt6l8agpmOfzlGWPA4WNUpwzwKTZ8br840ABawCuc_5JMebajKvVTer9R2Ylp-_Pkk/s1903/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-09-13%20180436.png)

  

An example of Docker logs:

```
2026-09-12 18:38:57 - ThermometerMonitor - DEBUG - DEBUG Started
2026-09-12T18:38:57.350702033Z 2026-09-12 18:38:57 - ThermometerMonitor - INFO - Version of app: 0.3.0
2026-09-12T18:38:57.359934978Z 2026-09-12 18:38:57 - ThermometerMonitor.db_writer - INFO - [DB] Database initialized successfully.
2026-09-12T18:38:57.360675789Z 2026-09-12 18:38:57 - ThermometerMonitor.start_scanning - INFO - [BLE] Attempting scan for sensors matching prefix 'Addresses: a4:c1:38:5e:db:77,a4:c1:38:67:99:5b' in passive mode...
2026-09-12T18:38:57.372897925Z 2026-09-12 18:38:57 - ThermometerMonitor.start_scanning - WARNING - Bluetooth 5.0 capabilities for passive scanning are not supported on this host
2026-09-12T18:38:57.373291280Z 2026-09-12 18:38:57 - ThermometerMonitor.start_scanning - INFO - Load 'aioblescan' library for access to HCI Event
2026-09-12T18:38:57.489054222Z 2026-09-12 18:38:57 - ThermometerMonitor.db_writer - INFO - [DB] Using SQLite database file: /app/data/ble_data.db
2026-09-12T18:38:57.489732913Z 2026-09-12 18:38:57 - ThermometerMonitor.db_writer - INFO - [DB] CLEANUP initialized every 86400 seconds.
2026-09-12T18:38:57.491186435Z 2026-09-12 18:38:57 - ThermometerMonitor.hci_passive_scanner_protocol - INFO - HCI Scanning started. Passive mode is True
2026-09-12T18:38:57.491967280Z 2026-09-12 18:38:57 - ThermometerMonitor - INFO - [BLE] Scanner and Watchdog are running. Waiting for events...
2026-09-12T18:38:57.492036156Z 2026-09-12 18:38:57 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Watchdog activated. Packet timeout: 600s.
2026-09-12T18:38:59.344335644Z 2026-09-12 18:38:59 - ThermometerMonitor - DEBUG - ATC_5EDB77: Payload(format='pvvx_15b', temperature_c=3.54, humidity_pct=73.42, battery_mv=2481, battery_pct=31, frame_counter=209)
2026-09-12T18:39:07.363636679Z 2026-09-12 18:39:07 - ThermometerMonitor - DEBUG - ATC_67995B: Payload(format='pvvx_15b', temperature_c=10.0, humidity_pct=57.25, battery_mv=2212, battery_pct=1, frame_counter=252)
2026-09-12T18:39:37.498625124Z 2026-09-12 18:39:37 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 2 sensor readings.
2026-09-12T18:40:47.120679260Z 2026-09-12 18:40:47 - ThermometerMonitor - DEBUG - ATC_67995B: Payload(format='pvvx_15b', temperature_c=9.94, humidity_pct=54.07, battery_mv=2212, battery_pct=1, frame_counter=253)
2026-09-12T18:41:17.124603662Z 2026-09-12 18:41:17 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 1 sensor readings.
2026-09-12T18:42:28.899410458Z 2026-09-12 18:42:28 - ThermometerMonitor - DEBUG - ATC_5EDB77: Payload(format='pvvx_15b', temperature_c=3.48, humidity_pct=68.42, battery_mv=2477, battery_pct=30, frame_counter=210)
2026-09-12T18:42:58.902991650Z 2026-09-12 18:42:58 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 1 sensor readings.
2026-09-12T18:43:17.494130145Z 2026-09-12 18:43:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 0.8s
2026-09-12T18:44:17.495541003Z 2026-09-12 18:44:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 1.0s
2026-09-12T18:44:46.466798366Z 2026-09-12 18:44:46 - ThermometerMonitor - DEBUG - ATC_67995B: Payload(format='pvvx_15b', temperature_c=9.9, humidity_pct=51.74, battery_mv=2212, battery_pct=1, frame_counter=254)
2026-09-12T18:45:16.470402816Z 2026-09-12 18:45:16 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 1 sensor readings.
2026-09-12T18:45:17.496783166Z 2026-09-12 18:45:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 1.1s
2026-09-12T18:46:17.498617776Z 2026-09-12 18:46:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 1.3s
2026-09-12T18:46:28.387498121Z 2026-09-12 18:46:28 - ThermometerMonitor - DEBUG - ATC_5EDB77: Payload(format='pvvx_15b', temperature_c=3.41, humidity_pct=64.89, battery_mv=2472, battery_pct=30, frame_counter=211)
2026-09-12T18:46:58.392286358Z 2026-09-12 18:46:58 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 1 sensor readings.
2026-09-12T18:47:17.500400475Z 2026-09-12 18:47:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 1.4s
2026-09-12T18:48:17.501516782Z 2026-09-12 18:48:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 1.6s
2026-09-12T18:48:35.873004448Z 2026-09-12 18:48:35 - ThermometerMonitor - DEBUG - ATC_67995B: Payload(format='pvvx_15b', temperature_c=9.82, humidity_pct=49.94, battery_mv=2215, battery_pct=2, frame_counter=255)
2026-09-12T18:49:05.877105617Z 2026-09-12 18:49:05 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 1 sensor readings.
2026-09-12T18:49:17.503958019Z 2026-09-12 18:49:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 9.5s
2026-09-12T18:50:17.506543240Z 2026-09-12 18:50:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 1.9s
2026-09-12T18:50:37.830438991Z 2026-09-12 18:50:37 - ThermometerMonitor - DEBUG - ATC_5EDB77: Payload(format='pvvx_15b', temperature_c=3.37, humidity_pct=62.56, battery_mv=2475, battery_pct=30, frame_counter=212)
2026-09-12T18:51:07.834376599Z 2026-09-12 18:51:07 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 1 sensor readings.
2026-09-12T18:51:17.508360633Z 2026-09-12 18:51:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 2.1s
2026-09-12T18:52:17.510009732Z 2026-09-12 18:52:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 2.3s
2026-09-12T18:52:35.225375205Z 2026-09-12 18:52:35 - ThermometerMonitor - DEBUG - ATC_67995B: Payload(format='pvvx_15b', temperature_c=9.82, humidity_pct=48.54, battery_mv=2215, battery_pct=2, frame_counter=0)
2026-09-12T18:53:05.228948108Z 2026-09-12 18:53:05 - ThermometerMonitor.db_writer - DEBUG - [DB] Saved 1 sensor readings.
2026-09-12T18:53:17.511740337Z 2026-09-12 18:53:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 0.0s
2026-09-12T18:54:17.513295203Z 2026-09-12 18:54:17 - ThermometerMonitor.watchdog - DEBUG - [Watchdog] Heartbeat check | Secs since last packet: 0.2s
...
2026-09-13 14:57:54 - ThermometerMonitor.handler_signal - INFO - 
2026-09-13T14:57:54.067448613Z [System] Received signal 15. Triggering graceful shutdown...
2026-09-13T14:57:54.068390758Z 2026-09-13 14:57:54 - ThermometerMonitor - INFO - [BLE] Stopping scanner...
2026-09-13T14:57:54.068574100Z 2026-09-13 14:57:54 - ThermometerMonitor - INFO - [DB] Flushing remaining queue items...
2026-09-13T14:57:54.068598045Z 2026-09-13 14:57:54 - ThermometerMonitor - INFO - [System] Finishing writer task
2026-09-13T14:57:54.068616309Z 2026-09-13 14:57:54 - ThermometerMonitor.db_writer - INFO - [DB] Cleanup worker shut down cleanly.
2026-09-13T14:57:54.068680651Z 2026-09-13 14:57:54 - ThermometerMonitor.db_writer - INFO - [DB] Shutdown event received, stopping worker...
2026-09-13T14:57:54.068774065Z 2026-09-13 14:57:54 - ThermometerMonitor.db_writer - INFO - [DB] Writer worker shut down cleanly.
2026-09-13T14:57:54.144978781Z 2026-09-13 14:57:54 - ThermometerMonitor - INFO - [System] Finishing cleanup task
2026-09-13T14:57:54.145064253Z 2026-09-13 14:57:54 - ThermometerMonitor - INFO - [System] Shutdown complete.
```

Repo: <https://github.com/lexsysko/ThermometerMonitor>
