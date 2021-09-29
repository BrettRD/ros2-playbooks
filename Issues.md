




* vcs lists '.repos' cannot be updated until packages are removed
    rosinstall-generator uses src to exclude packages from the list
    consider using a list concatenation of previous vcs packages passed to --deps-up-to instead, with a switch on empty to use --deps

* workspace location {{ ros2_workspace }} is fixed in the playbook, it needs to use a sane templated default derived from vars

* the test for whether the swapfile is mounted is very sensitive to a trailling slash in {{ ros2_workspace }}


