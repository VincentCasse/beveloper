---
title: "Don't trust CA? How to configure DANE to secure your SSL certificate"
date: 2014-11-13T22:18:48+02:00
draft: false
---

You think that your customers are secured against man-in-the-middle-attack using a SSL certificate validated by a Certificate Authorities? Indeed, only Certificate Authorities can ensure your certificate is linked with your website and was not build by a hacker.

Why Certificate Authorities are not safe?
=========================================

But, what about Certificate Authorities hacking... Internet history noticed yet [few compromises of CA](http://en.wikipedia.org/wiki/Certificate_authority#CA_compromise). And the real risk comes from possibility for all CA to generate SSL certificate for all websites, facebook.com or google.com too!

Imagine your government have a [secret contract](http://arabcrunch.com/2011/09/wikileaks-microsoft-accused-in-helping-bin-ali-monitor-tunisians-corruption-stifling-open-source.html) with one of 1800 CA. Imagine your government want to access to protestors gmails and facebook...

In this case, you cannot ensure there is no man-in-the-middle. You need to trust your certificate with an other way.
Simple way is to deploy certificate into your application. So, you don't need to ask to Certificate Authorities if the certificate is good. But, you need to update theses software at each time you modify your certificate, and that's doesn't work inside web application.

Another way is to deploy [DANE](http://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities). DANE is a protocol to allow certificate validation with DNSSEC instead of CA. Only your parent inside DNSSEC validation tree can be a risk for your security (For CA system, you trust all CA all around the world!).
DNSSEC ensure you are the owner of the domain. So, DANE allow to announce public key of your certificate inside your DNS.

Why DANE is not deploy?
=======================

Currently, DANE is not deploy by default inside browsers. Chrome and firefox have plugin to check DNSSEC and DANE validity but doesn't provide a way to authorize by default self-signed certificate announced by DANE.
For Firefox, there is a [bugzilla](https://bugzilla.mozilla.org/show_bug.cgi?id=672600) about these feature.

How to configure DANE and your HTTP server (nginx)?
===================================================

If your website is for a big audience, the good way is to configure DANE with a SSL certificate validated by a CA. If you won't pay for a SSL certificate, you can create your own certicate (self-signed)

## Generate your self-signed certificate

First, generate your key
```bash
openssl genrsa -aes256 -out /etc/nginx/ssl/key_beveloper.key 4096
```

With these key, you can generate a certification request for your website
```bash
openssl req -new -sha256 -key /etc/nginx/ssl/key_beveloper.key -out /etc/nginx/ssl/key_beveloper.csr
```

Then, you need to remove your passphrase. If you don't do it, you will type your passphrase each time you restart your web server.
```bash
openssl rsa -in /etc/nginx/ssl/key_beveloper.key -out /etc/nginx/ssl/key_beveloper.key
```

And to finish, you can generate your self-signed certificate
```bash
openssl x509 -sha256  -req -days 120 -in /etc/nginx/ssl/key_beveloper.csr -signkey  /etc/nginx/ssl/key_beveloper.key -out  /etc/nginx/ssl/key_beveloper.crt
```

## Configuring Nginx with you SSL certificate

Configuration to add SSL certificate inside nginx is pretty simple:
```bash
server {
    server_name         key.beveloper.fr;
    listen              443 ssl;
    listen              [::]:443 ssl;

    ssl on;
    ssl_certificate     /etc/nginx/ssl/key.beveloper.crt;
    ssl_certificate_key /etc/nginx/ssl/key.beveloper.key;
    ssl_session_timeout 5m;
    ssl_prefer_server_ciphers on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ALL:!aNULL:!eNULL:!LOW:!EXP:!RC4:!3DES:+HIGH:+MEDIUM;

    ssl_dhparam /etc/ssl/private/dh2048.pem;
    add_header Strict-Transport-Security max-age=2678400;
}
```

If you want to disable HTTP, you can add a redirect from HTTP to HTTPS
```bash
# redirect HTTP traffic to HTTPS
server {
    listen              80;
    listen              [::]:80;

    server_name         key.beveloper.fr;
    return              301 https://key.beveloper.fr$request_uri;
}
```

## Configuring DANE into your DNS

Configure DANE is very simple. You need just add a zone into your DNS configuration.
First, you need to get fingerprint of your SSL certificate
```bash
openssl x509 -noout -fingerprint -sha256 < /etc/nginx/ssl/key_beveloper.crt | tr -d :
```

It's time to announce your SSL certificate inside your DNS configuration.
Just add a zone like this
```bash
443._tcp.key.beveloper.fr. 3600 IN TLSA ( 3 0 1 EF1FEDA7862ED0474521DE0C7CDF25A27DB9A1E99129423D0BB0D6C2B621B2A7)
```

* 443 is port use by your program to access to your service
* _tcp is network protocol used to access to your service
* key.beveloper.fr is the subdomain secured by SSL certificate with DANE 
* TLSA field is the specific field defined by DANE protocol, is cached here for 3600 second
* 3 0 1 defines that's the hash is calculated from entire certificate and sha-256 (see [netfuture](http://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities) or [protocol](http://tools.ietf.org/html/rfc6698) to have more informations)
* EF1F[...]1B2A7 is the hash of my certificate for one of my subdomains.

Reload your DNS zone and don't forgot to sign your zone with DNSSEC!

If you want to test your new record, you could use dig (>9.8) to check TLSA fields
```bash
dig _443._tcp.key.beveloper.fr tlsa +short
```

## Use DANE to build a safe service

DANE is easy to configure if you already have a SSL certificate. So why not using it to ensure with your customer are not attacked by man-in-the-middle?

The real risk is like IPv6: there is not a lot of users who checks theses records and if you don't monitor DANE, you could have bad configuration during a lot of time. That's why I develop a small [check for nagios/shinken](https://github.com/VincentCasse/check_dane_certificate)!

Be the new explorers of online resources trusting!

__Note:__
* All examples are validated by [ssllabs](https://www.ssllabs.com/). Grade is not A because there is no validation from CA...
* These blog is DANE compliant, but certificate is also trusted by a CA
* These system could be use by other applications than web to ensure server is not a fake ;)

__Links:__
* https://netfuture.ch/2013/06/how-to-create-dnssec-dane-tlsa-entries/
* http://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities
* http://tools.ietf.org/html/rfc6698