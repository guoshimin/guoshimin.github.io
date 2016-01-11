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

Project unkilled, will install java 8 manually.


universe/bin/create_images.py: chanage baseami

universe/ami/ansible/roles/common/tasks/main.yml: comment out java8


universe/ami/ansible/roles/*/tasks/main.yml: remove z from rsync_opts, add compress=\"no\"

ami/charms/worker/var/lib/lxc/spark_template/rootfs/install-java: remove java8

configs/account_ids: accounts that the image will be shared with

./bin/create_images.py -k gov-shard-amis -r false -i /Users/harold/Downloads/gov-shard-amis.pem -b your_branch_created_by_pushall

bin/create_images.py -k shimin-dev -r false -i ~/.ssh/shimin-dev.pem -b 2.10.1

ami/ansible/roles/worker/tasks/main.yaml: utopic -> trusty
