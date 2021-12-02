# VCC EndSem  - TICK Stack

## Team Members
- Garvit Chugh (M20CS018)
- Samanyu A Saji (M20CS013)

## What security measures after setting up TICK Stack ?
- HTTPS Security - Running TICK stack by default work on http, instead of https, which is not secure. Hence, we will be trying to make our connection secure by converting it into https. This will be done by using a self generated certificate using openssl. More detailed instructions in Setup section.
- Authentication for for InfluxDB and Kapacitor - After setting up HTTPS, we setup authentication for InfluxDB as explained in the below section.

## Setup
Since TICK Stack is the combination of 4 components, in order to run and handle them simultaneously using Docker, we can make use of the docker-compose file. 

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
After setting up HTTPS, goto https://localhost:8888. This leads us to a prompt, saying 'Get Started'. Click on that to go to a page, where we will be able to set up authentication for InfluxDB. Replace the 'connection URL' with the name of the of the InfluxDB container (influxdb in our case) and create new username and password to access InfluxDB.

Similarly, they will be asking for connection to kapacitor as well. Here too, replace the connection url with 'http://kapacitor:9092', where kapacitor is the container name and 9092 is the port number. Along with it, add the username and password too.

This further tightens up the overall security of the TICK Stack system.

So now, once you move into the dashboard, select any host present in the HostList. We will be able to see their visualization, signalling that TICK  Stack is working properly.

## References
- [Manage Chronograf security](https://docs.influxdata.com/chronograf/v1.9/administration/managing-security/#configure-tls-transport-layer-security-and-https)
- [Setting Up HTTPS](http://cactusprojects.com/setup-https-for-grafana/) 
