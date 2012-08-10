B21
===

The B21r is the big red dustbin on wheels/expensive tripod. Partially outdated information on it can be found [here](http://www.cs.bham.ac.uk/resources/courses/robotics/research/B21r/). Also, there are manuals for the B21 in the filing cabinet.

We sometimes refer to the B21 as [Mr Chips](http://www.youtube.com/watch?v=qtM0-ZFwiNo).


PCs
---

The B21 contains two PCs that were recently upgraded by support. More information on the upgrade is [here](http://www.cs.bham.ac.uk/~sjt/B21r/).

We are currently only using a single PC. It is labelled as `irobot0` on the robot and is accessible via the hostname `irobot0-wlan` on the `ccarobot` network (both wired and wireless). `irobot0` is running Ubuntu 12.04 LTS. If you want an account on the machine, please contact Nick.

When working on the command line on the B21 PCs, you might need to set a proxy variable to access the internet:

<pre>
export http_proxy="http://webcache.cs.bham.ac.uk:3128"
</pre>



ROS 
---

ROS fuerte is installed on `irobot0` via binary packages into `/opt/ros/fuerte`. Additional packages which are available for all users are installed under `/opt/ros/local`. I have the following in my `.bashrc` to setup ROS correctly for this configuration.

```bash
# standard ROS install
source /opt/ros/fuerte/setup.bash
# machine-wide ROS packages
source /opt/ros/local/setup.sh
# user-local ROS packages
export ROS_PACKAGE_PATH=~/ros-nah:${ROS_PACKAGE_PATH}
# make sure the external-facing hostname is used for roscore
export ROS_MASTER_URI=http://irobot0-wlan:11311
```

Charging 
--------


Start-up
--

The B21 is started up in multiple stages. First off you need to turn on the fuses to the "Base" and "Enclosure". These are located behind the base door to the right of the door containing the charger. Once these are on you can power up the robot by pressing in the black rotary switch on the back right of the raised platform on top of the robot. After a few seconds the RFlex screen on the top of the enclosure should illuminate. Finally, to turn on the PCs,  press the black push switches on the back of the raised platform  next to the serial ports. You don't need to turn both PCs on if you only require one of them. 

Shut-down
--

First off, power down the PCs. To do this press the push switch for the corresponding PC. This will cleanly halt the machine and power it off. Next, power off the robot using the RFlex screen. Use the rotary switch (which you pressed to turn the thing on) to select the "PWR" box (or "Host Console", they go to the same place), then select the "Kill PWR" box (ignoring the "Shutdown PCs" box). Finally, switch off the fuses on the robot base.

Devices
-------

There are a number of devices connected to irobot0 over serial links. There is a 
serial card with 8 ports [Rocket Port Universal 8-port RJ11 PCI card](ftp://ftp.comtrol.com/html/RPuPCI_docs.htm) (it becomes visible when you open the 2 front doors of the top of the  robot). To open doors, lift them gently before pulling towards you). The port towards the centre of the robot is /dev/ttyR7 and the port furthest away from the centre is /dev/ttyR0. Make sure the device descriptors in the configuration files match the actual wiring.


Speech
------

When logged into the robot, type the following into the terminal:
```
echo <what you want to say> > /dev/ttyR0
```

For example, typing the following will say "My name is Mr.Chips"
```
echo my name is mr.chips > /dev/ttyR0
```


Laser Scanner
-------------

On `/dev/ttyR2` is a Sick PLS laser scanner. Code to run this in ROS is installed on the robot. This should work for you if you run the `sick_pls_wrapper` node, and a long delay in connecting is usual, but it should usually start streaming data after 15 or 20 seconds. However, occasionally the driver will fail to connect to the scanner. In this case, disconnect the laser from the power (open the robot, and break the connector on the power wire leading to the laser) then reconnect it. The driver should now connect fine.

The laser isn't properly started until you see the following message printed out.

```
   Requesting measured value data stream...
   	      Data stream started!
```

RFlex
-----

The B21 is controlled by sending commands to the RFlex controller on `/dev/ttyR5`. There is an odd problem that this doesn't work correctly on start-up, causing the B21 to move in a jerky and unpredictable fashion. To fix this, start up the rflex driver:

```
rosrun rflex b21 _port:=/dev/ttyR5
``` 

Then run the following command:%

```
jpnevulator --ascii --timing-print --tty /dev/ttyR5 --read
```

Then kill both processes. After this the rflex driver will be fine the next time you run it. Thanks to [ROS Answers](http://answers.ros.org/question/10900/rflexb21-cmd_vel-not-working-as-expected/) for this.

Launching
---------

To launch the basic B21 sensor drivers and tf components, run:

```
 roslaunch bham_b21_launch b21.launch 
```

and remember to wait for

```
	Requesting measured value data stream...
		   Data stream started!
```


Once this is complete, you can bring up the navigation stack on the B21 with the command:

```
roslaunch bham_b21_2dnav research-lab.launch
```

This runs the path planning components plus AMCL with a map of the research lab. You may also find other maps in that package (as people like you make them).
