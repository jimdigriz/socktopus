`socktopus` provides a high-throughput (high latency and non-guaranteed) datagram (UDP) socket between two systems over the Internet.  It does this with [multipathing](http://en.wikipedia.org/wiki/Multipath_routing) over multiple stream (TCP) sockets supporting [SOCKS](http://en.wikipedia.org/wiki/SOCKS_(protocol)) and [HTTP CONNECT](http://en.wikipedia.org/wiki/HTTP_tunnel#HTTP_CONNECT_Tunneling) proxies.

The topology requires `socktopus` to be installed on the endpoints and a SOCKS or HTTP proxy server on a number of nodes spread across the Internet.  `socktopus` then uses these nodes to multipath a flow to its destination, relying on TCP back pressure and non-blocking writes to do the magic under the hood.

Currently under development, it is being written to assist with streaming across the Internet as the author is finding out the hard way how slow the Internet is when you are trying to stream in real time over one billion HTTP access log lines a day.

Anyone finding they are hitting a choke point between their endpoints whilst trying to stream, or batch send, large amounts of data will hopefully find `socktopus` useful.

## Related Links

 * [Multipath TCP](http://www.multipath-tcp.org/)
 * [Proxy-based Multpath Routing](http://cs.uccs.edu/~cs691/secureRouting/YuCaiPhd_proposal_presentation.ppt) by [Yu Cai](http://www.mtu.edu/technology/school/faculty/cai/)
 * [Phoebus](http://damsl.cs.indiana.edu/projects/phoebus/):
  * [A system for high throughput data movement](http://wiki.martin.lncc.br/ziviani-cursos-gb-500-2011-2/file/06-vivian-phoebus.pdf)

# Pre-flight

`socktopus` requires the following libraries:

 * [Perl 5.14 or higher](https://www.perl.org/)
 * [URI](http://search.cpan.org/~ether/URI/lib/URI.pm)
 * [Data::GUID](http://search.cpan.org/~rjbs/Data-GUID/lib/Data/GUID.pm)

Start off by fetching the source:

    git clone https://github.com/jimdigriz/socktopus.git

## Debian 'wheezy' 7.x

    sudo apt-get update
    sudo apt-get install -yy --no-install-recommends perl liburi-perl libdata-guid-perl

# Usage

Connector (active) end that calls out to a passive instance:

    env UDPLOC=23461:127.0.0.1:1234 UDPREM=23461:192.0.2.8:1234 ./socktopus tcp://192.0.2.1:23461 ...

Listener (passive) end that receives connections:

    env PORT=23461 ./socktopus

Notes of interest:
 - if no arguments are present, the process goes into listener mode
 - arguments when present are of the form of a TCP URI with port
 - `PORT` is where the local TCP socket where the connector reaches out to (over multiple paths) - this is the first argument passed to the connector
 - `UDPLOC` is the local end of the port to set up by the connector
 - `UDPREM` is the port number to request the listener relays the traffic out on

One use case for the UDP data channel created is to run [VTun](http://vtun.sourceforge.net/) providing you with a high-throughput network interface.

## Node URIs

The URI looks like (path componment is ignored):

    socks://node.example.com:1234

The following URI schemes are supported:

 * **`tcp` (default port: 23461):** connecting to this raw TCP socket will proxy/NAT you to your destination but of course requires you to set up something like [iptables DNAT](http://linux-ip.net/html/nat-dnat.html) or [xinetd redirect](http://azouhr.wordpress.com/2012/06/21/port-forwarding-with-xinetd/) on the node to do the forwarding for you
 * **`socks` (default port: 1080):** SOCKS5 server but you can also use [`socks4`](http://en.wikipedia.org/wiki/SOCKS_(protocol)#SOCKS4)
 * **`http` (default port: 8080):** HTTP proxy that supports the [CONNECT method](http://en.wikipedia.org/wiki/HTTP_tunnel#HTTP_CONNECT_Tunneling)
