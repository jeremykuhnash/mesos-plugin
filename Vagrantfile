# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

 # Exit on any errors.
 set -e

 # Setup
 apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E56151BF
 DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')
 CODENAME=$(lsb_release -cs)
 echo ${DISTRO}
 echo ${CODENAME}

 # Add the repository
 echo "deb http://repos.mesosphere.com/${DISTRO} ${CODENAME} main" | \
   sudo tee /etc/apt/sources.list.d/mesosphere.list
 apt-get -y update

 # Library to do xpath and extract version from pom.xml
 apt-get -y install libxml-xpath-perl

 # Look up mesos version from the pom.xml
 MESOS_VERSION=$(cat /vagrant/pom.xml | xpath -q -e /project/properties/mesos.version/text\\(\\) 2>/dev/null)
 echo $MESOS_VERSION
 if [ $MESOS_VERSION == 1.0.0-rc2 ] 
  then 
    wget http://repos.mesosphere.com/ubuntu/pool/main/m/mesos/mesos_1.0.0-1.0.73.rc2.ubuntu1404_amd64.deb  
    apt-get -y install zookeeperd
    apt-get -y install gdebi-core
    gdebi --option=APT::Get::force-yes=1,APT::Get::Assume-Yes=1 -n mesos_1.0.0-1.0.73.rc2.ubuntu1404_amd64.deb
    rm -f *.deb
  else  
 # Install
 apt-get -y install --force-yes --install-recommends mesos=${MESOS_VERSION}\*
   fi
 
 #echo "Starting mesos master"
 service mesos-master restart

 #echo "Starting mesos slave"
 service mesos-slave restart

 echo "Installing prerequisite packages for mesos-jenkins plugin..."
 apt-get -y --force-yes install openjdk-7-jdk maven

 # Acquire latest code (either from local source on host, or latest release download) of mesos-jenkins plugin.
 MESOS_PLUGIN_VERSION="local"
 #MESOS_PLUGIN_VERSION="0.9.0"


 if [ $MESOS_PLUGIN_VERSION = "local" ]; then
    mkdir -p mesos-plugin-mesos-${MESOS_PLUGIN_VERSION}
    rm -rf mesos-plugin-mesos-${MESOS_PLUGIN_VERSION}/src
    cp -r /vagrant/src mesos-plugin-mesos-${MESOS_PLUGIN_VERSION}/src
    cp -r /vagrant/pom.xml mesos-plugin-mesos-${MESOS_PLUGIN_VERSION}/pom.xml
    chown -R vagrant:vagrant mesos-plugin-mesos-${MESOS_PLUGIN_VERSION}
 else
   echo "Downloading mesos-jenkins ${MESOS_JENKINS_VERSION}..."
   wget https://github.com/jenkinsci/mesos-plugin/archive/mesos-${MESOS_PLUGIN_VERSION}.tar.gz
   tar -zxf mesos-${MESOS_PLUGIN_VERSION}.tar.gz && rm -f mesos-${MESOS_PLUGIN_VERSION}.tar.gz
   chown -R vagrant:vagrant mesos-plugin-mesos-${MESOS_PLUGIN_VERSION}
 fi

 # Build mesos-jenkins plugin.
 echo "Building mesos-jenkins plugin"
 # TODO(vinod): Update the mesos version in pom.xml.
 su - vagrant -c "cd mesos-plugin-mesos-${MESOS_PLUGIN_VERSION} && mvn package -DskipTests"
 echo "Done"

 echo "****************************************************************"
 echo "Successfully provisioned the machine."
 echo "You can run the Jenkins server with plugin installed as follows:"
 echo "> vagrant ssh"
 echo "> cd mesos-plugin-mesos-${MESOS_PLUGIN_VERSION}"
 echo "> mvn hpi:run"
 echo "****************************************************************"
 echo "NOTE: Configure the plugin as follows:"
 echo "From local host go to localhost:8080/jenkins/configure"
 echo "Mesos native library path: /usr/local/lib/libmesos.so"
 echo "Mesos Master [hostname:port]: zk://localhost:2181/mesos"
 echo "****************************************************************"


SCRIPT


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Forward mesos ports.
  config.vm.network "forwarded_port", guest: 5050, host: 5050
  config.vm.network "forwarded_port", guest: 5051, host: 5051

  # Forward jenkins port.
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  # Provision the system.
  config.vm.provision "shell", inline: $script

  config.vm.provider :virtualbox do |vb|
     # Use VBoxManage to customize the VM. For example to change memory:
     vb.customize ["modifyvm", :id, "--memory", "2048"]
     vb.customize ["modifyvm", :id, "--cpus", "2"]
  end

end
