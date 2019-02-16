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
$ export MY_PUBLIC_FQDN_DNS_NAME="some.domain.com"
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
..................................................+++
.........................................................+++
writing new private key to 'env/lab/2019-02-16-144649-some.domain.com.key'
-----
$ openssl req -noout -text \
> -in env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.csr
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=US, ST=Tranquility, L=Peace, O=Justice For All, CN=some.domain.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:95:a8:88:a6:ef:97:fd:17:64:74:aa:6c:fe:8a:
                    92:5e:f9:df:5d:7b:0a:27:3e:13:97:93:28:92:8f:
                    11:05:b0:0b:78:a0:de:d0:53:bb:7f:40:da:d4:2c:
                    52:7a:71:c1:09:a8:9c:ad:fc:c1:95:b4:9c:d9:2f:
                    e6:61:5e:ff:4d:aa:99:5a:61:28:91:35:b1:7b:4d:
                    45:42:62:3f:71:5b:5d:89:4a:5d:2d:20:8d:07:c6:
                    66:70:69:41:0c:94:5c:54:ff:a5:00:19:94:b6:ab:
                    3d:e6:69:8d:09:25:7d:1e:4d:df:07:fd:ae:f3:9b:
                    ec:d3:6b:ef:de:71:bd:9e:9e:53:4a:21:f0:24:ef:
                    c9:b8:84:ca:db:1f:30:34:ec:a3:85:b8:1f:89:58:
                    ef:0b:23:91:79:1e:8b:ab:09:6b:b7:8e:53:4e:38:
                    3d:6f:cb:e8:f7:0e:c4:8e:7a:d4:ee:85:53:6b:e7:
                    e8:ed:cb:c0:18:0e:8b:5a:08:40:31:1c:24:bc:c7:
                    b1:40:62:b6:59:22:74:4f:cf:21:e5:1b:ec:bf:ef:
                    cf:5d:24:b1:f1:e6:66:c5:3d:f8:71:d2:a6:13:9a:
                    4c:95:8b:f5:83:a9:ee:47:d4:33:7c:72:67:08:5b:
                    44:0f:ff:92:98:6c:4e:68:85:37:27:1c:fc:09:5d:
                    89:59
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name:
                DNS:docker-registry.default.svc.cluster.local, DNS:docker-registry.default.svc
    Signature Algorithm: sha256WithRSAEncryption
         8b:d6:64:c2:ea:80:c6:eb:e0:7b:d7:23:12:05:75:b7:59:cd:
         96:7a:a4:12:73:c5:24:9a:a6:0e:f1:78:d7:ae:bb:c6:a6:b2:
         dc:64:da:c2:89:db:14:d4:41:e7:a0:bb:ba:73:f3:fd:05:99:
         ce:c7:98:5c:df:ea:bc:66:1f:bd:6c:65:d4:de:72:7d:2e:cd:
         60:4b:b9:48:31:58:27:d3:6a:09:de:d0:37:56:6c:54:b1:a5:
         cc:c9:0f:8b:cb:c9:23:cf:a6:b1:6f:a4:8c:5c:16:a0:29:c7:
         b6:e3:a0:1a:f8:6b:d0:09:c2:40:c9:86:06:50:a9:51:1b:33:
         63:7b:23:51:35:59:48:44:09:99:6d:d8:9e:bf:b4:30:58:43:
         ce:b5:74:f4:13:14:35:e2:f1:30:d1:b2:5a:72:cd:fd:d5:3d:
         67:bf:be:70:cb:39:6d:ce:ec:2e:f2:d0:74:a0:6c:49:39:f6:
         bb:21:22:2f:59:cd:65:0e:31:68:67:c5:13:77:2a:26:46:b0:
         d0:b4:ac:f5:7b:22:f2:d9:7b:e0:bd:66:eb:78:6c:31:0d:f6:
         53:9d:2d:73:65:4c:6c:37:55:52:b9:48:bb:fa:ea:f6:f8:ea:
         ef:ad:3f:55:1d:5d:b0:23:f3:68:fd:b4:a0:3c:68:41:46:06:
         96:f8:5f:af
```
