# WebRTC in a Nutshell
The purpose of this document is to provide an overview of WebRTC and a quick guide about how to get start with it. The document is based upon the documentation and information available on the 1st quarter of 2015.

[TOC]

<!--## Table of Contents
* [What is WebRTC](#What is WebRTC)
* [Browser Support](#Browser Support)
* [Application Workflow](Application Workflow)
* [Signaling and Session Control](Signaling and Session Control)
* [Creating a Peer Connection](Creating a Peer Connection)
* [Streaming Media](Streaming Media)
* [Data Channels](data_channels)
* [Desktop Sharing](desktop_sharing)
* [Libraries and Resources](libraries_and_resources)-->

#### What is WebRTC
WebRTC is a framework made up by several components (network, audio, video, etc.) that enables Real Time Communication between devices connected to internet. In the browser the components are exposed to developers through a Javascript API that simplifies and standardizes the access to a quite complex layer.
WebRCT enables peer to peer (P2P) video, audio and data communication between different devices. WebRTC is a browser (javascript) API for interacting with a local media stack, defined by W3C.

#### Browsers and Devices Support
When you are using a WebRTC enabled browser (Chrome, Firefox, Opera, Android Browser and Chrome for Android) you don't have to rely on any external plugin. There is a [plugin](https://temasys.atlassian.net/wiki/display/TWPP/WebRTC+Plugins) available to enable WebRTC in IE and Safari but there aren't many vendors that implement the tweaks needed to detect the plugin, making its usage pretty difficult and unpleasant. 
There is another possible solution, [named WebRCT 4 All](https://code.google.com/p/webrtc4all/), that seems more reliable and robust but it's just for Windows.
However, while Microsoft is planning to introduce the WebRTC API pretty soon in their [browser](https://status.modern.ie/webrtcobjectrtcapi?term=webrtc), Apple's plans regarding the introduction of the Javascript API in Safari aren't clear enough yet. 
In order to enable WebRTC on mobile devices you have to build the [native code](http://www.webrtc.org/native-code) and embed it in your application, currently iOS and Android are the only supported mobile platforms.

#### WebRTC Components and Application Workflow
WebRTC works by connecting two devices through the RTCPeerConnection. When two devices want to speak to each other, the first thing to do for them is to exchange contact information such as their IP addresses. In order to accomplish this goal the first thing to do is to setup a server (called STUN server) to help potential peers to locate each other.
Today's Internet configuration is pretty complex, clients share the same IP address, local networks have firewalls, most networks are behind a NAT, etc. In order to recover the device information in this complex scenario the STUN server uses the ICE protocol to find out and inform a peer of all the possible ways it can be reached on the Internet.  
Every potential way to connect to a device is called *ICE Candidates*, once the candidate is received it needs to be communicated to the other device(s). It is right now that a **Signaling Server** comes into the picture.
The purpose of the signaling server is to pass information between the peers at the start of the session. When a device wants to establish a connection, it generates a message called *offer* and sends it to a peer using the Signaling server. In the offer there are information about the device, such as which protocols it knows how to speak and which communication methods it knows.
The information are encoded using the [Session Description Protocol (SDP)](http://en.wikipedia.org/wiki/Session_Description_Protocol), the complexity of the SDP answers are hidden behind the Javascript API.

**Session Description Protocol format**

```
v=0
o=- 7614219274584779017 2 IN IP4 127.0.0.1
s=-
t=0 0
a=group:BUNDLE audio video
a=msid-semantic: WMS
m=audio 1 RTP/SAVPF 111 103 104 0 8 107 106 105 13 126
c=IN IP4 0.0.0.0
a=rtcp:1 IN IP4 0.0.0.0
a=ice-ufrag:W2TGCZw2NZHuwlnf
a=ice-pwd:xdQEccP40E+P0L5qTyzDgfmW
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
a=mid:audio
a=rtcp-mux
a=crypto:1 AES_CM_128_HMAC_SHA1_80 inline:9c1AHz27dZ9xPI91YNfSlI67/EMkjHHIHORiClQe
a=rtpmap:111 opus/48000/2
…
```

Let's quickly recap the steps to be performed to start a P2P connection using WebRTC:

* A device uses a Signaling server to relay an SDP offer to a potential peer
* The device asks an ICE server to find out how it can be reached from the public Internet
* The potential peer replies with its own connection offer
* The peers use the Signaling server to exchange ICE candidates, attempting to use them to open a connection
* Once a working ICE candidate is found the connection is established

First, the browsers are connected through signaling, by passing the Session Description Protocol (SDP). This signaling discovers where the two users are and how to connect. Once the RTCPeerConnection is open, video, audio, and data communication can be sent between the two browsers.

#### Signaling and Session Control
WebRTC signaling refers to the process of setting up, controlling, and terminating a communication session. In order for two endpoints to begin talking to one another, three types of information must be exchanged:

* *Session control information* determines when to initialize, close, and modify communications sessions; session control messages are also used in error reporting
* *Network Data* reveals where endpoints are located on the Internet (IP address and port) so that callers can find callees
* *Media Data* is required to determine the codecs and media types that the callers and callees have in common.

Signaling is not part of the WebRTC specification, the responsibility to design this layer of the app is left entirely to the developer. The signaling server can be completely custom or can be built using one of the several open source solutions available. An interesting implementation of the Signaling server is the one provided by [RTC-IO](https://github.com/rtc-io/rtc-signal) or the one provided by [Easy-RTC](https://github.com/priologic/easyrtc/blob/master/docs/easyrtc_client_tutorial.md), Easy-RTC is a complete suite of tools and [examples](http://demo.easyrtc.com/demos/index.html) that can be used to quick start with WebRTC.

#### Creating a PeerConnection
The `RTCPeerConnection` Javascript API is the main tool to use in order to implement the steps previously described to start a P2P connection between two devices. The first thing to do in order to use successfully the `RTCPeerConnection` API is to handle possible prefixes manually by defining a polyfill script for the `RTCPeerConnection`:
```
var PeerConnection = window.RTCPeerConnection || window.mozRTCPeerConnection || window.webkitRTCPeerConnection;
```
`PeerConnection` is now the constructor function to use when creating a new `RTCPeerConnection` object, as arguments it's possible to specify a list of ICE servers in order to create properly the offer message
```
var pc = new PeerConnection({ "iceServers": [{ "url": "stun:stun.l.google.com:19302" }, { "url": "stun:stun.services.mozilla.com" }]});
```
Once the `RTCPeerConnection` is instantiated the next step is to access the local media objects and prepare a complete offer to send. The Javascript API to use to access the local media is `getUserMedia`, however this API is prefixed in the different browsers. In order to properly handle the `getUserMedia` API another polyfill is needed in order to determine the *string* to then use it with the `navigator` object

```
var GET_USER_MEDIA = navigator.getUserMedia ? "getUserMedia" :
                     navigator.mozGetUserMedia ? "mozGetUserMedia" :
                     navigator.webkitGetUserMedia ? "webkitGetUserMedia" : "getUserMedia";
```

In order to access the `getUserMedia` API is now enough to use `navigator[GET_USER_MEDIA]` passing to the function the 
appropriate restrictions

```
navigator[GET_USER_MEDIA]({video: true, audio: true}, onSuccess, onError);
```

It's in the `onScuccess` handler that the offer is created because it's only at that time that the capabilities available for this connection are clear. The `createOffer` method defined on the `RTCPeerConnection` object accepts as arguments a success and an error handler, in the success handler a new `RTCSessionDescription` object is created. In order to be able to run the code on different browsers another polyfill is required
```
var SessionDescription = window.RTCSessionDescription || window.mozRTCSessionDescription || window.webkitRTCSessionDescription;
```
It's now possible to create a new description of the session using the SDP protocol and send it to the other device(s) involved in the P2P connection

```
pc.createOffer(function(offer) {
    pc.setLocalDescription(new SessionDescription(offer), function() {
    ....
```

Following the complete script, it can be used with Chrome, Firefox, Opera and other WebRTC compliant browsers

```
var PeerConnection = window.RTCPeerConnection || window.mozRTCPeerConnection || window.webkitRTCPeerConnection;
var SessionDescription = window.RTCSessionDescription || window.mozRTCSessionDescription || window.webkitRTCSessionDescription;
var GET_USER_MEDIA = navigator.getUserMedia ? "getUserMedia" :
                     navigator.mozGetUserMedia ? "mozGetUserMedia" :
                     navigator.webkitGetUserMedia ? "webkitGetUserMedia" : "getUserMedia";

var pc = new PeerConnection({ "iceServers": [{ "url": "stun:stun.l.google.com:19302" }, { "url": "stun:stun.services.mozilla.com" }]});

navigator[GET_USER_MEDIA]({video: true, audio: true}, function(stream) {

  pc.onaddstream = onaddstream;
  // Adding a local stream won't trigger the onaddstream callback
  pc.addStream(stream);

  pc.createOffer(function(offer) {
    pc.setLocalDescription(new SessionDescription(offer), function() {
      // send the offer to a server to be forwarded to the friend you're calling
      console.log(offer);
    }, localDescriptionError);
  }, createOfferError);
}, function(error) {
     console.log("The following error occurred: " + error.error);
   });

function localDescriptionError (error){

  console.log('localDescriptionError', arguments)

}

function createOfferError (error){

  console.log('createOfferError', arguments)

}

function onaddstream(event) {
  if (event) {
    console.log(event.stream)
  }
}
```

#### Streaming Audio and Video
In order to render audio and video from a connected peer it's mandatory to add to the page two `video` elements. Doing this the user can see his/her camera and the connected peer audio/video stream.
When the page load it's pretty simple to add the required video elements to the page and store them in two variables

```
var localVideo = document.createElement('video');
var remoteVideo = document.createElement('video');

document.body.appendChild(localVideo);
document.body.appendChild(remoteVideo);
```

In order to render the local video it's enough to modify the success handler sepcified as an argument in the `getUserMedia` call and assign to the local video a new source
```
localVideo.src = window.URL.createObjectURL(stream);
```
The last step required to render the remote video is to modify the `onaddstream` function in order to set up the appropriate source for the remote video elements

```
function onaddstream(event) {
  if (event) {
    remoteVideo.src = window.URL.createObjectURL(event.stream);
  }
}
```

A pretty simple demo is available [here](https://shanetully.com/2014/09/a-dead-simple-webrtc-example/) and a more detailed one is available [here](http://blog.carbonfive.com/2014/10/16/webrtc-made-simple/)
#### Data Channels
Currently data channels are supported in Google Chrome, Firefox & Opera, using Data Channels is possible to exchange data between connected devices.
The `RTCDataChannel` API are pretty different from *JSON*, *WebSockets* and *Server Events* in fact it works with the RTCPeerConnection API and therefore enables a low latency P2P data exchange.
The `RTCDataChannel` API uses the *Stream Control Transmission Protocol* , shorten [SCTP](http://en.wikipedia.org/wiki/Stream_Control_Transmission_Protocol), allowing configurable delivery semantics such as *out-of-order delivery* and *retransmit configuration*.
The `RTCDataChannel` API supports a flexible set of data types like strings as well as some of the binary types in JavaScript such as Blob, ArrayBuffer and ArrayBufferView. These types can be helpful when working with file transfer and multiplayer gaming.
In order to create a data channel connection it's mandatory to initiate a P2P connection first, once the connection is successfully established it's possible to create a channel with the desired options

```
// Establish your peer connection using your signaling channel here
var dataChannel = pc.createDataChannel('dataChannelName', {
  										ordered: false, // do not guarantee order
 					 					maxRetransmitTime: 3000, // in milliseconds
									});
```
Once the data channel is created it's possible to define different handlers in order to manage every single event that happens during the data exchange such as `dataChannel.onerror`, `dataChannel.onmessage`, `dataChannel.onopen` and `dataChannel.onclose`.
A very complete demo about Data Channels usage in Firefox is available [here](http://mozilla.github.io/webrtc-landing/data_test.html), [here](https://fosterelli.co/file/getting-started-with-webrtc-data-channels/webrtc.html) instead there is an example working with Chrome. A possible polyfill is available on [GitHub](https://github.com/ShareIt-project/DataChannel-polyfill).
#### Desktop Sharing
Desktop sharing through the browser without any plugin is possible with the cutting edge relases of Chrome and Firefox checking the `enable-usermedia-screen-capture` in the browser's flags. For older versions of Chrome and Firefox there are two *open source* extensions available [here](https://www.webrtc-experiment.com/Pluginfree-Screen-Sharing/).
Once the flag is enabled it's possible to access the device screen setting the appropriate constraints when using the `getUserMedia` API, the `video` property can now also contain nested properties

```
navigator.getUserMedia({
	audio: false,
	video: {
   	mandatory: {
       	chromeMediaSource: 'screen',
       	maxWidth: 1280,
       	maxHeight: 720
   	},
   	optional: []
	}
}, function(stream) {
   //We've got media stream
 }, function() {
   //Error, no stream provided
 }
)
```

#### Conclusions 
2015 is the year of WebRTC, browsers are supporting the standard more consistently and Microsoft is moving forward in this direction. The most reasonable and cost effective approach is to use an open source library on the client and rely on existing backend infrastructures. The general approach used in most of the WebRTC SDKs is to offer an abstraction layer to simplify the usage of the WebRTC API.
Among the various available solutions the following are the most documented and relatively easy to use:

* [Twilio](http://www.twilio.com/client/api)
* [TokBox](http://tokbox.com/opentok)
* [Phono](http://phono.com/docs)
* [PeerJS](http://peerjs.com/docs/#api)
* [EasyRTC](http://easyrtc.com)
* [PubNub](http://www.pubnub.com/knowledge-base/)
* [Freeswitch](http://www.freeswitch.org/node/376)
* [HookFlash](http://hookflash.com)
* [OnSip](http://www.onsip.com)
* [SIPML5](http://sipml5.org)
* [webRTC.io](https://github.com/webRTC/webRTC.io)

In order to stay updated about WebRTC it's strongly recommended to follow [webrtchacks.com](https://webrtchacks.com/), [Mozilla Hacks](https://hacks.mozilla.org/category/webrtc) and the [updates web site](http://updates.html5rocks.com/) of the HTML5Rocks project.
Any feedback, suggestions or improvements related to this document are more than welcome!