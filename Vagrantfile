Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.box_version = "20170503.1.0"  # should be able to use any version
  config.vm.box_check_update = false  # can enable to auto-update

  config.vm.network "forwarded_port", guest: 50010, host: 50010
  config.vm.network "forwarded_port", guest: 50020, host: 50020
  config.vm.network "forwarded_port", guest: 50070, host: 50070
  config.vm.network "forwarded_port", guest: 50075, host: 50075
  config.vm.network "forwarded_port", guest: 50090, host: 50090

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: <<-SHELL
    # run update for good measure
    sudo apt update

    # install java
    sudo apt install openjdk-8-jre-headless --yes
    sudo sh -c "echo 'export JAVA_HOME=\"/usr/lib/jvm/java-8-openjdk-amd64\"' >> /etc/environment"
    source /etc/environment

    # setup python
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1

    # download and extrat hadoop
    wget -q 'http://mirror.cc.columbia.edu/pub/software/apache/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz'
    tar -zxf hadoop-2.8.0.tar.gz
    
    # setup the config files
    cd hadoop-2.8.0
    mv ./etc/hadoop/core-site.xml ./etc/hadoop/core-site.xml_dist
    mv ./etc/hadoop/hdfs-site.xml ./etc/hadoop/hdfs-site.xml_dist
    cp /vagrant/core-site.xml ./etc/hadoop/
    cp /vagrant/hdfs-site.xml ./etc/hadoop/
    cd ~
    sudo chown ubuntu:ubuntu * -R
   
    # setup passwordless ssh 
    cp id_rsa id_rsa.pub known_hosts authorized_keys ~/.ssh
    chmod 0600 ~/.ssh/authorized_keys
    chmod 0600 ~/.ssh/id_rsa
    chmod 0444 ~/.ssh/id_rsa.pub
    chmod 0444 ~/.ssh/known_hosts
    sudo chown ubuntu:ubuntu ~/.ssh -R
    
    # format the name node and start HDFS
    bin/hdfs namenode -format
    sbin/start-dfs.sh

    # download and untar spark
    cd ~
    wget -q 'http://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.7.tgz'
    tar -zxf spark-2.1.1-bin-hadoop2.7.tgz

    # start spark
    cd ~/spark-2.1.1-bin-hadoop2.7
    sbin/start-master.sh
    
  SHELL
end
