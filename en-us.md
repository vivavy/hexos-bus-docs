# HexOS GBus - HexOS Global Bus

## What is it?

Hexos Global Bus is a global bus system for the HexOS platform, used to communicate between different entities of OS.

## How to use it?

You can put intents onto the bus queue, and OS will take care of the rest.

## local buses

You can create local buses, and put intents onto them. You can append several entities to the bus, and OS will broadcast the intent to all of them.

### Creating a local bus

```cpp
/*/
/* Simple example of a LocalBus broadcaster
/*/

#include <hexos-consts>
#include <hexos-bus>

hexos::bus::LocalBus lbus;

int main() {
    lbus.init("dbus-name");
    lbus.count(2); // accept maximum 2 entities (including the initiator)
    lbus.require(2); // require at least 2 entities (including the initiator)
    lbus.accept(); // accept entities
    lbus.wait(); // wait for the entities to be connected

    // put an intent on the bus
    lbus.put(hexos::bus::Intent(HEXOS_BROADCAST_TEXT, "Hello world!")); // broadcast a text message to all entities
    lbus.put(hexos::bus::Intent(HEXOS_BROADCAST_COMMAND, "show textbox")); // broadcast a command to all entities

    lbus.put(hexos::bus::Intent(HEXOS_DISCONNECT)); // disconnect the bus
    lbus.wait(); // wait for all entities to disconnect
    lbus.destroy(); // destroy the bus
}

```

```cpp
/*/
/* Simple example of a LocalBus receiver
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>
#include <string>

hexos::bus::LocalBus lbus;
std::string message;


int main() {
    lbus.connect("dbus-name"); // connect to the bus

    if (!lbus.success) {
        // error
        return -1;
    }

    while (true) {
        hexos::bus::Intent intent = lbus.get(); // get an intent from the bus
        if (intent.type == HEXOS_BROADCAST_TEXT)
            // handle a text message
            message = intent.data;
        else if (intent.type == HEXOS_BROADCAST_COMMAND)
            // handle a command message
            if (std::string(intent.data) == "show textbox")
                // show a textbox
                hexos::gui::showTextbox(message);
        else if (intent.type == HEXOS_DISCONNECT)
            // handle a disconnect message
            break;
    }

    lbus.close(); // close connection to the bus
}

```

## virtual buses (Operating System-hosted queues)

You can create virtual buses, and put any data onto them. Anyone who have a connection to the bus can get the data. by default, host process has access to the bus.

### Creating a virtual bus

```cpp
/*/
/* Simple example of a VirtualBus broadcaster and receiver
/*/

#include <hexos-consts>
#include <hexos-bus>

hexos::bus::VirtualBus vbus;

int main() {
    vbus.init("dbus-name");

    vbus.put("Hello world!"); // broadcast strings
    vbus.put("Blah blah blah");
    vbus.put("Hi there!");

    while (vbus.notEmpty()) {
        std::string message = vbus.get(); // get a string from the bus
        hexos::gui::showTextbox(message);
    }

    vbus.close(); // close the bus
}

```

## errors

When you initialize a bus, you can get an error, and there is a way to properly handle it.

```cpp
/*/
/*  We want to create a bus with a reserved name
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::VirtualBus vbus;

int main() {
    vbus.init("osgbus0"); // try to create a bus with this name
    if (!vbus.success) {
        // error
        hexos::gui::showTextbox("Error: " + std::string(vbus.errorMessage));
        return -1;
    }
}

```

will show a textbox with the error message:

```
Error: Bus with name "osgbus0" already exists
```

You have several consts to check for errors:

```cpp
// hexos-const part
#define HEXOS_BUS_ERROR_NONE 0
#define HEXOS_BUS_ERROR_NAME_EXISTS 1
#define HEXOS_BUS_ERROR_ALREADY_CONNECTED 2
#define HEXOS_BUS_ERROR_NOT_CONNECTED 3
#define HEXOS_BUS_ERROR_INVALID_DATA 4
#define HEXOS_BUS_ERROR_VIOLATION 5
#define HEXOS_BUS_ERROR_UNKNOWN 6
#define HEXOS_BUS_ERROR_TIMEOUT 7
```

## OS-wide Gbus

OS-wide Gbus is a special bus that is created by the OS, and is used to communicate between different processes and system API. It is not a virtual bus, and it is not created by the user.

### Connecting to OS-wide Gbus

