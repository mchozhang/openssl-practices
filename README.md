# openssl-practices

openssl practices
Root CA: often be offline

## preparation

create directory structure and index files for ca folder
```
mkdir -p {root-ca,sub-ca,server}/{certs,crl,csr,private}
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
openssl genrsa -aes256 -out sub-ca/private/ca.key 4096

# create private key for server
openssl genrsa -out server/private/sub-ca.key
```

## public key
create public configuration
```
# create cert
openssl req -config root-ca.conf -key private/ca.key -new -x509 -days 7500 -sha256 -extensions v3_ca -out certs/ca.crt

# check the cert
openssl x509 -text -in certs/ca.crt
```


