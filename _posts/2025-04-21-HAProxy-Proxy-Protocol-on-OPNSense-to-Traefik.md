---
layout: post
title: OPNsense HAProxy Proxy Protocol to Traefik with original IP
date: 2025-04-21 11:33:00 -0700
categories: [Networking, ReverseProxy]
tags: [opnsense, splitdns, multisite, reverseproxy, traefik, haproxy, proxyprotocol, security, networking, privacy]
pin: false
image:
  path: /assets/img/header/header--opnsense--haproxy-proxy-protocol-multiple-sni-routing-splitdns.jpg
  alt: HAProxy on OPNsense can use the Proxy Protocol to send full header information to selective backends
---


# HAProxy proxy protocol section


Different backends based on URL.

Domain specific routing with selective backend endpoints. 

Using HAProxy to route traffic based on SNI headers. 

HAProxy to Traefik. 

HAProxy forward to Traefik.

Traefik Proxy Protocol v2 setup.

## Intro

The HAProxy proxy protocol, despite its name, is quite different from traditional proxies. Its primary purpose is to chain proxies and reverse-proxies without losing client information.

It is built by HAProxy's CTO [Willy Tarreau](https://twitter.com/willytarreau), to allow the client’s original IP address to be passed along to the server, regardless of how the traffic is routed - through tunnels, TCP proxies, load balancers, transparent proxies, or other intermediaries. This is especially useful for the server to identify where the request truly originated.

The Proxy Protocol is protocol-agnostic, so it can be used with any layer-7 protocol. It doesn’t require changes to your network infrastructure, works seamlessly with NAT firewalls, and scales well.

It’s important to note that the HAProxy protocol is straightforward but must be supported by the receiving server. You can’t simply enable it on the client side if the server isn’t configured to accept it. Supported endpoints might include proxies, reverse proxies, load balancers, application servers, or web application firewalls (WAFs). In our case: Traefik.


### HAProxy protocol version

There are two version of the HAProxy Protocol.

- The newer version of the protocol is binary, rather than text so is faster to parse.

But you can always run into issues with your version, for example: 

- curl only supports the haproxy protocol v1.

