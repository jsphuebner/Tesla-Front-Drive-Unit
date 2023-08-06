# How To Build Your Own V8 Board

The following is a step-by-step guide for building an open inverter SDU V8 logic board.

## Prerequisites

- A basic understanding and ability to work with Arduino-like microcontrollers
- Computer w/internet access
- Visual Studio Code with Platform IO extension installed
- ST-Link V2 USB adapter
- ST-Link Utility software
- USB to Serial TTL adapter
- Soldering iron and solder
- Hot air gun (optional)

## 1. Download the Design Files:

1. [Gerber.zip](gerber.zip) (PCB Design) - You may need to explore the repo history to find it.
2. [Tesla_FDU_V8_BOM_JLC.xls](Tesla_FDU_V8_BOM_JLC.xls) (Bill of Materials)
3. [Tesla_FDU_V8_CPL.csv](Tesla_FDU_V8_CPL.csv) (Pick and Place)

## 2. Have PCBs made

The most popular approach is to have the PCBs made and assembled by an online service like JLCPCB. You usually have to order at least five boards, but you can choose to assemble only one if you want to save some money. The boards are delivered right to your door with all the components pre-soldered.

Alternatively, you can order empty PCBs from JLCPCB and manually solder all components yourself to further reduce your costs. You must purchase all components separately using the "Bill of Materials" (BOM) file as your guide. Fortunately, you can upload the BOM file to [LSCS](https://www.lcsc.com/bom) to quickly order all the components.

### To order fully assembled boards from JLCPCB.

1. Go to `jlcpcb.com` and click the "Instant Quote" button.
2. Add/upload PCB design file: `Gerber.zip`
3. Customize PCB options (optional)
4. Select "PCB Assembly" option
5. Upload Bill of Materials file: `Tesla_FDU_V8_BOM_JLC.xls`
    - NOTE: You may need to rename the "Package" column to "Footprint" for it to upload correctly.
6. Upload Pick and Place file: `Tesla_FDU_V8_CPL.csv`
7. Resolve any component stock issues (i.e., select replacements)
8. Finalize & order boards

## 3. Order plastic connectors and current sensors

_NOTE: Quantity is per board. Sensors are optional if you're re-using the existing sensors_

- 2x Current Sensors: https://www.mouser.com/ProductDetail/Melexis/MLX91209LVA-CAA-002-SP?qs=f9yNj16SXrLPMXdIg%2FG8eQ%3D%3D
- 2x 2-pin elbow connector: https://www.mouser.com/ProductDetail/306-S02BA-AIT2-1AK
- 1x 6-pin elbow connector: https://www.mouser.com/ProductDetail/306-S06B-AIT2-1AK
- 1x 20-pin Molex connector: https://www.mouser.com/ProductDetail/538-75757-5101
- 1x 18-pin low profile socket: https://www.mouser.com/ProductDetail/200-CES11201LD
- 1x 20-pin Molex harness connector: https://www.mouser.com/ProductDetail/538-33472-2002
- 20x Molex pins: https://www.mouser.com/ProductDetail/538-33012-2002-LP

## 4. Program ESP32 Wifi Module

- Clone or download the [esp32-web-interface](https://github.com/Pete9008/esp32-web-interface/tree/johu) repo and open it in VSCode
- **Switch to `johu` branch!**
- **Make sure the STM32 module is erased or held in reset mode!**
- Ensure the logic board is powered off
- Connect the USB Serial Adapter to the 6-pin ESP32 header: **Tx->Tx**, **Rx->Rx**, **GND->GND**.
- Run the following terminal command to determine what port the ESP32 is showing up on: `pio device list`
- Create a copy of the `platformio-local-override.ini.example` file in the root of the project and rename it to `platformio-local-override.ini`. 
- Modify the `platformio-local-override.ini` file to use the correct port from the step above.
- Ensure `PROG` and `GND` on the 6-pin header are bridged to boot into programming mode upon powering up.
- In **VSCode** > **Platform IO Tab** > **Project Tasks** > **Release** > **General**, click the `Monitor` task and wait for the monitor terminal to open and connect
- Now power on the board with +12V power at the main 20-pin connector (pins: 13 & 19)
- The text "waiting for download" should appear in the monitor terminal.
  - If you don't see the "waiting for download" text, make sure `PROG` and `GND` are bridged and the 6-pin header has a good connection. Also, make sure the STM32 is NOT running (see above). Leave the `Monitor` terminal open and reboot the board. Proceed to the next step only after you see the text.
- In **VSCode** > **Platform IO Tab** > **Project Tasks** > **Release** > **General**, click the `Upload` task and wait for a success message
- After success, the module will reboot back into normal mode. 
- Re-enter programming mode by rebooting the board with `PROG` and `GND` still bridged. Once again, you should see the "waiting for download" text in the `Monitor` terminal.
- Close the monitor terminal and any other instance connected to the ESP32.
- In **VSCode** > **Platform IO tab** > **Release** > **Platform**, click the `Upload Filesystem Image` task and wait for the success message
  - If this fails, double-check that you don't have any other processes connected to the ESP32
- Disconnect the USB Serial adapter and un-bridge `PROG` and `GND.`

## 5. Program STM32 Module

- Download STM32 assets from the [stm32-sine](https://github.com/jsphuebner/stm32-sine/releases) repository
  - `stm32-bootloader.hex`
  - `stm32-sine.hex`
    - You may need to browse the history of assets to find the latest versions
- Connect ST-Link 2 Adapter to 3-pin STM32 header: **SWCLK->CLK**, **SWDIO->DAT**, **GND->GND**
- Run [**ST-Link Utility**](https://www.st.com/en/development-tools/stsw-link004.html)
- **ST-Link Utility** > **Connect**
- **ST-Link Utility** > **Open** > select the `stm32-sine.hex` file
- **ST-Link Utility** > **Target** > **Program & verify** > confirm
- Repeat the last two steps with the `stm32-loader.hex` file
- **Red LED on board should start blinking**
- Disconnect the ST-Link adapter

## 6. Verify Successful Programming

- Connect to the ESP_XXXXXX wifi network
- Browse to `192.168.4.1`
- Verify the web interface is accessible
- Verify you can see and modify parameters
- Verify changes saved to Flash persist between reboots

## 7. Assemble plastic connectors

- Solder 18-pin IGBT connector to **UNDERSIDE** of board
- Solder 20-pin main wiring harness connector to **UNDERSIDE** of board
- Solder both 2-pin thermistor connectors
- Solder single 6-pin thermistor connector

## 8. Remove Tesla Board and Metal Tray

- Remove the inverter from the Tesla SDU
- Unscrew the two green T20 Torx screws from the big orange plastic phase wire holder
- Remove the big orange plastic phase wire holder from the three orange phase wires by _carefully_ working it **down**, towards the inverter body (away from wire ends) until you can get it off without breaking it, which can take a bit of finessing.
- Disconnect the remaining two thermistor connectors
- Unscrew the five remaining green T20 Torx screws
- Unscrew the three yellow T15 Torx screws surrounding the blue Molex connector retaining clip.
- Pop the metal tray with the board attached free from the 18-pin connector underneath using a plastic trim tool
- Carefully remove the tray and logic board up and away from the inverter and free it from the phase wires.

## 9. Separate Tesla Board From Metal Sensor Ring Tray

You can remove the stock Tesla board from the sensor ring metal tray in two different ways:

1. Leave sensors embedded in the sensor rings

   - Use a razor blade to carefully scrape away the resin coating on both sides of the PCB around the base of the sensor legs. 

   - Remove all but one of the remaining yellow T15 Torx screws. Keep one screw very loosely connected on the opposite side of the board. This will help prevent the board from rotating and damaging the sensor legs.

   - Use a plastic trim tool in-between the metal tray and board to apply slight upward pressure to the board. Be VERY careful not to apply too much upward pressure. Just enough so that when the sensors become unsoldered, the board will naturally lift off them.

   - Desolder the sensors from the board using a solder wick, flux, etc. This process can be challenging, so take your time. If you are struggling, you may need to scrape off more resin coating.

   - Gently lift the board off of the sensor legs, leaving the sensors embedded in place in the sensor rings

2. Remove sensors from the sensor rings

   - Using a hot air gun, blow hot air directly on the current sensors embedded in the sensor rings while applying gentle upward pressure on the board. The resin/goop in the sensor rings should start to melt, and the sensors should come free.

     - Be careful not to get the sensors too hot. Just hot enough so that the sensors come out of resin/goop.

   - If using new sensors, you can skip to step 10. 

   - With the board separated from the metal tray, use a razor blade to carefully scrape away the resin coating on both sides of the PCB around the base of the sensor legs.

   - Desolder and remove the sensors from the Tesla board, using a solder wick, flux, etc

## 10. Swap Large Molex Connector

Remove the large brown 20-pin Molex connector from the Tesla board. The blue retaining clip on the underside of the PCB must be released first. Use a tiny flathead screwdriver to remove the blue and brown plastic pieces from each other. These plastic pieces are very brittle and can break easily, so be careful.

Reinstall the large brown plastic Molex connector and blue retaining clip onto the open inverter board in reverse order.

## 11. Open Inverter Board Installation

- If you removed the sensors from the sensor rings, either re-install them or place the new ones into the holes left behind after removing them. **Be sure to orient the sensors so that the angled side of the sensor faces AWAY from the board.**

  - You can put a dab of hot glue or silicone in the hole with the sensors to help secure them.

- Align the eight tiny holes on the Open Inverter board with the eight tiny sensor legs and lower the board so that the sensor legs poke thru the holes in the board perfectly.

- Solder the sensor legs to the board.

- Re-install the yellow T15 Torx screws to secure the board to the metal tray.

- Double-check all connectors and make sure everything looks aligned correctly, with no bent pins, etc. When satisfied, re-install the tray and board back into the inverter.

- Re-install the big orange plastic phase wire holder and finally the green T20 Torx screws.

## Congratulations!

Your inverter has been unlocked and can be re-installed back into the motor!!