# Membuat self sign certificate dan  menjadi Certificate Authority CA lokal untuk lokal development

## reference

- <https://www.ssl.com/faqs/what-is-a-certificate-authority/>
- <https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/>
- <https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html>
- <https://www.golinuxcloud.com/add-x509-extensions-to-certificate-openssl/>

## prerequisite

- sudah terinstall openssl
- [cara install OpenSSL di Windows](/)

## menjadi CA (lokal)

- untuk menjadi CA lokal caranya cukup mudah,
- generate key dan generate certificate
- kemudian sign request menggunakan cert dan key CA

## generate private key untuk CA

``` shell
# openssl genrsa -des3 -out ngcpidCA.key 2048
openssl genpkey -algorithm RSA -out ca/private/ngcpidCA.key.pem -aes256
```

passphrase : 12345678 # ganti dengan yang lebih aman

## generate a CSR

```sh
mkdir ca/csr
openssl req -new -key ca/private/ngcpidCA.key.pem -out ca/csr/ngcpidCA.csr.pem -config ca/config/ngcpidCA.cnf
```

## generate root certificate

``` shell
openssl req -x509 -new -nodes -key ca/private/ngcpidCA.key.pem -days 3650 -extensions v3_ca -out ca/certs/ngcpidCA.cert.pem -config ca/config/ngcpidCA.cnf

openssl x509 -in ca/certs/ngcpidCA.cert.pem -noout -text
```

untuk memudahkan kita akan membuat file config

<details>
  <summary>ngcpidCA.cnf</summary>

    ```cnf
    [ ca ]
    # `man ca`
    default_ca = CA_default

    [ CA_default ]
    # Directory and file locations.
    dir               = .ca
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
    default_md        = sha512

    name_opt          = ca_default
    cert_opt          = ca_default
    default_days      = 3750
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
    default_bits       = 4096
    default_md         = sha512
    default_keyfile    = ngcpidCA.pem
    prompt             = no
    encrypt_key        = yes
    string_mask         = utf8only

    # Extension to add when the -x509 option is used.
    x509_extensions     = v3_ca

    # base request
    distinguished_name = req_distinguished_name

    # distinguished_name
    [ req_distinguished_name ]
    countryName            = "ID"                     # C=
    stateOrProvinceName    = "DKI Jakarta"                 # ST=
    localityName           = "Jakarta"                 # L=
    postalCode             = "10110"                 # L/postalcode=
    streetAddress          = "Merdeka"            # L/street=
    organizationName       = "ngcpidCA"        # O=
    organizationalUnitName = "Departemen Keamanan"          # OU=
    commonName             = "ngcpid CA local"            # CN=
    emailAddress           = "webmaster@ngcpid.local"  # CN/emailAddress=

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
</details>

## Verify the root certificate

```sh
# openssl x509 -noout -text -in ca/certs/ngcpidCA.cert.pem
```

```sh
echo "" > ca/index.txt
echo "1000" > ca/serial
```

windows

```sh
"" | Out-File -encoding ascii -NoNewline "ca/index.txt"
"00" | Out-File -encoding ascii -NoNewline "ca/serial"
```

## Create the intermediate pair

create directory

```sh
cd certs
echo 1000 > intermediate\crlnumber
```

## Create the intermediate key

```sh
openssl genrsa -aes256 -passout file:intermediate/config/password.starganCA.local -out intermediate/private/stargan.intermediateCA.key.pem 4096
```

## Create the intermediate certificate and CSR

```sh
openssl req -config intermediate/config/starganCA.cnf -new -sha512 -key intermediate/private/stargan.intermediateCA.key.pem -out intermediate/csr/stargan.intermediateCA.csr

# -subj "/CN=KBRI stargan Intermediate CA"

openssl req -text -noout -verify -in intermediate/csr/stargan.intermediateCA.csr

```

To create an intermediate certificate, use the root CA with the v3_intermediate_ca extension to sign the intermediate CSR. The intermediate certificate should be valid for a shorter period than the root certificate. Ten years would be reasonable.

> warning
> > This time, specify the root CA configuration file
>

```sh

cd certs

openssl ca -config ca/config/ngcpidCA.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha512  -in intermediate/csr/stargan.intermediateCA.csr -out intermediate/certs/stargan.intermediateCA.crt 

openssl x509 -noout -text -in intermediate/certs/stargan.intermediateCA.crt

```

## Create the certificate chain file

```sh
cd certificate\certs

cat intermediate/certs/stargan.intermediateCA.crt ca/certs/ngcpidCA.cert.pem > intermediate/certs/ca-chain.cert.pem
```

windows powershell

```sh
Get-Content intermediate\certs\stargan.intermediateCA.crt, ca\certs\ngcpidCA.cert.pem | Set-Content intermediate\certs\ca-chain.crt
```

## Install root CA

untuk menjadi CA sebenernya, kita perlu mempublish root certificate kita ke semua device di dunia

tetapi, untuk lokal development kita hanya perlu menginstall certificate kita di device lokal kita

### windows

1. Windows + R
2. ketikkan mmc
3. klik __File > Add/Remove Snap-in__
4. pilih __Certificate__ klik __Add__
5. pilih __Computer Account__ klik __Next__
6. pilih __Local Computer__ kemudian klik __Finish__
7. klik __OK__ dan kembali ke jendela __MMC__
8. dobel klik __Certificates (local computer)__
9. pilih __Trusted Root Certification Authorities__ , klik kanan __Certificates__ dikolom sebelah kanan, pilih __All Tasks__ kemudian __Import__
10. klik __Next__ kemudian __Browse__ , ubah dropdow certificate extension ke __All Files (\*.*)__ kemudian cari lokasi ngcpidCA.pem , klik __Open__ dan __Next__
11. pilih __Place all certificates in the following store__ di ```Trusted Root Certification Authorities store``` . klik __Next__ dan __Finish__

jika semua lancar seharusnya kita sudah bs melihat certificate kita di bawah
__Trusted Root Certification Authorities > Certificates__
