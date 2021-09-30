# ROS2 Ansible Playbooks

These playbooks are intended for helping with remote management of large numbers of cheap robots in an early prototyping setting.
### Set up ROS2 from source and populate the workspace with one command
`ansible-playbook ros2-setup.yml --extra-vars "nodes=robots" -K`

### Build ROS2 on a pi zero
This playbook will automatically configure a temporary swapfile and reduce CPU load
`ansible-playbook ros2-build.yml --extra-vars "nodes=robots ros2_colcon_use_swap=true ros2_colcon_single_core=true" -K`



## Features
* Streamline ros2 source builds
* Automatically configure workspaces full of packages on all nodes at once
* Coordinate updating robots from local git repositories
* Clean and rebuild ROS2 workspaces on multiple remote machines
* Start a launchfiles at boot using systemd (untested)
* Partial rebuilds using colcon package-selection commands

## Planned features
* Install packages from the ROS2 build farm
* Copy built packages between machines

## Caveats
* Relying on having a VPN into client site is a bad way to begin your startup journey, use a real fleet management service.
* This package offers no logging or monitoring features.


# Getting started
These instructions are for the Raspberry Pi Zero and Zero-W.
If you have more than about 4GB of ram, there are some shortcuts for you.

* define your desired workspace layout in `vars/workspace-config.yml`  I like to have subdirectories to separate the major parts of my applications, it helps me avoid damaging and rebuilding all of ros-core on a raspberry pi zero.

* list your robots in `/etc/ansible/hosts`
    ```
    all:
    children:
        robots:
        hosts:
            raspberrypi.local:
            ansible_user: pi
            ansible_python_interpreter: auto
            renamedpi.local:
            ansible_user: pi
            ansible_python_interpreter: auto
    ```
    
#### remote workspace setup
set up ROS and download sources with `ansible-playbook ros2-setup.yml --extra-vars "nodes=robots ros2_colcon_needs_latomic=true" -K`
    Here we use `-K` to tell ansible that it will need to use sudo, and it should ask us for the password.
    `nodes=robots` refers to the list of machines we made in `/etc/ansible/hosts`
    `ros2_colcon_needs_latomic=true` sets some tedious build flags to deal with a RaspiOS peculiarity
    This step will set up apt keys, apt sources, download build tools, prepare a workspace, and install system dependencies for packages in your workspace, and set some comfortable defaults for developing with colcon.

#### remote build
start a build with `ansible-playbook ros2-build.yml --extra-vars "nodes=robots ros2_colcon_needs_latomic=true ros2_colcon_single_core=true ros2_colcon_use_swap=true" -K`
    `ros2_colcon_single_core=true` instructs colcon to only use a single thread to cut back on processor resources
    `ros2_colcon_use_swap=true` will make a 4GB swap file to get through some rough patches in the build cycle, and then remove the swap file when it's finished.

#### launch at boot (and immediately)
launch ros at boot `ansible-playbook ros2-systemd-launch.yml -K -v --extra-vars "nodes=navbeacons launch_service_name=monitor launchpackage=monitoring launchfile=monitor.launch.py remove=false"
    This creates a systemd service called `ros2_monitor.service` that starts `monitor.launch.py` from the  package `monitoring`, immediately, and on boot.


#### stop the launchfile
remove a launch service  `ansible-playbook ros2-systemd-launch.yml --extra-vars "nodes=navbeacons launch_service_name=ros2monitor remove=true" -K`
    This stops the launch service and removes the systemd entry we created above.






