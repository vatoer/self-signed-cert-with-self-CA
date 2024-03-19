# goals

- create CA
- create intermediate CA
- create cert untuk web
- create PKI 12 untuk encrypt pdf

## docs

## [create self signed certificate with local ca](/docs/create_self_sign_certificate_with_local_ca.md)

## Preparation

## in windows

## Replace all variable

```sh
cd certs
cd docs

Get-ChildItem -Path "." -Recurse -File | ForEach-Object { (Get-Content $_.FullName) | ForEach-Object { $_ -replace 'stargan', 'yourdomain' } | Set-Content $_.FullName }

Get-ChildItem -Path "." -Recurse -File | ForEach-Object { (Get-Content $_.FullName) | ForEach-Object { $_ -replace 'change.with.your', 'your.name' } | Set-Content $_.FullName }

Get-ChildItem -Path "." -Recurse -File | ForEach-Object { (Get-Content $_.FullName) | ForEach-Object { $_ -replace 'webmaster@starganteknologi.com.id', 'mail@yourdomain.com' } | Set-Content $_.FullName }

```

## rename all folders

```sh
cd certs
Get-ChildItem -Path "." -Recurse | Where-Object { $_.Name -like "*stargan*" } | Rename-Item -NewName { $_.Name -replace 'stargan', 'yourdomain' }

Get-ChildItem -Path "." -Recurse | Where-Object { $_.Name -like "*change.with.yourname*" } | Rename-Item -NewName { $_.Name -replace 'change.with.yourname', 'yourname' }

```

## LINUX

## replace all variable in linux

In Linux, you can use the find and sed commands to recursively replace text in files. Here's how you can do it:

```sh
cd certs
find . -type f -exec sed -i 's/stargan/yourdomain/g' {} \;
find . -type f -exec sed -i 's/change.with.your/your.name/g' {} \;
```

## rename all folders in Linux

```sh
cd certs
find . -depth -name '*stargan*' -execdir bash -c 'mv -- "$1" "${1//stargan/yourdomain}"' bash {} \;
```
