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


- bin/create_images.py: chanage baseami
- ami/ansible/roles/common/tasks/main.yml: comment out java8
- ami/ansible/roles/*/tasks/main.yml: remove z from rsync_opts, add compress=\"no\"
- ami/charms/worker/var/lib/lxc/spark_template/rootfs/install-java: remove java8
- configs/account_ids: accounts that the image will be shared with


```
bin/create_images.py -k shimin-dev -r false -i ~/.ssh/shimin-dev.pem -b 2.10.1
```
`-r false` is supposed to say "don't remove ssh keys" but the code is broken so it doesn't work.

You must specify AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY as environtment variables. It doesn't search ~/.aws/credentials.

try 1: failed because lxc trying to download utopic image

ami/ansible/roles/worker/tasks/main.yaml: utopic -> trusty

try 2: success but ssh key deleted even though -r false was specified

built ami ami-f55b4194

launched instance with ami: i-b31d9869

Install Java 8:

- get java tarball from http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

- tar zxf jdk-8u66-linux-x64.gz

- sudo mv jdk1.8.0_66 /usr/lib/jvm
- add /usr/lib/jvm/.jdk1.8.0_66.jinfo
```
name=jdk1.8.0_66
priority=1081
section=main

hl rmid /usr/lib/jvm/jdk1.8.0_66/jre/bin/rmid
hl java /usr/lib/jvm/jdk1.8.0_66/jre/bin/java
hl keytool /usr/lib/jvm/jdk1.8.0_66/jre/bin/keytool
hl jjs /usr/lib/jvm/jdk1.8.0_66/jre/bin/jjs
hl pack200 /usr/lib/jvm/jdk1.8.0_66/jre/bin/pack200
hl rmiregistry /usr/lib/jvm/jdk1.8.0_66/jre/bin/rmiregistry
hl unpack200 /usr/lib/jvm/jdk1.8.0_66/jre/bin/unpack200
hl orbd /usr/lib/jvm/jdk1.8.0_66/jre/bin/orbd
hl servertool /usr/lib/jvm/jdk1.8.0_66/jre/bin/servertool
hl tnameserv /usr/lib/jvm/jdk1.8.0_66/jre/bin/tnameserv
hl jexec /usr/lib/jvm/jdk1.8.0_66/jre/lib/jexec
jre policytool /usr/lib/jvm/jdk1.8.0_66/jre/bin/policytool
jdk jconsole /usr/lib/jvm/jdk1.8.0_66/bin/jconsole
jdk jdeps /usr/lib/jvm/jdk1.8.0_66/bin/jdeps
jdk wsimport /usr/lib/jvm/jdk1.8.0_66/bin/wsimport
jdk rmic /usr/lib/jvm/jdk1.8.0_66/bin/rmic
jdk jinfo /usr/lib/jvm/jdk1.8.0_66/bin/jinfo
jdk jsadebugd /usr/lib/jvm/jdk1.8.0_66/bin/jsadebugd
jdk native2ascii /usr/lib/jvm/jdk1.8.0_66/bin/native2ascii
jdk jstat /usr/lib/jvm/jdk1.8.0_66/bin/jstat
jdk javac /usr/lib/jvm/jdk1.8.0_66/bin/javac
jdk javah /usr/lib/jvm/jdk1.8.0_66/bin/javah
jdk idlj /usr/lib/jvm/jdk1.8.0_66/bin/idlj
jdk jstack /usr/lib/jvm/jdk1.8.0_66/bin/jstack
jdk jrunscript /usr/lib/jvm/jdk1.8.0_66/bin/jrunscript
jdk javadoc /usr/lib/jvm/jdk1.8.0_66/bin/javadoc
jdk javap /usr/lib/jvm/jdk1.8.0_66/bin/javap
jdk jar /usr/lib/jvm/jdk1.8.0_66/bin/jar
jdk extcheck /usr/lib/jvm/jdk1.8.0_66/bin/extcheck
jdk schemagen /usr/lib/jvm/jdk1.8.0_66/bin/schemagen
jdk jps /usr/lib/jvm/jdk1.8.0_66/bin/jps
jdk xjc /usr/lib/jvm/jdk1.8.0_66/bin/xjc
jdk jarsigner /usr/lib/jvm/jdk1.8.0_66/bin/jarsigner
jdk jmap /usr/lib/jvm/jdk1.8.0_66/bin/jmap
jdk appletviewer /usr/lib/jvm/jdk1.8.0_66/bin/appletviewer
jdk jstatd /usr/lib/jvm/jdk1.8.0_66/bin/jstatd
jdk jhat /usr/lib/jvm/jdk1.8.0_66/bin/jhat
jdk jdb /usr/lib/jvm/jdk1.8.0_66/bin/jdb
jdk serialver /usr/lib/jvm/jdk1.8.0_66/bin/serialver
jdk wsgen /usr/lib/jvm/jdk1.8.0_66/bin/wsgen
jdk jcmd /usr/lib/jvm/jdk1.8.0_66/bin/jcmd
```

- cat /usr/lib/jvm/.jdk1.8.0_66.jinfo | awk 'NF == 3 {print "/usr/bin/" $2 " " $2 " " $3 " 30 \n"}' | xargs -t -n4 sudo update-alternatives --install


- sudo update-java-alternatives  -s jdk1.8.0_66

image: ami-1d776d7c

Also need to install latest kernel

```
- name: update kernel
  tags: kernel
  apt: pkg=linux-image-4.2.0-23-generic state=installed
```

instance: i-9e748759
