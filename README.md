# nx-libmodbus
An implementation of the Modbus library for the switch (https://github.com/stephane/libmodbus)

The documentation can be found here: https://libmodbus.org/

# Dependencies
This library uses libnx (https://github.com/switchbrew/libnx)

# Example
Here is a small example that allows you to read the registers 0 to 10
```C
#include <stdio.h>
#include <stdlib.h>
#include <switch.h>
#include <modbus.h>
int main(int argc, char *argv[])
{
    u16 msgresp[10] = {0x00};
    int len = 0;
    modbus_t *ctx = NULL;
    consoleInit(NULL);

    // Configure our supported input layout: a single player with standard controller styles
    padConfigureInput(1, HidNpadStyleSet_NpadStandard);

    // Initialize the default gamepad (which reads handheld mode inputs as well as the first connected controller)
    PadState pad;
    padInitializeDefault(&pad);
    socketInitializeDefault(); // Initialize sockets
    //Connect to a modbus server with the address 192.168.1.1:502
    ctx = modbus_new_tcp("192.168.1.1", 502);

    if (modbus_connect(ctx) == -1)
    {
        printf("Fail\n");
    }

    while (appletMainLoop())
    {
        // Scan the gamepad. This should be done once for each frame
        padUpdate(&pad);
        // padGetButtonsDown returns the set of buttons that have been
        // newly pressed in this frame compared to the previous one
        u64 kDown = padGetButtonsDown(&pad);
        if (kDown & HidNpadButton_Plus)
        {
          break; // break in order to return to hbmenu
        }
        
        len = modbus_read_registers(ctx, 0, 10, msgresp);
        for (int i = 0; i < len; i++)
        {
            printf("%d\n", msgresp[i]);
        }
     // Update the console, sending a new frame to the display
        consoleUpdate(NULL);
    }
    modbus_close(ctx);
    modbus_free(ctx);
    socketExit(); // Cleanup
    // Deinitialize and clean up resources used by the console (important!)
    consoleExit(NULL);
    return 0;
}
```
