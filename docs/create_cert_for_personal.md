# readme

GitHub Copilot: To create a certificate for personal use, you can use OpenSSL. Here are the steps:

1. **Generate a private key**: This command generates a new private key. The key will be 2048 bits long.

    ```sh
    openssl genrsa -out personal/change.with.your.key 2048
    ```

2. **Create a Certificate Signing Request (CSR)**: This command generates a new CSR. The CSR includes the public key from the private key we just created, and some additional information about the user.

    ```sh
    openssl req -new -key personal/change.with.your.key -out personal/change.with.your.csr
    ```

    During this step, you'll be asked to enter some information about yourself. This information will be included in the certificate.

3. **Sign CSR**: This command uses the CSR to be signet by intermedia CA . The certificate will be valid for 365 days.

    ```sh
    openssl x509 -req -days 365 -in personal/change.with.your.csr -CA intermediate/certs/stargan.intermediateCA.crt -CAkey intermediate/private/stargan.intermediateCA.key.pem -CAcreateserial -out personal/change.with.your.crt -extensions usr_cert -extfile intermediate/config/starganCA.cnf
    ```

After running these commands, you should have a new self-signed certificate at `personal/change.with.your.crt`. This certificate can be used to sign and encrypt files.

To sign a file, you can use the `openssl dgst` command:

```sh
openssl dgst -sha256 -sign personal/change.with.your.key -out file.sig file.txt
```

This command creates a SHA-256 digest of the file `file.txt`, signs the digest with the private key `personal/change.with.your.key`, and outputs the signature to `file.sig`.

To encrypt a file, you can use the `openssl enc` command:

```sh
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass file:personal/change.with.your.key
```

This command encrypts the file `file.txt` with AES-256-CBC, using the private key `personal/change.with.your.key` as the password, and outputs the encrypted file to `file.enc`.

Here's the command to decrypt the file:

```sh
openssl enc -aes-256-cbc -d -in file.enc -out file_decrypted.txt -pass file:personal/change.with.your.key
```
