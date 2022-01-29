# openssl-practices

openssl practices
Root CA: often be offline
Sub CA: intermediary CA that takes signing requests from server

## preparation

create directory structure and index files for ca folder
```
mkdir -p {root-ca,sub-ca,server}/{certs,crl,csr,private}
mkdir -p {root-ca,sub-ca}/newcerts
touch {root-ca,sub-ca}/index
```

create serial files for identities
```
openssl rand -hex 16 > root-ca/serial
openssl rand -hex 16 > sub-ca/serial
```

## create self-signed server private and public key
```
cd server/private

# create server certificate
openssl req -new --newkey rsa:2048 -days 365 -nodes -x509 -subj '/CN=www.example.com' -keyout server.key -out server.crt

# check the cert
openssl x509 -text -in server.crt -noout
```

## private key
```
# create private key for root ca and sub ca, AES pass phrase needed
openssl genrsa -aes256 -out- root-ca/private/ca.key 4096
openssl genrsa -aes256 -out sub-ca/private/sub-ca.key 4096

# create private key for server
openssl genrsa -out server/private/server.key
```

## public key
create public key(cert) for root ca from configuration file
```
# create cert
openssl req -config root-ca.conf -key root-ca/private/ca.key -new -x509 -days 7500 -sha256 -extensions v3_ca -out root-ca/certs/ca.crt

# check the cert
openssl x509 -text -noout -in root-ca/certs/ca.crt
```

create public key(cert) for sub ca from configuration file
```
# create cert signing request for sub ca from configuration file
openssl req -config sub-ca/sub-ca.conf -new -key sub-ca/private/sub-ca.key -sha256 -out sub-ca/csr/sub-ca.csr

# create sub cert from signing request and sign with root-ca
openssl ca -config root-ca/root-ca.conf -extensions v3_intermediate_ca -days 3650 -notext -in sub-ca/csr/sub-ca.csr -out sub-ca/certs/sub-ca.crt

# check cert
openssl x509 -text -noout -in sub-ca/certs/sub-ca.crt
```

## server certificate
create and sign server certificate
```
# create signing request for server
openssl req -key server/private/server.key -new -sha256 -out server/csr/server.csr

# sign with sub-ca
openssl ca -config sub-ca/sub-ca.conf -extensions server_cert -days 365 -notext -in server/csr/server.csr -out server/certs/server.crt

# check cert
openssl x509 -text -noout -in server/certs/server.crt
```

create chained cert, required by e.g. Nginx
```
cat server/certs/server.crt sub-ca/certs/sub-ca.crt > server/certs/chained.crt
```

## test certificates
add dummy host to /etc/hosts
```
echo "127.0.0.2 www.example.com" >> /etc/hosts
```




