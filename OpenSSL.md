[toc]

#### # Ways to present a certificate

- PEM - Privacy Enhanced Mail is a base-64 encoded format.
  - Can include public certs, public keys, entire chain, and private keys
  - May contain a CSR in PKCS10 format
  - Can be put inside email body
  - Has a `BEGIN CERTIFICATE` and `END CERTIFICATE`.
- DER is the parent of PEM and in a binary format.
- PKCS7 is used by Java and doesn't contain private key.
- PKCS12 is used by windows and can contain private key.

#### # Types and Transformations

- x509 v3 is RFC 5280
- File Extensions:
  - DER is binary and contains raw format using asn.1 encoding
  - PEM is ascii and contains base64 encoding using begin / end
  - CRT is extension for certificates and MS format. This may be encoded as DER or PEM.
  - CER is synonym for CRT
  - KEY is used for both public and private PKCS#8 keys and may be encoded as DER or PEM.
  - PKCS#7 is a complex format design for transport of signed or encrypted data
- Defined RFC 2315. Usually .p7b and .p7c extension. Supported by java keytool.
  - contains --BEGIN PKCS line
  - contains only certs, chain certs
  - doesn't contain private keys
  - base64 encoded ascii file
- PKCS#12 - binary format to store and protect server key along with entire chain
  - .pfx, .p12
  - stores server certs, any intermediate certs, private key in one encrypted file
  - used to import into browser
  - used to convert into jks using keytool
- PKCS#8 is a newer and recommended format for RSA keys

### OpenSSL

#### # Install OpenSSL

```bash
# install
brew update
brew install openssl

brew cask install keystore-explorer

# upgrade
mv /usr/bin/openssl /usr/bin/openssl_old
brew unlink openssl
brew link -force openssl
ln -s /usr/local/Cellar/../openssl /usr/bin/openssl

# validate
openssl version -a
which openssl

# building
wget http://www.openssl.org.source/openssl-.tar.gz ./config --prefix=/opt/openssl --openssldir=/opt/openssl enable-ec nistp_64_gcc_128
make depend
make
sudo make install

# config
openssl help
cd /usr/lib/../openssl.conf
// look inside misc

# list of supported ciphers
openssl ciphers -v 'ALL:COMPLEMENTOFALL'
```

#### # Test SSL Certificate of a URL

```bash
openssl s_client -connect subject.com:443 -showcerts

# check cert expiry date of ssl url
openssl s_client -connect subject.com:443 2>/dev/null | openssl x509 -noout -enddate

# check if ssl or tls is accepted on a url
openssl s_client -connect secureurl.com:443 -ssl2
openssl s_client -connect secureurl.com:443 –ssl3
openssl s_client -connect secureurl.com:443 –tls1
openssl s_client -connect secureurl.com:443 –tls1_1
openssl s_client -connect secureurl.com:443 –tls1_2

# verify if a particular cipher is accepted on url
openssl s_client -cipher 'ECDHE-ECDSA-AES256-SHA' -connect secureurl:443

# verify if the cert and the key belong to each other.
# checksum should match
openssl x509 -noout -modulus -in server.crt | openssl md5
openssl rsa -noout -modulus -in server.key | openssl md5
```

#### # Setup Root CA

- [JamieLinux Guide](https://jamielinux.com/docs/openssl-certificate-authority/introduction.html)
- [DigitalOcean Guide](https://www.digitalocean.com/community/tutorials/java-keytool-essentials-working-with-java-keystores)

```bash
mkdir root/ca
cd root/ca

mkdir csr certs crl newcerts private public
chmod 700 private

touch index.txt
echo 1000 > serial

// Configure file 'root/ca/openssl.cnf'
[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /root/ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/ca.key.pem
certificate       = $dir/certs/ca.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`), are applied when creating certs or csr.
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = GB
stateOrProvinceName_default     = England
localityName_default            =
0.organizationName_default      = Alice Ltd
#organizationalUnitName_default =
#emailAddress_default           =

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
// end of config file

```

#### # Generate Root CA Private Key

```bash
openssl genrsa -aes256 -out private/ca.key.pem -passout pass:changeit 4096

# read-only for owner
chmod 400 private/ca.key.pem

# To verify enter the pass phrase
openssl rsa -in private/ca.key.pem -check
```

#### # Generate Root CA Certificate

```bash
# Specify a conf file to use with the -config option
# otherwise openssl defaults to /etc/pki/tls/openssl.cnf

# generate csr and sign it
openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem -passin pass:changeit

# read access to all
chmod 444 certs/ca.cert.pem

# alternative inline option
-subj '/C=UA/O=myCompany/CN=myCompanyRootCA'

# verify the cert
openssl x509 -noout -text -in certs/ca.cert.pem

# verify the cert dates
openssl x509 -noout -dates -in certs/ca.cert.pem

# verify the cert signing authority
openssl x509 -noout -issuer -issuer_hash -in certs/ca.key.pem

# verify the cert's hash
openssl x509 -noout -text -in certs/ca.key.pem | grep 'Signature Algorithm'

# verify signature
openssl x509 -noout -in certs/ca.cert.pem -sha256 -fingerprint

# verify the checksum of private key and cert match, vouching the cert is signed by private key
openssl x509 -noout -modulus -in certs/ca.cert.pem | openssl md5
openssl rsa -noout -modulus -in private/ca.key.pem -passin pass:changeit | openssl md5
```

#### # Generate Root CSR and sign it to generate Root CA Certificate

```bash
# generate csr
openssl req -config openssl.cnf \
  -key private/ca.key.pem -new -days 7300 -sha256 \
  -out csr/ca.csr.pem -passin pass:changeit

# sign csr and generate certificate
openssl req -config openssl.cnf \
  -in csr/ca.csr.pem -key private/ca.key.pem -x509 -sha256 \
  -extensions v3_ca -out certs/ca.cert2.pem -passin pass:changeit

# every one has read access
chmod 444 certs/ca.cert2.pem
```

#### # Extract Root CA Public Key

```bash
# extract rootCA public key using private key
openssl rsa -in private/ca.key.pem -pubout -out public/ca.public.key.pem

# extract rootCA public key using certificate
openssl x509 -pubkey -noout -in certs/ca.cert.pem > public/ca.public.key.pem
```

#### # Generate self-signed Certificate

```bash
# generate self-signed certificate and key file with 4096 bit rsa
# this will generate cert valid for one year
openssl req -x509 -sha256 -nodes -newkey rsa:4096 -keyout selfsigned.key -out selfsignedcert.pem

# self-signed valid for 2 years
openssl req -x509 -sha256 -nodes -days 730 -newkey rsa:4096 -keyout selfsigned.key -out selfsignedcert.pem

# how do you know if a certificate is self-signed?
# The issuer and subject attributes will be identical
```

#### # Intermediate CA

```bash
# intermediate
mkdir ca/intermediate
cd intermediate
mkdir certs crtl csr newcerts private public
chmod 700 private
touch index.txt
echo 1000 > serial
echo 1000 > crlnumber

// Copy to `ca/intermediate/openssl.cnf`**.

[ ca ]
# `man ca`
default_ca = CA_default

[ CA_default ]
# Directory and file locations.
dir               = /Users/manish/Documents/openssl/ca/intermediate
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# The root key and root certificate.
private_key       = $dir/private/intermediate.key.pem
certificate       = $dir/certs/intermediate.cert.pem

# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-2.
default_md        = sha256

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_loose

[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only

# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256

# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca

[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = US
stateOrProvinceName             = Arizona
localityName                    = Chandler
0.organizationName              = Deliwala Ltd
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

# Optionally, specify some defaults.
countryName_default             = US
stateOrProvinceName_default     = Arizona
localityName_default            = Chandler
0.organizationName_default      = Deliwala Ltd
organizationalUnitName_default  =
emailAddress_default            =

[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection

[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

#### # Generate Intermediate CA

An intermediate ca signs certs on behalf of the root ca. The root ca signs the intermediate cert, forming a chain of trust.

```bash
# create the intermediate key
openssl genrsa -aes256 \
  -out intermediate/private/intermediate.key.pem 4096
chmod 400 intermediate/private/intermediate.key.pem

# create the intermediate certificate
openssl req -config intermediate/openssl.cnf \
  -new -sha256 \
  -key intermediate/private/intermediate.key.pem \
  -out intermediate/csr/intermediate.csr.pem

# sign the intermediate certificate, use the root ca conf 
# with appropriate extensions
# make sure you say 'y' to sign the certificate.
openssl ca -config openssl.cnf \
  -extensions v3_intermediate_ca -days 3650 -notext -md sha256 \
    -in intermediate/csr/intermediate.csr.pem \
    -out intermediate/certs/intermediate.cert.pem

chmod 444 intermediate/certs/intermediate.cert.pem

# verify the intermediate certificate
openssl x509 -noout -text -in intermediate/certs/intermediate.cert.pem

# verify the intermediate against the root
openssl verify -CAfile certs/ca.cert.pem intermediate/certs/intermediate.cert.pem

# create the certificate chain file
cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
chmod 444 intermediate/certs/ca-chain.cert.pem
```

#### # Other Commands

```bash
# view a certificate
openssl x509 -in some.crt -noout -text

# check hash value of a cert
openssl x509 –inform der –in sslcert.der –out sslcert.pem

# remove passphrase from key
openssl rsa -in certkey.key -out nopassphrase.key

# generate csr and 2048-bit private rsa key
openssl req -out some.csr -newkey rsa:2048 -nodes -keyout private.key

# create intermediate CA private key
openssl genrsa -out intermediateCA.key -aes256 -passout pass:changeit 4096

# create intermediate CA certificate request
openssl req -new -key intermediateCA.key -out intermediateCA.csr -subj '/C=UA/O=MyCompany/CN=MyCompanyIntermediateCA' -passin pass:changeit

# create v3 x509 extensions
cat <<EOF >v3_ca.ext
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
basicConstraints=CA:true //this means that certificate can sign other certificates
EOF

# review text version of csr
openssl x509 -intermediateCA.csr -noout -text

# sign intermediate CA csr with root CA private key and add x509 v3 extensions
openssl x509 -req -in intermediateCA.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -CAserial rootCA.srl -extfile v3_ca.ext -out intermediateCA.crt -days 365 -sha256 -passin pass:changeit

# verify intermediate CA certificate
openssl verify -CAfile rootCA.crt intermediateCA.crt

# if required, generate csr from a certificate
openssl x509 -x509toreq -in intermediateCA.crt -out intermediateCA-2.csr -signkey rootCA.key -passin pass:changeit

# verify if a certificate was issued by a specific CA.
openssl verify -verbose -CAfile cacert.pem  server.crt
server.crt: OK

# generate server private key
openssl genrsa -out server.key -aes256 -passout pass:changeit 4096

# generate csr
openssl req -new -key server.key -out server.csr -subj '/C=UA/O=MyCompany/CN=example.com' -passin pass:changeit

# sign csr using intermediate CA private key
openssl x509 -req -in server.csr -CA intermediateCA.crt -CAkey intermediateCA.key -CAcreateserial -CAserial intermediateCA.srl -out server.crt -days 365 -sha256 -passin pass:changeit

# decrypt server private key to allow webserver 
# to use it without prompting for password
mv server.key server.key.secure
openssl rsa -in server.key.secure -out server.key -passin pass:changeit

```

#### # Sign Server Certificate using Intermediate CA

```bash
# create a private key
openssl genrsa -aes256 -out intermediate/private/www.myhost.com.key.pem 2048
chmod 400 intermediate/private/www.myhost.com.key.pem

# create a CSR
openssl req -config intermediate/openssl.cnf -key intermediate/private/www.myhost.com.key.pem -new -sha256 -out intermediate/csr/www.myhost.com.csr.pem

# sign the CSR
openssl ca -config intermediate/openssl.cnf -extensions server_cert -day 375 -notext -md sha256 -in intermediate/csr/www.myhost.com.csr.pem -out intermediate/certs/www.myhost.com.cert.pem
chmod 444 intermediate/certs/www.myhost.com.cert.pem

# verify
openssl x509 -noout -text -in intermediate/certs/www.myhost.com.cert.pem
  
# verify against CA cert chain
openssl verify -CAfile intermediate/certs/ca-chain.cert.pem intermediate/certs/www.myhost.com.cert.pem

# text version of certificate
openssl x509 -in rootCA.crt -text

# view pem certificates
openssl x509 -in cert.pem -text -noout
openssl x509 -in cert.cer -text -noout
openssl x509 -in cert.crt -text -noout
# you may see error if you are using 
# pem command to view the certificate
unable to load certificate
12626:error:0906D06C:PEM routines:PEM_read_bio:no start line:pem_lib.c:647:Expecting: TRUSTED CERTIFICATE

# view der certificates
openssl x509 -in certificate.der -inform der -text -noout

# convert der (.crt, .cer, .der) to pem
openssl x509 -inform der -in cert.cer -out cert.pem

# pem to der
openssl x509 -in cert.crt -outform der -out cert.der
openssl x509 -in cert.pem -outform der -out cert.der

# pem to p7b
openssl crl2pkcs7 -nocrl -certfile certificate.cer -out certificate.p7b -certfile CAcert.cer

# pem to pfx
openssl pkcs12 -export -out certificate.pfx -inkey privateKey.key -in certificate.crt -certfile CAcert.crt

# pem to crt
openssl x509 -outform der -in cert.pem -out cert.crt

# der to pem
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem

# p7b to pem
openssl pkcs7 -print_certs -in certificate.p7b -out certificate.cer

# p7b to pfx
openssl pkcs7 -print_certs -in certificate.p7b -out certificate.cer
openssl pkcs12 -export -in certificate.cer -inkey privateKey.key-out certificate.pfx -certfile CAcert.cer

# pfx to pem
openssl pkcs12 -in certificate.pfx -out certificate.cer -nodes
# command will put all certs and private key into a single file
# you will need to copy each certificate and private key including the
# begin/end statements to its individual text files and save them as .cer

# convert cert and private key to pkcs#12 format
openssl pkcs12 –export –out sslcert.pfx –inkey key.pem –in sslcert.pem

# convert pkcs12 to pem
openssl pkcs12 –in cert.p12 –out cert.pem
openssl pkcs12 -in keystore.pfx -out keystore.pem -nodes

# convert pkcs12 to pem - private key only
add -nocerts as option to above

# convert pkcs12 to pem - cert only
add -nokeys as option to above

# check contents of pkcs12 format cert
openssl pkcs12 –info –nodes –in cert.p12

# der to text
openssl x509 -in cert.der -inform der -text -noout

# pem to text
openssl x509 -in cert.pem -inform dem -text -noout

# pem to pkcs12 (.pfx, .p12)
# convert pem cert and private key to pkcs12 format
openssl pkcs12 -export -out cert.pfx -inkey privatekey.key -in cert.crt -certfile CACert.crt

# combination - combine multiple x509 assets into a single file
```

#### # Client Certificate

```bash
# generate client private key
openssl genrsa -out client.key -aes256 -passout pass:changeit 4096

# generate client certificate request
openssl req -new -key client.key -out client.csr -subj '/C=UA/O=My Company/CN=My Name/emailAddress=my_name@example.com' -passin pass:changeit

# sign client certificate request with intermediate CA private key
openssl x509 -req -in client.csr -CA intermediateCA.crt -CAkey intermediateCA.key -CAcreateserial -CAserial intermediateCA.srl -out client.crt -days 365 -sha256 -passin pass:changeit

# create CA certificate chain file
cat intermediateCA.crt rootCA.crt > CAchain.pem
export client certificate to PKCS#12 file to be able to import it into the browser
openssl pkcs12 -export -passout pass:changeit -in client.crt -inkey client.key -certfile CAchain.pem -out client.p12 -passin pass:changeit

# verify server and client certs against 
# the cert chain and print their contents
openssl verify -CAfile CAchain.pem server.crt
openssl x509 -in server.crt -text
openssl verify -CAfile CAchain.pen client.crt
openssl x509 -in client.crt -test
```

#### # Digest

```bash
# sha1 for a file
openssl dgst -sha1 file.txt
openssl dgst -out digest.txt file.txt

# sha256 for a file
openssl dgst -sha256 file.txt
openssl dgst -sha256 file.txt -out digest.txt

# dgst options
# run with unknown command to get more details
openssl dgst -help

# -c for colons
# -hex for hexdump
# -binary
# -sign: sign the digest with a private key
# -verify: verify a signature using private key

# sign the digest of file.txt using DSA or RSA private key
openssl dgst -dss1 -sign dsakey.pem -out dsasign.bin file.txt
openssl -sha1 -sign rsaprivate.pem out rsasign.bin file.txt

# verify signature
openssl dgst -dss1 -prverify dsakey.pem -signature dsasign.bin file.txt
openssl -sha1 -verify rsapublic.pem -signature rsasign.bin file.txt
```

#### Encrypt in DES3/CBC mode

```bash
# encrypt in des3/cbc mode
openssl enc -des3 -salt -in plaintext.doc -out ciphertext.bin
openssl enc -des3-ede-ofb -d -in ciphertext.bin -out plaintext.doc -passin pass:changeit
```

#### # DSA Key Generation

```bash
openssl ecparam -genkey -name secp256r1 | openssl ec -out ec.key -aes128
```

#### # Self-signed Certificates, Generating Keystore and Truststore

```bash
# use openssl to generate  private key and certificate
# import into keystore and truststore

# generate server private key and verify it
openssl genrsa -aes256 -out private/server.key.pem -passout pass:s123 4096
chmod 400 private/server.key.pem
openssl rsa -in private/server.key.pem –check

# generate csr and sign it
openssl req -key server.key.pem -new -x509 -days 30 -sha256 -out server.cert.pem -passin pass:s123

# verify the certificate
openssl x509 -noout -text -in server.cert.pem

# verify the dates in the certificate
openssl x509 -noout -dates -in server.cert.pem

# verify the cert signing authority
openssl x509 -noout -issuer -issuer_hast -in server.cert.pem

# verify the cert's hash
openssl x509 -noout -text -in server.cert.pem | grep 'Signature Algorithm'

# convert server cert from pem to pkcs12
openssl pkcs12 -export -name server-cert -in server.cert.pem -inkey server.key.pem -out server.keystore.p12

# convert pkcs12 keystore into a jks keystore
keytool -importkeystore -destkeystore server.keystore.jks -srckeystore server.keystore.p12 -srcstoretype pkcs12 -alias server-cert

# import server's certificate into server truststore
keytool -import -alias server-cert -file server.cert.pem -keystore server.truststore.jks

# import client's certificate into server's truststore
keytool -import -alias client-cert -file client.cert.pem -keystore server.truststore.jks

# check which certificates are in the keystore
keytool -list -v -keystore server.keystore.jks -storepass sss123 //this shows fingerprint

# check which certificates are in the truststore
keytool -list -v -keystore server.truststore.jks -storepass sss123

# generate client side
openssl genrsa -aes256 -out private/client.key.pem -passout pass:c123 4096
openssl req -key client.key.pem -new -x509 -days 30 -sha256 -out client.cert.pem -passin pass:c123
openssl pkcs12 -export -name client-.cert -in client.cert.pem -inkey client.key.pem -out client.keystore.p12
openssl pkcs12 -export -name server-cert -in client.cert.pem -inkey client.key.pem -out client.keystore.p12
keytool -importkeystore -destkeystore client.keystore.jks -srckeystore client.keystore.p12 -srcstoretype pkcs12 -alias client-cert
keytool -import -alias client-cert -file client.cert.pem -keystore client.truststore.jks

# import server cert into client truststore
keytool -import -alias server-cert -file server.cert.pem -keystore client.truststore.jks
keytool -list -v -keystore client.keystore.jks -storepass ccc123
keytool -list -v -keystore client.truststore.jks -storepass ccc123
```

###  keytool

[Guide](http://javarevisited.blogspot.com/2012/09/difference-between-truststore-vs-keyStore-Java-SSL.html)

> keystore stores private keys and certs corresponding to public keys and truststore store third-party certificates

```bash
rm \*.jks 2\> /dev/null
rm \*.pem 2\> /dev/null


# Generate private key for RootCA
keytool -genkeypair -alias rootCA -dname cn=MyCompanyRootCA -validity 10000 -keyalg RSA -keysize 4096 -ext bc:c -keystore rootCA.jks -keypass changeit -storepass changeit

# Generate private key for IntermediateCA
keytool -genkeypair -alias intermediateCA -dname cn=MyCompanyIntermediateCA -validity 10000 -keyalg RSA -keysize 4096 -ext bc:c -keystore intermediateCA.jks -keypass changeit -storepass changeit

# Generate root certificate
keytool -exportcert -rfc -keystore rootCA.jks -alias rootCA -storepass changeit \> rootCA.pem

# Generate a certificate for intermediateCA and signed by rootCA, use pipeline between two commands
keytool -keystore intermediateCA.jks -storepass changeit -certreq -alias intermediateCA | keytool -keystore rootCA.jks -storepass changeit -gencert -alias rootCA -ext bc=0 -ext san=dns:intermediateCA -rfc \> intermediateCA.pem

# Import intermediateCA cert chain into intermediateCA.jks
keytool -keystore intermediateCA.jks -storepass changeit -importcert -trustcacerts -noprompt -alias rootCA -file rootCA.pem

keytool -keystore intermediateCA.jks -storepass changeit -importcert -alias intermediateCA -file intermediateCA.pem

# Generate private keys for server
keytool -genkeypair -alias server -dname cn=server -validity 10000 -keyalg RSA -keysize 4096 -keystore my-keystore.jks -keypass changeit -storepass changeit

# Generate a certificate for server signed by intermediateCA 
keytool -keystore my-keystore.jks -storepass changeit -certreq -alias server | keytool -keystore intermediateCA.jks -storepass changeit -gencert -alias intermediateCA -ext ku:c=dig,keyEnc -ext san=dns:localhost -ext eku=sa,ca -rfc \> server.pem

# Import server cert chain into my-keystore.jks
keytool -keystore my-keystore.jks -storepass changeit -importcert -trustcacerts -noprompt -alias rootCA -file rootCA.pem

keytool -keystore my-keystore.jks -storepass changeit -importcert -alias intermediateCA -file intermediateCA.pem

keytool -keystore my-keystore.jks -storepass changeit -importcert -alias server -file server.pem

# Generate truststore.jks
keytool -keystore my-truststore.jks -storepass changeit -importcert -trustcacerts -noprompt -alias rootCA -file rootCA.pem

keytool -keystore my-truststore.jks -storepass changeit -importcert -alias intermediateCA -file intermediateCA.pem

keytool -keystore my-truststore.jks -storepass changeit -importcert -alias server -file server.pem
```

### ssh-gen

[Guide](https://www.ssh.com/ssh/keygen)

[Guide](https://www.freecodecamp.org/news/the-ultimate-guide-to-ssh-setting-up-ssh-keys/)

```bash
# Creating an SSH Key Pair for User Authentication 
ssh-keygen

# With different ciphers
ssh-keygen -t rsa -b 4096
ssh-keygen -t dsa
ssh-keygen -t ecdsa -b 521
ssh-keygen -t ed25519

ssh-keygen -f ~/tatu-key-ecdsa -t ecdsa -b 521

# Copy public key to server
ssh-copy-id -i ~/.ssh/tatu-key-ecdsa user@host

# Copy public key to clipboard
pbcopy < ~/.ssh/id_rsa.pub
```

