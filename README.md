# Waveshare ESP32-S3 Touch LCD 4.3 working with ESPHome

This repo documents the **correct way to control the backlight and implement proper sleep/wake** on:

Waveshare ESP32-S3-Touch-LCD-4.3  
https://www.waveshare.com/esp32-s3-touch-lcd-4.3.htm?sku=25948

---

## ✅ Final result

- Backlight fully OFF when idle
- Screen wakes on touch
- First touch does NOT press a button
- LVGL layout unaffected
- Works with Home Assistant entities

---

## ❗ The key discovery

The backlight is **NOT connected to an ESP32 GPIO**

It is controlled by the **CH422G IO expander**


