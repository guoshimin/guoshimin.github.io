Making an AMI based on Ubuntu 14.04
==========

Prep instructions from harold:
```
brew tap homebrew/dupes (to grab rsync 3.1)
brew install rsync (installs rsync 3.1)
brew install ansible (if on linux and need >1.6, sudo apt-get update && sudo apt-get install software-properties-common, sudo apt-add-repository ppa:ansible/ansible, sudo apt-get update && sudo apt-get install ansible )
sudo pip install boto
sudo pip install paramiko
```

ami-06e30735: Ubuntu trusty 14.04

Project killed (for now) because 14.04 doesn't have java 8.
