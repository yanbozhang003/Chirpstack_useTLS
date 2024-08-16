### This repo provides a direct guidance on how to set up Chirpstack with MQTT TLS.
1. **Configuring MQTT Broker (using mosquitto)**

   1. **Creating a username and password for MQTT Broker access**

      ```
      mosquitto_passwd -c /etc/mosquitto/passwd <your username>
      ```

      **Modifying the file owner:**

      ```
      sudo chown mosquitto:mosquitto /etc/mosquitto/passwd
      ```

   2. **Editing the configuration file (/etc/mosquitto/mosquitto.conf) of mosquitto**

      ```
      # Enable username and password
      password_file /etc/mosquitto/passwd

      # Enable TLS/SSL
      listener 8883
      cafile /etc/mosquitto/ca_certificates/<your CA certificate>
      certfile /etc/mosquitto/certs/<your Broker certificate>
      keyfile /etc/mosquitto/certs/<your Broker private key>

      # Keep using an unencrypted connection
      listener 1883
      allow_anonymous false
      ```

      For information on generating certificates and keys, refer to [this link](https://github.com/chirpstack/chirpstack-certificates).

      **Modifying the permission of loading the private key:**

      ```
      # Change the file owner
      sudo chown mosquitto:mosquitto /etc/mosquitto/certs/<your Broker private key>

      # Change the mode
      sudo chmod 600 /etc/mosquitto/certs/<your Broker private key>
      ```

      If needed (error still occurs when loading the private key), remove the passphrase from the private key. This is common in server environments where services need to start without manual intervention:

      ```
      openssl rsa -in /etc/mosquitto/certs/<your Broker private key> -out /etc/mosquitto/certs/<your Broker private key>
      ```

      This command will prompt you for the current passphrase and then save an unencrypted version of the key (i.e., without the passphrase) back to the same file.

   3. **Restart mosquitto to load the modifications:**

      ```
      sudo systemctl restart mosquitto.service
      ```

      Check the status to ensure a successful restart.

2. **Configuring Chirpstack**

   1. **Modifying the "integration" section of "chirpstack.toml":**

      ```toml
      [integration]

      # Enabled integrations (global).
      enabled = ["mqtt"]

      # MQTT integration configuration.
      [integration.mqtt]

      # MQTT server (e.g. scheme://host:port where scheme is tcp, ssl or ws)
      server = "ssl://<your mqtt broker ip address>:8883/"

      # Username and password for MQTT broker connection
      username = "your_username"
      password = "your_password"

      # Specifying CA certificate if it is self-signed
      ca_cert = "<path to the CA certificate of MQTT broker>"
      ```
      Note that the mqtt broker address needs to be registered when requesting the server certificate from CA. Find out how-to from the instruction of [chirpstack certificate scripts](https://github.com/chirpstack/chirpstack-certificates). 

   2. **Modifying the "regions.gateway.backend.mqtt" section of "region_XXXX.toml":**

      ```toml
      [regions.gateway.backend.mqtt]

      # MQTT server (e.g. scheme://host:port where scheme is tcp, ssl or ws)
      server = "ssl://<your mqtt broker ip address>:8883"

      # Username and password for MQTT broker connection
      username = "your_username"
      password = "your_password"

      # Specifying CA certificate if it is self-signed
      ca_cert = "<path to the CA certificate of MQTT broker>"
      ```
   3. **Restarting the Chirpstack service:**

      ```
      sudo systemctl restart chirpstack.service
      ```
      Check out the status to verify whether the connection to MQTT broker is successful.

3. **Configuring the Gateway**

   1. **Modifying the "mqtt" section of the MQTT configuration file:**

      ```toml
      # MQTT server (e.g. scheme://host:port where scheme is tcp, ssl or ws)
      server = "ssl://<your mqtt broker ip address>:8883"

      # Username and password for MQTT broker connection
      username = "your_username"
      password = "your_password"

      # Specifying CA certificate if it is self-signed
      ca_cert = "<path to the CA certificate of MQTT broker>"
      ```

   2. **Starting the MQTT connection with the [Chirpstack MQTT Forwarder](https://github.com/chirpstack/chirpstack-mqtt-forwarder):**

      ```
      ./chirpstack-mqtt-forwarder -c <path to your MQTT configuration file>
      ```

      Check the log files to verify a successful MQTT connection.

### Note
Using self-signed CA certificate (e.g., using openssl or cfssl) is a cost-effective method for a fully controllable local network, but may not be recognized by many public client services (e.g., browsers, operating systems). For commercial/public usage, please obtain your certificate from authorized CA such as DigiCert and [Let's Encrypt](https://letsencrypt.org/) (free). 
