# haproxy

## health-check

### nginx health-check

Default health-check in `haproxy` ...

```
backend local-nginx
  server nginx 127.0.0.1:80 check
```

... can cause error messages in `error_log` of `nginx`:

```
2000/01/01 00:00:00 [error] 123456#123456: accept4() failed (53: Software caused connection abort)
```

It's even described in `nginx` official documentation - http://nginx.org/en/docs/faq/accept_failed.html.
One of solutions is to use [option httpchk](http://cbonte.github.io/haproxy-dconv/1.7/configuration.html#4-option%20httpchk) in `haproxy`'s backend configuration.
By default it will use "OPTIONS" method, which isn't allowed in `nginx`, but this can be worked around.

* solution using "OPTIONS" method:  
   ```
   backend local-nginx
     option httpchk
     server nginx 127.0.0.1:80 check
   ```
   
   ```
   location / {
     if ($request_method = OPTIONS) {
       add_header Content-Length 0;
       add_header Content-Type text/plain;
       return 200;
     }
     ...
   }
   ```

* solution using "HEAD" method:

    ```
    backend local-nginx
      option httpchk HEAD / HTTP/1.0
      server nginx 127.0.0.1:80 check
    ```

In either solution `access_log` of `nginx` will get filled with either
```
127.0.0.1 - - [01/Jan/2000:00:00:00 +0100] "OPTIONS / HTTP/1.0" 200 0 "-" "-"
```

or

```
127.0.0.1 - - [01/Jan/2000:00:00:00 +0100] "HEAD / HTTP/1.0" 200 0 "-" "-"
```

To filter these logs out, add log filtering (_example for "HEAD" method filtering_):
```
http {
    map "$request_method:$request_uri:$remote_addr" $loggable {
        "HEAD:/:127.0.0.1" 0;
        default 1;
    }
    access_log /var/log/nginx/access.log combined if=$loggable;
    ...
}
```
