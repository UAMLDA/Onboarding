# About the UAMLDA Computing Resources 

All of the computing facilities in the UAMLDA given a codename, and these codenames are characters from the Game of Thrones series. There is currently only one server in the group which is described below. Any group member can `ssh` into the server; however, you must be connected to the Univeristy of Arizona's VPN if you are off campus. 


# The Research Server

This a private server to the students working in the University of Arizona's Machine Learning and Data Analytics Lab. The specifications are 

* Intel Xeon E7-4830 v3 Dodeca-core (12 Core) 2.10 GHz
* RAM 16 SM 16GB DDR3 SDRAM RAM 16 GB (1 x 16 GB) DDR3 SDRAM-1600 MHz 
* Management 1 Integrated IPMI 2.0 & KVM with Dedicated LAN
* Controller 1 Dual-Port Intel i350 GigE plus 2 Ports 6Gb/s SATA and 4 Ports 3Gb/s 
* RAID controllers 1 LSI 9361-8i (8-Port Int, PCIe 3.0, 1GB Cache, Dual Core)
* RAID controllers 1 LSI 9361 KIT CACHEVAULT
* Hot-Swap Drive 5 Seagate Constellation 3 TB 3.5" Internal Hard Drive - SAS - 7200 rpm
* Power Supply 1 Redundant 1400W Power Supply with DSC and PMBus 
* Management SW 1 IPMI Support for Intelligent Platform Management Interface v.2.0 

# Configuring and Accesing the Server

## `ssh` Configuration 

If you're lazy you can make the `ssh` process a bit easier by adding the following to the `~/.ssh/config` file on your local machine.
```
Host khaleesi 
  User UAnetID
  Hostname <server name>.ece.arizona.edu
```

To make your life even easier, add your ssh key to the server. 
```
# do not add the <>
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
$ ssh-add ~/.ssh/id_rsa
$ ssh-copy-id <UAnetID>@<server name>.ece.arizona.edu
```

## Setting Up Your Paths 

Here is an example, however, you will need to make the appropriate changes for your Anaconda installation. 

```
. /home/skel/bash/bashrc

export PATH=~/bin:${PATH}
export PATH=/usr/local/MATLAB/R2015b/bin/:${PATH}
export PATH=/scratch/ditzler/anaconda2/bin/:${PATH}
```


# Accessing RStudio

The `ssh` accounts are dealt with via the active directories that are mounted from the University of Arizona; however, the [RStudio](https://www.rstudio.com/) accounts are managed locally on the servers. The usernames for Rstudio are `rs_netid` (e.g., `rs_ditzler`). The default path for the Rstudio accounts are `/localhome/rs_netid`. Note that your Rstudio and standard `ssh` accounts are different and cannot read/write to both accounts. I recommend using Git to manage your projects in the `localhome` directory. 

The RStudio accounts can be accessed by `http://<server name>.ece.arizona.edu:8787`. 


# Setting up Anaconda for Python 

The [Anaconda](https://www.continuum.io/) distribution of Python is recommended rather than having an admin install specific packages of Python. There are plenty of tutorials out there.  

