# IRB DroneLab Crazyflie + Crazyswarm Vicon Instructions

**RAAS Lab**: Iribe Center 3207 <br/>
**Faculty**: Pratap Tokekar <tokekar@umd.edu> <br/>
**Authors**: Griffin Bonner <gbonner1@umd.edu>, Guangyao Shi <gyshi@umd.edu> <br/>

## Description
These instructions serve to explain the process and issues faced while testing the Crazyflie 2.1 micro drone with the Vicon motion capture system in the IRB drone lab. 

## Hardware Dependencies
| Device | Link |
| ------ | ------ |
| **Crazyflie 2.1** | https://store.bitcraze.io/collections/kits/products/crazyflie-2-1 |
| **Motion Capture Marker Deck** | https://www.bitcraze.io/products/motion-capture-marker-deck/ |
| **Motion Capture Markers (6.5mm)** | https://store.bitcraze.io/collections/positioning/products/reflective-markers |
| **Crazyradio PA** | https://www.bitcraze.io/products/crazyradio-pa/ |
| **USB Hub** | (many options on Amazon.com) |

## Software Dependencies
| Name | Source |
| ------ | ------ |
| **Crazyswarm** | https://crazyswarm.readthedocs.io/en/latest/index.html |
| **Crazyflie Client** | Ubuntu Software Center |

## System Dependencies
| Name | Version |
| ------ | ------ | 
| **Ubuntu** | 20.04 | 
| **Python** | 3.7 |
| **ROS** | Noetic |

## Vicon Dependencies
| Name | Source |
| ------ | ------ |
| **Vicon Callibration Guide** | https://youtu.be/tfNm0jKSwk0 | 

### Operating System (Ubuntu 20.04 LTS)
It is recommended to install Ubuntu 20.04 natively for Crazyflie and Crazyswarm development, windows system for linux (WSL) is not supported and virtual machines may introduce error inducing latency. The crazyswarm package also supports Ubuntu 18.04, Python 2.7, and ROS Melodic, but these were not tested personally and I would recommend using the versions listed above.

### Crazyflie Python Virtual Environment
It is recommended to create a separate python virtual environment for using the bitcraze provided tools including the crazyflie client and software provided to clash the crazyradios. These tools use a different version of python compared with crazyswarm, so for ease of use it is recommended to create a python virtual environment which uses the correct version which should be activated before using the tools mentioned above, but must be deactivated before running the crazyswarm package. Explicit instructions are given below.

### Crazyswarm Installation
Download the crazyswarm package and follow the instructions provided on the *installation* page. The instructions should be straightforward, choose the *Physical Robots and Simulation* option. It is possible that the one or more dependencies may fail due to improper versioning, if this occurs it will need to be debugged on a per case basis. 

### Crazyswarm Configuration
Proceed to the *configuration* page of the crazyswarm documentation. Every step before *configure external tracking system* should be followed according to the documentation. It is recommended to download the crazyflie client separately (not included in the crazyswarm package) from the ubuntu software center, you can also build it from source, but this is unecessary and may add complications. Some additional details are given below. 

#### Crazyflie Software Configuration
In keeping with the configuration documentation, flash the crazyflies with the crazyswarm compatible firmware, assign unique identifiers (following the enumeration convention discribed) and assign radio channels (which need not be unique). The assigned channel corresponds to a single *Crazyradio PA*, thus if you are running two crazyflies, setting them to the same channel will default to a single radio whereas setting them to unique channels will require two physical radios. We have experimentally determined that at maximum two crazyflies should be assigned the same radio address, thus if you run with four crazyflies you will need two crazyradios. Firmware, identification, and radio channels will not be assigned automatically, you need to launch the crazyflie client, plug in the crazyflie via usb, enter *bootloader mode* by holding the power button for two full seconds until two blue lights appear on the rear arms, then from the top menu select *crazyflie* -> *bootloader* select the approved firmware version, we used *version 2021.6* which is the newest approved version, and select flash both STM32 + nRF51. After flashing the firmware, powercycle the crazyflie. The connection method in the top left of the client should auto-populate with USB://dev.. select connect and then from the top drop down menu find *crazyflie* -> *configure* and enter the chosen identifier and radio channel. Continue with the crazyswarm configuration documentation...

