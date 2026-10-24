# Onboarding steps

This will be everyone in Robotic Arm software's first task. This task must be completed before any other tasks can be started. To submit the task, DM the software lead (currently Quinn Thiessen) a screenshot of the GUI that pops up at the final step!

Questions can be asked by Discord DM, in the tasks thread for "Software Onboarding", or in the robotic arm discord channel.

## 1. Dual boot linux
Instructions will not be provided at this time.

## 2. Install programs

### 2.1. Install docker and set up your tokens.
Once docker is installed, set up your GitHub Container Registry (ghcr) tokens.
This will allow you to download our docker container images from github through the docker command line.

Instructions are located at https://github.com/umroboticsteam/umrt-build

As soon as you are done this step, start running the following command as it might take a while:

```shell
docker pull ghcr.io/umroboticsteam/umrt-build-dev:babbler
```

### 2.2. You will also need to configure a git access token to upload your code changes.

If you install the github command line (gh) this is an easy way to do it.

You can also log into VSCode and it should just start working? I don't use this so I don't know.

Personally: I generated ssh keys and uploaded them onto github's setting page. 
They have instructions for this on their website.

### 2.3. Install vscode and the extension "Dev Containers"

### 2.4. Install the "can-utils" package.

Each time you start your computer, if you don't have a real can device

```shell
sudo ip link add can0 type vcan
sudo ip link set up can0
```

To test if your can device is working

1. In one terminal: `candump can0`
2. In another terminal: `cansend can0 123#123123`
3. To send a lot of messages very quickly: `cansequence -p can0`

You should see a lot of messages going by. You can stop these programs now.

## 3. Download the umrt-build github repository.
### 3.1. Clone and switch branches to `quinndt/add-umrt-start-arm-script`

```shell
git clone https://github.com/umroboticsteam/umrt-build.git
cd umrt-build
git checkout quinndt/add-umrt-start-arm-script
```

You can press tab to autocomplete the branch name and it should work.

## 4. Start the docker container

To forward usb devices, and any CAN devices, and networks:

```shell
./scripts/umrt-start-arm --privileged --net=host --volume="/dev/input:/dev/input"
```

This may take a while if you didn't start the download earlier since it is downloading a large docker image >1GB.

## 5. Set up inside the docker container

### 5.1. At this point in time, you can connect the VS Code Dev Containers extension

1. In VS Code, press CTRL-P, and type `>Attach to running container` and press enter.
2. Select `/umrt-arm`.
3. Open the folder `/workspace` in vscode, in the container. It won't open the usual file picker because the file isn't accessed from your PC's file system.
4. This folder is mapped to `~/Projects/umrt-arm-docker-workspace` on your computer. Any files in this folder will be saved. Any files NOT in this folder may be lost at any time.

### 5.2. Inside the docker container run the following command:

```shell
usermod -aG sudo rosdev
```

You won't need to use this terminal any more.

### 5.3. SSH to the docker container with X11 forwarding

```shell
ssh -X -Y rosdev@127.0.0.1 -p 50000
```

If you get an error saying something about connection refused, try port number `12345`. If port number `12345` works then your container isn't in privileged mode. Delete the container and run the start script again. It won't need to download again.

The password is `cannonmenunickel`.

### 5.4. Forward X11 as root.

The commands -X and -Y above forward X11 (the display protocol) from rosdev to your computer. We will make this work for root as well. Source: https://www.simplified.guide/ssh/x11-forwarding-as-root (steps 4,5)

```shell
sudo sh -c 'umask 077; touch /root/.Xauthority'
xauth extract - "$DISPLAY" | sudo XAUTHORITY=/root/.Xauthority xauth merge -
```

Now, switch to the root user. Remember which terminal this one is, because it is the only one which will have X11 forwarding.

```shell
sudo su
```

### 5.4b. Extra: SSHing without X11 forwarding.

Using this option means you won't get any display.

```shell
ssh rosdev@127.0.0.1 -p 50000
sudo su
```

Key differences:
    You don't need -X or -Y.
    You don't need to forward the X11 credentials.

### 5.5. Clone the repositories

```shell
cd /workspace
git clone https://github.com/umroboticsteam/umrt-arm-encoder-driver
git clone https://github.com/umroboticsteam/umrt-arm-ros-firmware
git clone https://github.com/umroboticsteam/umrt-arm-firmware-lib
git clone https://github.com/umroboticsteam/umrt-arm-joystick-operator
git clone https://github.com/umroboticsteam/umrt-project-perry-config
```

Switch branches. This step is only necessary for now, since some code hasn't been merged yet.

```shell
git -C umrt-arm-encoder-driver checkout quinndt/verified-working
git -C umrt-arm-ros-firmware checkout circ2026
git -C umrt-arm-joystick-operator checkout circ2026
git -C umrt-project-perry-config checkout circ2026
```

### 5.6. Source the ros workspace

```shell
source /opt/ros/humble/setup.bash
```

This step is required for every new terminal, otherwise the `ros2` command will not be found.

### 5.7. Temporary: Delete `/opt/umrt-arm-encoder-driver`

```shell
rm -r /opt/umrt-arm-encoder-driver
```

This is the "newer" version that doesn't work properly anymore. The code was updated in a way which wasn't meant to change anything, but somehow it stopped working. It will be looked into!

### 5.8. Build the project
```shell
colcon build --symlink-install
```
Or: to only build one project (just for your information)
```shell
colcon build --symlink-install --packages-select <package>
# for example
colcon build --symlink-install --packages-select umrt-arm-ros-firmware
```

If you get any failiures at this step, try building umrt-arm-encoder-driver first, doing step 5.9, then building the rest, and step 5.9 again.

The `--symlink-install` isn't required, but it means if you edit a text file you won't have to recompile.

### 5.9. Source the built workspace

```shell
source /workspace/install/setup.bash
```

This will make all of the compiled programs available to you on the terminal.

### 5.10. Run the demo
```shell
ros2 launch umrt-project-perry-moveit-config demo.launch.py
```

It should launch a GUI. If it doesn't I probably forgot a step.

Congrats!

# TODO

Changes necessary:
 - [ ] Update the script in step 3 and move it to it's own repository. (Where to comms and embedded store their start scripts?)
 - [ ] Modify dockerfile to do step 5.2
 - [ ] Potentially allow ssh as root or make script to make step 5.4 unnecessary as it is likely confusing.
 - [ ] Merge a ton of branches so 5.5 doesn't need switching branches.
 - [ ] Fix 5.7
