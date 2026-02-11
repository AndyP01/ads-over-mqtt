# Create CA certificate and key

## Step 1 - Create a private certificate key for the root CA
```bash
openssl genrsa -out ca.key 4096
```

## Step 2 - Create certificate of the root CA
```bash
openssl req -new -x509 -noenc -key ca.key -sha256 -days 3650 -out ca.crt
```

## Step 3 - Add the CA certificate to the trusted root certificates
TODO What needs to be done here?

# Create certificate and key for the server side

## Step 1 - Create a private certificate key for the service (e.g. Mosquitto)
```bash
openssl genrsa -out mosquitto.key 4096
```

## Step 2 - Create a service certificate request
```bash
openssl req -new -key mosquitto.key -out mosquitto.csr

	Country Name : AU
	State or Province Name : Victoria
	Locality Name : Melbourne
	Organization Name : BAAU
	Organizational Unit Name : <empty>
	Common Name : nt-baa-vm03
	Email Address : <empty>
	
	Challenge password : <empty>
	Optional company name : <empty>
```

## Step 3 - Sign the certificate
```bash
openssl x509 -req -in mosquitto.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mosquitto.crt -days 365 -sha256
```
	
## Step 4 - Deploy the certificate
Copied mosquitto.key and mosquitto.crt to C:\Program Files\Mosquitto\certs

# Create certificate and key for the client side

## Step 1 - Create a private certificate key for the client
```bash
openssl genrsa -out mosquitto_client_AndrewPa.key 4096
```

## Step 2 - Create a client certificate request
```bash
openssl req -new -key mosquitto_client_AndrewPa.key -out mosquitto_client_AndrewPa.csr

	Country Name : AU
	State or Province Name : Victoria
	Locality Name : Melbourne
	Organization Name : BAAU
	Organizational Unit Name : <empty>
	Common Name : AndrewPa
	Email Address : <empty>
	
	Challenge password : <empty>
	Optional company name : <empty>

## Step 3 - Sign the certificate
```bash
openssl x509 -req -in mosquitto_client_AndrewPa.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mosquitto_client_AndrewPa.crt -days 365 -sha256
```

## Step 4 - Deploy the certificate
Copied to directory on client side and pointed client (e.g. MQTTX app) to look at them.
