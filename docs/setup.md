# Setup

# Pre-Requisites

- Required:
    - Decent spec computer, I’m using a Lenovo Legion 5, Geforce 3050
    - Unreal Engine 5.0
    - 1 HTC Vive Base Station
    - 1 HTC Vive Tracker
    - Green Screen, preferably a large wrapping background with floor.
    - Hot Shoe Mount adapter for vive tracker. Here’s the one that I made and tailored this project to.
    - DSLR camera with a hot shoe mount and HDMI out capabilities
    - Mini HDMI to HDMI adapter
    - Video capture card, I’m using Cam link 4k
- Highly Recommended:
    - 2 Tripods with 0.25 inch mounting screws for the base station and camera calibration
    - Studio Lights, for lighting your green screen and your subject
    - AC adapter for your camera, so that battery life is not a factor

# Stage Setup

- Create a green screen studio setup
- Create a dimensionally accurate 3D model of your green screen.

# Vive Setup

- Install Steam, SteamVR
    - If using a tracker without a headset, you must change a few configuration files. These changes can be found [here](https://medium.com/@Adamcbrz/vive-tracker-with-out-hmd-part-1-572b7a09ce49). This will need to be done on every new install you make of Steam VR
- plug in base station
- plug in usb bluetooth dongle for the tracker to communicate with
- make sure tracker is charged and turned on (goes to sleep after inactivity)
- Attach the vive tracker to our camera using the hot shoe mount and the adapter, make sure that the tracker is rotated so that the indicator light is rotated toward the bottom. On the back the gold pins should be on top

# Unreal, Level Setup

- Enable plugins
    - Virtual production Utilities (Live Link XR)
    - Steam VR
    - Procedural Mesh Component
    - MIDI Device Support
- (**TODO, be more clear add detail, test with Unreal 5.1)** Switch to DirectX (11?) in the project settings, the video players (Wmf) needed to display the camera feed live on our material currently do not work in the latest DirectX.
    - This also means that some of Unreal 5’s latest features such as Nanite will not work. It appears that realtime lighting with Lumen does work.
- Add vive tracker in the “live link” window (Must be done each time Unreal is opened)
- import master audio wav file and all stems
- create audio synesthesia audio analysis assets for each audio file you’d like to visualize
- Add in the data broadcaster and the virtual physical bridge
- Set the broadcast event delay to the same value as your XR tracking offset (var is called “XR Delay” in Data Broadcaster)
- …

# Blueprint Setup

## Virtual Physical Bridge

- This is the Blueprint that handles syncing the “Cine Camera Actor” and our real world physical camera
- Video Plane component is a piece of geometry that is used to bring our live video feed from our physical camera into the scene
- Setup and Usage
    - ***TODO: Before Runtime (OUTDATED, New Calibration Settings Added Which allows baked calibration)***
        - Focal Length must be set to match the focal length and sensor size of the physical camera before play starts, this setting can be found under the “All” section in the details panel. Currently sensor size is set to match a Nikon d5600 camera.
        - Marked settings are settings that should match the physical camera’s settings at the beginning of play, but can be manipulated during runtime
            - Sensor Size
            - Focal Length*
            - Aperture* (opening size)
            - Focus Distance*
            - Distance to Subject (not applicable when “use cloud green screen” option is enabled)
            - Aperture Location relative to Tracker
                - This is the XYZ coordinates (in centimeters) of the aperture, camera pinhole, from our vive tracker in physical space.
                - Because the tacker is mounted above our camera in the hot shoe, the aperture’s relative location is below the tracker, the Z coordinate will be negative.
                - The X coord will indicate how far forward (positive) or behind (negative) the aperture is from our tracker. The aperture is found in our lens, so the X coord will be positive.
                - The Vive Tracker is designed so that its coordinate origin is at the mount point (the surface of where the screw screws in). Use this as your reference point to measure from.
        - Once you load an instance of the virtual physical bridge in the level, go to the details panel of that instance and set the “auto possess player” setting to “player 0” in order to lock the viewport to the view of our cine camera and capture user input.
        - Additional Settings can be found in the “All” panel of the Details Menu for this actor in the level
            - If you have a vive tracker and live link set up
                1. provide the name of your tracker in the “LiveLinkSubjectName” variable (should be a drop down)
                2. Enable the “use vive tracker” setting in the details panel
            - There are other checkboxes that can be enabled in order to see debug messages during runtime
            - Hologram option will change the video plane material to a hologram
        - **Baked Calibration** (optional, but highly recommended makes things a lot easier for running multiple times)
    - ***During Runtime***
        - When the game is started, the live video source will need to be set again, during runtime.
            1. If your mouse is captured by the game: press “Shift+F1” to release it. 
            2. Open the asset “Content>Media>NewMediaPlayer” and set the source in the top left of the preview window to “Cam Link 4k” or whatever video source is being used.
        - If “Use Cloud Green Screen” is enabled **TODO: AND BAKED CALIBRATION IS NOT USED**
            1. first, set the tracker down in the center of the room, then press “right alt”
            2. now set the tracker down in the center of the back wall on the floor, then press “right alt” again
            - Ensure that the tracker is visible to the base station during this process by looking at the tracker’s status in the live link window
            - If calibrated correctly, the virtual camera will be looking at the back wall of the virtual green screen when the tracker is facing the back wall of the physical green screen.
            - If extra perfectly calibrated and tracking is in sync with video, the virtual green screen will only show stuff that is between the camera and the green screen, no walls outside the green screen, the floor, etc.
        - Once the camera is placed somewhere it is directly facing the subject. Set the reference transform for the tracker with “right alt” (make sure that input is being captured by the game clicking the viewport will capture the mouse and keyboard input, mouse should not be visible).
            - from now on, your physical movement of the camera and tracked will be measured in reference to the position you just set with “right alt”. the video plane location will be automatically be placed the distance in front of the camera you specified under “dist Cm to subject”.
        - The settings that were asterisked earlier can be adjusted with various keyboard controls or mapped MIDI controls. You can check the blueprint for these, but it should be the number keys, “O←→P”, “K←→L”. Enable the “Debug Virtual Camera”  option before runtime to interactively see the mappings as you press the keys.

# Problems, Improvements on the Horizon

- The media player does not work with DirectX 12, so if we are using the live video plane with the video material, we cannot use other DirectX 12 dependent features such as Nanite
- Green screen should be calibrated according to positions on the back wall (marked with slightly different green pieces of masking tape), rather than floor positions. This will make our calibration more accurate, easier to diagnose
- There are other aspects of the virtual production project not included in repo yet which allow for music playback and visual effects.