[curl source: everything.curl.dev](https://everything.curl.dev/usingcurl/proxies/haproxy.html)


#### Who all is currently using Proxy Protocol V2?

The following software, services, and devices are known to support the Proxy Protocol:

[up-to-date list of other ppl using the HAProxy Proxy Protocol V2: haproxy.com](https://www.haproxy.com/blog/use-the-proxy-protocol-to-preserve-a-clients-ip-address)



## HA Proxy Protocol - HTTP

Yes. You can proxy TCP traffic. Yes, that's one of the great features of HAProxy, but it's not a feature for today's show.

TCP is really for like, a load balanced MYSQL database, extra security, or 1:1 port tranlsation like an SSH connection for Git.

This write-up is trying to use HTTP mode (Layer 7 Proxy Mode).

OPNsense's HAProxy in HTTP mode allows for more options. Like access to HTTP-specific options for both requests and responses. This allows you to configure routing based on information like the URL path, query string, or HTTP headers. 

For this write-up, we will direct traffic to different backends based on the domain or URL in the request.






## OPNsense HAProxy Setup

These are directions to modify HAProxy to work with the Proxy Protocol.


### Step 1: Edit Real Server

- Enable 'Advanced mode' in the upper left hand corner of this window

- Scroll to the bottom

- Option pass-through: send-proxy-v2



### Step 2: Edit Backend

- Enable 'Advanced mode' in the upper left hand corner of this window

- Mode: TCP (Layer 4)

- Proxy Protocol: Version 2

- Option pass-through: option tcp-smart-connect


### Step 3: Edit Condition

- Condition type: Host Contains

- Host Contains: <subdomain.domain.tld>



### Step 4: Edit Rule

- Test Type: IF

- Select Conditions: <Condition-name-from-above>

- Logical operator for conditions: AND

- Execute function: Use specified Backend Pool

- Use backend pool: <Backend-name-from-above>



### Step 5: Edit Public Service

- Enable 'Advanced mode' in the upper left hand corner of this window

- Type: SSL / HTTPS (TCP mode)

- Option pass-through:

```
tcp-request content accept if { req_ssl_hello_type 1 }
tcp-request inspect-delay 5s
option tcp-smart-accept
timeout client 30s
```

- Select the rules to use <Rule-name-from-above>






* * *

## What did you enter into OPNsense?

Let's break it down:


### TCP content requires req_ssl_hello_type

- `tcp-request content accept if { req_ssl_hello_type 1 }` - fixes ERR_SOCKET_NOT_CONNECTED issue

> This forces HAProxy to buffer incoming data during the connection's initial phase. Although designed for SSL/TLS inspection, it indirectly stabilizes HTTP traffic by ensuring that HAProxy waits for enough data (at least 5 bytes) before forwarding the request to the backend. This prevents premature socket closures and incomplete packet forwarding, which are common causes of this error in TCP passthrough mode.

[answer to fix ERR_SOCKET_NOT_CONNECTED source: serverfault.com](https://serverfault.com/questions/1029930/having-trouble-with-tcp-mode-on-haproxy-running-on-opnsense)


#### What’s the Problem? Why is this required?

HAProxy sometimes faces a situation where it doesn't wait long enough for all the data from a request to come in, causing problems like `ERR_SOCKET_NOT_CONNECTED` — which is basically a connection error that happens when things get cut off too early or not properly handled.

This is especially true in environments where you're only using regular HTTP (not HTTPS/TLS). HAProxy is trying to forward the data but is sometimes doing it too soon, and the backend (the server that HAProxy sends data to) ends up rejecting it.


#### How Does the requires req_ssl_hello_type Rule Help?  

The rule `tcp-request content accept if { req_ssl_hello_type 1 }` tricks HAProxy into behaving like it’s handling encrypted traffic (even if it’s not), by forcing it to wait a little longer and make sure it has enough data to send a complete request. 

- It waits for at least 5 bytes of data (just enough to detect an SSL handshake, even though it's not happening).

- It gives HAProxy up to 5 seconds to collect the data (this is called the `inspect-delay`).

- This extra time makes sure that the whole request (like the `GET` or `POST` command) is buffered properly before being sent to the backend.


#### In Simpler Terms:

Think of it like sending a text message:

You want to send a long text, but instead of waiting until you finish typing it, you accidentally hit "send" halfway through. The person on the other end gets a message that says "Hey, I want to go to the..." and then nothing else, so they get confused and don’t reply.

Now, imagine you have a friend who’s really careful and waits until you’ve typed the whole message before they send it. They make sure they’ve got the full message in one go, so the person on the other side gets the whole thing and understands exactly what you mean.

That’s what this rule does. It’s like making sure HAProxy waits for the whole message (the full request) before it sends anything out, so the backend server doesn’t get confused by incomplete data.



* * *

### Impact of inspect-delay

This was added to ensure you can get a full handshake, even if it takes 5 seconds.

It delays any routing decision for up to 5 seconds to allow sufficient data to be buffered for inspection. This ensures that HAProxy has enough information about the incoming request before forwarding it, preventing incomplete packet forwarding.

```yaml
tcp-request inspect-delay 5s  
```

1. Creates a 5-second window for data collection  

2. Pauses HAProxy’s decision-making until:  

   - 5 seconds elapse, **or**  

   - Buffer contains ≥5 bytes (SSL handshake minimum)  

3. Maintains TCP socket state during this period  



* * *

### Rule Processing Flow

```text
[TCP Connection Open]  
  └─ Wait for data (up to inspect-delay)  
     ├─ If data matches { req_ssl_hello_type 1 }: Accept  
     └─ Else: Fall through to default ACCEPT  
```

Even though your HTTP traffic **never** satisfies `req_ssl_hello_type 1`, the rule:  

- Creates an explicit evaluation point  

- Forces HAProxy to use the **buffered data** for routing decisions  

- Prevents premature connection closure  







#### HAProxy option tcp-smart-accept

- Improves how HAProxy handles incoming connections by optimizing the accept() system call.

- This can help resolve issues related to socket exhaustion or connection instability under high load.

<details>

<summary>Documentation for tcp-smart-accept</summary>  

When an HTTP connection request comes in, the system acknowledges it on
behalf of HAProxy, then the client immediately sends its request, and the
system acknowledges it too while it is notifying HAProxy about the new
connection. HAProxy then reads the request and responds. This means that we
have one TCP ACK sent by the system for nothing, because the request could
very well be acknowledged by HAProxy when it sends its response.

For this reason, in HTTP mode, HAProxy automatically asks the system to avoid
sending this useless ACK on platforms which support it (currently at least
Linux). It must not cause any problem, because the system will send it anyway
after 40 ms if the response takes more time than expected to come.

During complex network debugging sessions, it may be desirable to disable
this optimization because delayed ACKs can make troubleshooting more complex
when trying to identify where packets are delayed. It is then possible to
fall back to normal behavior by specifying "no option tcp-smart-accept".

It is also possible to force it for non-HTTP proxies by simply specifying
"option tcp-smart-accept". For instance, it can make sense with some services
such as SMTP where the server speaks first.

It is recommended to avoid forcing this option in a defaults section. In case
of doubt, consider setting it back to automatic values by prepending the
"default" keyword before it, or disabling it using the "no" keyword.

[tcp-smart-accept source: docs.haproxy.org](https://docs.haproxy.org/3.1/configuration.html#4.2-option%20tcp-smart-accept)

</details>





### HAProxy timeout client 30s

- Set the maximum inactivity time on the client side.

- Sets a timeout for client connections. If a client does not send data within this period, the connection is closed.

- Prevents hanging sockets from consuming resources unnecessarily.

<details>

<summary>Documentation for timeout client</summary>  

The inactivity timeout applies when the client is expected to acknowledge or
send data. In HTTP mode, this timeout is particularly important to consider
during the first phase, when the client sends the request, and during the
response while it is reading data sent by the server. That said, for the
first phase, it is preferable to set the "timeout http-request" to better
protect HAProxy from Slowloris like attacks. The value is specified in
milliseconds by default, but can be in any other unit if the number is
suffixed by the unit, as specified at the top of this document. In TCP mode
(and to a lesser extent, in HTTP mode), it is highly recommended that the
client timeout remains equal to the server timeout in order to avoid complex
situations to debug. It is a good practice to cover one or several TCP packet
losses by specifying timeouts that are slightly above multiples of 3 seconds
(e.g. 4 or 5 seconds). If some long-lived sessions are mixed with short-lived
sessions (e.g. WebSocket and HTTP), it's worth considering "timeout tunnel",
which overrides "timeout client" and "timeout server" for tunnels, as well as
"timeout client-fin" for half-closed connections.

This parameter is specific to frontends, but can be specified once for all in
"defaults" sections. This is in fact one of the easiest solutions not to
forget about it. An unspecified timeout results in an infinite timeout, which
is not recommended. Such a usage is accepted and works but reports a warning
during startup because it may result in accumulation of expired sessions in
the system if the system's timeouts are not configured either.

[timeout client source: docs.haproxy.org](https://docs.haproxy.org/3.1/configuration.html#4-timeout%20client)

</details>




### HAProxy (Optional) option tcpka

- Sends keep-alive packets to ensure a tunnel doesnt break.

- This is very Zero-Tier, Wireguard, Tailscale, NetBird, TwinGate, esque.

<details>

<summary>Documentation for tcpka</summary> 

When there is a firewall or any session-aware component between a client and
a server, and when the protocol involves very long sessions with long idle
periods (e.g. remote desktops), there is a risk that one of the intermediate
components decides to expire a session which has remained idle for too long.

Enabling socket-level TCP keep-alives makes the system regularly send packets
to the other end of the connection, leaving it active. The delay between
keep-alive probes is controlled by the system only and depends both on the
operating system and its tuning parameters.

It is important to understand that keep-alive packets are neither emitted nor
received at the application level. It is only the network stacks which sees
them. For this reason, even if one side of the proxy already uses keep-alives
to maintain its connection alive, those keep-alive packets will not be
forwarded to the other side of the proxy.

Please note that this has nothing to do with HTTP keep-alive.

Using option "tcpka" enables the emission of TCP keep-alive probes on both
the client and server sides of a connection. Note that this is meaningful
only in "defaults" or "listen" sections. If this option is used in a
frontend, only the client side will get keep-alives, and if this option is
used in a backend, only the server side will get keep-alives. For this
reason, it is strongly recommended to explicitly use "option clitcpka" and
"option srvtcpka" when the configuration is split between frontends and
backends.

See also : "option clitcpka" - enables the emission of TCP keep-alive probes on the client side of a connection
and      : "option srvtcpka" - enables the emission of TCP keep-alive probes on the server side of a connection

[tcpka source: docs.haproxy.org](https://docs.haproxy.org/3.1/configuration.html#4.2-option%20tcpka)

</details>




### HAProxy (Optional) option tcplog

- Enables detailed logging for TCP connections, providing visibility into connection states, errors, and timeouts.

- Useful for debugging socket-related issues.


<details>

<summary>Documentation for TCP log format</summary>  

The TCP format is used when "option tcplog" is specified in the frontend, and
is the recommended format for pure TCP proxies. It provides a lot of precious
information for troubleshooting. Since this format includes timers and byte
counts, the log is normally emitted at the end of the session. It can be
emitted earlier if "option logasap" is specified, which makes sense in most
environments with long sessions such as remote terminals. Sessions which match
the "monitor" rules are never logged. It is also possible not to emit logs for
sessions for which no data were exchanged between the client and the server, by
specifying "option dontlognull" in the frontend. Successful connections will
not be logged if "option dontlog-normal" is specified in the frontend.

[TCP log format source: docs.haproxy.org](https://docs.haproxy.org/2.4/configuration.html#8.2.2)

</details>


### HAProxy (Optional) option httplog

- option httplog activates detailed HTTP logging.

- It provides more information than the default or TCP log formats.


<details>

<summary>Documentation for HTTP log format</summary>  

The HTTP format is the most complete and the best suited for HTTP proxies. It
is enabled by when "option httplog" is specified in the frontend. It provides
the same level of information as the TCP format with additional features which
are specific to the HTTP protocol. Just like the TCP format, the log is usually
emitted at the end of the session, unless "option logasap" is specified, which
generally only makes sense for download sites. A session which matches the
"monitor" rules will never logged. It is also possible not to log sessions for
which no data were sent by the client by specifying "option dontlognull" in the
frontend. Successful connections will not be logged if "option dontlog-normal"
is specified in the frontend.

[HTTP log format source: docs.haproxy.org](https://docs.haproxy.org/2.4/configuration.html#8.2.3)

</details>







## Settings on the HAProxy backend

The backend pool is where you connect HAProxy to your real servers. We're using a TCP connection, so it's a good idea to add tcp-smart-connect.

* * *

### HAProxy option tcp-smart-connect

- This option optimizes TCP connection handling by ensuring that connections are efficiently established with backend servers.

- It can reduce latency and prevent connection drops caused by backend server delays or mismatched TCP settings.


<details>

<summary>Documentation for tcp-smart-connect</summary>  

On certain systems (at least Linux), HAProxy can ask the kernel not to
immediately send an empty ACK upon a connection request, but to directly
send the buffer request instead. This saves one packet on the network and
thus boosts performance. It can also be useful for some servers, because they
immediately get the request along with the incoming connection.

This feature is enabled when "option tcp-smart-connect" is set in a backend.
It is not enabled by default because it makes network troubleshooting more
complex.

It only makes sense to enable it with protocols where the client speaks first
such as HTTP. In other situations, if there is no data to send in place of
the ACK, a normal ACK is sent.

[tcp-smart-connect source: docs.haproxy.org](https://docs.haproxy.org/3.1/configuration.html#4.2-option%20tcp-smart-connect)

</details>






## Settings on the HAProxy server

HAProxy needs to be told you're trying to send proxy protocol traffic to rour real server. There is a distinction to make, VERSION 1 or VERSION 2.

- **"send-proxy"** - Sends data using text

- **"send-proxy-v2"** - Sends data using binary

* * *

### HAProxy send-proxy-v2

- send-proxy-v2 makes HAProxy send the PROXY protocol version 2 header to the real servers.


<details>

<summary>Documentation for send-proxy-v2</summary>  

The "send-proxy-v2" parameter enforces use of the PROXY protocol version 2
over any connection established to this server. The PROXY protocol informs
the other end about the layer 3/4 addresses of the incoming connection, so
that it can know the client's address or the public address it accessed to,
whatever the upper layer protocol. It also send ALPN information if an alpn
have been negotiated. This setting must not be used if the server isn't aware
of this version of the protocol. See also the "no-send-proxy-v2" option of
this section and send-proxy" option of the "bind" keyword.

[send-proxy-v2 source: docs.haproxy.org](https://docs.haproxy.org/3.1/configuration.html#5.2-send-proxy-v2)

</details>



* * *

### **Why These Configurations Help**

#### **Buffering and Inspection (`tcp-request inspect-delay`)**
In TCP mode, HAProxy operates at Layer 4 and does not parse HTTP headers. Without buffering, fragmented packets may be forwarded prematurely, leading to incomplete requests being sent to the backend. The `inspect-delay` ensures that enough data is collected before routing decisions are made.

#### **Optimized Connection Handling (`tcp-smart-connect` and `tcp-smart-accept`)**
These options improve how HAProxy manages connections:
- `tcp-smart-connect`: Ensures backend servers are ready to handle requests before connections are established.
- `tcp-smart-accept`: Optimizes how incoming connections are accepted, reducing the likelihood of socket-related errors under high traffic.











### Summary of Placement

| Option                  | Frontend | Backend |
|-------------------------|----------|---------|
| `tcp-request inspect-delay` | ✅       | ❌       |
| `option tcplog`          | ✅       | ❌       |
| `option tcp-smart-accept` | ✅       | ❌       |
| `timeout client`         | ✅       | ❌       |
| `option tcp-smart-connect` | ❌       | ✅       |
| `timeout server`         | ❌       | ✅       |
| `option tcpka`         | ✅       | ✅       |

---

### Additional Notes

- The frontend manages traffic coming from clients, so options like `inspect-delay`, `tcplog`, and `tcp-smart-accept` are applied here.
- The backend manages traffic going to servers, so options like `tcp-smart-connect`, `timeout server`, and PROXY protocol directives are applied there.

This structure ensures that HAProxy efficiently handles both incoming and outgoing traffic while addressing socket-related issues.




# Traefik v3 using proxy protocol v2


## Docker-Compose Command

The label will always be required.

```
services:
  traefik:
    image: traefik
    command:
      - "--entrypoints.web.proxyProtocol.insecure=true"
    labels:
      - "traefik.tcp.services.traefik.loadbalancer.proxyprotocol.version=2"
```

When you set `entrypoints.web.proxyProtocol.insecure=true`, you're telling Traefik that it's okay to accept PROXY protocol headers from sources that might not be trusted (i.e., those that might not be using a secure, reliable proxy path).

This is useful in environments where **you are routing traffic from an internal proxy** or trusted service that might forward requests using PROXY protocol.

This is like a "trust me" flag to Traefik, saying "I know this traffic could be coming from an unreliable source, but I still want to accept PROXY protocol headers in this case."

Instead of blindly accepting PROXY headers from anywhere (insecure=true), you can explicitly tell Traefik which IP addresses are allowed to send PROXY protocol headers. This is what `trustedIPs` is for.

#### Example trustedIPs Traefik config

Production will have the static IP of HAProxy in a pool. When doing this, you can just remove the insecure line, as the default is secure.

```yaml
entryPoints:
  web:
    address: ":80"
    proxyProtocol:
      trustedIPs:
        - "192.168.1.10"  # IP of HAProxy
  websecure:
    address: ":443"
    proxyProtocol:
      trustedIPs:
        - "192.168.1.10"  # IP of HAProxy
```
