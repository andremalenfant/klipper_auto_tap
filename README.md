# Klipper Auto TAP
 Klipper module for automatic z-offset configuration on [Voron TAP](https://github.com/VoronDesign/Voron-Tap)

This module calculates the distance your toolhead lifts to actuate TAP. 
Doing this enables *automatic z-offset calculation*. i.e. make `G0 Z0.2` put the nozzle at 0.2mm above the bed. 
This is a fairly slow process since the toolhead needs to move by *step* (0.005), check if the endstop is still triggered, and repeat. 
Check [how it works](https://github.com/anonoei/klipper_auto_tap#how-does-it-work) for more information. 

On textured PEI, the offset may need to be slightly lowered to get proper first-layer squish. 
YMMV.

This is only known to work on QGL based printers, namely the Voron 2. 
If you use a different printer and want to help add support, please create an [issue](https://github.com/Anonoei/klipper_auto_tap/issues), or message me on Discord. 
Please include Auto TAP's console output so I can try to fix the issue.

**This module is under development**: Please ensure the calculated offset seems reasonable for your printer!

# Table of Contents
 - [Overview](https://github.com/anonoei/klipper_auto_tap#overview)
 - [How does it work](https://github.com/anonoei/klipper_auto_tap#how-does-it-work)
 - [Usage examples](https://github.com/anonoei/klipper_auto_tap#usage_examples)
 - [Using Klipper Auto TAP](https://github.com/anonoei/klipper_auto_tap#using-klipper-auto-tap)
   - [Installation](https://github.com/anonoei/klipper_auto_tap#installation)
   - [Configuration](https://github.com/anonoei/klipper_auto_tap#configuration)
   - [Macro](https://github.com/anonoei/klipper_auto_tap#macro)
   - [Moonraker Update Manager](https://github.com/anonoei/klipper_auto_tap#moonraker-update-manager)

## Overview
 - License: MIT


## How does it work?
1. Home your printer
   - When your printer homes Z, your toolhead is lifted until the endstop is triggered, making that lifted point your z0
2. Move to park position
3. For each sample, tap!
   1. Probe the bed, and keep it at the position where endstop triggers
   2. Slowly lift the toolhead by *step* until the endstop isn't triggered
   3. Calculate travel distance *travel* = `abs(probe z - measure distance)`
   4. Save the resulting probe, measure distance, and travel
4. Calculate Z-Offset based on `CALC_METHOD`
   - QGL: `travel_mean \* 2`
   - STA: `measure_mean + (travel_mean/2)` (under development)

## Usage examples
One and done:
 - Run `AUTO_TAP`, and save the z-offset. Adjust as needed based on build surface material


Before starting print:
 - In your `PRINT_START` macro, add `AUTO_TAP` after homing and leveling have been complete

## Using Klipper Auto TAP
### Installation
To install this module, you need to clone the repository and run the `install.sh` script

#### Automatic installation
```
cd ~
git clone https://github.com/Anonoei/klipper_auto_tap.git
cd klipper_auto_tap
chmod +x install.sh
./install.sh
```
#### Manual installation
 1. Clone the repository
    1. `cd ~`
    2. `git clone https://github.com/Anonoei/klipper_auto_tap.git`
    3. `cd klipper_auto_tap`
 2. Link auto_tap to klipper
    1. `ln -sf ~/klipper_auto_tap/auto_tap.py ~/klipper/klippy/extras/auto_tap.py`
 3. Restart klipper
    1. `sudo systemctl restart klipper`

### Configuration
Place this in your printer.cfg
```
[auto_tap]
```
The values listed below are the defaults Auto TAP uses. You can include them if you wish to change their values.
```
[auto_tap]
x: Unset             ; X position to probe, should be the middle of your bed
y: Unset               ; Y position to probe, should be the middle of your bed
z: 10                ; Z position to park
probe_to: -2         ; Lower probe until it triggers, or reaches this value
set: True            ; Set probe offset after calculation
settling_probe: True ; Perform a dummy probe before starting
calc_method: Unset   ; Defaults to your printers leveling method, "QGL", or "STA". You probably don't want to change this
stop: 2.0            ; Lift Z up to this amount for TAP to de-actuate
step: 0.005          ; Lift Z by this amount each check
samples: Unset       ; Number of samples to take, Defaults to your config's probe sample count
retract: Unset       ; Lift up by this amount at end, Defaults to your config's probe retract distance
probe_speed: Unset   ; Probe at this speed, Defaults to your config's probe travel speed
lift_speed: Unset    ; Retract at this speed, Defaults to your config's probe lift speed
travel_speed: 1000   ; Speed for travel to park position
```
### Macro
Run the klipper command `AUTO_TAP`. You can also use the arguments below
Argument       | Default | Description
-------------- | ------- | -----------
X              | Unset   | X position to probe, Defaults to the middle of the x rail `(max - min)/2`
Y              | Unset   | Y position to probe, Defaults to the middle of the y rail `(max - min)/2`
Z              | 10      | Z position to park
PROBE_TO       | -2      | Lower probe until it triggers, or reaches this value
SET            | 1       | Set probe offset after calculation
SETTLING_PROBE | 1       | Perform a dummy probe before starting
CALC_METHOD    | Unset   | Defaults to your printers leveling method, "QGL", or "STA". You probably don't want to change this
STOP           | 2.0     | Lift Z up to this amount for TAP to de-actuate
STEP           | 0.005   | Lift Z by this amount each check
SAMPLES        | Unset   | Number of samples to take, Defaults to your config's probe sample count
RETRACT        | Unset   | Lift up by this amount at end, Defaults to your config's probe retract distance
PROBE_SPEED    | 1.0     | Probe at this speed
LIFT_SPEED     | Unset   | Retract at this speed, Defaults to your config's probe lift speed
TRAVEL_SPEED   | 1000    | Speed for travel to park position
FORCE          | 0       | Force AUTO_TAP to run, even if it was calculated previously

If you set values under *Configuration*, those will become the defaults.

If you run `AUTO_TAP` again, it will set the z-offset to the last calculated value unless you run `AUTO_TAP FORCE=1`.

### Moonraker Update Manager
```
[update_manager klipper_auto_tap]
type: git_repo
path: ~/klipper_auto_tap
origin: https://github.com/anonoei/klipper_auto_tap.git
primary_branch: main
install_script: install.sh
managed_services: klipper
```
