
# Klipper Notes

## Neptune 4 Pro

The [Klipper Documentation](https://www.klipper3d.org/Overview.html) is great. Very detailed but gets pretty technical.

One of the main skills to figure out is bed leveling. The Klipper docs have a [Bed Leveling](https://www.klipper3d.org/Bed_Level.html) page. Although you should have good luck with just the Benchie that comes on the SD card.

### KIAUH

Easy to install other things in Klipper using Kiauh https://docs.mainsail.xyz/setup/getting-started/kiauh

### Moonraker

The Neptune came with the Fluidd interface. I used KIAUH to install Moonraker on port 81 alongside the default Fluid on port 80.

### OctoEverywhere

https://octoeverywhere.com/getstarted?source=dash_control_panel

They start you out with a free trial but it's super cheap and allows for remote monitoring and control of the printer. Super cool software.

### MobileRaker

Going along with the Moonraker interface, the MobileRaker app connects with OctoEverywhere.

### Prusa Slicer tips

Definitely enable G-code thumbnails. 
https://docs.mainsail.xyz/overview/slicer/prusaslicer

Change Prusa Slicer file name output
https://help.prusa3d.com/article/list-of-placeholders_205643
> {input_filename_base}_{layer_height}mm_{filament_type[0]}_{printer_preset}_[print_time].gcode

I've uploaded my prusa slicer config bundle in this repo. It has my `Neptune-4 Pro (0.4 mm nozzle)` printer preset as well as some filament presets that I've found work well. But you're also welcome to experiment and ignore my settings.

### Camera setup

- Crowsnest
https://crowsnest.mainsail.xyz/

- Legacy crowsnest on Buster https://crowsnest.mainsail.xyz/faq/use-legacy-branch-on-buster

- Configuration for Legacyhttps://github.com/mainsail-crew/crowsnest/tree/legacy/v3#simple-configuration

- Moonraker Timelapse https://github.com/mainsail-crew/moonraker-timelapse/blob/main/docs/installation.md

- Example timelapse setup https://www.youtube.com/watch?v=Z2ut7lGHWuQ

### Manual Focus

Configuring on Neptune 4 Pro

Using Logitech C525 USB HD Webcam required some manual focus changes

https://www.kurokesu.com/main/2016/01/16/manual-usb-camera-settings-in-linux/

SSH into the printer and then run:
```bash
sudo apt update
sudo apt-get install v4l-utils
```

Then check the available devices. On the Neptune 4 a few Rockhip devices are listed, but the camera is the last one.
```
> v4l2-ctl --list-devices
  ...
  HD Webcam C525 (usb-xhci-hcd.0.auto-1):
        /dev/video4
        /dev/video5
```

Check the available controls
```
> v4l2-ctl -d /dev/video4 --list-ctrls

User Controls

                     brightness 0x00980900 (int)    : min=0 max=255 step=1 default=128 value=128
                       contrast 0x00980901 (int)    : min=0 max=255 step=1 default=32 value=32
                     saturation 0x00980902 (int)    : min=0 max=255 step=1 default=32 value=32
        white_balance_automatic 0x0098090c (bool)   : default=1 value=1
                           gain 0x00980913 (int)    : min=0 max=255 step=1 default=64 value=107
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=2 value=2
      white_balance_temperature 0x0098091a (int)    : min=2800 max=6500 step=1 default=5500 value=6365 flags=inactive
                      sharpness 0x0098091b (int)    : min=0 max=255 step=1 default=22 value=150
         backlight_compensation 0x0098091c (int)    : min=0 max=1 step=1 default=1 value=1

Camera Controls

                  auto_exposure 0x009a0901 (menu)   : min=0 max=3 default=3 value=3
         exposure_time_absolute 0x009a0902 (int)    : min=3 max=2047 step=1 default=166 value=664 flags=inactive
     exposure_dynamic_framerate 0x009a0903 (bool)   : default=0 value=1
                   pan_absolute 0x009a0908 (int)    : min=-36000 max=36000 step=3600 default=0 value=0
                  tilt_absolute 0x009a0909 (int)    : min=-36000 max=36000 step=3600 default=0 value=0
                 focus_absolute 0x009a090a (int)    : min=0 max=255 step=5 default=60 value=95
     focus_automatic_continuous 0x009a090c (bool)   : default=1 value=0
                  zoom_absolute 0x009a090d (int)    : min=1 max=5 step=1 default=1 value=1
```

Disable autofocus, set focus to 95, set sharpness to 150

```
> v4l2-ctl -d /dev/video4 --set-ctrl=focus_automatic_continuous=0
> v4l2-ctl -d /dev/video4 --set-ctrl=focus_absolute=95
> v4l2-ctl -d /dev/video4 --set-ctrl=sharpness=150
```


# Ender 3 V2 Neo

## Resonance Compensation

https://www.klipper3d.org/Resonance_Compensation.html

"Compute the ringing frequency of X axis as V · N / D (Hz), where V is the velocity for outer perimeters (mm/sec). For the example above, we marked 6 oscillations, and the test was printed at 100 mm/sec velocity, so the frequency is 100 * 6 / 12.14 ≈ 49.4 Hz."

- V * N / D
- X: 80 * 3 / 5.6 = 42.86 Hz
- Y: 80 * 3 / 5.5 = 43.63 Hz

## Extruder 

full_steps_per_rotation

1. actual_extrude_distance = <initial_mark_distance> - <subsequent_mark_distance>
2. rotation_distance = <previous_rotation_distance> * <actual_extrude_distance> / <requested_extrude_distance>
  
Test 1
- 53.4 = 60 - 6.6
- 26.2194 = 24.55 * 53.4 / 50

Test 2
- 50.45 = 60 - 9.55
- 26.4553746 = 26.2194 * 50.45 / 50

Test 3
- 43.31 = 97.87 - 54.56
- 26.38044547852 = 30.4553746 * 43.31 / 50

`guestimating` the actual vs. requested because the Resonance Compensation print doesn't have the expected gap
- 29.018490026372 = 26.38044547852 * `55` / 50
- 27.4356632976608 = 26.38044547852 * `52` / 50
- `28.25` = 27.4356632976608 * 51.48 / 50



