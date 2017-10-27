---
layout: post
title: ONVIF / gSOAP in C++ by example [Pt-2]
description: Demystifying ONVIF
date: '2017-10-27 21:09:21'
tags: onvif soap gsoap c++
permalink: onvif-gsoap-in-c-by-example-pt-2/
---

## Part 2 of the ONVIF - gSOAP - C++ series. <br/>This post attempts to demystify ONVIF by discussing the actual data exchange and gSOAP's role in it.

Apparently, [ONVIF / gSOAP in C++ by example [Pt-1]]({% post_url 2016-10-21-onvif-gsoap-in-c-by-example %}) is my most viewed post (which is quite unexpected). Looks like there's still a lot of interest in ONVIF - gSOAP, and getting started is still not easy.<br/>
That is why I decided to write this post. Here, I map the ONVIF protocol specifications to actual data exchange and explain the role of the gSOAP toolkit therein.
 
> <u>Note:</u><br/>
> This post is a sequel of [ONVIF / gSOAP in C++ by example [Pt-1]]({% post_url 2016-10-21-onvif-gsoap-in-c-by-example %}) and makes multiple references to the [example code used there]({% post_url 2016-10-21-onvif-gsoap-in-c-by-example %}#the-example). 

***

## Enable debugging
To begin with, we need to enable debugging and message logging for the example application. This is done by defining the `DEBUG` macro, either in a source file or in the `Makefile` by adding `-DDEBUG` to `CPPFLAG`s.<br/>
```make
CPPFLAG = ... -DWITH_DOM -DWITH_OPENSSL -DDEBUG ...
```
Now, rebuild the application and run it. This will create three logfiles, namely - 
* SENT.log 
* RECV.log
* TEST.log
<br/><br/>
For this post, we are mostly concerned with the `SENT.log` and `RECV.log`, which contain the SOAP content sent and received by the application respectively.

***

## Decoding the logs
Open up the `SENT.log` file and you'll see lots of XML stuff. If you've worked with HTTP and REST services before, it should look quite familiar.<br/>
Below, I have separated out a part of this file which represents a single sent request.

```xml
POST /onvif/device_service HTTP/1.1
Host: 192.168.1.53
User-Agent: gSOAP/2.8
Content-Type: application/soap+xml; charset=utf-8; action="http://www.onvif.org/ver10/device/wsdl/GetDeviceInformation"
Content-Length: 2447
Connection: close
SOAPAction: "http://www.onvif.org/ver10/device/wsdl/GetDeviceInformation"

<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://www.w3.org/2003/05/soap-envelope" xmlns:SOAP-ENC="http://www.w3.org/2003/05/soap-encoding" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:wsa="http://schemas.xmlsoap.org/ws/2004/08/addressing" xmlns:wsdd="http://schemas.xmlsoap.org/ws/2005/04/discovery" xmlns:chan="http://schemas.microsoft.com/ws/2005/02/duplex" xmlns:wsa5="http://www.w3.org/2005/08/addressing" xmlns:netrm="http://schemas.microsoft.com/ws/2006/05/rm" xmlns:wsrm="http://docs.oasis-open.org/ws-rx/wsrm/200702" xmlns:c14n="http://www.w3.org/2001/10/xml-exc-c14n#" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" xmlns:xenc="http://www.w3.org/2001/04/xmlenc#" xmlns:wsc="http://schemas.xmlsoap.org/ws/2005/02/sc" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:xmime="http://tempuri.org/xmime.xsd" xmlns:xop="http://www.w3.org/2004/08/xop/include" xmlns:tt="http://www.onvif.org/ver10/schema" xmlns:wsrfbf="http://docs.oasis-open.org/wsrf/bf-2" xmlns:wstop="http://docs.oasis-open.org/wsn/t-1" xmlns:wsrfr="http://docs.oasis-open.org/wsrf/r-2" xmlns:ns1="http://www.onvif.org/ver10/advancedsecurity/wsdl" xmlns:tdn="http://www.onvif.org/ver10/network/wsdl" xmlns:tds="http://www.onvif.org/ver10/device/wsdl" xmlns:tev="http://www.onvif.org/ver10/events/wsdl" xmlns:wsnt="http://docs.oasis-open.org/wsn/b-2" xmlns:timg="http://www.onvif.org/ver20/imaging/wsdl" xmlns:tmd="http://www.onvif.org/ver10/deviceIO/wsdl" xmlns:tptz="http://www.onvif.org/ver20/ptz/wsdl" xmlns:trt="http://www.onvif.org/ver10/media/wsdl"><SOAP-ENV:Header><wsse:Security SOAP-ENV:mustUnderstand="true"><wsu:Timestamp wsu:Id="Time"><wsu:Created>2017-10-24T10:48:41Z</wsu:Created><wsu:Expires>2017-10-24T10:48:51Z</wsu:Expires></wsu:Timestamp><wsse:UsernameToken><wsse:Username>admin</wsse:Username><wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordDigest">P8tZAba/bPqWRkOe9f8zIh23NyQ=</wsse:Password><wsse:Nonce>iRrvWUB8sARgRnLD1mu/Yo4lzOc=</wsse:Nonce><wsu:Created>2017-10-24T10:48:41Z</wsu:Created></wsse:UsernameToken></wsse:Security></SOAP-ENV:Header><SOAP-ENV:Body><tds:GetDeviceInformation></tds:GetDeviceInformation></SOAP-ENV:Body></SOAP-ENV:Envelope>
```

That above is a [HTTP POST request](https://en.wikipedia.org/wiki/POST_(HTTP)). In fact, you could use a browser based REST client (like Postman for Chrome) to send it to an ONVIF complaint camera and it should work in exactly the same way. The blank line separates the header from the request body. The header consists of key-value pairs separated by `:`.<br/>
There's a lot of information in the body and we'll come back to that in a moment, but atleast we now know what it means when someone says SOAP uses application layer protocols (like HTTP here).

### The request header
Note the `SOAPAction` key and its value, specifically the `GetDeviceInformation` part. This means that this is a `GetDeviceInformation` request. 
Open up the `SOAPAction` URL (http://www.onvif.org/ver10/device/wsdl) in a browser and you'll see it redirects to `https://www.onvif.org/ver10/device/wsdl/devicemgmt.wsdl`. This is a WSDL file which is part of the actual ONVIF specification. 

<img src="https://user-images.githubusercontent.com/7275476/32126472-ed01c9d0-bb81-11e7-81e3-b37b0979f39c.png" width="100%">

The browser actually formats WSDL files, so what you're seeing is probably the formatted version. View the page source and you'll see that WSDL is actually very much like XML; infact it *is* based on XML.
Search for `GetDeviceInformation` and you'll find this:

```xml
<!--===============================-->
<xs:element name="GetDeviceInformation">
	<xs:complexType>
		<xs:sequence/>
	</xs:complexType>
</xs:element>
<xs:element name="GetDeviceInformationResponse">
	<xs:complexType>
		<xs:sequence>
			<xs:element name="Manufacturer" type="xs:string">
				<xs:annotation>
					<xs:documentation>The manufactor of the device.</xs:documentation>
				</xs:annotation>
			</xs:element>
			<xs:element name="Model" type="xs:string">
				<xs:annotation>
					<xs:documentation>The device model.</xs:documentation>
				</xs:annotation>
			</xs:element>
			<xs:element name="FirmwareVersion" type="xs:string">
				<xs:annotation>
					<xs:documentation>The firmware version in the device.</xs:documentation>
				</xs:annotation>
			</xs:element>
			<xs:element name="SerialNumber" type="xs:string">
				<xs:annotation>
					<xs:documentation>The serial number of the device.</xs:documentation>
				</xs:annotation>
			</xs:element>
			<xs:element name="HardwareId" type="xs:string">
				<xs:annotation>
					<xs:documentation>The hardware ID of the device.</xs:documentation>
				</xs:annotation>
			</xs:element>
		</xs:sequence>
	</xs:complexType>
</xs:element>
<!--===============================-->

<!--             ...               -->
<wsdl:operation name="GetDeviceInformation">
	<wsdl:documentation>This operation gets basic device information from the device.</wsdl:documentation>
	<wsdl:input message="tds:GetDeviceInformationRequest"/>
	<wsdl:output message="tds:GetDeviceInformationResponse"/>
</wsdl:operation>
```

So the ONVIF protocol specification can be thought of as an interface for requests & responses that complaint devices must implement. The protocol also specifies the data contained in these requests and responses. In the above example, the request (or input) has no attributes, while the response (or output) has `Manufacturer`, `Model`, `FirmwareVersion`, `SerialNumber` and `HardwareId`.

### The request body
Here's a prettified version of the body:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://www.w3.org/2003/05/soap-envelope" 
xmlns:SOAP-ENC="http://www.w3.org/2003/05/soap-encoding" 
xmlns:c14n="http://www.w3.org/2001/10/xml-exc-c14n#" 
xmlns:chan="http://schemas.microsoft.com/ws/2005/02/duplex" 
xmlns:ds="http://www.w3.org/2000/09/xmldsig#" 
xmlns:netrm="http://schemas.microsoft.com/ws/2006/05/rm" 
xmlns:ns1="http://www.onvif.org/ver10/advancedsecurity/wsdl" 
xmlns:tdn="http://www.onvif.org/ver10/network/wsdl" 
xmlns:tds="http://www.onvif.org/ver10/device/wsdl" 
xmlns:tev="http://www.onvif.org/ver10/events/wsdl" 
xmlns:timg="http://www.onvif.org/ver20/imaging/wsdl" 
xmlns:tmd="http://www.onvif.org/ver10/deviceIO/wsdl" 
xmlns:tptz="http://www.onvif.org/ver20/ptz/wsdl"
xmlns:trt="http://www.onvif.org/ver10/media/wsdl" 
xmlns:tt="http://www.onvif.org/ver10/schema" 
xmlns:wsa="http://schemas.xmlsoap.org/ws/2004/08/addressing" 
xmlns:wsa5="http://www.w3.org/2005/08/addressing" 
xmlns:wsc="http://schemas.xmlsoap.org/ws/2005/02/sc" 
xmlns:wsdd="http://schemas.xmlsoap.org/ws/2005/04/discovery" 
xmlns:wsnt="http://docs.oasis-open.org/wsn/b-2" 
xmlns:wsrfbf="http://docs.oasis-open.org/wsrf/bf-2" 
xmlns:wsrfr="http://docs.oasis-open.org/wsrf/r-2" 
xmlns:wsrm="http://docs.oasis-open.org/ws-rx/wsrm/200702" 
xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" 
xmlns:wstop="http://docs.oasis-open.org/wsn/t-1" 
xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" 
xmlns:xenc="http://www.w3.org/2001/04/xmlenc#" 
xmlns:xmime="http://tempuri.org/xmime.xsd" 
xmlns:xop="http://www.w3.org/2004/08/xop/include" 
xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
   <SOAP-ENV:Header>
      <wsse:Security SOAP-ENV:mustUnderstand="true">
         <wsu:Timestamp wsu:Id="Time">
            <wsu:Created>2017-10-24T10:48:41Z</wsu:Created>
            <wsu:Expires>2017-10-24T10:48:51Z</wsu:Expires>
         </wsu:Timestamp>
         <wsse:UsernameToken>
            <wsse:Username>admin</wsse:Username>
            <wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordDigest">
            	P8tZAba/bPqWRkOe9f8zIh23NyQ=
            </wsse:Password>
            <wsse:Nonce>iRrvWUB8sARgRnLD1mu/Yo4lzOc=</wsse:Nonce>
            <wsu:Created>2017-10-24T10:48:41Z</wsu:Created>
         </wsse:UsernameToken>
      </wsse:Security>
   </SOAP-ENV:Header>
   <SOAP-ENV:Body>
      <tds:GetDeviceInformation />
   </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

As you can see, there's an envelope element (`SOAP-ENV:Envelope`) which contains a header element (`SOAP-ENV:Header`) and a body element (`SOAP-ENV:Body`).
The header provides the `Security` information including the `Username` and hashed `Password`.
Note that the body is comprised of a single `tds:GetDeviceInformation` self closing tag, this is just as the specification required.

***

## The code
Now here's the relevant code that generated the above request:
```c++
struct soap *soap = soap_new();
if (SOAP_OK != soap_wsse_add_UsernameTokenDigest(proxyDevice.soap, NULL, "admin", "admin"))
	return -1;

if (SOAP_OK != soap_wsse_add_Timestamp(proxyDevice.soap, "Time", 10)) 
	return -1;

// Get Device info
_tds__GetDeviceInformation *tds__GetDeviceInformation = soap_new__tds__GetDeviceInformation(soap, -1);
_tds__GetDeviceInformationResponse *tds__GetDeviceInformationResponse = soap_new__tds__GetDeviceInformationResponse(soap, -1);

if (SOAP_OK != proxyDevice.GetDeviceInformation(tds__GetDeviceInformation, tds__GetDeviceInformationResponse))
    return -1;
```

So, no serialization/deserialization, no evident RPC calls, no network programming and socket stuff and no need to manually maintain data structures. 

***

## gSOAP
It all just works because `gSOAP` is handling everything for us. It generated the boiler-plate code for translating the specs to data structures and functions, it did all the XML data binding, it wrapped up all the memory handling and it also handles the tcp socket communication. Isn't that cool?

All this was done by the `wsdl2h` and `soapcpp2` tools. Each ONVIF operation specified in the WSDL maps to a function and associated data structures. 
See for example the respective code for `GetDeviceInformation`:
```c++
// soapStub.h : line 28181 onwards
#ifndef SOAP_TYPE___tds__GetDeviceInformation
#define SOAP_TYPE___tds__GetDeviceInformation (2381)
/* Operation wrapper: */
struct __tds__GetDeviceInformation
{
public:
	_tds__GetDeviceInformation *tds__GetDeviceInformation;	/* optional element of type tds:GetDeviceInformation */
public:
	int soap_type() const { return 2381; } /* = unique type id SOAP_TYPE___tds__GetDeviceInformation */
};
#endif

// ...

// onvif.h : line 41822 onwards
//gsoap tds  service method-protocol:	GetDeviceInformation_ SOAP
//gsoap tds  service method-style:	GetDeviceInformation_ document
//gsoap tds  service method-encoding:	GetDeviceInformation_ literal
//gsoap tds  service method-input-action:	GetDeviceInformation_ http://www.onvif.org/ver10/device/wsdl/GetDeviceInformation
//gsoap tds  service method-output-action:	GetDeviceInformation_ http://www.onvif.org/ver10/device/wsdl/GetDeviceInformationResponse
int __tds__GetDeviceInformation_(
    _tds__GetDeviceInformation*         tds__GetDeviceInformation,	///< Input parameter
    _tds__GetDeviceInformationResponse* tds__GetDeviceInformationResponse	///< Output parameter
);

// ...

// soapH.h : line 36170 onwards
inline _tds__GetDeviceInformation * soap_new__tds__GetDeviceInformation(struct soap *soap, int n = -1) { return soap_instantiate__tds__GetDeviceInformation(soap, n, NULL, NULL, NULL); }
```

So, to use an ONVIF function, all we need to do is to look for the associated data structure, fill it and use it with the associated API. 

***

## Footnotes

* [Genivia](https://www.genivia.com) has some very nice resources to learn gSOAP. In particular, their [SOAP documentation](https://www.genivia.com/doc/soapdoc2.html) is very helpful.
* Taking the time to read `onvif.h` would be a good idea.

