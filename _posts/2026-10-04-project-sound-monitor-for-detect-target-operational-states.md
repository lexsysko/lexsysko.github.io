---
layout: post
title: "Project Sound Monitor for detect target operational states"
date: 2026-10-04 16:05:52 +0000
tags: ["Docker", "Grafana", "headless", "Linux", "numpy", "pyaudyo", "Python", "scipy", "SQLite", "Ubuntu"]
blogger_orig_link: https://lexsysko.blogspot.com/2026/10/project-sound-monitor-for-detect-target.html
---

Sound Monitor is an IoT sound and acoustic pattern monitoring tool designed to capture live audio, analyze spectral characteristics (PSD / Power), detect target operational states (e.g., compressor/device ON/OFF), and record state transitions to a SQLite database.

[![](/assets/images/blog/f4a165559d7990cd-6271e5171eb5b19b.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEge3a-ahriQet8UIocaLjqskeT23QCXlOdiL0jVIo1AxiJMSngGD2-OHG1nZnNMYdUsBUuV1lMqvePwcrlLHcGg8RSIfqaLz4Qu_pUJhq78Kl6YpWG5ijJ-aAZV89N__zgv3zXr0DenZpg6WWs7TYzNWS4y0R3BYJPE876Mz_qHMEeIARgqGjQ5cPU-TJA/s1440/compressor_spectrogram.png)  
*STFT spectrogram of a compressor sound recorded in advance using a smartphone’s digital microphone.*

[![](/assets/images/blog/b860c26e8dd1524f-c1482fd10c210eee.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjo1jJTcgHoCiF0s7basqKMhFXaMq1ejAgVpxTbentsNIpIQzkKCo5hQj7ucXh7BYHP-PCqeBmXOQ9Ghn2gfySzX9Z8p25kAwMViornvV09XcBg-PCPhYlzQK6HNI7WvRJ7wTGXQYkEFTzXqi8dw0AGGXk1OjVAOQxRPXhWZ1d-4uj_4mPdP5mHmJnCVgQ/s1500/diagram.png)  
*PSD Welch 60-1000Hz*

  

[![](/assets/images/blog/c553e2980bae0684-7b930ee639580d92.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhPIHwk8dIQTmjeyXeT7Kk1bGWxEio-hGSLa9w8sU3bNcct8eTqaHNt9cJn6sJf7UZzk3HFbPDgpwBKJ8wmfLimCe7n2INtmXyRMPJF4WAyzFJ_EVKBAxilTbqUKggdtr4MEpK9NDDh9OKrmdT65g_EZttsmXR4uQTe26IE8Qovrvr1Hg-Z9M_oDo5yznM/s1440/compressor_psd.png)  
*Welch PSD*

  

### Environmental conditions for measuring sound

[![](/assets/images/blog/0fa7616e7ca4470d-88dc60ad9b92d627.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjZbGkMv0gTWmaD8EjgMaGLhALBtX1SQ7l84J4i_dbF3BBG-ony4W0baLYBcwJmIVEWm8owEkVTcm40jpN7IebkoAMA6MfKCoY9zaFhOB8LzobfGh0T0Dsg7O_YCVRokwfitzSIzZoOrBHjsZ5Ra225tSUzEIbhB9sS8zpg7UzV6DfKx5e8CJJupCUKSTE/s2196/20260918_213347.jpg)  
*Laptop with a built-in microphone*

  

A laptop running Ubuntu Server with Docker installed was used for the setup. For measurements, only basic tools were available: an HP laptop and its built-in microphone.

Also, you can see the surge voltage protection device that will be used in the next project - [Seven Segment Monitor](https://github.com/lexsysko/SevenSegmentMonitor).

### Predictors & Detection Methods

SoundMonitor uses modular predictors inheriting from BasePredictor to classify audio states from Power Spectral Density (PSD) analysis.  
  
Historical project began for different version of application and trey different solution since main problem is detect off state when is real-life environment sound and it will generate false positive events in day time. Now dev version v.0.6.0 start to implement for sound analyze CNN torch audio with VGGish models based on audio sets from YouTube-8M.   
The sound quality of the built-in microphone is so poor and noisy that I’ve decided to stop the project and switch to a more reliable method: visually checking the refrigerator’s power consumption by reading the amperage, where the OFF state is 00.0A and the ON state is 00.7A - [Seven Segment Monitor](https://github.com/lexsysko/SevenSegmentMonitor).

## Power Predictor (Currently in use)

Mechanism: Computes the band power in dB from the PSD signal (compute\_band\_power\_db).

Classification: Compares the computed dB level against configured threshold levels (THRESHOLD\_ON\_POWER\_DB, with hysteresis bounds THRESHOLD\_ON\_POWER\_UP and THRESHOLD\_ON\_POWER\_DOWN).

Scoring: Generates normalized confidence scores [0.0, 1.0] based on relative power levels between the ON and OFF bounds and applies penalty filtering for out-of-bounds power surges when evaluating smoothed states.

  

[![](/assets/images/blog/dcbe36773c84afbc-8c66c1fba43b4e33.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgVEQbe03Xvz1kDg7oOrT9giAJJ7dq597TsXN63nF3WBpoQZuWzBIJEsje1XtjU2hnffU0Gch3uv8eGzIepU-YrXlShKHQAXZ1sU3oKBLl0-F4DYs0UIFL-_XbI7HfRsiZr6bZHA3u6q-MZP6dZ8enOVkdT8biLFAXFCnenLEo2VijnCLOxiRCV1NQOKZ0/s1500/live_power_off.png)  
*Power measurements for state OFF*

  

[![](/assets/images/blog/ed679e3a475a92cc-3137f609032af221.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgJCGEkwg2NG3mdFEFUlrt9PE4sJq_dEEFHjgks2q2oR9gXKBH5wISS5zskZPNQTqkw0HI8f2nb0oHZCWbxv1h0sYvob1r5UR6IRvO70DXxSHZGjrQLDWXJvnSeT1eEGELN_g3739idwo2bRHT0zG8wZsR4V7ID0olonySrQn2zSaDE4IEtzFykOt_l1mI/s1500/live_power_on.png)  
*Power measurements for state ON*

## PSD KNN (Alternative / Not currently active)

Mechanism: Template-matching k-Nearest Neighbors classifier on normalized PSD features.

Features & Alignment:

Employs signal alignment (compute\_aligned\_signals\_batch) via cross-correlation against baseline templates.

Normalizes PSD vectors (e.g. L2 normalization and standard scaling) and extracts peak frequencies and frequency band metrics.

Computes cosine similarity / distance against recorded training templates (template\_on\_\*.npz and template\_off\_\*.npz).

Status: Maintained in codebase as an alternative template-based classifier; PowerPredictor is currently instantiated in detector.py.

[![](/assets/images/blog/7a913b050c3b2467-ff3d42ce4aa93fd1.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiksNmT8j7pZQnd5kycp-AuoBH-PKVDAM0pvTEMB-Jt0d_mr8s7p-R4su52KJumzPHJacA9GwV6p-0anZ860v__JUVdvavFID80i5jSRmCs9TdXwgg9OYLQG6UGUg-YIC75WkOynPm2ONE7CMZrnWRqu6VkU_sCfjDLDm0gIPRu31Ha7ezwbr_BD9x70ys/s1500/compressor_on.png)  
*PSD Spectrum for compressor ON state*

[![](/assets/images/blog/5e4102ba331be743-dafc6aa7a3b35263.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgCKLaq2glYgRKxvDfPx8To-AohrjRrW1mw1T55uEMsXrUDP806rRVxHjEKCU8IiHp2gKzGqvkztz9LinaQvn2JTcWsr_AQIW1H88SOxJBAc8Bw3CXR1iJwyisJKqduKIbTMmvvrMKNGxFumhst0CZ1jmjQUfsplssE4a5pwVQ5XJJdOxkWF55qtprwU1Y/s1500/compressor_off.png)  
*PSD Spectrum for compressor OFF state*

  

### Diagrams

[![](/assets/images/blog/50761a72434f1704-e6154e094a6e0c2c.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhDaiJYTowX1ffSHw1_UX_uOzonhpitUBUmnRdo2bGgMXZ9PI1yXl7tWR8icoD3J6iHAzNL_4msatVuSZL1Zo5SWKL_-eOFgNoI1hJILCO70gHPTXXRue2ritoiBbDo_e6xTKb1ioXdFgdLA-Fw3NzZALppIFXcj7QNG_frCUElYBKeclULTewSur09beM/s1567/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202026-09-18%20200606.png)  
*Mix temperature with ON/OFF states*

[![](/assets/images/blog/88def8d87176bb8f-f085a4cfef2dbdd7.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgyvLXPjaKGV28ptwF7O4Twbyfj2TCbDVXhI0U_0hgrVOVqUw07Wp_glcaXuX60wcuNhLTH3CPtVjhiqK4XCGEgQ2X5Yuuw-IwuKxRFb8f95ZtssAHDVdj8IwrkSfihihQ6-IoG9Vi9zSJuvJcCX3Bd9cXdvTzyxgBHO-chjliqiYzu0W6G4EakOSwvdbw/s1596/tSM0lZ28Yx652qxR.png)  
*False positive errors occurring in the ON state*

Repositories: 

* [Analyzing sound using PSD and FFT tools for trigger event monitoring.](https://github.com/lexsysko/SoundMonitor)
* [Thermometer Monitor is an asynchronous Bluetooth Low Energy (BLE) environmental monitor and telemetry logger.](https://github.com/lexsysko/ThermometerMonitor)
* [Seven Segment Monitor](https://github.com/lexsysko/SevenSegmentMonitor)
