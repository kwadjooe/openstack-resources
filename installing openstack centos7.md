## INSTALLING OPENSTACK ON CENTOS 7
This tutorial will guide you on how you can deploy your own private cloud infrastructure with OpenStack installed on a single node in CentOS 7 or RHEL 7 or Fedora distributions by using rdo or openstack-stein   repositories, although the deployment can be achieved on multiple nodes.
### PREPARING MACHINE

    ss -tulpn ---> to list all running services  
    systemctl stop firewalld NetworkManager  
    systemctl disable firewalld NetworkManager  
    sed -i 's/^SELINUX=.*/SELINUX=permissive/g'  /etc/selinux/config  
    setenforce 0  
    getsetenforce  
    yum update -y  
    yum -y install vim wget curl telnet bash-completion nano  
    hostnamectl set-hostname cloud.centos.lab  ---> setting the hostname
    hostname ---> will display the hostname  
    yum install ntpdate  ---> install ntpdate to synchronize with a NTP server  
    ----  
    # systemctl stop postfix firewalld NetworkManager
    # systemctl disable postfix firewalld NetworkManager
    # systemctl mask NetworkManager
    # yum remove postfix NetworkManager NetworkManager-libnm

#### install the openstack repository  

    yum install -y centos-release-openstack-stein  
    yum install -y openstack-packstack

#### Another Alternative repos  
    yum install https://www.rdoproject.org/repos/rdo-release.rpm  
    yum install -y centos-release-openstack-mitaka (Non existent)  
    yum update -y

#### Generate the answer file for the deployment  
    
    packstack --gen-answer-file /root/answers.txt  
    or 
    packstack --gen-answer-file=`date +"%d.%m.%y"`.conf  

#### Edit the Answer file 
    vi /root/answer.txt

#### edit these settings  
    CONFIG_NTP_SERVERS=0.pool.ntp.org,1.pool.ntp.org,2.pool.ntp.org
    CONFIG_CONTROLLER_HOST=
    CONFIG_KEYSTONE_ADMIN_PW=B@ckD00r!
    CONFIG_KEYSTONE_DEMO_PWD=S3cu71+y  
    CONFIG_MARIADB_PW=B@ckD00r!
    CONFIG_NAGIOS_PW=B@ckD00r!
    CONFIG_PROVISION_DEMO=n



#### Formatting drive for swift storage
    lsblk  ---> List all volumes
    fdisk /dev/sdb
    select n  
    select p  ---> create a primary partition
    select 1  ---> to create one partition  
    set first sector to 2048   
    set the last sector +40GB ---> create a 40GB Partition  
    select w to write changes
    mkfs.ext4 /dev/sdb1
    mount /dev/sdc1 
    mkdir /mnt/STORAGE
    mount /dev/sdc1 /mnt/STORAGE
    make a permanent entry in the /etc/fstab for auto mount on reboot

    fdisk /dev/sdc
    select n  
    select p  ---> create a primary partition
    select 1  ---> to create one partition  
    set first sector to 2048   
    set the last sector +100GB ---> create a 100GB Partition  
    select w to write changes
    mkfs.ext4 /dev/sdc1
    mkdir /mnt/DATA
    mount /dev/sdc1 /mnt/DATA
    make a permanent entry in the /etc/fstab for auto mount on reboot

    vgcreate cinder-volumes /dev/sdb1
    lvcreate -l 100%FREE -T cinder-volumes/cinder-volumes-pool

--- MAKE THIS CHANGES IN THE ANSWER FILES  
    CONFIG_SWIFT_STORAGES=/dev/sdb
    CONFIG_CINDER_VOLUMES_CREATE=n

##### Allow root Login in sshd_config
    vi /etc/ssh/sshd_config  
    PermitRootLogin yes
    systemctl restart sshd

    cat keystonerc_admin


#### Adding Images 
    source keystonerc_admin
    cd /var/lib/glance/images  

    openstack image create \  
> --container-format bare \  
> --disk-format qcow2 \  
> --file cirros-0.4.0-x86_64-disk.img \  
> Cirros-0.4.0-x86_64  

#### Creating a CentOS7-x86_64 image
    openstack image create --container-format bare --disk-format qcow2 --file CentOS-7-x86_64-GenericCloud.qcow2 CentOS-7-x86_64  

    The installation log file is available at: /var/tmp/packstack/20230224-092544-Hf02e4/openstack-setup.log
