# Let’s Encrypt

This is written based on FreeBSD and nginx, 
using secure Let’s Encrypt client [acme-client](https://kristaps.bsd.lv/acme-client/).  

## install
Download and install `acme-client`:  
```
curl -O https://kristaps.bsd.lv/acme-client/snapshots/acme-client-portable.tgz
tar -xvzf acme-client-portable.tgz
cd acme-client-portable-0.1.11
gmake install clean
```

Create necessary directories:  
```
mkdir -p /var/www/letsencrypt
mkdir -p /etc/ssl/letsencrypt
chmod 700 /etc/ssl/letsencrypt
```

## pre-config
#### nginx
In `server` block add following:
```
  location /.well-known/acme-challenge {
          alias /var/www/letsencrypt;
  }
```

## wrapper
For more convenient usage, create wrapper script `acme-client.sh`:  
```
#!/bin/sh
_ssldir="/etc/ssl/letsencrypt"
_chldir="/var/www/letsencrypt"

if [ "$(id -u)" -ne "0" ]; then
        echo "error: must be run as root." >&2
        exit 1
fi
if [ X"$1" = X"" ]; then
        echo "usage: $0 domain [args]" >&2
        exit 1
fi
_domain="$1"
shift
_args="$*"

cd "${_ssldir}"
if [ $? -ne 0 ]; then
        echo "error: ${_ssldir} does not exist." >&2
        exit 1
fi
if [ ! -f "${_ssldir}/privkey.pem" ]; then
        _newacc="-n"
else
        _newacc=""
fi

mkdir -p "${_ssldir}/private/${_domain}"
mkdir -p "${_ssldir}/public/${_domain}"
chmod 700 "${_ssldir}/private/${_domain}"
chmod 700 "${_ssldir}/public/${_domain}"
if [ ! -f "${_ssldir}/private/${_domain}/privkey.pem" ]; then
        echo "info: generating domain key"
        openssl genrsa 2048 > "${_ssldir}/private/${_domain}/privkey.pem"
        if [ $? -ne 0 ]; then
                exit 1
        fi
fi

acme-client \
-C "${_chldir}" \
-f "${_ssldir}/privkey.pem" \
-c "${_ssldir}/public/${_domain}" \
-k "${_ssldir}/private/${_domain}/privkey.pem" \
${_newacc} ${_args} ${_domain}

if [ X"${_newacc}" = X"-n" ]; then
        chmod 600 "${_ssldir}/privkey.pem"
fi
chmod 600 "${_ssldir}/private/${_domain}/privkey.pem"

```

## usage
```
usage: acme-client.sh domain [args]  
```
To request certificate:  
```
acme-client.sh example.com -v
```
To revoke certificate:  
```
acme-client.sh example.com -rv
```

## post-config
#### nginx
In `server` block add following:  
```
  ssl_certificate     /etc/ssl/letsencrypt/public/example.com/fullchain.pem;
  ssl_certificate_key /etc/ssl/letsencrypt/private/example.com/privkey.pem;
```
