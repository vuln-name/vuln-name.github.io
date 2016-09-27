---
layout: post
author: phosphore
title: "UnluckyHMAC"
category: vulnerability
date: "05/2016"
---

## UnluckyHMAC  
UnluckyHMAC (also know LuckyMinus20, FreezerBurn) was the original name given by Juraj Somorovsky, the researcher who discovered the vulnerability.  

>because our HMAC is sad not to be able to validate bytes :)  

<!-- more -->
Source: [Juraj Somorovsky](http://web-in-security.blogspot.it/2016/05/curious-padding-oracle-in-openssl-cve.html)  

LuckyMinus20 is the name given by Filippo Valsorda in [his post on the CloudFlare's blog](https://blog.cloudflare.com/yet-another-padding-oracle-in-openssl-cbc-ciphersuites/).  
	
# CVEs (NVD/MITRE)
[CVE-2016-2107](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2107) (Padding oracle in AES-NI CBC MAC check), a consequence of CVE-2013-0169's incorrect fix.

# Detection
**Detected by:** Juraj Somorovsky  

## Timeline
05/03/2016 - First detection  
05/04/2016 - Public disclosure

# Description

## TL;DR
The AES-NI implementation in OpenSSL before 1.0.1t and 1.0.2 before 1.0.2h does not consider memory allocation during a certain padding check, which allows remote attackers to obtain sensitive cleartext information via a padding-oracle attack against an AES CBC session, NOTE: this vulnerability exists because of an incorrect fix for CVE-2013-0169.

## Full description
Very long story short, the CBC cipher suites in TLS have a design flaw: they first compute the HMAC of the plaintext, then encrypt:  

`plaintext || HMAC || padding || padding length`  

using CBC mode. The receiving end is then left with the uncomfortable task of decrypting the message and checking HMAC and padding without revealing the padding length in any way. If they do, we call that a padding oracle, and a MitM can use it to learn the value of the last byte of any block, and by iteration often the entire message. You can learn more about CBC padding oracles and their history in much more detail on [this blog post].  
The best way to explain the problem is to give an example of a message triggering a different alert. 

Given an attacker sends to the server a message decrypting to two blocks with an incomplete padding, the server responds with a RECORD_OVERFLOW alert. Example gives a message with 32 bytes of 0xFF padding.  

| FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF |
| -- |:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|--: |
| FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF | FF |

As you can correctly see, this message lacks more than 200 padding bytes (0xFF means there should be 257 padding bytes), and a MAC. Therefore, it should be  rejected. However, this is not the case for the OpenSSL AES-NI implementation. This implementation does not check whether there are enough data for MAC and padding validation. It just checks whether all the decrypted padding bytes are correct. If there are not enough padding bytes, it just omits further padding and MAC validation and proceeds with further internal steps in the record processing function. See the [incorrect fix here](https://github.com/openssl/openssl/blob/b1a07c38540105077878ad80a6a87c96fdbc18f1/crypto/evp/e_aes_cbc_hmac_sha1.c#L739).  


If such a decrypted message is further processed by the function **tls1_cbc_remove_padding** (file s3_cbc.c), this function resets the message size and computes it as follows:

     `rr->length -= padding_length + 1;`  

Note that in our case the record length is 32 (two 16 byte blocks) and the padding length is 256. Thus, the record length variable underflows and becomes

     `rr->length = 4294967264`


This also explains why we get the record overflow alert, triggered from s3_pkt.c. There, we have a check whether the data decrypted message data is too long (probably this check was introduced by validate the decompressed data):

    `if (rr->length > SSL3_RT_MAX_PLAIN_LENGTH + extra) {
        al = SSL_AD_RECORD_OVERFLOW;
        SSLerr(SSL_F_SSL3_GET_RECORD, SSL_R_DATA_LENGTH_TOO_LONG);
        goto f_err;
    }`  

Since our record length variable is now really huge, the RECORD_OVERFLOW is returned.




# CVSS (v2 base score)
 2.6 (LOW) by 10 (HIGH)

# Resources
[NVD NIST](https://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2016-2107)  
[MITRE](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2107)  
["Official" website](http://web-in-security.blogspot.it/2016/05/curious-padding-oracle-in-openssl-cve.html)