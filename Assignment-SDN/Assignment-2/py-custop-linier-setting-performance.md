# mininet - Linear Topology (with Performance Settings)

In addition to basic behavioral networking, Mininet provides performance limiting and isolation
features, through the CPULimitedHost and TCLink classes.
There are multiple ways that these classes may be used, but one simple way is to specify them
as the default host and link classes/constructors to Mininet(), and then to specify the
appropriate parameters in the topology.

<img src="https://github.com/syaifulahdan/mininet/blob/master/image/linier_topology.png" width="400" height="400" align="center" title="syaifulahdan/mininet" />

##### Creating Topology Linear Topology (with Performance Settings)

<pre>
#!/usr/bin/python

from mininet.topo import Topo
from mininet.net import Mininet
from mininet.node import CPULimitedHost
from mininet.link import TCLink
from mininet.util import irange,dumpNodeConnections
from mininet.log import setLogLevel

class LinearTopo(Topo):
    "Linear topology of k switches, with one host per switch."

    def __init__(self, k=2, **opts):
    """Init.
        k: number of switches (and hosts)
        hconf: host configuration options
        lconf: link configuration options"""

    super (LinearTopo, self).__init__(**opts)

    self.k = k


    lastSwitch = None
    for i in irange(1, k):
          host = <b>self.addHost('h%s' % i, cpu=.5/k)</b>
          switch = self.addSwitch('s%s' % i)
          # 10 Mbps, 5ms delay, 1% loss, 1000 packet queue
          self.addLink( host, switch, bw=10, delay='5ms', loss=1,
max_queue_size=1000, use_htb=True)
          if lastSwitch:
              <b>self.addLink(switch, lastSwitch, bw=10, delay='5ms', loss=1,max_queue_size=1000, use_htb=True)</b>
            lastSwitch = switch


def perfTest():
    "Create network and run simple performance test"
    topo = LinearTopo(k=4)
    net = Mininet(topo=topo,
                  host=CPULimitedHost, link=TCLink)
net.start()

print "Dumping host connections"
    dumpNodeConnections(net.hosts)
    print "Testing network connectivity"
    net.pingAll()
    print "Testing bandwidth between h1 and h4"
    h1, h4 = net.get('h1', 'h4')
    net.iperf((h1, h4))
    net.stop()
if __name__ == '__main__':
    setLogLevel('info')
    perfTest()

</pre>

##### Description Scrip Python

- <b>self.addHost(name, cpu=f)</b> : This allows you to specify a fraction of overall system CPU resources which will be allocated to the virtual host.

- <b>self.addLink( node1, node2, bw=10, delay='5ms', max_queue_size=1000,loss=1, use_htb=True)</b>: adds a bidirectional link with bandwidth, delay and loss characteristics, with a maximum queue size of 1000 packets using the Hierarchical Token
Bucket rate limiter and netem delay/loss emulator. The parameter bw is expressed as a number in Mb/s; delay is expressed as a string with units in place (e.g. '5ms', '100us', '1s'); loss is expressed as a percentage (between 0 and 100); and max_queue_size is expressed in packets. You may find it useful to create a Python dictionary to make it easy to pass the same parameters into multiple method calls, for example:
