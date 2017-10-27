---
layout: post
title: ONVIF / gSOAP in C++ by example [Pt-1]
description: Retrieving a snapshot from an ONVIF complaint IP Camera
date: '2016-10-21 20:33:40'
tags: onvif soap gsoap c++
permalink: onvif-gsoap-in-c-by-example/
---

## Retrieve a snapshot from an ONVIF complaint IP Camera using a client application written in C++. 

If all you need is the code, head [here](#the-example). To learn more about the ONVIF specification and how exactly gSOAP works, see [ONVIF / gSOAP in C++ by example [Pt-2]]({% post_url 2017-10-27-onvif-gsoap-in-c-by-example-pt-2 %}).

***

### Disclaimer
When I first started working with ONVIF and SOAP about a year ago, I was completely lost. I just couldn't figure out where to start. 
I'm writing this post with the intention of providing other developers (who find themselves in a similar situation today that I was in a few months back) with a head-start. 
This post therefore, is *NOT* meant to be a complete tutorial. 

---


#### ONVIF
ONVIF is a global and open industry forum. The ONVIF specification aims to achieve interoperability between network video products regardless of manufacturer. <sup>[1](#onvif-wikipedia)</sup>
ONVIF specifications are available in the form of [WSDL XML](https://en.wikipedia.org/wiki/Web_Services_Description_Language) (Web Services Description Language) files. 


#### SOAP
SOAP (Simple Object Access Protocol) is a protocol specification for exchanging structured information in the implementation of web services in computer networks. It uses XML Information set for its message format.<sup>[2](#soap-wikipedia)</sup>
SOAP relies on HTTP (or SMTP) for transmission. 

Now, just in case you were wondering what's so *simple* about SOAP, rest assured you're not alone. 


#### gSOAP
So now, we have a specification (in the form of WSDL files) and we have the protocol (SOAP) for the server-client communication. All we need now is to translate these specs to code. 
Heres where gSOAP comes in.

gSOAP is a C and C++ software development toolkit for SOAP/XML web services and generic XML data bindings.<sup>[3](#gsoap-wikipedia)</sup>

Basically, gSOAP first translates the WSDL spec to a header file with the data binding interface. This is done by the `wsdl2h` tool.
Then, the `soapcpp2` tool runs a preprocessor on this header file and generates the data binding implementation with XML serializers to implement Web services and XML data bindings.
The runtime engine handles HTTP and XML transport over any IO device and sockets and is responsible for memory allocation.<sup>[4](#gsoap-genivia)</sup>

---

### Using gSOAP with ONVIF
For this topic, I couldn't do a better job than what the folks at [Genivia](https://www.genivia.com) have already done on their website's [*Developer Center*](https://www.genivia.com/resources.html#How_do_I_use_gSOAP_with_the_ONVIF_specifications?). Their website is full of awesome resources on gSOAP, so please read about this there. 

---

<a name="the-example"></a>
### The example
In this example, I demonstrate how to retrieve a snapshot URI from an ONVIF complaint IP camera and download it locally.
I've done all the `wsdl2h` and `soapcpp2` stuff described above, just so *you* don't have to do it. 

The code is well commented and should be easy to read.

#### Prerequisites
You will need [curl](https://curl.haxx.se/) to build the app.


#### Usage
Clone the example code from [github](https://github.com/Sufi-Al-Hussaini/onvif-gsoap-by-example):

```
git clone https://github.com/Sufi-Al-Hussaini/onvif-gsoap-by-example
```

Now, all you have to do is:
```
cd onvif-by-example
make
```

Run the program:
```
./ipconvif -cIp '<camera-ip>' -cUsr '<camera-username>' -cPwd '<camera-password>'
```

---

##### Credits
The base code for this project was adapted from the [gsoap-onvif github repo](https://github.com/tonyhu/gsoap-onvif) originally written by [tonyhu](https://github.com/tonyhu). 


##### References:

<a name="onvif-wikipedia">1. </a>[ Wikipedia: ONVIF](https://en.wikipedia.org/wiki/ONVIF)<br/>
<a name="soap-wikipedia">2. </a>[ Wikipedia: SOAP](https://en.wikipedia.org/wiki/SOAP)<br/>
<a name="gsoap-wikipedia">3. </a>[ Wikipedia: gSOAP](https://en.wikipedia.org/wiki/GSOAP)<br/>
<a name="gsoap-genivia">4. </a>[ Genivia: gSOAP overview](https://www.genivia.com/dev.html#overview)<br/>
