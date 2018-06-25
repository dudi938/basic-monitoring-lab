# Basic Monitoring
Lab: Basic monitoring using Docker containers

---

## Setting up the environment


### Initialize the environment

 - Clone the repository to get the docker-compose file

```
$ git clone https://github.com/leonjalfon1/basic-monitoring-lab.git
$ cd basic-monitoring-lab
```

 - Set up the environment by run:

```
$ docker-compose up -d
```


### Exploring Grafana


 - Browse to the Grafana portal:

```
$ http://localhost:3000
```

 - Login using the credentials below:

```
username: admin
password: admin
```

 - You will be asked for set a new password, set the password below:

```
password: admin
```

 - Explore Grafana on your own
 
 

### Exploring Graphite


 - Browse to the Graphite portal:

```
$ http://localhost:5000
```

 - Login using the credentials below:

```
username: root
password: root
```

 - Explore Graphite on your own
 
 

### Exploring the application

 - Browse to the application page:

```
$ http://localhost:80
```

 
 
### Configure collectd in the application server

 - Move to the application container terminal:

```
$ docker exec -it app /bin/bash
```

 - Install collectd tool:

```
$ apt-get update
$ apt-get install -y collectd collectd-utils
```

 - Install vim tool:

```
$ apt-get install -y vim
```

 - Configure collectd to send metrics to Graphite:

```
$ vim /etc/collectd/collectd.conf
```

 - Enable the following plugins:

```
LoadPlugin cpu
LoadPlugin df
LoadPlugin interface
LoadPlugin load
LoadPlugin memory
LoadPlugin ping
LoadPlugin write_graphite
```

 - Configure the following plugins as below:

```
<Plugin ping>
        Host graphite
#       Host "host.baz.qux"
        Interval 1.0
        Timeout 0.9
        TTL 255
#       SourceAddress "1.2.3.4"
#       Device "eth0"
        MaxMissed -1
</Plugin>

<Plugin write_graphite>
        <Node "graphite">
                Host graphite
                Port "2003"
                Protocol "tcp"
                LogSendErrors true
#                Prefix "collectd"
#               Postfix "collectd"
                StoreRates true
                AlwaysAppendDS false
                EscapeCharacter "_"
        </Node>
</Plugin>
```

 - For changes to take affect, please restart collectd service :

```
$ /etc/init.d/collectd restart
```

 - Browse to Graphite and look for the new metrics:

```
$ http://localhost:5000/
```

### Add a Data Source and Create a Grafana Dashboard

 - Click Create data source in the main dashboard and fill in the form as follows:

```
Name: Graphite
Type: Graphite
URL: http://graphite:5000
Access: Server (Default)
Version: <Select the newest available>
```

 - Click New dashboard to create and customize a new panel:



### Jenkins Configuration

 - Browse to the jenkins portal:

```
http://jenkins.sela.com:8080
```

 - Unlock jenkins using the administrator password, use the command below to retrieve it:

```
$ docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

![Image 1](Images/lab01-jenkins-01.png)

 - Select "Install suggested plugins" and wait for the plugins to being installed: 

![Image 2](Images/lab01-jenkins-02.png)
 
 - Set "http://jenkins.sela.com:8080" as the Jenkins URL: 

![Image 3](Images/lab01-jenkins-03.png)

 - You will be asked for credentials, set the details below:

```
Username: administrator
Password: administrator
FullName: administrator
Email: administrator@administrator.com
```

![Image 4](Images/lab01-jenkins-04.png)
 
 - You can update the user details from "Manage Jenkins"/"Manage Users"/<settings icon>




### Jenkins Slave Configuration

 - Create a new folder to be used for the Jenkins slave:

```
$ sudo mkdir /home/jenkins
$ sudo chmod 777 /home/jenkins
```

 - Configure the ubuntu server as a jenkins slave, start creating a new node in the jenkins portal:

```
"Manage Jenkins" / "Manage Nodes" / "New Node"
Set "slave" as name and select "Permanent Agent"
number of executors: 5
Remote root directory: /home/jenkins
Labels: slave
Launch method: Launch agent via java web start
```
 
![Image 5](Images/lab01-slave-01.png)
 
![Image 6](Images/lab01-slave-02.png)

 - Run the slave using the secret shown in the node page and the "agent.jar" from the "infrastructure" repository:

```
$ cd ~/infrastructure
$ java -jar -jnlpUrl http://jenkins.sela.com:8080/computer/slaves/slave-agent.jnlp -secret <secret> -workDir "/home/jenkins"
```
 
![Image 7](Images/lab01-slave-03.png)

 - If you exit from the terminal (or interrupt the process), the slave will be disconnected.


### GitLab Configuration

 - Browse to the gitlab portal:

```
http://gitlab.sela.com
```

 - The first time you will asked for set the admin password, set the password below:

```
administrator
```

![Image 8](Images/lab01-gitlab-01.png)

 - Then, to login use the following credentials:

```
Username: root
Password: administrator
```

![Image 9](Images/lab01-gitlab-02.png)




### Environment Sanity Test

 - Create a new repository to test the configuration by click "Create a Project":

![Image 10](Images/lab01-sanitiy-01.png)

 - Set the repository name (set-up-test) with "public" visibility:

![Image 11](Images/lab01-sanitiy-02.png)

 - Add a README file to the repository:

```
README.md --> "Hello World!"
```

![Image 12](Images/lab01-sanitiy-03.png)

 - Copy the repository URL":

![Image 13](Images/lab01-sanitiy-04.png)

 - Browse to Jenkins and create a new "Freestyle Job" with the configuration below:

```
Name: set-up-test
SCM --> Git: http://gitlab.sela.com/root/set-up/test.git
Restrict where this project can be run: slave
Execute Shell: cat README.md
```

![Image 14](Images/lab01-sanitiy-05.png)

![Image 15](Images/lab01-sanitiy-06.png)

![Image 16](Images/lab01-sanitiy-07.png)




## Tips

 - You can look for the ubuntu server IP to access to jenkins and gitlab from the host instead of from the VM:

```
$ sudo ifconfig | grep inet
http://<server-ip>:8080 (jenkins)
http://<server-ip>:80 (gitlab)
```