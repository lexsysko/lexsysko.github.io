---
layout: post
title: "Project Seven Segment Monitor - visually monitoring the refrigerator’s power consumption"
date: 2026-10-02 21:29:35 +0000
tags: ["Docker", "Grafana", "headless", "Linux", "OpenCV", "Python", "SQLite", "Ubuntu"]
blogger_orig_link: https://lexsysko.blogspot.com/2026/10/project-seven-segment-monitor.html
---

🛎️ A hobby involving collecting provisions includes monitoring passive
Bluetooth temperature, analyzing sound with PSD and FFT tools, and now adding
visual monitoring through a webcam focused on seven-segment indicators.

### Why

The [previous project](/2026-10-04-project-sound-monitor-for-detect-target-operational-states.md) for detecting the on/off state of the kitchen refrigerator by sound analyses has been stopped. The built-in microphone’s sound quality was so poor and noisy that I decided to abandon it and switch to a more reliable method: visually monitoring the refrigerator’s power consumption by reading the amperage, with 00.0A indicating OFF and 00.7A or 00.8A indicating ON.  

### How

📼 Seven Segment Monitor is a computer vision and IoT tool that
continuously captures images from a USB or V4L2 camera, or static frames, of
seven-segment displays. It uses OpenCV to extract and recognize numeric
readouts, then stores the parsed readings in an SQLite database with automated
retention and cleanup.

[![](/assets/images/blog/1430588b9c11ae36-eabc68f2f0f023aa.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjcoZqJBX3bfKDcEn722UnQInPPYzAdft6v2d_MJ-sLrNTjMob20fp714kdAhA9q9hrXkzdMQJ34ddAWUnti9Wcia7-fcqGH2utiEePLCWGoCGAQRIYtnVAZ2UAd37PA5iBZkjSW_s8aiScmuNuD9HskyagsNI0I7jsDj3P0yebZzcaqpecBIdWv90Lo-o/s1237/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-10-03%20000629.png)  
*Example of a development result*

[![](/assets/images/blog/04bb0d1b61afe7a1-19315812964e6374.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg-rgLcACaACARbOyk22CfRlHwKcaVTaAvH9ygjfq_mdYzYlNimRKYsGE084Bdcis6n88kp0Cw5iKFhB_E7E6tOhwcjMFBH1CqgGZ1pCwD4mMqZBdMlZENDndN7P51kYj3yFgkZs6x_ze0XX0lQciq1hroyHwvD8OHS5cTJeVZeaRw3__NptqfjP37-FFA/s2828/20260930_183333.jpg)  
*A development laptop with sunglasses used as a pre-filter for the webcam to reduce lighting from seven-segment LED indicators on a surge voltage device.*

### Features

* Multi-threaded Architecture:

+ Camera Producer: Captures frames asynchronously from a V4L2/USB camera or
  loads them from disk for testing or simulation.
+ Vision Worker: Handles automatic rotation correction, color and brightness
  filtering, shear de-skewing, digit box slicing, and segment state
  recognition.
+ Database Writer Worker: Batches writes and saves timestamped readings and
  alert/state statuses to SQLite.

+ Database Cleanup Worker: Purges records older than a configurable
  retention period automatically.

* Multiple Color & Mask Extraction Algorithms:

+ WITHOUT\_GREEN: Difference-based channel masking for red LED displays.
+ RED: Dual-band HSV color masking.
+ RED\_ADAPTIVE: Adaptive thresholding based on the green-to-red pixel ratio.

* Display Layout Customization:

+ Configurable rows and digits per row (NUM\_DIGITS\_ROWS,
  NUM\_DIGITS\_PER\_ROW).
+ Shear angle compensation for slanted italicized digits
  (DIGIT\_SHEAR\_ANGLE).
+ Digit density and segment activation threshold tuning.

* Deployment Ready:

+ Run directly via Python CLI (sevensegmentmonitor).
+ Run containerized via headless Docker Compose with optional Grafana
  integration.

Repositories: 

* [Seven Segment Monitor](https://github.com/lexsysko/SevenSegmentMonitor)
* [Analyzing sound using PSD and FFT tools for trigger event monitoring.](https://github.com/lexsysko/SoundMonitor)
* [Thermometer Monitor is an asynchronous Bluetooth Low Energy (BLE)
  environmental monitor and telemetry logger.](https://github.com/lexsysko/ThermometerMonitor)

### Diagrams

[![](/assets/images/blog/50761a72434f1704-e6154e094a6e0c2c.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhDaiJYTowX1ffSHw1_UX_uOzonhpitUBUmnRdo2bGgMXZ9PI1yXl7tWR8icoD3J6iHAzNL_4msatVuSZL1Zo5SWKL_-eOFgNoI1hJILCO70gHPTXXRue2ritoiBbDo_e6xTKb1ioXdFgdLA-Fw3NzZALppIFXcj7QNG_frCUElYBKeclULTewSur09beM/s1567/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-09-18%20200606.png)  
*Mix temperature with ON/OFF states*

### Logs

```
Setup webcam video brightness to -30


RUNNING  DEBUG...
2026-10-02 03:38:03 [INFO] SevenSegmentMonitor: APP_VERSION='0.1.1'
2026-10-02 03:38:03 [INFO] SevenSegmentMonitor.services.db_writer_worker: [DB] init_db done
2026-10-02 03:38:03 [INFO] SevenSegmentMonitor.services.camera_producer: camera producer is ready. headless: True
2026-10-02 03:38:03 [INFO] SevenSegmentMonitor.services.vision_worker: vision worker is ready
2026-10-02 03:38:03 [INFO] SevenSegmentMonitor.services.db_writer_worker: [DB] Worker is ready
2026-10-02 03:38:03 [INFO] SevenSegmentMonitor: Multithreaded Headless Monitor Running...
2026-10-02 03:38:04 [DEBUG] SevenSegmentMonitor.vision.vision_libs: Auto rotated by 0.00deg
2026-10-02 03:38:05 [DEBUG] SevenSegmentMonitor.services.vision_worker: readout='234 001'
2026-10-02 03:38:05 [INFO] SevenSegmentMonitor.services.db_writer_worker: [DB] LOGGED: [1790912285] state=False, raw_data='234 001'
2026-10-02 03:38:13 [INFO] SevenSegmentMonitor.services.db_writer_worker: [DB] CLEANUP initialized every 86400 seconds.
2026-10-02 03:38:35 [DEBUG] SevenSegmentMonitor.vision.vision_libs: Auto rotated by 0.00deg
2026-10-02 03:38:35 [DEBUG] SevenSegmentMonitor.services.vision_worker: readout='234 001'
2026-10-02 03:39:06 [DEBUG] SevenSegmentMonitor.vision.vision_libs: Auto rotated by 0.00deg
2026-10-02 03:39:06 [DEBUG] SevenSegmentMonitor.services.vision_worker: readout='233 001'
2026-10-02 03:39:06 [INFO] SevenSegmentMonitor.services.db_writer_worker: [DB] LOGGED: [1790912346] state=False, raw_data='233 001'
...
2026-10-02 03:45:15 [INFO] SevenSegmentMonitor.services.db_writer_worker: [DB] LOGGED: [1790912715] state=False, raw_data='237 001'
2026-10-02 03:45:23 [INFO] SevenSegmentMonitor.handler_signal: 
[System] Received signal 15. Triggering graceful shutdown...
2026-10-02 03:45:24 [INFO] SevenSegmentMonitor.services.db_writer_worker: [DB] Worker stopped.
```
