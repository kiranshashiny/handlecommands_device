Embedded C Client Library - Introduction
========================================

Embedded C client for interacting with the Internet of Things Foundation.

Dependencies
------------

1.  [Embedded C MQTT Client]

  [Embedded C MQTT Client]: http://www.eclipse.org/paho/clients/c/embedded/
  

Embedded C Client Library - Devices
===================================

*iotfclient* is client for the Internet of Things Foundation service.
You can use this client to connect to the service, publish events and
subscribe to commands.

Initialize
----------

There are 2 ways to initialize the *iotfclient*.

### Passing as parameters

The function *initialize* takes the following details to connect to the
IoT Foundation service

-   client - Pointer to the *iotfclient*
-   org - Your organization ID
-   type - The type of your device
-   id - The ID of your device
-   authmethod - Method of authentication (the only value currently
    supported is “token”)
-   authtoken - API key token (required if auth-method is “token”)

``` {.sourceCode .c}
#include "iotfclient.h"
   ....
   ....
   Iotfclient client;
   //quickstart
   rc = initialize(&client,"quickstart","iotsample","001122334455",NULL,NULL);
   //registered
   rc = initialize(&client,"orgid","type","id","token","authtoken");
   ....
```

### Using a configuration file

The function *initialize\_configfile* takes the configuration file path
as a parameter.

``` {.sourceCode .c}
#include "iotfclient.h"
   ....
   ....
   char *filePath = "./device.cfg";
   Iotfclient client;
   rc = initialize_configfile(&client, filePath);
   ....
```

The configuration file must be in the format of

``` {.sourceCode .}
org=$orgId
type=$myDeviceType
id=$myDeviceId
auth-method=token
auth-token=$token
```

Connect
-------

After initializing the *iotfclient*, you can connect to the Internet of
Things Foundation by calling the *connectiotf* function

``` {.sourceCode .c}
#include "iotfclient.h"
   ....
   ....
   Iotfclient client;
   char *configFilePath = "./device.cfg";

   rc = initialize_configfile(&client, configFilePath);

   if(rc != SUCCESS){
       printf("initialize failed and returned rc = %d.\n Quitting..", rc);
       return 0;
   }

   rc = connectiotf(&client);

   if(rc != SUCCESS){
       printf("Connection failed and returned rc = %d.\n Quitting..", rc);
       return 0;
   }
   ....
```

Handling commands ( these are commands being published from the application ) 
---------------------------------------------------------------------------

When the device client connects, it automatically subscribes to any
command for this device. To process specific commands you need to
register a command callback function by calling the function
*setCommandHandler*. The commands are returned as

-   commandName - name of the command invoked
-   format - e.g json, xml
-   payload

``` {.sourceCode .c}
#include "iotfclient.h"

void myCallback (char* commandName, char* format, void* payload)
{
   printf("The command received :: %s\n", commandName);
   printf("format : %s\n", format);
   printf("Payload is : %s\n", (char *)payload);
}
 ...
 ...
 char *filePath = "./device.cfg";
 rc = connectiotfConfig(filePath);
 setCommandHandler(myCallback);

 yield(1000);
 ....
```

**Note** : *yield* function must be called periodically to receive commands.


Disconnect Client
------------------

Disconnects the client and releases the connections

``` {.sourceCode .c}
#include "iotfclient.h"
 ....
 rc = connectiotf (org, type, id , authmethod, authtoken);
 char *payload = {\"d\" : {\"temp\" : 34 }};

 rc= publishEvent("status","json", payload , QOS0);
 ...
 rc = disconnect();
 ....
```


Compile on the Raspberry pi ( Mine was a Raspberry Pi Model B )
---------------------------

cd samples

make handlecommands

sudo ./handlecommands


A typical output when requests are received should look like:
`
pi@raspberrypi ~/nodetest/iot-handlecommands/samples $ sudo ./handlecommands 
 Calling connectiotf() 
Connecting to registered service with org 8ubmht
The auth token being used is [kg&z3L4pxKd?_BRtZm]
 before mqtt connecting 
Waiting for commands from application or the Bluemix IoT Foundation------------------------------------
The command received :: blink format string payload (char *)critical 
Payload received is : [critical]
  received critical, turning PIN HIGH
 ------------------------------------
------------------------------------
The command received :: blink format string payload (char *)safeical 
Payload received is : [safeical]
  received safe, turning Pin LOW
 ------------------------------------
`



Trouble spots and some resolutions:
----------------------------------
Often times I had difficulty connecting to the Bluemix IoT Foundation as shown.
Repeatedly trying it over again ( 1 or 2 times ) fixed it .


`
pi@raspberrypi ~/nodetest/iot-signal/samples $ sudo ./handlecommands 
 Calling connectiotf() 
Connecting to registered service with org 8ubmht
the auth token being used is [kg&z3L4pxKd?_BRtZm]
Connection failed and returned rc = -1.  <<<<<<<<<<<<<<<<<<
 Quitting..

pi@raspberrypi ~/nodetest/iot-signal/samples $ sudo ./handlecommands 
 Calling connectiotf() 
Connecting to registered service with org 8ubmht
the auth token being used is [kg&z3L4pxKd?_BRtZm]
`


The request from the application-> Bluemix -> to the device is not coming thru !
Check that you have set the yield call waiting for incoming requests.
