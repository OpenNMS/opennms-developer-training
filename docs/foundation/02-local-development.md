
# Local Development

## Goal

Compile OpenNMS from source and perform remote debugging.

## Hands On

### 1) Vagrant

We will walk through setting up a development environment using a new Ubuntu 20.04.1 LTS (Focal) virtual machine created using [Vagrant](https://www.vagrantup.com/).

The remaining instructions asume that Vagrant is installed and that we are using [VirtualBox](https://www.virtualbox.org/) as the target hypervisor on the system.

In order to install the VirtualBox Guest Additional automatically in the Vagrant VM, install the following plugin:
```
vagrant plugin install vagrant-vbguest
```

On the host system, clone this source tree:
```
git clone https://github.com/OpenNMS/opennms-developer-training
cd opennms-developer-training/devlab
```

And start the VM using Vagrant:
```
vagrant up
```

### 2) System configuration

Add swap:
```
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Install development tools:
```
sudo apt install openjdk-8-jdk postgresql-12 nsis firefox snmp snmp-mibs-downloader
```

Configure Java:
```
echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
```

Setup PostgreSQL:
```
sudo systemctl enable postgresql
```

Edit `/etc/postgresql/12/main/pg_hba.conf`:
```
%s/peer/trust/g
%s/md5/trust/g
```

Restart PostgreSQL:
```
sudo systemctl restart postgresql
```

Install Eclipse:
```
sudo snap install --classic eclipse
```

Setup Eclipse:
```
sudo cp /snap/eclipse/current/eclipse.ini /etc/eclipse.ini
```

Edit `/etc/eclipse.ini` and set -Xmx to `-Xmx2g`.

Open up the MenuEditor and change the command in the path from `/snap/bin/eclipse %U` to `/snap/bin/eclipse  --launcher.ini /etc/eclipse.ini %U`

Start Eclipse using menu icon.

### 3) Build OpenNMS

Clone the source tree for Horizon 26.2.2:
```
mkdir ~/git && cd ~/git
git clone -b opennms-26.2.2-1 https://github.com/OpenNMS/opennms.git
```

Open the following URL in Firefox and download the archive (1.4GB):
```
https://drive.google.com/file/d/1paV6fXvdLjwVNOU9PoXbnx4R0DIVl9Qa/view?usp=sharing
```

Extract the archive and use it to prime the Maven repository:
```
tar zxvf h26-m2-repo-seed.tgz
mkdir ~/.m2
mv repository ~/.m2
rm -f h26-m2-repo-seed.tgz
```

Start compilation:
```
cd ~/git/opennms
./compile.pl -DskipTests=true
```

Create the assembly:
```
cd ~/git/opennms
./assemble.pl -DskipTests=true
```

Extract the assembly:
```
cd ~/git/opennms/target
mkdir opennms-26.2.2
tar zxvf opennms-26.2.2.tar.gz -C opennms-26.2.2
ln -s opennms-26.2.2 opennms
```

Setup helpers:
```
echo 'export OPENNMS_HOME=/home/vagrant/git/opennms/target/opennms' >> ~/.bashrc
echo 'alias oh="cd $OPENNMS_HOME"' >> ~/.bashrc
source ~/.bashrc
```

Start OpenNMS:
```
oh
sudo -E ./bin/runjava -s
sudo ./bin/install -dis
sudo ./bin/opennms -t start
tail -f logs/manager.log
```

> Wait for `Starter: Startup complete` message.

Open Firefox and navigate to `http://localhost:8980`, login with username `admin` and password `admin`.

### 4) OpenNMS in Eclipse

Import the project by opening the import dialog `File -> Import`.

In the import dialog select `Maven -> Existing Maven Projects`.

Use this path `/home/vagrant/git/opennms/features/events`.

> We only import a subset of the projects here for expediency.

Once imported create a new launch configuration by selection `Run -> Debug Configuration`.

Select `Remove Java Application` and click `New Launch Configuration`.

Enter these values:
```
Name: opennms debug
Host: localhost
Port: 8001
```

Click `Apply` and `Close`.

Open the `TrapSinkConsumer` class in the `org.opennms.netmgt.trapd` package of the `org.opennms.features.events.traps` module.

Set a breakpoint on line 103:
```
final Log eventLog = toLog(messsageLog);
```

Start debugging with the `opennms debug` configuration.

Switch to the `Debug` perspective.

Switch to a shell and send a trap with:
```
snmptrap -v 1 -c public localhost '' localhost 6 1 ''
```

Our breakpoint should be hit.