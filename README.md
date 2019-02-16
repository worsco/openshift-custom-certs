# Generating a Custom Cert for OpenShift registry

| Env | Description             |
| --- | ----------------------- |
| lab | Lab                     |
| sb  | Sandbox                 |
| np  | Non-Prod                |
| qa  | QA                      |
| uat | User Acceptance Testing |
| prf | Performance             |
| pd  | Production              |
|     |                         |

# Initialize environment
```
export myenv=lab
# export myenv=sb
# export myenv=np
# export myenv=qa
# export myenv=uat
# export myenv=prf
# export myenv=pd
```

# Initialize variables for environment
```
export MY_PUBLIC_FQDN_DNS_NAME="registry.intranet.domain.com"
export MY_COUNTRY_CODE="US"
export MY_STATE="Tranquility"
export MY_CITY_OR_LOCALITY="Peace"
export MY_ORGANISATION="Justice For All"
export MY_OPENSSL_CONFIG="${MY_PUBLIC_FQDN_DNS_NAME}-san.cnf"
```

# Create the OpenSSL config with SAN
```
cat > env/$myenv/${MY_OPENSSL_CONFIG} << END_OF_FILE
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[ dn ]
C=${MY_COUNTRY_CODE}
ST=${MY_STATE}
L=${MY_CITY_OR_LOCALITY}
O=${MY_ORGANISATION}
CN=${MY_PUBLIC_FQDN_DNS_NAME}

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = docker-registry.default.svc.cluster.local
DNS.2 = docker-registry.default.svc
END_OF_FILE
```

# To generate the csr and key
```
MYDATE=`date +%Y-%m-%d-%H%M%S` && \
openssl req -new -newkey rsa:2048 -nodes \
-out env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.csr \
-keyout env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.key \
-config env/$myenv/${MY_OPENSSL_CONFIG}
```

# To view the CSR from the command line
```
openssl req -noout -text \
-in env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.csr
```

# Convert a p7b into the required pem format
```
openssl pkcs7 -print_certs -in ca.p7b -out ca.pem
```
# View the resulting ca.pem in plaintext
```
openssl x509 -in ca.pem -noout -text
```

# Next steps

Use your CSR to generate the certificate from your certificate provider.
Ensure they do not over-write your SAN or they duplicate your SAN with
whatever provisioning steps they use.

