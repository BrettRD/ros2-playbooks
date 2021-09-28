




* vcs lists '.repos' cannot be updated until packages are removed
    rosinstall-generator uses src to exclude packages from the list
    consider using a list concatenation of previous vcs packages passed to --deps-up-to instead, with a switch on empty to use --deps
* workspace location is fixed in the playbook
    needs to use a sane templated default derived from vars

* allow colcon swap undefined