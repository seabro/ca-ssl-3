# How to create Selfsigned CA and custom Wildcars certificate

I had so many trouble to create a custom CA and try to sign 2nd wildcard certificate with it. So... after some trial and error this is a solution.

We will need:
 1. Some password for CA (e.g. MyCACrtPassword)
 2. Config file for CSR (e.g. somename.cnf)

First, you have to create key for CA certificate. We are gonna name it **selfsignCA.key**. When key is generated you have to enter password and verify it.

```
openssl genrsa -des3 -out selfsignCA.key 4096
```

![image](https://github.com/seabro/ca-ssl-3/raw/main/img/capass.jpg)

Next step is to generate CA certificate itself with already created key. We are gonna name it **selfsignCA.crt**. When prompts, you have to enter key password, and after that you insert Certificate Data. You can use default values or your own, nevermind.

```
openssl req -new -x509 -days 3650 -key selfsignCA.key -out selfsignCA.crt
```

![image](https://github.com/seabro/ca-ssl-3/raw/main/img/cacrt.jpg)

### Now you are successfully created CA root certificate

Time to create wildcard custom domain certificate (**somedomain.crt**) and sign it with CA root.

```
openssl genrsa -out somedomain.key 2048
```
 
Now create config file (**openssl.cnf**) which we are gonna use for CSR and signing. You can use template, or create your own. For **commonName** value I used wildcard domain value *.somedomain.tld

```
[req]
default_md = sha256
prompt = no
req_extensions = req_ext
distinguished_name = req_distinguished_name

[req_distinguished_name]
commonName = *.yourdomain.com
countryName = US
stateOrProvinceName = No state
localityName = City
organizationName = LTD

[req_ext]
keyUsage=critical,digitalSignature,keyEncipherment
extendedKeyUsage=critical,serverAuth,clientAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=yourdomain.com
DNS.2=*.yourdomain.com
```

after that you have to use config file to create CSR.

```
openssl req -new -nodes -key somedomain.key -config openssl.cnf -out somedomain.csr
```
Check that you have **x509v3** section with **Subject Alternative Names** that you entered in openssl.cnf file. You can check it with:
```
openssl req -noout -text -in somedomain.csr
```

![image](https://github.com/seabro/ca-ssl-3/raw/main/img/x509v3.jpg)

Now, you can sign it with early created CA. We will name final CRT file **somedomain.crt**. Main thing is that we use `-extfile` flag with our config file and `-extensions` flag with section name from our config file with values of **subjectAltName** for Subject Alternative Names. To create successfully you will have to enter password from CA key file

```
openssl x509 -req -in somedomain.csr -CA selfsignCA.crt -CAkey selfsignCA.key -CAcreateserial -out somedomain.crt -days 1024 -sha256 -extfile openssl.cnf -extensions req_ext
```

For SSL to function properly you need to import CA certificate in your Trusted Root Certificate Repository.

If you use Nginx you can bundle CA and custom CRT into one file for better use.

Good luck!

References:
* [Certificate Tools for confgi files](https://certificatetools.com/)
* [Create Certificate Authority](https://www.golinuxcloud.com/create-certificate-authority-root-ca-linux/)
* [Setup SSL on NGINX](https://www.ssltrust.com.au/help/setup-guides/setup-ssl-nginx-configure-best-security)


