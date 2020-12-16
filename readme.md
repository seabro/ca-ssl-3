# How to create Selfsigned CA and custom Wildcars certificate

I had many trouble to create custom CA and try to sign 2nd wildcard certificate with it. So after some trial and error this is solution.

We will need:
 1. Some password for CA (e.g. MyCACrtPassword)
 2. Config file for CSR (e.g. somename.cnf)

First you have to create key for CA certificate. We are gonna name it **selfsignCA.key**. When key is generated you have to enter password and verify it.

```
openssl genrsa -des3 -out selfsignCA.key 4096
```

![image](https://github.com/seabro/ca-ssl-3/raw/main/img/capass.jpg)

Next step is to generate CA certificate itself with already created key. We are gonna name it **selfsignCA.crt**. When promts you have to ander key password, and after that you enter Certificate Data. You can use default values or your own, nevermind.

```
openssl req -new -x509 -days 3650 -key selfsignCA.key -out selfsignCA.crt
```

![image](https://github.com/seabro/ca-ssl-3/raw/main/img/cacrt.jpg)

### Now you are succesfuly created CA root certificate

Time to create wilcard custom domain certificate (**somedomain.crt**) and sign it with CA root.

```
openssl genrsa -out somedomain.key 2048
```
 
time to create config file (**openssl.cnf**) wich we are gonna use for CSR and signing. You can use template, or create your own. For **commonName** value I used wildcard domain value *.somedomain.tld

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

after that you have to use it to create CSR, and check that you have **x509v3** section with **Subject Alternative Names** that you entered in openssl.cnf file

```
openssl req -new -nodes -key somedomain.key -config openssl.cnf -out somedomain.csr
```
you can check it with:
```
openssl req -noout -text -in somedomain.csr
```

![image](https://github.com/seabro/ca-ssl-3/raw/main/img/x509v3.jpg)

If it is everting ok, you can sign it with early created CA. We will name final CRT file somedomain.crt. Main thing is that we use -extfile flag with out config file and -extensions flags with section from our config file with **subjectAltName** for Subject Alternative Names. To succesfully you will have to enter password from CA key created file

```
openssl x509 -req -in somedomain.csr -CA selfsignCA.crt -CAkey selfsignCA.key -CAcreateserial -out somedomain.crt -days 1024 -sha256 -extfile openssl.cnf -extensions req_ext
```

For SSL to funcion propery you need to import CA certificate in your Trusted Root Certificate Repository.

If you use Nginx you can bundle CA and custom CRT into one file for better use.

Good luck!
