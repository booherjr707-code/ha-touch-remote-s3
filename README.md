# Home Assistant Touch Remote

A 4" touchscreen remote for Home Assistant on the Raspberry Pi, built on
[openHASP](https://github.com/HASwitchPlate/openHASP). It shows the Sainlogic SA68
weather station, the driveway alarm and the lightning detector, and has buttons to
turn lights and switches on and off. Home Assistant sends it what to show over MQTT,
so you can change the screen later without reflashing.

## Parts

- ESP32-S3 DevKitC-1 **N16R8** (16 MB flash, 8 MB PSRAM) ([Amazon B0H7HWWVBN](https://www.amazon.com/dp/B0H7HWWVBN))
- Hosyond 4.0" 480x320 SPI touch display, ST7796S + XPT2046 touch ([Amazon B0CKRJ81B5](https://www.amazon.com/dp/B0CKRJ81B5))

## Wiring

Open `WIRING.html` for the diagram and a checklist.

| Display pin | ESP32-S3 pin |
|---|---|
| VCC | 5V |
| GND | GND |
| CS | 10 |
| RESET | 8 |
| DC/RS | 9 |
| SDI (MOSI) | 11 |
| SCK | 12 |
| LED | 7 |
| SDO (MISO) | not connected |
| T_CLK | 12 (same as SCK) |
| T_CS | 14 |
| T_DIN | 11 (same as SDI) |
| T_DO | 13 |
| T_IRQ | not connected |

## 1. Flash the firmware

`firmware/openhasp-hosyond-s3-full-16MB.bin` is openHASP 0.7 built for this board and
screen (build settings in `openhasp/hosyond-st7796-s3.ini`). Plug the S3 into your Mac
with the USB-C port labeled **COM** (or UART), then:

```sh
~/.platformio/packages/tool-esptoolpy/esptool.py --chip esp32s3 \
  write_flash 0x0 firmware/openhasp-hosyond-s3-full-16MB.bin
```

Only the first flash needs a cable. Later updates can go over Wi-Fi from the remote's
web page (Firmware > upload `openhasp-hosyond-s3-update.bin`).

## 2. First start

1. The remote makes its own Wi-Fi network named `HASP-` plus 6 letters/numbers, and
   the screen shows a QR code for it. Join it from your phone (password
   `haspadmin`) and enter your home Wi-Fi. It only works on 2.4 GHz Wi-Fi.
2. Open the address shown on the screen in a browser. That's the remote's web page.
3. **Display settings:** set Rotation to landscape (try 1 or 3) and run **Calibrate**
   so taps land where you touch.
4. **Time settings:** time zone `America/Chicago`.
5. **MQTT settings:** Plate name `dashboard`, and the Raspberry Pi's address, user and
   password for the Mosquitto add-on.
6. **File editor:** upload `pages.jsonl` from this folder, then restart the remote.

## 3. Home Assistant

1. Install the **Mosquitto broker** add-on (Settings > Add-ons) and the **MQTT**
   integration.
2. Install **HACS**, then the **openHASP** integration from it.
3. Copy `homeassistant/openhasp.yaml` into `/config/packages/` and restart Home
   Assistant. The file's header explains the one line `configuration.yaml` needs.
4. Replace the entity names marked `CHANGE ME` with your own (Settings > Devices &
   services > Entities). The lightning detector's names are already correct.

When the driveway alarm goes off, the remote jumps to the driveway page on its own.

## The pages

1. **Weather:** big temperature, feels-like, humidity, wind, gust and direction, rain
   today, and the National Weather Service conditions.
2. **Driveway & storms:** green "Driveway: clear" that turns red on motion, time of
   the last motion, and lightning distance and strike count.
3. **Controls:** six on/off buttons for lights, fans or switches.

Every page has the clock and date at the top and Back / Home / Next buttons at the
bottom. Swiping also changes pages.

## Credits

openHASP by the HASwitchPlate project (MIT license). Setup written by
[Claude Code](https://claude.com/claude-code), Anthropic's AI coding assistant, for
James Booher's ESP32 weather projects.
