* ROS2 Ansible Playbooks

These playbooks are intended for helping with remote management of large numbers of cheap robots in an early prototyping setting.

Set up ROS2 from source and populate the workspace with one command
`ansible-playbook ros2-setup.yml --extra-vars "nodes=robots" -K`

Build ROS2 on a pi zero, this playbook will automatically configure a temporary swapfile and reduce CPU load
`ansible-playbook ros2-build.yml --extra-vars "nodes=robots ros2_colcon_use_swap=true ros2_colcon_single_core=true" -K`



** Features
* Streamline ros2 source builds
* Automatically configure workspaces full of packages on all nodes at once
* Coordinate updating robots from local git repositories
* Clean and rebuild ROS2 workspaces on multiple remote machines
* Start a launchfiles at boot using systemd (untested)
* Partial rebuilds using colcon package-selection commands

** Planned features
* Install packages from the ROS2 build farm
* Copy built packages between machines

** Caveats
* Relying on having a VPN into client site is a bad way to begin your startup journey, use a real fleet management service.
* This package offers no logging or monitoring features.


* Getting started
These instructions are for the Raspberry Pi Zero and Zero-W.
If you have more than about 4GB of ram, there are some shortcuts for you.

* define your desired workspace layout in `vars/workspace-config.yml`  I like to have subdirectories for major parts of my applications.
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
* set up ROS and download sources with `ansible-playbook ros2-setup.yml --extra-vars "nodes=robots ros2_colcon_needs_latomic=true" -K`
    Here we use `-K` to tell ansible that it will need to use sudo, and it should ask us for the password.
    `nodes=robots` refers to the list of machines we made in `/etc/ansible/hosts`
    `ros2_colcon_needs_latomic=true` sets some tedious build flags to deal with a RaspiOS peculiarity
    This step will set up apt keys, apt sources, download build tools, prepare a workspace, and install system dependencies for packages in your workspace, and set some comfortable defaults for developing with colcon.

* start a build with `ansible-playbook ros2-setup.yml --extra-vars "nodes=robots ros2_colcon_needs_latomic=true ros2_colcon_single_core=true ros2_colcon_use_swap=true" -K`
    `ros2_colcon_single_core=true` instructs colcon to only use a single thread to cut back on processor resources
    `ros2_colcon_use_swap=true` will make a 4GB swap file to get through some rough patches in the build cycle, and then remove the swap file when it's finished.


* launch ros at boot (coming soon) `ansible-playbook ros2-systemd-launch.yml --extra-vars "nodes=robots launch_service_name=my_service launchpackage=my_package launchfile=my_application.launch.py" -K`
    This creates a systemd service that starts your launchfile.

* remove a launch service (coming soon) `ansible-playbook ros2-systemd-launch.yml --extra-vars "nodes=robots launch_service_name=my_service" -K`
    This stops the launch service and removes the systemd entry.
