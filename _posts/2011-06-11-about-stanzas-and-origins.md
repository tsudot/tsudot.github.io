--- 
layout: post
title: About stanzas' and origins' (Prosody)
location: Manipal
---

Here is a structure of the stanza property of an event,

{% highlight lua %}
stanza =  {name = "message";  
attr = {  from = "...";  to = "..."; };  };
{% endhighlight %}

A basic event I'm using to test my script has a stanza and a origin property.
The origin property looks like this.

{% highlight lua %}
origin =  { ip = "...";  
    send = function(stanza) 
    print(stanza) 
    end;  
};
{% endhighlight %}

We must now define the stanza property, doing it manually would require you to
set the metatables manually too. So heres an easy way of doing it. Use the
stanza util module.

{% highlight lua %}
st = require "util.stanza";
event.stanza = st.stanza("message", {
    from="robot@abuser.com/zombie", 
    to="innocent@victim.com"
});
{% endhighlight %}
