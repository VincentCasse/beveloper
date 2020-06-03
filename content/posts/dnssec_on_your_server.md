---
title: "How to configure DNSSEC on your server?"
date: 2014-08-27T22:18:48+02:00
---

Since few years, I use my own bind server to announce my domain names (I buy my domains on ovh.com and change the nameserver inside the control panel). I have two servers, one master and one slave in two differents datacenters in case of downtime.

Today, I want to configure DANE on my domains, but one prerequisites is to have DNSSEC configured on his server!

I thought configure DNSSEC is hard... No, a lot of tools can help us!

In this article, i will write my examples with liberetongeek.com, one of my domains. Zone liberetongeek.com is defined inside db.liberetongeek.com file.

First step: installations
=========================

My servers are on debian{6|7}. So, i use apt-get as installer.

```bash
apt-get install dnssec-tools
```

Second step: configuration of bind on master and slave servers
==============================================================

Edit the /etc/bind/named.conf.options and check the next line are inside and uncomment:

```bash
options {
    dnssec-enable yes;
    dnssec-validation yes;
    dnssec-lookaside auto;
}
include "/etc/bind/bind.keys";
```

The file bind.keys must contain key of dns root.

After a reboot of bind, your dns server could be send signed zone.

Third step : sign a zone on master
==================================

We have install a lot of DNSSEC tool. One of them is zonesigner, a tool to generate keys and sign zones.

```bash
zonesigner -genkeys -usensec3 -zone liberetongeek.com db.liberetongeek.com
```
This command generate keys. If it's successful, there are a lot generated. Three keys (with private and public files)
* Kliberetongeek.com.+008+xx1.key
* Kliberetongeek.com.+008+xx1.private
* Kliberetongeek.com.+008+xx2.key
* Kliberetongeek.com.+008+xx2.private
* Kliberetongeek.com.+008+xx3.key
* Kliberetongeek.com.+008+xx3.private 

One is a signing key and two zone-signing keys.
* liberetongeek.com.krf with many informations about your zone key
* db.liberetongeek.com.signed. This is the signed zone file !

And to finish, edit the named.conf file to replace db.liberetongeek.com by db.liberetongeek.com.signed:
```bash
zone "liberetongeek.com" {
        type master;
        file "/etc/bind/db.liberetongeek.com.signed";
        notify yes;
        forwarders{};
};
```

Now, you can reload bind configuration on master.

Edit slave configuration
========================

On slave server, just edit your named.conf to replace db.liberetongeek.com by db.liberetongeek.com.signed. Reload your bind and your configuration is ready on your machines

Last step: send you key to your parent tld
==========================================

To ensure your dns record use the good key, you need to send at your parent domain your signing public key. Remember, that was one of the generated file.

On ovh control panel, this is possible to do on old control panel:
https://ovh.com/managerv3/ and in menu "Liberetongeek.com" > Domains > Secure delegation > Edit.

To fill the form, use information of your key-signing key. In my file, I have
```bash
; This is a key-signing key, keyid __21061__, for liberetongeek.com.
; Created: 20140827221947 (Thu Aug 28 00:19:47 2014)
; Publish: 20140827221947 (Thu Aug 28 00:19:47 2014)
; Activate: 20140827221947 (Thu Aug 28 00:19:47 2014)
liberetongeek.com. IN DNSKEY __257__ 3 __8__ ''AwEAAbn5h2dA0nYh/nymq3HoZ/Q5xvgbou1/ESgUQzQNy3/ZyRYfLRcN H9F3BTRo99XNtRSJhS3H9w+tgRmnMJsawrEO3X3btRxZMsYWbASxhloG xxx3GLnkFdtGXXHWutz0K7kkMObhWzMM/TfCf3d lRPYSPjJgbU=''
```

my id is: 21061
my flag is: 257
algorithm is: 8
et ma clef est :and my key is: AwEAAbn5h2dA0nYh/nymq3HoZ/Q5xvgbou1/ESgUQzQNy3/ZyRYfLRcN H9F3BTRo99XNtRSJhS3H9w+tgRmnMJsawrEO3X3btRxZMsYWbASxhloG xxx3GLnkFdtGXXHWutz0K7kkMObhWzMM/TfCf3d lRPYSPjJgbU=

After few minutes, your domains is secured by DNSSEC. You can now check if record of your DNS is yours or if a hackers try to get traffic from your domain.

Next step : how to configure DANE.