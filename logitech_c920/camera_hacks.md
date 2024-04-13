## To disable auto focus and auto exposure by default:

Add file to `/etc/init.d/` with name that makes it run last i.e `S999_cam_init` needs to at least run after the non-creality camera script

```
#!/bin/sh

echo "Initialise Logi C920"
v4l2-ctl -d /dev/video4 --set-ctrl=focus_auto=0
v4l2-ctl -d /dev/video4 --set-ctrl=focus_absolute=40
v4l2-ctl -d /dev/video4 --set-ctrl=exposure_auto=1
v4l2-ctl -d /dev/video4 --set-ctrl=exposure_absolute=500

exit 0
```

`v4l2-ctl -d /dev/video4 --list-ctrls`


## To have macros that can control the camera

i.e. in Mainsail:
![image](https://github.com/ajchellew/3dprinting/assets/17216760/6297a9c4-1770-4b5b-914f-d3c1a484f274)

Install https://guilouz.github.io/Creality-K1-Series/helper-script/klipper-gcode-shell-command/

In `printer.cfg`
```
[gcode_shell_command cam_focus_manual]
command: sh /usr/share/cam-scripts/cam_focus.sh
verbose: True

[gcode_shell_command cam_zoom_manual]
command: sh /usr/share/cam-scripts/cam_zoom.sh
verbose: True

[gcode_shell_command cam_exposure_manual]
command: sh /usr/share/cam-scripts/cam_exposure.sh
verbose: True


[gcode_shell_command pan_left]
command: v4l2-ctl -d /dev/video4 --set-ctrl=pan_absolute=-36000
verbose: True

[gcode_shell_command pan_centre]
command: v4l2-ctl -d /dev/video4 --set-ctrl=pan_absolute=0
verbose: True

[gcode_shell_command pan_right]
command: v4l2-ctl -d /dev/video4 --set-ctrl=pan_absolute=36000
verbose: True

[gcode_macro FOCUS_MANUAL]
gcode:
    {% set focus = params.VALUE|default(40)|int %}
    #{% set output_txt = ["Output Text:"] %}
    #{% set _dummy = output_txt.append("Focus: %s" % focus) %}
    #{action_respond_info(output_txt|join("\n"))}
    RUN_SHELL_COMMAND CMD=cam_focus_manual PARAMS={focus}

[gcode_macro ZOOM_MANUAL]
gcode:
    {% set zoom = params.VALUE|default(1)|int %}
    {% set zoom_actual = zoom * 100 %}
    RUN_SHELL_COMMAND CMD=cam_zoom_manual PARAMS={zoom_actual}

[gcode_macro EXPOSURE_MANUAL]
gcode:
    {% set exposure = params.VALUE|default(500)|int %}
    RUN_SHELL_COMMAND CMD=cam_exposure_manual PARAMS={exposure}

[gcode_macro PAN_LEFT]
gcode:
    RUN_SHELL_COMMAND CMD=pan_left

[gcode_macro PAN_CENTRE]
gcode:
    RUN_SHELL_COMMAND CMD=pan_centre

[gcode_macro PAN_RIGHT]
gcode:
    RUN_SHELL_COMMAND CMD=pan_right
```

Scripts I put in `/usr/share/cam-scripts` they don't need to be here.

`cam_zoom.sh`
```
#!/bin/sh

echo "zoom is: $1"
v4l2-ctl -d /dev/video4 --set-ctrl=zoom_absolute=$1
```

`cam_focus.sh`
```
#!/bin/sh

echo "focus is: $1"
v4l2-ctl -d /dev/video4 --set-ctrl=focus_auto=0
v4l2-ctl -d /dev/video4 --set-ctrl=focus_absolute=$1
```

`cam_exposure.sh`
```
#!/bin/sh

echo "exposure is: $1"
v4l2-ctl -d /dev/video4 --set-ctrl=exposure_auto=1
v4l2-ctl -d /dev/video4 --set-ctrl=exposure_absolute=$1
```
