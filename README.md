open62541
=========

An open-source communication stack implementation of OPC UA (OPC Unified Architecture) licensed under LGPL + static linking exception.

[![Ohloh Project Status](https://www.ohloh.net/p/open62541/widgets/project_thin_badge.gif)](https://www.ohloh.net/p/open62541)
[![Build Status](https://travis-ci.org/acplt/open62541.png?branch=master)](https://travis-ci.org/acplt/open62541)
[![Coverage Status](https://coveralls.io/repos/acplt/open62541/badge.png?branch=master)](https://coveralls.io/r/acplt/open62541?branch=master)
[![Coverity Scan Build Status](https://scan.coverity.com/projects/1864/badge.svg)](https://scan.coverity.com/projects/1864)

### What is currently working?
The project is in an early stage. We retain the right to break APIs until a first stable release.
- Binary serialization of all data types: 100%
- Node storage and setup of a minimal namespace zero: 100%
- Standard OPC UA Clients can connect/open a SecureChannel/open a Session: 100%
- Browsing/reading and writing attributes: 100%
- Advanced SecureChannel and Session Management: 75%

### Documentation
The developer documentation is generated from Doxygen annotations in the source code (http://open62541.org/doc).
Build instruction can be found under https://github.com/acplt/open62541/wiki/Building-open62541.

### Example Server Implementation
```c
#include <stdio.h>
#include <signal.h>

// provided by the open62541 lib
#include "ua_server.h"
#include "ua_namespace_0.h"

// provided by the user, implementations available in the /examples folder
#include "logger_stdout.h"
#include "networklayer_tcp.h"

UA_Boolean running = UA_TRUE;
void stopHandler(int sign) {
	running = UA_FALSE;
}

void serverCallback(UA_Server *server) {
    // add your maintenance functionality here
    printf("does whatever servers do\n");
}

int main(int argc, char** argv) {
	signal(SIGINT, stopHandler); /* catches ctrl-c */

    /* init the server */
	#define PORT 16664
	UA_String endpointUrl;
	UA_String_copyprintf("opc.tcp://127.0.0.1:%i", &endpointUrl, PORT);
	UA_Server *server = UA_Server_new(&endpointUrl, NULL);

    /* add a variable node */
    UA_Int32 myInteger = 42;
    UA_String myIntegerName;
    UA_STRING_STATIC(myIntegerName, "The Answer");
    UA_Server_addScalarVariableNode(server,
                 /* browse name, the value and the datatype's vtable */
                 &myIntegerName, (void*)&myInteger, &UA_TYPES[UA_INT32],
                 /* the parent node where the variable shall be attached */
                 &UA_NODEIDS[UA_OBJECTSFOLDER],
                 /* the (hierarchical) referencetype from the parent */
                 &UA_NODEIDS[UA_HASCOMPONENT]);

    /* attach a network layer */
	NetworklayerTCP* nl = NetworklayerTCP_new(UA_ConnectionConfig_standard, PORT);
	printf("Server started, connect to to opc.tcp://127.0.0.1:%i\n", PORT);

    /* run the server loop */
	struct timeval callback_interval = {1, 0}; // 1 second
	NetworkLayerTCP_run(nl, server, callback_interval, serverCallback, &running);
    
    /* clean up */
	NetworklayerTCP_delete(nl);
	UA_Server_delete(server);
    UA_String_deleteMembers(&endpointUrl);
	return 0;
}
```
