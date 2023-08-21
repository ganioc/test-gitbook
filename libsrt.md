## What is libsrt?

## Problem facing
Gstreamer 1.14.4 doesnt support streamid property in its srtserversink plugin.

### libsrt way of handling streamid

It seems to pass it to srt_setsockopt() function.

SRT doesn't do URI parsing at all, only describe the socket option, doesn't mention the URI.

It is a valid case to set a socket option of string type to a string value containing printable character. It is a valid case to call srt_setsockflag (or srt_setsockopt) function using SRT_STREAMID, as well as extract this option's value from the socket.

The srt socket option, 

srt-live-transmit application,

socket-groups.md,

srt-test-multiplex,

GStreamer application is using a different approach, 
```shell
srtserversink uri=srt://xx.xx.xx.xx:8080 latency=100
```

VLC and FFMPEG both support the use case, support setting streamid by option. 

### SRT API Socket Options
Support int32, int64, bool, string, linger,

SRT C API srt.h, is based on the legacy UDT API,

srt-live-transmit, srt-file-transmit,

1. setup and teardown,
   srt_startup(),
   srt-cleanup(),
2. Creating and Destroying a Socket,
   SRT socket, 
   srt_create_socket(),
   srt_close(STRSocket s)
   Use UDP socket underlying, 
3. Binding and Connecting
   srt_bind(SRTSOCKET u, struct sockaddr* name, int namelen)
   srt_bind_acquire(SRTSOCKET u, UDPSOCKET updsock)
   srt_listen()
   srt_accept()
   srt_connect()
   srt_connect_debug()
4. Rendezvous 
   SRTO_RENDEZVOUS flag, connect to rendezvous counterpart
   lsa, local ip/port,
   rsa, remote ip/port,
   srt_setsockopt(m_sock,0,SRTO_RENDEZVOUS, &yes, sizeof yes)
   srt_bind()
   srt_connect()
   HandleConnection(sock)
5. Sending and receiving,
   srt_send(),
   srt_recv(),
   srt_sendmsg(),
   srt_recvmsg(),
6. srt epoll
   srt_epoll_wait()
   srt_epoll_uwait()



### Some interesting points from Haivision github repo


    Accept URIs with standard encoding - in this case, e.g. 
    srt://example.com:9000?streamid=%23%21%3A%3Au=admin,r=foo [EDIT: fixed]

### srt source code, version 1.14.4
https://gitlab.freedesktop.org/gstreamer/gst-plugins-bad/-/tree/1.14.4

gstsrtclientsink.c

- gst_srt_client_sink_set_property()
  - priv->poll_timeout
  - priv->bind_address
  - 
- gst_srt_client_sink_start
  - gst_srt_client_connect_full

### srt source code, version 1.19
srt_params

srt_options[]

```shell
gst_srt_object_set_common_params()
    call srt_setsockopt()
    gst_srt_object_apply_socket_option()
        call function below, 
        srt_setsockopt()

gst_srt_object_set_socket_option()
    gst_srt_object_set_string_value()


```


### How to change the plugin code?


### Materials

SRT and RIST are recent streaming protocols, a growing list of alternatives to RTMP( among wich one can find webrtc, warp/quit etc.)

SRT can recover from packet loss up to range of 15%, RIST up to 50% packet loss. 