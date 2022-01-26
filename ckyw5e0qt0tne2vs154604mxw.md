## Kasm Workspaces: Addressing HTTPS error on the fresh installation

> Based on [Create Your Own SSL Certificate Authority for Local HTTPS Development](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development)

# Is it mandatory to have correct SSL certificate?

Yes and no. In some web projects, you can slip through the default "I know, just let me in" option in browsers. But nowadays, it is hard to find a website that doesn't load the JavaScript - and [browser won't let you load a JS from the external sources when certificate is not valid](https://web.dev/fixing-mixed-content/#upgrading_insecure_requests).

From the security point of view - you should not deliver solutions that with invalid certificate. There was a question on the Reddit asked by [cool-thinker](https://www.reddit.com/r/kasmweb/comments/s8tqyw/comment/htug4yo/?utm_source=share&utm_medium=web2x&context=3) - if such invalid certificate cases connection to be not encrypted. Let's see it now.

There is a great [step-by-step wiki](https://en.wikiversity.org/wiki/Wireshark/HTTPS) on the HTTPS traffic analysis with Wireshark. By applying this knowledge for the current Kasm Workspaces URL with invalid certificate, you can see that the data is encrypted. 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643122958096/XIqFNtsuC.png)

Don't let it cloud your judgement, though - danger does not come from the lack of encryption, but from the lack of knowledge if the party that signed that certificate can be trusted. 

# Why not Let's Encrypt?

Let's Encrypt is a well-known, trusted [Certificate Authority](https://letsencrypt.org/certificates/) that allows everyone with the accepted TLD (top-level domain, like `*.com` or `*.org`) to generate secure certificate for their websites' SSL connections.

From [Let’s Encrypt Community Support](https://community.letsencrypt.org/t/name-does-not-end-in-a-public-suffix/77293/4):
> These are two distinct “systems” with distinct choices (choose one per domain):
* Use a private certificate [private CA] for a private domain.
* Use a public certificate [LetsEncrypt] for a public domain.

Another, similar response: [Can I create a cert for a private domain?](https://community.letsencrypt.org/t/can-i-create-a-cert-for-a-private-domain/27264/3)

In our example, we have a **private** domain that would like to be trusted by the **public** certificate of Let's Encrypt - that won't work. If we tried to do that:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642899826522/T1XLpclsm.png)

So now, our plan is:
1. **Become private CA by generating private Root Certificate**.  
We will be using this certificate to trust other certificates in the trust chain. Our Root Certificate is the only certificate that needs to be installed on the all systems that will be accessing Kasm Workspaces.
2. **Create new certificate for the usage of Kasm Workspaces**.  
This is the certificate that will be imported to the Kasm. This one will show up when accessing Kasm Web UI. Trust is granted by our private CA Root Certificate.

# Acquire private Certificate Authority Root Certificate

%%[support-cta]

1. Generate CA private key:
```sh
$ openssl genrsa -aes256 -out cybethme-ca.key 2048
```

2. Generate CA Root Certificate (10ys validity):
```sh
$ openssl req -x509 -new -nodes -key cybethme-ca.key -sha256 -days 3650 -out cybethme-ca.pem
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642901945297/rEFKqWcZ3.png)

3. `ca-certificates` expects PEM files with `*.crt` extension, so let's give it to him:
```sh
$ sudo cp cybethme-ca.pem /usr/local/share/ca-certificates/cybethme-ca.crt
```
4. Update certificates database and verify:
```sh
$ sudo update-ca-certificates
# sudo update-ca-certificates --fresh / to rebuild from scratch
$ awk -v cmd='openssl x509 -noout -subject' '/BEGIN/{close(cmd)};{print | cmd}' < /etc/ssl/certs/ca-certificates.crt | grep Cyber
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642903752944/iaIkgRxYY.png)

# Acquire private certificate for Kasm Workspaces

%%[join-cta]

1. Generate private key:
```sh
$ openssl genrsa -out kasm.rpi.key 2048
```

2. Create Certificate Signing Request (CSR):
```sh
$ openssl req -new -key kasm.rpi.key -out kasm.rpi.csr
```

3. Create `ext` file (*kasm.rpi.ext*) to [supply](https://stackoverflow.com/a/43665244/6710729) during making a signing request.
```txt
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kasm.rpi
```

4. Create signed certificate for the Kasm:
```sh
$ openssl x509 -req -in kasm.rpi.csr -CA cybethme-ca.pem -CAkey cybethme-ca.key -CAcreateserial -out kasm.rpi.crt -days 730 -sha256 -extfile kasm.rpi.ext
```

Now you have the CRT certificate that can be used in application.

> You may notice that additional SRL file for Root CA is created. It is required by OpenSSL to track serial number of generated certificates - read more about it [here](https://stackoverflow.com/questions/66357451/why-does-signing-a-certificate-require-cacreateserial-argument).

# Upload certificates to Kasm and endpoints

1. SSH into Kasm server and replace certificate and the private key:
```sh
$ sudo /opt/kasm/bin/stop
$ sudo cp ~/.certs/kasm.rpi.crt /opt/kasm/current/certs/kasm_nginx.crt
$ sudo cp ~/.certs/kasm.rpi.key /opt/kasm/current/certs/kasm_nginx.key
$ sudo /opt/kasm/bin/start
```

2. Copy CA Root Certificate to the systems that will be using the Kasm Workspaces. It depends on the system, but on Windows - double-click the certificate and import it to Local Machine (or Current User) in Trusted Root Certification Authorities.

3. Change `hosts` entry (on endpoint) to point chosen Kasm address to the IP address.
```txt
192.168.0.105 kasm.rpi
```

Now we have access to the Kasm Workspaces without errors on the browser.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642982280406/hv5Dlbsls.png)

Unfortunately, this is not the end. When I tried to launch an image, [I've got an error message](https://www.reddit.com/r/kasmweb/comments/sbk7yp/nginx_failed_to_reload_after_generating_config/). Last step is changing the `Upstream Auth Address` for the default zone to the local IP address:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643033669128/Ko_b3PDmH.png)

# Conclusion

%%[follow-cta]

So far, we have a working Kasm Workspaces installation that we can connect via secure SSL connection. After making the device accessible over the Internet, you can perform containerized operations from any system using a browser. In the next tutorials, I will showcase how you can add more images to the Kasm and how you can persist data between sessions.

Check out other guides from the Kasm Workspaces Rapberry Pi series.
