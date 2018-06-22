---
language: en
layout: post
title: "Why time synchronization is important in CCTV"
excerpt: "and how you should fix your NTP server"
categories: articles
comments: true
tags: cctv onvif ntp security cybersecurity time synchronization
keywords: cctv onvif ntp security cybersecurity time synchronization
---

I am writing this article, because once a season I get a complain about the need of time synchronization in CCTV systems. Installers want something that _just works_, no matter what the implication for the customer may be.

As many of you know, Videotec is contributing member of ONVIF, and I attend the Face2Face meetings and contribute to the ONVIF specifications. ONVIF is nowadays the standard protocol for the CCTV market, thus it is the best choice for explaining the issues that may rise with poor time synchronization between a camera and a VMS. These consideration in general apply to proprietary protocols as well, so even if you are not using ONVIF you should not stop reading this article.

## How does password-based authentication work in ONVIF?

ONVIF is based on Web Services and the SOAP protocol. In the core specification, two methods are selected for authentication:
* WS-UsernameToken
* HTTP digest

[WS-UsernameToken](https://www.oasis-open.org/committees/download.php/13392/wss-v1.1-spec-pr-UsernameTokenProfile-01.htm) was the first authentication method for the ONVIF protocol. The credential is embedded in the header of the SOAP message used to transport.

[HTTP digest](https://tools.ietf.org/html/rfc2617) was added in a second phase of the protocol development. The main difference from the previous method is that it is based on the HTTP headers, thus it it not handled by the ONVIF protocol, but by the underlying web-server.

The [ONVIF Core Specification](https://www.onvif.org/specs/core/ONVIF-Core-Specification-v1712.pdf) explains in Section 5.12.1 how to make these two authentication methods live together in an ONVIF device. They key idea is that an ONVIF device can support both methods, and wrong time synchronization impacts differently both of them.

### Time synchronization in WS-UsernameToken

Let start by looking at an example of a payload of an authenticated SOAP message:

    <wsse:Security>
        <wsse:UsernameToken>
            <wsse:Username>NNK</wsse:Username>
            <wsse:Password Type="...#PasswordDigest">
                weYI3nXd8LjMNVksCKFV8t3rgHh3Rw==
            </wsse:Password>
            <wsse:Nonce>WScqanjCEAC4mQoBE07sAQ==</wsse:Nonce>
            <wsu:Created>2003-07-16T01:24:32Z</wsu:Created>
        </wsse:UsernameToken>
    </wsse:Security>

As we can see, three elements are sent unencrypted:
* the username
* the nonce
* the timestamp of the creation of the UsernameToken payload.

The password is the output of an hashing function that takes into account not only the plain text password, but also the nonce and the timestamp, with the following formula:

    Password_Digest = Base64 ( SHA-1 ( nonce + created + password ) )

Since the camera stored internally the value of the password, it can recalculate the Password_Digest and only if it gets the same result as the one sent by the client it can authorize the required action.

Why does it hashes also the nonce and the timestamp? They are both needed to avoid [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). If only the password is hashed, in practice there is no difference from using plain text password to access the service, as it happens in the [HTTP basic authentication schema](https://tools.ietf.org/html/rfc7617). In fact, if an attacker is able to get access to just one message, he could use the same password hash again and again to send malicious commands to the camera.

By adding a nonce and a time stamp, this risk is eliminated:
* nonces shall not be accepted by the camera more than once. To implement this, cameras usually have a buffer of used nonces, and every time they have a new request they check if the supplied nonce has already been used.
* since cameras are embedded systems with limited RAM and storage, they cannot store all the previously used nonces. The buffer is usually sized so that the camera can hold all the nonce of an interval. Once a nonce is older that a set number of seconds, it is considered expired and removed from the list of used nonces. An attacker could at this point replay the nonce to attack the camera. By checking also the created timestamp the replay attack will fail, because repeating the nonce will generate a different hash value.

As a final security measure, cameras verify that the created timestamp is synchronized with the internal clock with a certain clock skew tolerance. As an example, the default clock skew tolerance in gSoap is 300s. Any message generated with a difference of time bigger than the values is discarded because it could be either a replay attack or something trying to crack the camera's firmware.

**Trivia**: _how can an ONVIF tool configure a Profile-S camera that is just out from the box which implements only WS-UsernameToken as authentication method?_

The correct handshake procedure is:
1. the client invokes `GetSystemDateAndTime` to get the camera clock value;
2. the client checks its internal clock;
3. the client determines the time difference between the two values;
4. the client sends a `SetSystemDateAndTime` to synchronize the clock of the camera with the clock of the system;
5. the client configures all other parameters.

On the market there are VMSs that implement this handshake procedure when adding cameras into the system.

### Time synchronization in HTTP digest

Unlike WS-UsernameToken, the [HTTP digest](https://tools.ietf.org/html/rfc7616) authentication method does not explicitly include information about time in its definition. Still, in the RFC7616 the possibility of including the timestamp is suggested as an option to increase security.

Before entering the details for adding timestamp to the HTTP digest, it is necessary to explain how is works.

Let's suppose that the client is requesting `/dir/index.html` page on a web-server

    GET /dir/index.html HTTP/1.0
    Host: localhost

If the access to the page is restricted, the web-server will send back the following answer

    HTTP/1.0 401 Unauthorized
    Server: HTTPd/0.9
    Date: Sun, 10 Apr 2014 20:26:47 GMT
    WWW-Authenticate: Digest realm="testrealm@host.com",
                            qop="auth,auth-int",
                            nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                            opaque="5ccc069c403ebaf9f0171e9517f40e41"
    Content-Type: text/html
    Content-Length: 153

    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8" />
        <title>Error</title>
      </head>
      <body>
        <h1>401 Unauthorized.</h1>
      </body>
    </html>

The key part of the response is the `WWW-Authenticate` header. This header specifies a realm, which is the description of the restricted area that the user is trying to access, the quality of protection `qop`, which specifies that authentication can be either be used to perform authentication (`auth`) or to to perform authentication and verification of message integrity (`auth-int`), the specifies a number that can be used only once every time a `WWW-Authenticate` header is generated called `nonce` and an `opaque` value, which is a string of that that should not be modified by the client.

At this point the client repeats the request, by adding the `Authorization` header:

    GET /dir/index.html HTTP/1.0
    Host: localhost
    Authorization: Digest username="Mufasa",
                         realm="testrealm@host.com",
                         nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                         uri="/dir/index.html",
                         qop=auth,
                         nc=00000001,
                         cnonce="0a4f113b",
                         response="6629fae49393a05397450978507c4ef1",
                         opaque="5ccc069c403ebaf9f0171e9517f40e41"

As it can be easily seen, `realm`, `nonce` and `opaque` are sent back to the server without any modification, while the client chooses the `auth` method as quality of protection (which means only authentication)

Finally the web-server accepts the requests and gives a positive response:

    HTTP/1.0 200 OK
    Server: HTTPd/0.9
    Date: Sun, 10 Apr 2005 20:27:03 GMT
    Content-Type: text/html
    Content-Length: 7984

So far, so good, because time is not involved in the basic mechanism of the Digest authentication. However, this way of performing authentication is exposed to reply attacks. In fact, a client could re-transmit the same command to the camera with the unaltered `Authorization` field  and the camera could not discriminate if the sender is good or malicious. For this reason, RFC2617 states

> The contents of the nonce are implementation dependent. The quality
> of the implementation depends on a good choice. A nonce might, for
> example, be constructed as the base 64 encoding of
>
>     time-stamp H(time-stamp ":" ETag ":" private-key)
>
> where time-stamp is a server-generated time or other non-repeating
> value, ETag is the value of the HTTP ETag header associated with
> the requested entity, and private-key is data known only to the
> server.  With a nonce of this form a server would recalculate the
> hash portion after receiving the client authentication header and
> reject the request if it did not match the nonce from that header
> or if the time-stamp value is not recent enough. In this way the
> server can limit the time of the nonce's validity.

For this reason, devices that implement this strategy to prevent replay attacks must have a reliable source of time. Time must be monotone even after a reboot, to have this kind of protection working.

## What about encryption?

Since password-based authentication schemes are in general weak, the ONVIF protocol offers a way to totally bypass passwords in the [Advanced Security specifications](https://www.onvif.org/ver10/advancedsecurity/wsdl/advancedsecurity.wsdl).

In this schema, authentication and authorization are performed over x.509 certificates. However, certificates have a validity range, expressed in terms of Not Before and Not After. Thus, preventing the usage of expired certificates still requires time synchronization, to make sure certificates are accepted in the desired time interval.

## What's the best timezone?

Many users require burning date and time into the video at compression time, so that they can easily recognize when the video was recorded. In order to get a better user experience, users required to have time expressed in their time zone.

This choice, unfortunately, may introduce drawbacks, because in may time zones time is not monotone. In fact, daylight savings introduces two discontinuities a year. In one part of the year local clocks will be advanced by one hour, leaving a hole of 60 minutes. In the other discontinuity, the same hour will exist twice, exposing the system to replay attacks.

Daylight saving is the most immediate use case scenario to understand that holes in time may exist, but it is not the only one. Let's think about cargo ships: their journey is likely to span over different time zones.

## How to configure an authoritative time server in Windows Server

Windows offers an NTP server that can be used to synchronize the IP cameras. Unfortunately, in CCTV many systems are air-gapped, either because of security policies or simply because they are physically located in places where a connection to the Internet is not available.

If the Windows server is not able to synchronize with one or more server of a higher stratum, it will announce itself as a not authoritative NTP server. As a result, devices may refuse to synchronize with the server, preventing the CCTV from working.

Microsoft provides [instructions](https://support.microsoft.com/en-us/help/816042/how-to-configure-an-authoritative-time-server-in-windows-server) about how to configure an isolated NTP server:

1. Select Start > Run, type regedit, and then select OK.
2. Locate and then select the following registry subkey: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Config\AnnounceFlags`
3. In the right-pane, right-click `AnnounceFlags`, and then select Modify.
4. In Edit DWORD Value, type `A` in the Value data box, and then select OK.
5. Close Registry Editor.
6. At the command prompt, type the following command to restart the Windows Time service, and then press Enter: `net stop w32time && net start w32time`
