# ADS over MQTT

## Disclaimer

This guide is a personal project and not a peer-reviewed publication or sponsored document. It is provided “as is”, without any warranties — express or implied — including, but not limited to, accuracy, completeness, reliability, or suitability for any purpose. The author(s) shall not be held liable for any errors, omissions, delays, or damages arising from the use or display of this information.

All opinions expressed are solely those of the author(s) and do not necessarily represent those of any organization, employer, or other entity. Any assumptions or conclusions presented are subject to revision or rethinking at any time.

Use of this information, code, or scripts provided is at your own risk. Readers are encouraged to independently verify facts. This content does not constitute professional advice, and no client or advisory relationship is formed through its use.

## Overview

In addition to the regular method of creating a route to another device via ADS, TwinCAT also provides an in-built ability to create a route by using ADS over MQTT.

InfoSys provides the following [overview](https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_ads_over_mqtt/17769357707.html&id=4800773352293992935) for ADS over MQTT:

> *"Beckhoff ADS (Automation Device Specification) is a communication protocol developed by Beckhoff for efficient data exchange in industrial automation systems. It serves as the backbone for the integration of devices and software into the PC-based control technology from Beckhoff. From the perspective of the ADS protocol, ADS-over-MQTT is an additional transport channel over which ADS can be transported. Decoupling communication via an MQTT message broker results in a number of advantages, particularly in terms of scalability and flexibility when integrating additional ADS applications. Security mechanisms such as TLS can be used at the transport layer to secure the communication connection. With ADS-over-MQTT, the entire data exchange is transparent for the ADS applications, because only the ADS router needs to know and hold the corresponding information on the MQTT transport channel. In particular, this also enables easy retrofitting for existing applications. The main use case for ADS-over-MQTT is a classic remote maintenance and remote diagnostics scenario, where the TwinCAT engineering environment (TwinCAT XAE) needs to connect to one or more controllers for remote debugging."*

## ADS over MQTT

MQTT is a lightweight, publish–subscribe, machine-to-machine network protocol for message queue/message queuing service.

The MQTT protocol defines two types of network entities: a message broker and a number of clients. An MQTT broker is a server that receives messages from publishing clients and then routes the messages to the appropriate destination clients.

Both TwinCAT XAE and XAR installations have the in-built ability to act as an MQTT client to transport ADS payloads - no extra installation is required.

The relationship between XAE/XAR clients and a Broker can be seen in the following diagram:

![ADS-over-MQTT relationship](docs/images/ADS-over-MQTT-relationship.png)