The *Vicon* tab is the relevant tab because this is the physical hardware we have in the IRB drone lab. The vicon tracker software runs on the lab computer and sends the motion capture data over wifi or ethernet to the local computer running the crazyswarm software. The following modifications will be made in the *hover_swarm.launch* file with path specified by the documentation, this file is the ros launch file which will contain all crazyswarm configuration parameters. 

    # ros_ws/src/crazyswarm/launch/hover_swarm.launch
    motion_capture_type: "vicon" # one of none,vicon,optitrack,qualisys,vrpn
    object_tracking_type: "libobjecttracker" # one of motionCapture,libobjecttracker
    send_position_only: False # set to False to send position+orientation; set to True to send position only
    vicon_host_name: "192.168.78.11"

There are two *object tracking modes* provided by crazyswarm, these determine where the identification of and discrimination between drones will take place. If the *unique marker configurations* is selected, the drones must have different marker configurations, (which becomes very difficult for more than four drones) the vicon software will track the drones and each drone must be labeled within this software. If the *duplicated marker configurations* is selected, the drones must start at predetermined starting locations specified in the *crazyflies.yaml* configuration file. The crazyswarm software will discriminate which drone corresponds to which crazyflie ID during launch. It is recommended to select the duplicated marker option and follow the instructions given. A photo of a working and experimentally ideal marker configuration is attached to this repository.

    # ros_ws/src/crazyswarm/launch/hover_swarm.launch
    object_tracking_type: "libobjecttracker"

The user should create their own ros launch file and understand the implications of the parameter configurations described above, however, a sample version of the *hover_swarm.launch* file is provided in this repository which contains the configurations.

#### Define Crazyflie Types
The "default" marker configuration according to the crazyswarm documentation is not the same as the experimentally determined ideal configuration given in this tutorial. It is important that the crazyswarm package understands the geometry of the markers in the duplicated marker configuration. If a new configuration needs to be used, follow the instructions provided by the crazyswarm documentation. If you plan to use the configuration provided by this document, the points are given below and can be added to *crazyflieTypes.yaml*. It should be obvious that if a completely different drone is used with the crazyswarm software, many additional parameters will need to be changed as the shape and weight of any other drone will significantly affect the dynamics and the stock parameters are tuned for the crazyflie. 

    # ros_ws/src/crazyswarm/launch/crazyflieTypes.yaml
    markerConfigurations:
    "0":  # for standard Crazyflie
      numPoints: 4
      offset: [0.0, -0.01, -0.04]
      points:
        "0": [0.0053868,0.00298296,0.0345655]
        "1": [-0.0130016,0.022591,0.0253847]
        "2": [0.00677098,0.0471788,0.0234509]
        "3": [0.0358569,0.0226408,0.0231263]

