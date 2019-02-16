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
export MY_PUBLIC_FQDN_DNS_NAME="some.domain.com"
export MY_COUNTRY_CODE=US
export MY_STATE=Tranquility
export MY_CITY_OR_LOCALITY=Peace
export MY_ORGANISATION=Justice For All
export MY_OPENSSL_CONFIG=${MY_PUBLIC_FQDN_DNS_NAME}-san.cnf
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
$ export MY_COUNTRY_CODE=US
$ export MY_STATE=Tranquility
$ export MY_CITY_OR_LOCALITY=Peace
$ export MY_ORGANISATION=Justice For All
$ export MY_OPENSSL_CONFIG=${MY_PUBLIC_FQDN_DNS_NAME}-san.cnf
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
.........+++
...........................................................+++
writing new private key to 'env/lab/2019-02-16-142123-some.domain.com.key'
-----
$ openssl req -noout -text \
> -in env/$myenv/$MYDATE-${MY_PUBLIC_FQDN_DNS_NAME}.csr
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=US, ST=Tranquility, L=Peace, O=Justice, CN=some.domain.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:a0:b4:52:67:aa:65:12:7c:c4:20:a4:98:47:9e:
                    23:e3:20:ba:57:08:36:88:98:60:7c:0a:e6:14:e5:
                    cf:08:6c:1e:50:65:68:7d:72:93:47:e2:6f:17:7d:
                    c8:d3:25:a5:f0:84:60:cd:c4:cc:c0:5c:b2:83:6a:
                    a1:05:6b:ff:e0:52:d8:1c:e5:36:a3:00:63:77:02:
                    06:c4:19:9b:40:aa:ea:a9:d5:12:53:27:7a:d4:be:
                    8b:87:df:a4:9f:60:7e:78:4a:d6:a5:a0:86:04:c2:
                    42:5a:fa:f4:e2:96:70:f0:fb:e9:6b:42:14:27:c0:
                    02:3a:80:a3:69:21:1d:48:ca:2d:06:13:cb:a1:73:
                    c2:df:fa:2c:1f:dd:49:f4:a0:bf:4d:24:ab:72:d6:
                    c8:0e:14:a5:ee:68:38:4e:d4:cd:f6:62:dc:46:d6:
                    08:bc:a4:7f:68:75:85:fa:f4:a6:49:e9:d8:e7:c7:
                    3f:85:36:1a:46:4a:a1:95:8c:43:cf:eb:75:9a:41:
                    e0:b7:b2:d1:3f:97:77:12:65:ad:b5:46:bf:94:35:
                    5a:f3:20:06:13:3e:c4:a5:17:e7:52:1a:21:d0:46:
                    3d:58:5d:69:cc:91:21:23:61:2e:cb:9d:18:81:f0:
                    f9:49:94:95:e2:76:47:7c:8f:1d:5f:d6:b9:fa:09:
                    53:cd
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name:
                DNS:docker-registry.default.svc.cluster.local, DNS:docker-registry.default.svc
    Signature Algorithm: sha256WithRSAEncryption
         07:17:97:de:04:6b:f9:6b:33:14:35:c2:65:7f:f6:27:80:ee:
         68:87:05:54:8b:d8:bf:13:4a:5f:97:39:1f:e4:7c:ce:92:66:
         a4:3e:1a:83:c3:96:c3:53:41:1c:99:8c:a0:7d:07:ae:c3:21:
         98:d3:c4:23:93:60:7d:b6:5f:c5:33:5e:0b:9c:73:61:aa:cb:
         bb:ac:0c:ae:74:95:40:51:78:94:3f:c3:bc:d0:20:f6:bc:09:
         7d:70:c6:1a:12:95:44:6b:ed:3b:e7:f8:16:3d:4f:35:f4:af:
         a5:cc:4d:9f:d5:ab:96:97:9d:e3:82:f4:e3:46:aa:63:dd:25:
         42:f8:96:6d:dc:d7:29:4d:59:6d:c0:74:ee:ca:51:f5:13:ec:
         04:17:b5:65:8c:47:a4:f9:f4:60:70:40:f7:d3:62:06:24:7f:
         04:2b:fa:78:e9:f4:48:74:5e:b9:25:3b:ca:68:36:5e:09:d8:
         5f:21:cc:56:f7:52:48:81:33:79:ba:ce:57:d6:60:05:54:88:
         9f:e5:11:df:9e:3b:a9:2a:74:40:a0:25:96:67:0e:80:f6:be:
         0b:7b:bb:69:73:59:e3:7e:04:63:d3:b2:8d:fe:14:9e:51:e6:
         6d:0a:2e:08:b0:2e:c8:0e:3e:90:04:cb:2d:65:9f:d5:55:f1:
         36:eb:fc:0e
```
