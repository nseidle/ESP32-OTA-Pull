# ESP32-OTA-Pull
An Arduino library to facilitate simple ESP32 "**pull**"-based OTA updates

## "Pull" Design
There are a number of good Arduino libraries out there for OTA ("Over The Air") firmware updates.  Example: [ArduinoOTA](https://github.com/jandrassy/ArduinoOTA) and [AsyncElegantOTA](https://github.com/ayushsharma82/AsyncElegantOTA).  These libraries use a "push" technology, wherein you identify a target device you'd like to update and push/upload a new firmware to it.

[ESP32-OTA-Pull](https://github.com/mikalhart/ESP32-OTA-Pull) uses a different, "pull", strategy. In this scenario, you post new firmware image(s) to a webserver, and your WiFi-enabled devices find the new images and update themselves. It scales. You don't have to target each device individually; as long as your devices can connect to WiFi, they can update themselves.

## Setup
With ESP32-OTA-Pull, you post two files to your webserver:
- Your new compiled program ("**image**")
- A small human-readable (text) **JSON** format "filter" file that helps decide whether to download and install the image.

## Filtering
A minimal JSON filter file simply tells the library what the new image's version is and where to download it:

```
{
  "Configurations": [
    {
      "Version": "2.0.0",
      "URL": "https://example.com/myimages/example.esp32_dev.v2.bin"
    }
  ]
}
```

This example informs the library that Version 2.0.0 of the sketch has been posted at the specified location.  Sketches running a version less than 2.0.0 will be updated.  Sketches greater than 2.0.0 will not be, unless downgrades are enabled.

A more elaborate filter file might also constrain the update based on the "Board" name&mdash;by default the ARDUINO_BOARD symbol that is predefined based on which board type was selected at build&mdash;the "Device" name (the MAC address), and an optional user-specified "Config" string.  In the example below, a device will be updated only if it was built with the specified board profile (ESP32_DEV), its MAC address and Config strings match exactly, and it is currently running a sketch with version less than 2.

```
{
  "Configurations": [
    {
      "Board": "ESP32_DEV",
      "Device": "24:6F:28:AD:FF:04",
      "Config": "4MB",
      "Version": "2.0.0",
      "URL": "https://example.com/myimages/example.esp32_dev.v2.bin"
    }
  ]
}
```

## Getting started with ESP32-OTA-Pull
1. Install the [ESP32-OTA-Pull](https://github.com/mikalhart/ESP32-OTA-Pull) and [ArduinoJson](https://github.com/bblanchon/ArduinoJson) libraries
2. Make sure to choose a partition scheme that includes OTA when you build your sketch.
3. Generate the new firmware binary using the Arduino IDE's **Sketch/Export Compiled Binary** menu item.
4. Upload the firmware image .bin file to your webserver, e.g. "https://example.com/myimages/example.esp32_dev.v2.bin".
5. Open a text editor and create a small JSON file like the one above---the example sketch shows you what it should contain---that points to and documents the .bin image(s) you uploaded in step 4.
6. Post the JSON file on your webserver, e.g. http://example.com/myimages/example.json
7. In your sketch connect to WiFi, then call

```
       ESP32OTAPull ota;
       int ret = ota.CheckForOTAUpdate("http://example.com/myimages/example.json", VERSION);
```

If the JSON file says that the "Version" number of the posted image is greater than the current VERSION supplied in the call to CheckForOTAUpdate, then it will download and install it!

## How it works
The **CheckForUpdate** method downloads the specified JSON file and begins iterating through all the "Configurations" provided.  When it finds one that matches the Configuration of the current sketch, it compares its Version string with the value passed into the method.  If the posted Version is greater than the local one, an update is done, subject to the constraints of the "Action" parameter.  If the posted Version is less, then the update is only done if AllowDowngrades has been set.

## Extended techniques
See the "Further-OTA-Examples" sketch for examples on how you can:
- Add a callback function to report update progress (**SetCallback()**)
- Request the object NOT to do an update, but only report whether one is available. (Action parameter **ESP32OTAPull::DONT_DO_UPDATE**)
- Request it to download the update, but not do the necessary reset to trigger it. (Action parameter **ESP32OTAPull::UPDATE_BUT_NO_BOOT**)
- Specify a "Config" string to match any "Config" string in the JSON filter file. (**SetConfig()**)
- Permit downgrades. (**AllowDowngrades()**)
- Override the default Board or Device strings if needed.  (**OverrideBoard()** and **OverrideDevice()**)