# Example run:
```
$ export myenv=lab
$ export MY_PUBLIC_FQDN_DNS_NAME="registry.intranet.domain.com"
$ export MY_COUNTRY_CODE="US"
$ export MY_STATE="Tranquility"
$ export MY_CITY_OR_LOCALITY="Peace"
$ export MY_ORGANISATION="Justice For All"
$ export MY_OPENSSL_CONFIG="${MY_PUBLIC_FQDN_DNS_NAME}-san.cnf"
$ cat > env/$myenv/${MY_OPENSSL_CONFIG} << END_OF_FILE
> [ req ]
> default_bits       = 2048
> prompt             = no
> default_md         = sha256
> req_extensions     = req_ext
> distinguished_name = dn
> 
> [ dn ]
> C=${MY_COUNTRY_CODE}
> ST=${MY_STATE}
> L=${MY_CITY_OR_LOCALITY}
> O=${MY_ORGANISATION}
> CN=${MY_PUBLIC_FQDN_DNS_NAME}
> 
> [ req_ext ]
> subjectAltName = @alt_names
> 
> [ alt_names ]
> DNS.1 = docker-registry.default.svc.cluster.local
> DNS.2 = docker-registry.default.svc
> END_OF_FILE
$ MYDATE=`date +%Y-%m-%d-%H%M%S` && \
> openssl req -new -newkey rsa:2048 -nodes \
> -out env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.csr \
> -keyout env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.key \
> -config env/$myenv/${MY_OPENSSL_CONFIG}
Generating a 2048 bit RSA private key
.....+++
.....................................................+++
writing new private key to 'env/lab/2019-02-16-145400-registry.intranet.domain.com.key'
-----
$ openssl req -noout -text \
> -in env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.csr
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=US, ST=Tranquility, L=Peace, O=Justice For All, CN=registry.intranet.domain.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c0:ab:fb:c2:82:ba:bf:cf:e1:c5:18:cc:95:8a:
                    35:7f:57:16:c6:4f:41:69:8e:36:00:9b:84:9b:e9:
                    53:30:32:33:d0:2b:42:33:13:6d:c7:50:58:bf:be:
                    9e:a5:00:ec:45:d5:a3:46:d8:a6:6a:56:d1:44:b9:
                    4b:ae:f8:bc:ef:81:e3:18:33:9b:16:1d:af:1c:61:
                    8c:2c:a2:b3:0a:7b:7d:ce:d7:3d:2f:3f:92:e2:c6:
                    7c:8c:57:15:18:61:c2:4d:3d:bb:13:be:17:2b:c5:
                    1e:40:f9:09:5a:e7:76:48:aa:2e:37:e7:34:62:56:
                    0d:a8:56:e5:e8:38:b2:ad:62:1e:0a:aa:cd:ce:a3:
                    f8:32:62:05:ea:1b:ce:80:7c:d6:ab:8b:43:71:fa:
                    33:f8:ca:65:bf:4c:d4:0f:f7:aa:ba:aa:b6:ad:51:
                    0c:77:55:a6:c7:80:f5:fc:7d:5c:bf:1d:b6:6c:0d:
                    2c:02:0b:d6:b6:1c:25:ef:fc:c3:cd:d9:8a:96:7e:
                    32:bc:7d:a8:40:fd:c1:99:d3:38:86:4f:18:f4:41:
                    ed:54:3c:6c:48:4e:87:af:ec:2d:c6:2a:9d:b1:e0:
                    aa:20:d5:d7:c1:b4:e8:59:ce:ca:64:17:4d:ea:bc:
                    23:b7:7e:cb:df:1d:11:e0:ce:c1:f7:00:d8:df:2d:
                    75:c9
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name: 
                DNS:docker-registry.default.svc.cluster.local, DNS:docker-registry.default.svc
    Signature Algorithm: sha256WithRSAEncryption
         a0:60:0c:67:dc:eb:48:a4:02:5d:ad:25:1f:27:67:9b:94:4c:
         2d:40:38:6d:58:b1:ad:d1:f8:4c:9f:83:f5:0a:1e:e6:5e:ac:
         83:e0:0d:76:9d:34:53:d5:9b:66:8b:a3:42:1c:dc:01:da:c5:
         93:9d:b9:96:af:61:4d:30:80:7d:49:bd:e3:b5:22:20:b1:f8:
         e0:40:5d:09:31:91:e1:82:67:9d:53:53:fb:eb:f4:8f:fa:1a:
         4d:3a:c3:94:0d:16:41:08:84:c9:1b:a6:90:89:43:18:1d:7d:
         58:3e:91:6d:c6:61:37:94:d9:35:0c:44:1b:97:60:64:ea:ec:
         83:0a:9e:e4:af:7a:0d:09:a9:a1:e2:2d:08:6d:e1:47:0b:43:
         47:30:71:7d:8f:15:bb:40:14:68:3f:9c:2d:cc:65:7b:db:75:
         f9:8c:4f:06:f4:04:14:6e:2e:48:42:06:3c:5c:af:a4:32:57:
         75:b9:7c:18:a9:0f:4c:27:df:69:17:b2:11:f0:bc:f3:0c:2f:
         df:69:9f:89:5e:eb:61:12:67:42:08:f1:8e:72:df:6e:8a:95:
         11:0a:70:88:25:ff:f7:32:77:48:a0:7e:ce:55:a0:3f:f1:3c:
         05:2c:40:5f:aa:00:a3:0f:7e:16:f6:28:0b:f7:1d:5c:d1:5a:
         3c:5c:1b:7c
```
