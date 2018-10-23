# Video calling using Agora's JavaScript SDK
Having video streaming features integrated inside an application can be very tedious and time-consuming. Maintaining a low-latency video server, load balancing, listening to end-user events(screen off, reload etc.) are some really painful hassles not to mention having cross-platform compatibility.

Feeling dizzy already? Dread not! Agora's SDKs integrate video calling features into your application and will get you up and running within a matter of minutes. Oh!, and all the video server details are abstracted away. In this tutorial, we'll write a bare-bones web application with video calling features using vanilla javascript and AgoraWebSDK.

Let's start by signing up with Agora.
Go ahead to https://dashboard.agora.io/en/signup to make an account and login to the dashboard.
Navigate to the project list tab under projects and create a new project by clicking the green button as shown in the below image.

Create a new project and retrieve the app id. This will be used while coding the application.
Structure
This would be the structure of the application that we are developing
```
.
├── index.html
├── scripts
│ ├── AgoraRTCSDK-2.4.0.js
│ └── script.js
└── styles
  └── style.css
```  
## index.html
The application's structure is very straightforward.
One can download the latest version of the SDK from agora's SDK downloads page and integrate it like it has been done the tutorial or use the CDN version by using this instead
`<script src="http://cdn.agora.io/sdk/web/AgoraRTCSDK-2.4-latest.js"></script>`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Video Call Demo</title>
    <link rel="stylesheet" href="./styles/style.css">
</head>
<body>
<h1>
    Video Call Demo<br><small style="font-size: 14pt">Powered by Agora.io</small>
</h1>

<h4>My Feed :</h4>
<div id="me"></div>

<h4>Remote Feeds :</h4>
<div id="remote-container">

</div>

<script src="scripts/AgoraRTCSDK-2.4.0.js"></script>
<script src="scripts/script.js"></script>
</body>
</html>
```


There is a container with id `me` which is supposed to contain the video stream of the local user (you).

There is a container with id `remote-container` to hold the video feeds of the other remote users in the channel.

We add some basic styling to our app.

```css
*{
    font-family: sans-serif;
}
h1,h4{
    text-align: center;
}
#remote-container video{
    height: auto;
    position: relative !important;
}
#me{
    position: relative;
    width: 50%;
    margin: 0 auto;
    display: block;
}
#me video{
    position: relative !important;
}
#remote-container{
    display: flex;
}
```


### Let's get to the interesting parts already!
**Script.js**
First, let's spring up some helper functions to handle errors and trivial DOM operations.

```javascript
/**
 * @name handleFail
 * @param err - error thrown by any function
 * @description Helper function to handle errors
 */
let handleFail = function(err){
    console.log("Error : ", err);
};

// Queries the container in which the remote feeds belong
let remoteContainer= document.getElementById("remote-container");

/**
 * @name addVideoStream
 * @param streamId
 * @description Helper function to add the video stream to "remote-container"
 */
function addVideoStream(streamId){
    let streamDiv=document.createElement("div"); // Create a new div for every stream
    streamDiv.id=streamId;                       // Assigning id to div
    streamDiv.style.transform="rotateY(180deg)"; // Takes care of lateral inversion (mirror image)
    remoteContainer.appendChild(streamDiv);      // Add new div to container
}
/**
 * @name removeVideoStream
 * @param evt - Remove event
 * @description Helper function to remove the video stream from "remote-container"
 */
function removeVideoStream (evt) {
    let stream = evt.stream;
    stream.stop();
    let remDiv=document.getElementById(stream.getId());
    remDiv.parentNode.removeChild(remDiv);
    console.log("Remote stream is removed " + stream.getId());
}
```

The architecture of Agora video calls:
<img src="https://cdn-images-1.medium.com/max/660/0*y57wxspSOKciu4Rx.jpg">

So what's the diagram all about? 

Let me break it down a bit.

Channels are something similar to chatrooms and every appid can spawn multiple channels.

Users can join and leave a channel.

We'll be implementing the mentioned methods inside script.js .

So first we need to create a client object by calling the AgoraRTC.createClient constructor. We would have to pass in the parameters to set the video encoding and decoding format and the mode.
```javascript
// Client Setup
// Defines a client for RTC
let client = AgoraRTC.createClient({
    mode: 'live',
    codec: "h264"
});
```


Next up we initialize the method with the appid which we obtained previously.
```javascript
// Client Setup
// Defines a client for Real Time Communication
client.init("<---Your AppId here--->",() => console.log("AgoraRTC client initialized") ,handleFail);
```


This leaves us ready to join a channel. Let's use the client.join method to do the same.
```javascript

client.join(null,"any-channel",null, (uid)=>{

    // publish the video

},handleFail);
```


We pass in null parameters so that server can dynamically assign a user id and because we are using channel name instead of channel-id.
Next up, we are going to publish our video feed into the channel. To do this, we will first create a stream object by calling the AgoraRTC.createStream constructor and passing appropriate parameters.
After this, we publish the stream to the channel as shown below:

```javascript

client.join(null,"dsc-video-demo",null, (uid)=>{

    // Stream object associated with your web cam is initialized
    let localStream = AgoraRTC.createStream({
        streamID: uid,
        audio: true,
        video: true,
        screen: false
    });

    // Associates the stream to the client
    localStream.init(function() {

        //Plays the localVideo
        localStream.play('me');

        //Publishes the stream to the channel
        client.publish(localStream, handleFail);

    },handleFail);

},handleFail);
```

Now we need to display the other users in the channel (remote users). And handle the view appropriately if somebody exits the video call. To achieve this, we use a series of event listeners and handlers.

```javascript
//When a stream is added to a channel
client.on('stream-added', function (evt) {
    client.subscribe(evt.stream, handleFail);
});
//When you subscribe to a stream
client.on('stream-subscribed', function (evt) {
    let stream = evt.stream;
    addVideoStream(stream.getId());
    stream.play(stream.getId());
});
//When a person is removed from the stream
client.on('stream-removed',removeVideoStream);
client.on('peer-leave',removeVideoStream);
```

Shazam! Now we can place a successful video call inside our application.
Note: When you try to deploy this web app (or any other which uses Agora SDK) make sure the server has an SSL certificate. i.e connection happens over https.
Full code is available at https://github.com/technophilic/Agora-demo-web
