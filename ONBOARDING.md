# Onhoarding steps

## 1. Dual boot linux
## 2. Install docker, other programs you might need (like vscode and git)

### 2.1. Once docker is installed, we need to set up your tokens. Instructions are located at https://github.com/umroboticsteam/umrt-build
    --> As soon as you are done this step, start running the following command as it might take a while:

        $ docker pull ghcr.io/umroboticsteam/umrt-build-dev:babbler


### 2.2. You will also need to configure a git access token to upload your code changes.
     --> If you install the github command line (gh) this is an easy way to do it.
            Personally: I generated ssh keys and uploaded them onto github's setting page.
                        They have instructions for this on their website.

### 2.3. Install vscode and the extension "Dev Containers"

### 2.4. Install the "can-utils" package.

Each time you start your computer, if you don't have a real can device

    $ sudo ip link add can0 type vcan
    $ sudo ip link set up can0

To test if your can device is working

    In one terminal: `$ candump can0`
    In another terminal: `$ cansend can0 123#123123`
    To send a lot of messages very quickly: `$ cansequence -p can0`

You should see a lot of messages going by. You can stop these programs now.

## 3. Download the umrt-build github repository.
### 3.1. Switch branches to `quinndt/add-umrt-start-arm-script`

    git checkout quinndt/add-umrt-start-arm-script

You can press tab to autocomplete and it should work.

## 4. Start the docker container

Without forwarding any USB devices: (not recommended)

    $ scripts/umrt-start-arm

To forward usb devices, and any CAN devices, and networks:

    $ scripts/umrt-start-arm --privilaged --volume="/dev/input:/dev/input"

Either way, this may take a while since it is downloading a large docker image >1GB.

## 5. Set up inside the docker container

### 5.1. At this point in time, you can connect the VS Code Dev Containers extension

    In VS Code, press CTRL-P, and type ">Attach to running container" and press enter.
    Select "/umrt-arm".
    Open the folder "/workspace" in vscode, in the container.
    This folder is mapped to "~/Projects/umrt-arm-docker-workspace"
    Any files in this folder will be saved.

### 5.2. Inside the docker container run the following command:

    $ usermod -aG sudo rosdev

You won't need to use this terminal any more.

### 5.3. SSH to the docker container with X11 forwarding

Try:
    $ ssh -X -Y rosdev@127.0.0.1 -p 12345
or:
    $ ssh -X -Y rosdev@127.0.0.1 -p 50000

The first version will work for people who didn't make the container privilaged.
The second version will work for people who did.

The password is "cannonmenunickel".

### 5.4. Forward X11 as root.

The commands -X and -Y above forward X11 (the display protocol) from rosdev to your computer. We will make this work for root as well. Source: https://www.simplified.guide/ssh/x11-forwarding-as-root (steps 4,5)

    $ sudo sh -c 'umask 077; touch /root/.Xauthority'
    $ xauth extract - "$DISPLAY" | sudo XAUTHORITY=/root/.Xauthority xauth merge -

Now, switch to the root user. Remember which terminal this one is, because it is the only one which will have X11 forwarding.

    $ sudo su

### 5.4b. Extra: SSHing without X11 forwarding.

    $ ssh rosdev@127.0.0.1 -p (your port)
    where (your port) is 12345 or 50000, whichever worked before. Password is listed above.
    $ sudo su

Key differences:
    You don't need -X or -Y.
    You don't need to forward the X11 credentials.

### 5.5. Clone the repositories

    $ cd /workspace
    $ git clone https://github.com/umroboticsteam/umrt-arm-encoder-driver
    $ git clone https://github.com/umroboticsteam/umrt-arm-ros-firmware
    $ git clone https://github.com/umroboticsteam/umrt-arm-firmware-lib
    $ git clone https://github.com/umroboticsteam/umrt-arm-joystick-operator
    $ git clone https://github.com/umroboticsteam/umrt-project-perry-config

Switch branches. This step is only necessary for now, since some code hasn't been merged yet.

    $ git -C umrt-arm-encoder-driver checkout quinndt/verified-working
    $ git -C umrt-arm-ros-firmware checkout circ2026
    $ git -C umrt-arm-joystick-operator checkout circ2026
    $ git -C umrt-project-perry-config checkout circ2026

### 5.6. Source the ros workspace

    $ source /opt/ros/humble/setup.bash

This step is required for every new terminal, otherwise the "ros2" command will not be found.

### 5.7. Temporary: Delete /opt/umrt-arm-encoder-driver

    $ rm -r /opt/umrt-arm-encoder-driver

This is the "newer" version that doesn't work properly anymore. The code was updated in a way which wasn't meant to change anything, but somehow it stopped working. It will be looked into!

### 5.8. Build the project

    $ colcon build --symlink-install

Or: to only build one project (just for your information)

    $ colcon build --symlink-install --packages-select <package>
    for example
    $ colcon build --symlink-install --packages-select umrt-arm-ros-firmware

If you get any failiures at this step, try building umrt-arm-encoder-driver first, doing step 5.9, then building the rest, and step 5.9 again.

The --symlink-install isn't required, but it means if you edit a text file you won't have to recompile.

### 5.9. Source the built workspace

    $ source /workspace/install/setup.bash

This will make all of the compiled programs available to you on the terminal.

### 5.10. Run the demo

    $ ros2 launch umrt-project-perry-moveit-config demo.launch.py

It should launch a GUI. If it doesn't I probably forgot a step.
