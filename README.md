
Someting like:
```
docker run -ti --rm \
  -e cert_domains=sub.example.net \
  -e cert_email=webmaster@example.net \
  -e LETSENCRYPT_ENDPOINT=https://acme-staging.api.letsencrypt.org/directory \
  solsson/letsencrypt-once
```

This is my third iteration on Letsencrypt in Kubernetes, after
 * https://github.com/solsson/ssl-proxy-letsencrypt
 * https://github.com/solsson/docker-httpd-letsencrypt

Now it starts to look a lot like the idea I once failed to adapt:
 * http://blog.ployst.com/development/2015/12/22/letsencrypt-on-kubernetes.html

For non-`once`, could we have a server running with the following?
```
--standalone \
--standalone-supported-challenges http-01 \
```
