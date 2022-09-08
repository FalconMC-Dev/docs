# Networking

## Packet pipeline

To guarantee performance, the packet pipeline should be one of the most optimized components of FalconMC.

### Receiving process
Shown below is a flowchart of the receiving end of a connection:

```plantuml
@startuml
start
repeat
if (Check <b>packet_len</b>) then (Some(len))
    :Read max from socket/
else (None)
    :Read 8kb from socket/
    :Set <b>packet_len</b> to length\nread from buffer;
    :Reserve <b>packet_len</b> bytes in buffer;
    if (Is <b>packet_len</b> > 8kb?) then (yes)
        :Set <b>is_large</b> to true;
    else (no)
        if (Is <b>is_large</b>?) then (yes)
            :Set <b>is_large</b> to false;
            :Shrink read buffer to 8kb;
        else (no)
        endif
    endif
endif
if (Buffer length longer\nthan <b>packet_len</b>?) then (yes)
    :Process <b>packet_len</b> bytes\n as packet;
    :Set <b>packet_len</b> to "None";
else (no)
endif
repeat while (Stop?) is (no)
->yes;
stop
@enduml
```
The theoretical maximum size of a packet seems to be 2 MB (maximum size that fits a 3-byte varint).
Achieving such a packet appears to be ultimately rare and because of that
it doesn't seem wise to just let buffers grow indefinitely. This approach keeps the size
of the buffers just right.

**Note**: After a packet has been processed, the raw bytes of that packet **must** be dropped.
Doing so will minimize the amount of reallocations required.

### Sending process
Shown below is a flowchart of the sending end of a connection:

```plantuml
@startuml
start
:Initiate packet send;
:Write packet to buffer;
repeat
if (Buffer is empty?) then (no)
    :Send data through socket/
    if (Length of buffer\n < 8kb) then (yes)
        :Shrink write buffer to 8kb;
    else (no)
    endif
else (yes)
endif
repeat while (Stop?) is (no)
->yes;
stop
@enduml
```
Like receiving (see above), the maximum size of a packet is about 2 MB. This approach keeps the size of the buffers just right.

