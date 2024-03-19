# generate certificate utk server stargan

langkah-langkah setiap tahun

- generate key
- generate csr
- sign csr
- concat with chain certificate
- copy to server
- restart nginx

## Generate key

```sh
openssl genrsa -out stargan.local/private/stargan.local.key 2048
```

## generate csr

```sh
openssl req -new -key stargan.local/private/stargan.local.key -out stargan.local/csr/stargan.local.csr -config stargan.local/config/stargan.local.cnf
```

### check CSR

``` shell
openssl req -in stargan.local/csr/stargan.local.csr -text -noout
```

## sign csr

```sh
mkdir stargan.local/certs/

openssl x509 -req -in stargan.local/csr/stargan.local.csr -CA intermediate/certs/stargan.intermediateCA.crt -CAkey intermediate/private/stargan.intermediateCA.key.pem -CAcreateserial -out stargan.local/certs/bintang.stargan.local.crt -days 366 -sha512 -extfile stargan.local/config/bintang.stargan.local.ext

openssl x509 -noout -text -in stargan.local/certs/bintang.stargan.local.crt
```

## concat with chain certificate

linux

```sh
cat stargan.local/certs/bintang.stargan.local.crt intermediate/certs/ca-chain.cert.pem > fullchain.stargan.local.crt
```

or
windows

```sh
Get-Content -Path stargan.local/certs/bintang.stargan.local.crt, intermediate/certs/ca-chain.crt | Set-Content -Path stargan.local/certs/fullchain.stargan.local.crt
```

verify

```sh
openssl verify -CAfile intermediate/certs/ca-chain.crt stargan.local/certs/bintang.stargan.local.crt
openssl verify -CAfile intermediate/certs/ca-chain.crt stargan.local/certs/fullchain.stargan.local.crt
```

## copy to server

gunakan filezilla utk copy file key dan cert

copy file `stargan.local.key`

![copy file stargan.local.key](img/copy-private-key.png)

copy file `fullchain.stargan.local.crt`

![copy file fullchain.stargan.local.crt](img/copy-certs.png)

### ssh ke server

 ```psw
 > ssh keamanan@192.168.1.225
 ```

 ![ssh ke server](img/ssh-keamanan.png)

### lihat setting nginx

```sh
cat /etc/nginx/snippets/stargan.local.self-signedCA.conf
```

```cnf
ssl_certificate /etc/ssl/certs/fullchain.stargan.local.crt;
ssl_certificate_key /etc/ssl/private/stargan.local.key;
```

### copy dari dari folder ~/cert ke folder sesuai setting nginx

```sh
 sudo cp certs/fullchain.stargan.local.crt /etc/ssl/certs/
 sudo cp certs/stargan.local.key /etc/ssl/private/
 ```

## restart nginx

- test config

```sh
sudo nginx -t
```

restart nginx

```sh
sudo systemctl restart nginx
```

## additional reference

<https://devicetests.com/how-to-trust-certificate-on-iphone>
