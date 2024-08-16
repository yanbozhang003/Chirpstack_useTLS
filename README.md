
### This repo provides a direct guidance on how to set up Chirpstack with MQTT TLS. 

1. Configuring MQTT Broker (using mosquitto)
a. creating username and password for MQTT Broker access
      ```
      mosquitto_passwd -c /etc/mosquitto/passwd <your_username>
      ```
      Modifying file permission (need sudo)
      ```
      sudo chown mosquitto:mosquitto /etc/mosquitto/passwd
      ```
      
b. Editing the configuration file of mosquitto
   
3. Configuring Chirpstack
4. Configuring Gateway

### Troubleshooting
