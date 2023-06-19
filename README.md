## Welcome to Catalyst Wireless Telemetry

This small piece of code allows you to quickly create a Grafana Dashboard 
that shows information from the Catalyst Wireless Sensors embedded in the APs.

You can find the basic instructions to run the code in the section 
[Instructions](#Instructions).

## Architecture

It uses:
- Telegraf: server-based agent for collecting and sending metrics and events.
- InfluxDB 2.0: time series database.
- Grafana: dashboard.

## Screenshots

Containers in steady state:
![Containers in steady state](/images/ContainersSteady.png)

AP Sensor Dashboard:
![AP Sensor Dashboard](/images/APSensorDashboard.png)

## Instructions

### Requirements

- Docker installed (this app runs on Docker)

### Versions

This code has been developed and tested with Python 3.8.

### Steps

1. Clone the repository.
   ```
   git clone https://wwwin-github.cisco.com/nfitelop/Catalyst-Wireless-Telemetry.git
   ```

2. Go to the directory where the `docker-compose.yml` file is located, pull and
 build all images of the container:
   ```
    cd Catalyst-Wireless-Telemetry/
    docker compose build
   ```
   
3. _[Optional]_ If you wish to edit any settings, please edit the .env file.


4. Start all the containers in the background (-d). Grafana and InfluxDB will 
   be automatically configured. 
   ```
    docker compose up -d
   ```
   
   **Note:** there is a container named _influxdb_cli_ that just needs to run 
   once, this container will do all the setup work for you and afterward 
   it will stop.
   
   **Note 2:** the container _influxdb_cli_ needs the database 
   to be up and running, that takes some time, you might see both 
   containers restarting a few times before they can do their job.

   
5. Browse to http://localhost:3000/ to open Grafana and log in using:
   - **Username**: admin
   - **Password**: admin
   
   It will ask for a password change, you might skip it.
   

6. Inside Grafana, click on the _Search_ button on the left-side menu, you 
   will see one dashboard if everything worked correctly:
   - AP Sensor Data
   
   You should be able to access it and it **will be empty**.

7. Now you shall configure the Catalyst 9800 Wireless LAN Controller (WLC) in order
   to collect the AP sensor data. Access the WLC CLI and configure the ap profile:
   ```
    ap profile default-ap-profile
     description "Default AP Profile for IOS-XE"
     sensor environment air-quality
      no shutdown
     sensor environment temperature
      no shutdown
   ```
   Further instructions on how to do so can be found in the 
   [Cisco Catalyst 9800 Series Wireless Controller Software Configuration Guide, 
   Cisco IOS XE Cupertino 17.9.x](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/17-9/config-guide/b_wl_17_9_cg/m_environmental_sensors_on_aps.html?bookSearch=true).

8. The data must be sent from the WLC to telegraf, to do so the WLC uses streaming telemetry. Configure
   the following subscriptions:
   ```
   telemetry ietf subscription 20
    encoding encode-kvgpb
    filter xpath /wireless-access-point-oper:access-point-oper-data/ap-temp
    source-address w.x.y.z
    stream yang-push
    update-policy periodic 5000
    receiver ip address a.b.c.d 57000 protocol grpc-tcp
   telemetry ietf subscription 21
    encoding encode-kvgpb
    filter xpath /wireless-access-point-oper:access-point-oper-data/ap-air-quality
    source-address w.x.y.z
    stream yang-push
    update-policy periodic 5000
    receiver ip address a.b.c.d 57000 protocol grpc-tcp
   telemetry ietf subscription 22
    encoding encode-kvgpb
    filter xpath /wireless-access-point-oper:access-point-oper-data/ap-name-mac-map
    source-address w.x.y.z
    stream yang-push
    update-policy periodic 5000
    receiver ip address ab.c.d 57000 protocol grpc-tcp
   ```
   Replace `w.x.y.z` with the IP address of the WLC. Replace `a.b.c.d` with the IP address
   of the machine running telegraf. Ensure that there's no firewall in between blocking the port 57000.
   
   The subscription numbers are irrelevant as long as they don't overlap. The last subscription
   sends the corresponding hostnames for each ap mac address allowing for a nicer print.
    
### Cleanup

You can bring down all containers in this sample app with:
```
   docker compose down
```

To make sure they're gone, check with:
```
   docker compose ps
```

## License

Check the [LICENSE][LICENSE] file attached to the project to see all the 
details.

[LICENSE]: ./LICENSE.md