`socktopus` provides a high-throughput (high latency and non-guaranteed) UDP datagram socket between two systems over the Internet through [multipathing](http://en.wikipedia.org/wiki/Multipath_routing).

Inspired by [Multipath TCP](http://en.wikipedia.org/wiki/Multipath_TCP) which utilises multiple physical interfaces to create a faster channel, `socktopus` instead uses TCP 'reflectors' (via [xinetd](https://azouhr.wordpress.com/2012/06/21/port-forwarding-with-xinetd/), [iptables](https://thewiringcloset.wordpress.com/2013/03/27/linux-iptable-snat-dnat/), or application proxy servers) installed on a number of relay systems spread across the Internet.  These systems are then used to multipath a flow through to its destination, relying on TCP back pressure and non-blocking writes to do the magic under the hood.

Currently under development, it is being written to assist with streaming across the Internet as the author is finding out the hard way how slow the Internet is when you are trying to stream in real time over one billion HTTP access log lines a day.

Anyone finding they are hitting a choke point between their endpoints whilst trying to stream, or batch send, large amounts of data will hopefully find `socktopus` useful.

## Related Links

 * [Multipath TCP](http://www.multipath-tcp.org/)
 * [Proxy-based Multpath Routing](http://cs.uccs.edu/~cs691/secureRouting/YuCaiPhd_proposal_presentation.ppt) by [Yu Cai](http://www.mtu.edu/technology/school/faculty/cai/)
 * [Phoebus](http://damsl.cs.indiana.edu/projects/phoebus/):
  * [A system for high throughput data movement](http://wiki.martin.lncc.br/ziviani-cursos-gb-500-2011-2/file/06-vivian-phoebus.pdf)

# Pre-flight

`socktopus` requires [Perl 5.14 or higher](https://www.perl.org/) and the following libraries:

 * [URI](http://search.cpan.org/~ether/URI/lib/URI.pm)
 * [Data::GUID](http://search.cpan.org/~rjbs/Data-GUID/lib/Data/GUID.pm)
 * [Convert::ASN1](http://search.cpan.org/~gbarr/Convert-ASN1/lib/Convert/ASN1.pod)
 * [AnyEvent](http://software.schmorp.de/pkg/AnyEvent.html)

Start off by fetching the source which requires you to [have git installed on your workstation](http://git-scm.com/book/en/Getting-Started-Installing-Git):

    git clone https://github.com/jimdigriz/socktopus.git

## Debian 'wheezy' 7.x

    sudo apt-get update
    sudo apt-get install -yy --no-install-recommends perl liburi-perl libdata-guid-perl libconvert-asn1-perl libanyevent-perl cpanminus
    sudo cpanm AnyEvent::Handle::UDP

# Usage

Connector (active) end that calls out to a passive instance:

    env AE_LOG=main=+log UDPLOC=1234 UDPREM=1235:1236 ./socktopus tcp://127.0.0.1

Listener (passive) end that receives connections:

    env AE_LOG=main=+log SERVICE=23461 ./socktopus

Notes of interest:

 - logging is controlled by the environment variable [`AE_LOG`](http://pod.tst.eu/http://cvs.schmorp.de/AnyEvent/lib/AnyEvent/Log.pm#CONFIGURATION_VIA_ENV_PERL_ANYEVENT_)
 - if no arguments are present, the process goes into listener mode
 - arguments are URIs
 - the first URI is the destination host you are connecting to and *must* be a `tcp` URI
 - `SERVICE` is the TCP listening socket, takes the syntax `[host:][serv]` where `host` defaults to `[::]` and `serv` to `23461`
 - `UDP{LOC,REM}` take the syntax `[host:][serv:][rhost:]rserv`, where `(r)host` defaults to `localhost` and `serv` defaults to `23461`
 - `UDPLOC` describes the local source host/port and destination host/port tuple
 - `UDPREM` describes the remote source host/port and destination host/port tuple

One use case for the UDP data channel created is to run [VTun](http://vtun.sourceforge.net/) providing you with a high-throughput network interface.

## URIs

Example URIs are:

 * `tcp://host.example.com:1234`
 * `socks://user:pass@socks.example.com:1111`

The following URI schemes are supported:

 * **`tcp` (default port: 23461):** connecting to this raw TCP socket will proxy/NAT you to your destination but of course requires you to set up something like [iptables DNAT](http://linux-ip.net/html/nat-dnat.html) or [xinetd redirect](http://azouhr.wordpress.com/2012/06/21/port-forwarding-with-xinetd/) on the node to do the forwarding for you
 * **`socks` (default port: 1080) [not supported]:** [SOCKS server](http://en.wikipedia.org/wiki/SOCKS_(protocol))
 * **`http` (default port: 8080) [not supported]:** HTTP proxy that supports the [CONNECT method](http://en.wikipedia.org/wiki/HTTP_tunnel#HTTP_CONNECT_Tunneling)

For the non-`tcp` schemes, the application proxy will be asked to connect to the target system in the first argument.
