
## Letsencrypt for a specific Kubernetes scenario

..and a generic Docker image for [standalone](https://letsencrypt.readthedocs.org/en/latest/using.html#standalone) mode:

```
docker run -ti --rm \
  -e cert_domains=sub.example.net \
  -e cert_email=webmaster@example.net \
  -e LETSENCRYPT_ENDPOINT=https://acme-staging.api.letsencrypt.org/directory \
  solsson/letsencrypt-once
```

The [once](https://hub.docker.com/r/solsson/letsencrypt-once/) image is meant to be fired up once per certificate request/renew, proxied by the actual service that wants the cert.

```
---
apiVersion: v1
kind: Service
metadata:
  name: letsencrypt
spec:
  ports:
    - port: 80
  selector:
    role: letsencrypt
---
apiVersion: v1
kind: Pod
metadata:
  name: letsencrypt-once
  labels:
    role: letsencrypt
spec:
  restartPolicy: Never
  #volumes:
  containers:
    - name: letsencrypt-example
      image: solsson/letsencrypt-once
      env:
        - name: LETSENCRYPT_ENDPOINT
          value: https://acme-staging.api.letsencrypt.org/directory
        - name: cert_email
          value: webmaster@example.net
        - name: cert_domains
          value: sub.example.net
      ports:
        - name: http
          containerPort: 80
      #volumeMounts:
```

The commented out `volume` stuff is where you need to share, with the https service, `/etc/letsencrypt/live` with the resulting certs.

If your web server is apache, your http VirtualHost should do something like:
```
ProxyPass /.well-known/acme-challenge/ http://letsencrypt/.well-known/acme-challenge/
```

When it works and you want a real cert, comment out the LETSENCRYPT_ENDPOINT env.

### changelog

This is a third iteration on Letsencrypt for Kubernetes, after:
 * https://github.com/solsson/ssl-proxy-letsencrypt
 * https://github.com/solsson/docker-httpd-letsencrypt

Now it starts to look a lot like the setup in:
 * http://blog.ployst.com/development/2015/12/22/letsencrypt-on-kubernetes.html

The [current build](https://hub.docker.com/r/solsson/letsencrypt-once/builds/bcfjlfkyrb78ve7muabkezf/) sleeps for 1 hour after cert renewal. Two reasons for that:
 * Mistakes (like accidental pod replication) won't exhaust your letsencrypt weekly limit so fast.
 * Until you figured out how to share volumes you can extract `/etc/letsencrypt/live` certs manually.
