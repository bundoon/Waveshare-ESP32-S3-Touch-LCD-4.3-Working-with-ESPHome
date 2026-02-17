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

## 🔑 The key discovery

The backlight is **NOT connected to an ESP32 GPIO**

It is controlled by the **CH422G IO expander**

EXIO2 → DISP → Backlight enable

🔌 Backlight control (working)


So PWM brightness control will not work.

Use ON/OFF control instead.

---

## 🔌 Backlight control (working)

```yaml
i2c:
  sda: GPIO8
  scl: GPIO9
  scan: true
  id: bus_a

ch422g:
  - id: exio
    i2c_id: bus_a

output:
  - platform: gpio
    id: backlight_en_out
    pin:
      ch422g: exio
      number: 2

light:
  - platform: binary
    id: disp_backlight
    name: Display Backlight
    output: backlight_en_out
    restore_mode: ALWAYS_ON


💤 Sleep / wake logic (no ghost presses)

We wake into a blank LVGL page for 500ms.

globals:
  - id: last_touch_ms
    type: uint32_t
    initial_value: "0"

  - id: is_sleeping
    type: bool
    initial_value: "false"
script:

  - id: user_activity
    mode: restart
    then:
      - lambda: 'id(last_touch_ms) = millis();'

      - if:
          condition:
            lambda: 'return id(is_sleeping);'
          then:
            - lambda: 'id(is_sleeping) = false;'
            - light.turn_on: disp_backlight
            - lvgl.page.show: wake_page
            - delay: 500ms
            - lvgl.page.show: main_page
          else:
            - light.turn_on: disp_backlight


  - id: go_to_sleep
    then:
      - lambda: 'id(is_sleeping) = true;'
      - lvgl.page.show: wake_page
      - light.turn_off: disp_backlight

interval:
  - interval: 500ms
    then:
      - if:
          condition:
            lambda: |-
              if (id(is_sleeping)) return false;
              return millis() - id(last_touch_ms) > 30000;
          then:
            - script.execute: go_to_sleep


🖥 LVGL blank wake page

lvgl:
  pages:

    - id: wake_page
      bg_color: 0x000000
      widgets: []

    - id: main_page
      # your full UI

⚠ Strapping pin warnings

You will see:

GPIO3 is a strapping pin
GPIO46 is a strapping pin

This is normal for this display.

Safe to ignore as long as you don’t add external pull-ups/downs.

📸 Working result


![IMG20260217210120](https://github.com/user-attachments/assets/d941fe42-7c47-4634-8ca1-b6a0c3c1f4e7)


## ✅ Tested with

- ESPHome 2026.x
- Home Assistant 2026.1.
- Waveshare ESP32-S3 Touch LCD 4.3 (SKU 25948)






