
### This repo provides a direct guidance on how to set up Chirpstack with MQTT TLS. 

1. Configuring MQTT Broker (using mosquitto)
   1) Creating username and password for MQTT Broker access
      ```
      mosquitto_passwd -c /etc/mosquitto/passwd <your username>
      ```
      Modifying file owner:
      ```
      sudo chown mosquitto:mosquitto /etc/mosquitto/passwd
      ```
   2) Editing the configuration file (/etc/mosquitto/mosquitto.conf) of mosquitto
      ```
      # Enable username and password
      password_file /etc/mosquitto/passwd
      
      # Enable TLS/SSL
      listener 8883
      cafile /etc/mosquitto/ca_certificates/<your CA certificate>
      certfile /etc/mosquitto/certs/<your Broker certificate>
      keyfile /etc/mosquitto/certs/<your Broker private key>
      
      # Keep using unencrypted connection
      listener 1883
      allow_anonymous false
      ```
      About generating certificates and keys, check out [here](https://github.com/chirpstack/chirpstack-certificates).

      Modifying the permission of loading private key:
      ```
      # Change file owner
      sudo chown mosquitto:mosquitto /etc/mosquitto/certs/<your Broker private key>

      # Change mode
      sudo chmod 600 /etc/mosquitto/certs/<your Broker private key>
      ```

      If needed (error still happens on loading the private key), remove the passphrase from the private key (which is common for server environments where services need to start without manual intervention):
      ```
      openssl rsa -in /etc/mosquitto/certs/server.key -out /etc/mosquitto/certs/<your Broker private key>
      ```
      This command will prompt you for the current passphrase and then save an unencrypted version of the key (i.e., without the passphrase) back to the same file.

      Re-start mosquitto to load the modifications:
      ```
      sudo systemctl restart mosquitto.service
      ```
      Check out the status to make sure successful restart. 
   
3. Configuring Chirpstack
4. Configuring Gateway

### Troubleshooting
