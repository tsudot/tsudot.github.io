--- 
layout: post
title: Jingle Relay Nodes
location: Manipal
---
A couple of weeks ago,[Thiago](http://twitter.com/xmppjingle) helped me out
understand the basics of Jingle Relay Nodes. He explained that, the server
needs to open 4 ports and transfer data coming into them with each one of the
other. His exact email

{% highlight text %} 
In fact your server will have to open 4 ports in total, 1rtp + 1rtcp  for each
end point.  Lets call them A, A', B, B' respectively.  Every packet that
arrives at A needs to be sent through B to whichever address(IP:Port) is
sending to B. Every packet that arrives at A' needs to be sent through B' to
whichever address(IP:Port) is sending to B'.  

Every packet that arrives at B needs to be sent through A to whichever
address(IP:Port) is sending to A.  Every packet that arrives at B' needs to be
sent through A' to whichever address(IP:Port) is sending to A'.  Exception:  *
If no packet was received in a given port, packets to be sent through that port
should be discarded.  Remarks: * There is no process to verify that a sender to
a given port is the real sender of the stream.  

The recommended mitigation is to create a race of packets, for the latest 10
received packets in a given port, only relay and update Address to the one
sender holding the majority of the packets.
{% endhighlight %}

I ended up implementing the same in TCP. [mod jingle relay
nodes](https://code.google.com/p/prosody-gsoc/source/browse/mod_jinglerelaynodes.lua)
and [mod jingle
channel](https://code.google.com/p/prosody-gsoc/source/browse/mod_jinglechannel.lua)