Any MQTT broker can be used. A typical installation uses [Mosquitto](https://mosquitto.org/).

The broker can be installed anywhere on a network, including the 'cloud' via the internet. This allows for a truly distributed solution. If a client can ‘find’ the broker on the network, then it can be used. This is a networking problem to solve, and not a TwinCAT problem.

Once a client can connect with the broker, it immediately registers itself by publishing and subscribing to defined topics. These are observable via any other ADS-over-MQTT clients that connect to the broker. Routes are then immediately available for selection from with XAE without the need to use the TwinCAT Route search dialog.

See [InfoSys](https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_ads_over_mqtt/17768315019.html&id=8793359276566777531) here.

## Configuration

### Initial configuration – TwinCAT

An important point to note: ***Before establishing an ADS-over-MQTT connection, any existing 'regular' ADS routes to the target must be removed.*** For example, if an XAE has previously connected to an XAR on a controller, the static ADS route must be deleted on ***both systems*** before attempting to create the MQTT based route.

Next, an XML file must be created in the ***“%TWINCAT%\3.1\Target\Routes\”*** directory. The “Routes” directory itself may need to be created manually if it does not already exist.

The ***“%TWINCAT%”*** installation path depends on the build version of TwinCAT being used.

| Build | Path |
|---|---|
| <4026 | C:\TwinCAT\3.1\Target\Routes |
| >=4026 | C:\Program Files (x86)\Beckhoff\TwinCAT\3.1\Target\Routes | 

This file can have any name but must have an .xml extension. A suggested filename is ***“mqtt.xml”***.

The following initial basic format assumes the broker has been installed locally to the runtime and is being used over an unsecured connection.

#### mqtt.xml
```xml
<?xml version="1.0" encoding="ISO-8859-1"?> 
<TcConfig xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.beckhoff.com/schemas/2015/12/TcConfig"> 
<RemoteConnections>
  <Mqtt>
    <Address Port="1883">127.0.0.1</Address>
    <Topic>VirtualAmsNetwork1</Topic>
    </Mqtt>
  </RemoteConnections>
</TcConfig>
```

Broker located at 127.0.0.1, i.e. local home address.
Port 1883 is the default port number for unsecured MQTT communications.
```xml
<Address Port=”1883”>127.0.0.1</Address>
```

This is the topic used by this MQTT connection to communicate with other clients that share the same topic name. This allows for ‘virtual’ networks to be created within the broker.
```xml
<Topic>VirtualAmsNetwork1</Topic>
```

Any changes to this file must be activated with a restart of the runtime for the changes to come into effect.

Any ADS-over-MQTT client that uses ***"VirtualAmsNetwork1"*** as the topic name can form a route to each other.

From InfoSys:

![ADS-over-MQTT virtual networks](docs/images/ADS-over-MQTT-virtual-networks.png)

### Initial configuration – Mosquitto

Mosquitto uses a configuration file named ***”mosquito.conf”***.

In its most basic form, it has the following format which allows anonymous connections and configures the broker to listen on port 1883, which is the default for unsecure conections:

#### mosquitto.conf
```bash
allow_anonymous true
listener 1883
```

### Further Configuration - TwinCAT

The setup described above allows for an unsecure connection between MQTT clients and the broker.

Further configuration of both TwinCAT and the broker can be made to enable more secure connections.

The ADS route can be made unidirectional. This may be an important consideration for ‘engineering’ clients, such as using an XAE on a laptop. This allows only ADS commands to be made to a target from XAE but will not allow commands in the reverse direction from the target to XAE. This setting is directly analogous the the 'Unidirectional' setting on the regular Route search dialog box.

To enable this, the ***'Unidirectional'*** attribute can be added to the mqtt.xml file:

```xml
<Mqtt Unidirectional="true">
```

#### mqtt.xml
```xml
<?xml version="1.0" encoding="ISO-8859-1"?> 
<TcConfig xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.beckhoff.com/schemas/2015/12/TcConfig"> 
<RemoteConnections>
   <Mqtt Unidirectional="true">
   <Address Port="1883">127.0.0.1</Address>
    <Topic>VirtualAmsNetwork1</Topic>
    </Mqtt>
  </RemoteConnections>
</TcConfig>
```

Beckhoff also provide a plugin for Mosquitto named ***“TcMqttPlugin.dll”***.
This was developed to enable the definition of access rights between the individual TwinCAT ADS routers.
***However, development and support of this plugin has been discontinued, so it is not recommended for future use.***

Secure TLS connection on the TwinCAT side can be configured requiring the use of certificates and keys.

The ***%TWINCAT%\3.1\Target\Certificates\*** directory may need to be manually added in the same way as for the 'Routes' directory.

```xml
<?xml version="1.0"?>
<TcConfig xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.beckhoff.com/schemas/2015/12/TcConfig">
	<RemoteConnections>
		<Mqtt Unidirectional="true">
			<Address Port="8883">nt-baa-vm03</Address>
			<Topic>VirtualAmsNetwork1</Topic>
			<Tls IgnoreCn="false">
				<Ca>C:\Program Files (x86)\Beckhoff\TwinCAT\3.1\Target\Certificates\ca.crt</Ca> 			
          <Cert>C:\Program Files (x86)\Beckhoff\TwinCAT\3.1\Target\Certificates\mosquitto_client_VM-XAE.crt</Cert> 
          <Key>C:\Program Files (x86)\Beckhoff\TwinCAT\3.1\Target\Certificates\mosquitto_client_VM-XAE.key</Key>
      </Tls> 
		</Mqtt>
	</RemoteConnections>
</TcConfig>
```

Configure the broker to use port 8883, which is the default for secure connections.
```xml
<Address Port="8883">nt-baa-vm03</Address>
```

Add the Tls section an point to the directory holding the certificates and key files.
```xml
<Tls IgnoreCn="false">
  <Ca>C:\Program Files (x86)\Beckhoff\TwinCAT\3.1\Target\Certificates\ca.crt</Ca> 			
  <Cert>C:\Program Files (x86)\Beckhoff\TwinCAT\3.1\Target\Certificates\mosquitto_client_VM-XAE.crt</Cert> 
  <Key>C:\Program Files (x86)\Beckhoff\TwinCAT\3.1\Target\Certificates\mosquitto_client_VM-XAE.key</Key>
</Tls>
```

Set the ***IgnoreCn*** attribute to false to ensure verification of the CommonName (CN) proprty of the server certificate.
```xml
<Tls IgnoreCn="false">
```

### Further Configuration - Mosquitto

Secure TLS connection on the server side can be configured requiring the use of certificates and keys.

#### mosquitto.conf
```bash
allow_anonymous false
listener 8883
cafile C:\certs\ca.crt
certfile C:\Program Files\mosquitto\certs\mosquitto.crt
keyfile C:\Program Files\mosquitto\certs\mosquitto.key
require_certificate true
use_identity_as_username true
```

***'require_certificate true'*** means a client must have a valid Tls certificate to be able to connect to the broker.
***'use_identity_as_username'*** means the CommonName (CN) proprty of the client certificate is used as a user name within Mosquitto.
In conjunction with ***'allow_anonymous false'*** this means that a user name ***must*** be provided.

## OpenSSL

Certificates and keys must be used to facilitate the use of TSL secure connections.

OpenSSL can be used to generate these files. Some notes can be found [here](./docs/openssl.md).