![logo](https://github.com/GriffinBonner/crazyswarm-iribe-dronelab/blob/main/CF_MarkerConfig_1.jpg)
![logo](https://github.com/GriffinBonner/crazyswarm-iribe-dronelab/blob/main/CF_MarkerConfig_2.jpg)

## Vicon Instructions
The vicon system includes the motion capture cameras, windows pc running vicon tracker software, and the Iribe drone lab network. Instructions on how to operate safely in the lab and use the equipment including the vicon system should be received from Ivan Penskiy <ipenskiy@umd.edu> before obtaining swipe access to the Iribe drone lab.

### Vicon Camera Temperature
The accuracy and consistency of the vicon camera measurements vary with the hardware temperature. The vicon nexus cameras have extensive onboard processing for the IR strobes and cameras. Thus, they heat up significantly when powered on. The camera temperature during callibration should match the camera temperature during motion capture use. It is important to enter the lab aproximately 30 minutes before testing to plug in the cameras and log in to the lab pc. You can view the temperature of each camera by selecting them individually in the left configuration window of vicon tracker software. If you plan to take a break from using the vicon cameras for less than three hours it is recommended to leave the cameras powered on, otherwise unplug them before leaving the lab, the components have a finite lifetime. 

### Initial Positioning and Coordinate System
A custom calibration is required to set the volume origin for your experiments. If you will be using multiple crazyflies, it is recommended to use the existing calibration file, or recalibrate the system by setting the origin and x-y axis according to the existing black tape L in the netted area. Currently, the tape configuration is setup for a maximum of nine crazyflie drones. The origin and eight other vertices form a 2x2 meter square corresponding to starting positions (x,y,x): [0,0,0], [0,1,0],... ect. relative to the origin. You can reference the *allCrazyflies.yaml* file in this repository if this is unclear.

### Vicon Calibration 
The vicon system is very sensitive and can be affected by inaccurate or outdated callibration, camera temperature, building vibrations, incidental reflective materials inside the netted area, and possibly other unknown factors. If after following these instructions you are still experiencing marker instability, the first step should be to recallibrate the vicon cameras. The callibration process should have described during the Iribe drone lab tutorial, but it is recommended to review the video tutorial *Vicon Callibration Guide* linked above. The current callibration sets to postitive +x axis in the direction of the windows and the positive +y axis in the direction of the wall mounted screen. After following the steps given in the guide, save your callibration file to a local file on the lab pc. You will need to load this file in the vicon tracker software before use to set the origin of the coordinate system. If you perform a new callibration make sure to read the above section regarding camera temperature first. It is recommended to turn the wall mounted screen on before calibrating the system, you will be able to see the coverage of the space from the perspective of each camera, an accurate calibration requires each camera to get many detections spread across its field of view.

### Recognizing Instability
When the drones or any object with a reflective marker is placed in the netted area it should appear stationary in the vicon tracker software. If any of the markers appear to be shaking or instantaneously alternating between two points quickly while the marker is stationary, it is recommended to return to the *vicon calibration* section and perform another full camera calibration. There may be other factors causing this issue, but at the point of writing this document, the two main causes are discrepancy between camera temperature during calibration and testing, and poor calibration. 

## Operating Procedure

**1.** Turn on the vicon cameras and login to the lab pc. Login instructions are given in the lab tutorial, contact Ivan Penskiy <ipenskiy@umd.edu>. <br/>
**2.** Wait until the camera temperature has risen to ~50-55 degrees, visible in the vicon tracker software. <br/>
**3.** On the ubuntu computer that is running the crazyswarm software, join the Iribe DroneLab wifi network or connect to the PC running vicon tracker with LAN via the blue ethernet cable. <br/>
**4.** In the terminal on the ubuntu computer source the crazyswarm setup bash file, location given in crazyswarm configuration documentation. Creating a bash alias is recommended. <br/>
**5.** Place all crazyflies that you intend to use at their predetermined starting locations within the netted area facing the +x axis. The crazyflie with identification 1 *must* always be used. <br/>
**6.** Plug in all required crazyradios observing the heuristic that there should be a maximum of two crazyflies on a given radio channel. <br/>
**7.** Turn on all crazyflies in the netted area, be careful to maintain their location. <br/>
**8.** Run the *chooser.py* script with the following command and select all active crazyflies by identification number, click reboot, make sure that the correct ID's are displayed to the terminal. <br/>

    # ros_ws/src/crazyswarm/scripts/chooser.py
    python3 chooser.py

**9.** Run the crazyswarm ros launch file with the following command. Confirm that all active crazyflies appear at the correct initial positions with correct x,y,z orientation relative to the global coordinate system. <br/>

    # ros_ws/src/crazyswarm/launch/hover_swarm.launch
    roslaunch crazyswarm hover_swarm.launch

**10.** Run a provided test with the following command to make sure that the system is configured properly. <br/>

    # ros_ws/src/crazyswarm/scripts/niceHover.py
    python3 niceHover.py
