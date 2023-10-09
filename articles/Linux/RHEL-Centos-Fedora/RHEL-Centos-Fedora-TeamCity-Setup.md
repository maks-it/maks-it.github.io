# TeamCity Setup

## Install OpenJDK JRE 17


```bash
sudo dnf install -y java-17-openjdk
```

Ensure that JRE or JDK are stalled and the `JAVA_HOME` environment variable is pointing to the Java installation directory 

ls -lh /usr/lib/jvm

export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-17.0.8.0.7-1.fc38.x86_64

echo $JAVA_HOME

```bash
sudo nano /etc/profile
```

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-17.0.8.0.7-1.fc38.x86_64
export PATH=$PATH:$JAVA_HOME/bin
```


Save the file and close the editor. Reload the file into the bash shell to take the changes into effect.

```bash
source /etc/profile
```

## Teamcity group and user

It is highly recommended running the TeamCity server under a dedicated user account.


```bash
groupadd -r teamcity
useradd -M -r -g teamcity teamcity

mkdir -p /home/teamcity
chown teamcity:teamcity /home/teamcity -R
chmod 700 /home/teamcity -R

passwd teamcity
```

## Install git

```bash
dnf install git -y
```

## Install Teamcity

Go to the JetBrains website and download the .tar.gz distribution with the "portable" version of the TeamCity server.

```bash
tar xfz TeamCity<version number>.tar.gz
```

Copy the folder to `teamcity` user's home directory

```bash
cp TeamCity /home/teamcity/
```

Reapply folder permissions

```bash
chown teamcity:teamcity /home/teamcity -R
chmod 700 /home/teamcity -R
```

## Start server

To start/stop the TeamCity server only, use the teamcity-server scripts and pass the required parameters. Start the script without parameters to see the usage instructions. The teamcity-server scripts support the following options for the stop command:

stop n — sends the stop command to the TeamCity server and waits up to n seconds for the process to end.

stop n -force — sends the stop command to the TeamCity server, waits up to n seconds for the process to end, and terminates the server process if it did not stop.

The TeamCity server will restart automatically if the server process exits (crashes or is killed) without invoking the teamcity-server stop script.

/home/teamcity/TeamCity/bin/teamcity-server.sh start

or

runuser -l  teamcity -c /home/teamcity/TeamCity/bin/teamcity-server.sh start


## Firewall to access UI

The TeamCity UI can be accessed via a web browser. The default addresses are http://localhost:8111


## Initialization

Yo can leave default Data Directory location on the TeamCity server machine:

/home/teamcity/.BuildServer

Continue with internal db, migration to Postgres we will see later, as maybe you do not need it

Check
 Accept license agreement

Uncheck
 Send anonymous usage statistics to TeamCity development team (can be turned off at any time)

 Create your user account for webui

## TeamCity agent setup

Follow the same

## Install and configure Docker

```bash
dnf install docker -y
```

```bash
sudo systemctl enable --now docker
```

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
sudo usermod -aG docker teamcity
```

## Add custom container registry root certificate

By following these steps, you ensure that Docker uses the custom CA certificate when connecting to the Harbor registry with the specified hostname. This approach is particularly useful for self-hosted or private registries like Harbor that require custom CA certificates.

```bash
sudo mkdir -p /etc/docker/certs.d/<cr_hostname>
```

```bash
sudo cp /path/to/custom-certs/ca.crt /etc/docker/certs.d/myregistry.example.com/ca.crt
```

> Replace /path/to/custom-certs/ca.crt with the actual path to your custom CA certificate.

```bash
sudo systemctl restart docker
```

You can now use Docker with your custom CA certificate for the specific hostname. You can pull images from the Harbor registry as follows:

```bash
docker pull myregistry.example.com/myimage:tag
```

Docker will use the custom CA certificate located in the `/etc/docker/certs.d/<cr_hostname>/` directory for this specific hostname.

## Install Teamcity Agent

```bash
dnf install unzip -y
```

Extract the downloaded file into an arbitrary directory.


```bash
sudo unzip buildAgentFull.zip -d /home/teamcity/TeamCityAgent
```


```bash
chown teamcity:teamcity /home/teamcity -R
chmod 700 /home/teamcity -R
```

Open the <installation path>\conf directory and rename the buildAgent.dist.properties file to buildAgent.properties.

Edit the buildAgent.properties file to specify the TeamCity server URL (HTTPS is recommended, see these notes) and the name of the agent. Refer to this article for details on the agent configuration.

On Linux, you may need to give execution permissions to the bin/agent.sh shell script.


```bash
/home/teamcity/TeamCityAgent/bin/agent.sh start
```