# VCC EndSem  - TICK Stack

## What is TICK Stack 
TICK Stack is a collection of open source components that combine to deliver a platform for easily storing, visualizing and monitoring time series data such as metrics and events. The components are: 
- Telegraf, a server agent for collecting and reporting metrics; 
- InfluxDB, a high-performance time series database; 
- Chronograf, a user interface for the platform and; 
- Kapacitor, a data-processing engine that can process, stream and batch data from InfluxDB. 

## What are we doing after setting up TICK Stack ?
- HTTPS Security - Running TICK stack by default work on http, instead of https, which is not secure. Hence, we will be trying to make our connection secure by converting it into https. This will be done by using a self generated certificate using openssl. More detailed instructions in Setup section.
- Security for InfluxDB - After setting up HTTPS, we setup authentication for InfluxDB as explained in the below section.

## Setup
Since TICK Stack is the combination of the above 4 components, in order to run and handle them simultaneously using Docker, we can make use of the docker-compose file. 

### docker-compose file
Let's have a quick overview of what's present in the docker-compose.yaml.
- Containers for TICK Stack - Obviously, it will have the container details for all 4 components of TICK Stack.
- Network - We have made a custom network, called 'mynetwork'.
- Persistant Storage - We will be having a persistant storage for chronograf as it is the user interface of TICK stack. Here, we will be storing the certificates required for making the connection, https. Persistant storage is setup using the volumes keyword. 
- Environment Variables - We will be setting up environment variables for both chronograf and kapacitor. In the case of chronograf, we will be setting up the TLTLS_CERTIFICATE and TLS_PRIVATE_KEY. We will also be setting a value for the first URL in the first InfluxDB configuration group using KAPACITOR_INFLUXDB_0_URLS_0.

### Certificate Generation and conversion to https
We will be making use of openssl to generate own own certificates for chronograf. We are using RSA-2048 secure encryption scheme for this. 
```
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
```
(Output for certificate generation has been attached in this [file](openssl_output.txt)).

After creating the certificates, we change key permissions using chmod and move them to a newly created directory in home.
```
sudo chmod -R 777 cert.pem
sudo chmod -R 777 key.pem
sudo mkdir /home/certificates
sudo mv *.pem /home/certificates
```
This location of '/home/certificates' will be made as a permanant storage so that we don't need to carry out the setup for certificates over and over again.

After this, run the docker-compose file.
```
docker-compose up -d
```

According to our setup, TICK Stack UI, i.e, chronograf, is assigned with the port 8888. Now, try accessing http://localhost:8888. We won't be able to connect to it. Instead let's try https://localhost:8888. There you go, it starts loading now!. So we have added https security to our local TICK Stack system.

### Setting up InfluxDB security
After setting up HTTPS, goto https://localhost:8888. This leads us to a prompt, saying 'Get Started'. Click on that to go to a page, where we will be able to set up authentication for InfluxDB. Replace the 'connection URL' with the IP address of the InfluxDB container and create new username and password to access InfluxDB.

In order to get IP address of the InfluxDB container, use the below docker command.
```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' {containerID}
```
, where you should replace {containerID} with the container ID of InfluxDB container. 

Finally, when setting up kapacitor connection, rename connection url from as http://kapacitor:9092. This helps us to establish the connection with kapacitor too.

This will help us to create another set of password protected authentication, further securing our system.

## References
- [Manage Chronograf security](https://docs.influxdata.com/chronograf/v1.9/administration/managing-security/#configure-tls-transport-layer-security-and-https)
- [Setting Up HTTPS](http://cactusprojects.com/setup-https-for-grafana/) # vccendsem