```cpp
/*/
/*  We want to connect to OS-wide Gbus
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // connect to OS-wide Gbus

    if (!gbus.success) {
        // error
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    // we want to broadcast a message to absolutely everyone in the system
    gbus.put(HEXOS_BROADCAST_TEXT, "Hello world!", append_credentials = true);
    /*                                             ^^^^^^^^^^^^^^^^^^
     *                                             append credentials
     *                                             to the message
     *
     *                                             this will add the
     *                                             sender's type and
     *                                             ID to the message
     */

    gbus.close(); // close the connection
}

```

### Receiving messages from OS-wide Gbus

```cpp
/*/
/*  We want to receive messages from OS-wide Gbus
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // connect to OS-wide Gbus

    if (!gbus.success) {
        // error
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // get an intent from the bus
        if (intent.type == HEXOS_BROADCAST_TEXT)
            // handle a text message
            hexos::gui::showTextbox(intent.data);
        else if (intent.type == HEXOS_DISCONNECT)
            // handle a disconnect message
            break;
    }

    gbus.close(); // close the connection
}

```

## diffence between syscall system API (HWSAPI) and Gbus system API (SWSAPI)

### syscall system API (HWSAPI)

HWSAPI is a *HardWare System API*, it uses `syscall` instruction to communicate with kernel.

```cpp
/*/
/*  We want to call system function
/*/

#include <hexos-consts>
#include <hexos-assembly>

int main() {
    hexos::assembly::syscall(HEXOS_DEBUG_OUT, "Hello world!");
}

////////////////////  ALTERNATIVE  ////////////////////
/*/
/*  We want to call system function
/*/

#include <hexos-debug>

int main() {
    // hexos::internals::SWSAPI is false by default
    hexos::debug::out("Hello world!");
}

```

### Gbus system API (SWSAPI)

SWSAPI is a *SoftWare System API*, it uses Gbus system API to communicate with kernel.

For ones who interested what the difference between direct HWSAPI access to functions and software-wrapped HWSAPI,
Gbus doesn't use `syscall` instruction, it mapped to static RAM address as GPIO.

```cpp
/*/
/*  We want to call system function using Gbus system API
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.connect(); // connect to OS-wide Gbus

    if (!gbus.success) {
        // error
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    gbus.put(HEXOS_EXECSWSAPI, HEXOS_DEBUG_OUT, "Hello world!");

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // get an intent from the bus
        if (intent.type == HEXOS_DISCONNECT)
            // handle a disconnect message
            break;
    }

    gbus.close(); // close the connection
}

```

Also, using HWSAPI, you can't broadcast info between different system entities, but using Gbus, you can.
So, if you want to use, for instance, debugger, you can set up a Gbus encryption to selectively broadcast debug info to only debugger,
and use SWSAPI instead of HWSAPI to communicate with kernel.

example:

```cpp
/*/
/*  We want to call system function using SWSAPI to debugging
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-internals>
#include <hexos-bus>
#include <hexos-debug>

hexos::bus::GlobalBus gbus;

int main() {
    hexos::internals::SWSAPI(true); // set SWSAPI flag to true

    gbus.setEncryptionKey("debug"); // set encryption key
    gbus.connect(); // connect to OS-wide Gbus

    if (!gbus.success) {
        // error
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    // we can use this form to represent system call, but will just set flag `SWSAPI` to true
    // gbus.put(HEXOS_EXECSWSAPI, HEXOS_DEBUG_OUT, "Hello world!");
    hexos::debug::out("Hello world!");

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // get an intent from the bus
        if (intent.type == HEXOS_DISCONNECT)
            // handle a disconnect message
            break;
    }

    gbus.close(); // close the connection
}

```

debugger code:

```cpp
/*/
/*  We want to call system function using SWSAPI to debugging
/*/

#include <hexos-consts>
#include <hexos-gui>
#include <hexos-bus>

hexos::bus::GlobalBus gbus;

int main() {
    gbus.setDecryptionKey("debug"); // set encryption key
    gbus.connect(); // connect to OS-wide Gbus

    if (!gbus.success) {
        // error
        hexos::gui::showTextbox("Error: " + std::string(gbus.errorMessage));
        return -1;
    }

    while (gbus.notEmpty()) {
        hexos::bus::Intent intent = gbus.get(); // get an intent from the bus
        hexos::debug::out(intent.data.repr()); // print the binary syscall representation
    }

    gbus.close(); // close the connection
}

```

## Gbus encryption

If you set EncryptionKey, all your messages will be encrypted with it, you can read all messages from the bus.
If you set DecryptionKey, you can only read messages that were encrypted with it, your messages will not be encrypted with it.

P.S.: You can set both keys at the same time, and use them to read and write messages. (This method is recommended to create
subenv-wide tunneled communication throw Gbus), but better to use local buses for this purpose.

Don't forget that you can create several Gbus clients in the same process, so you can access both global scope and encrypted local scope.
