--- 
layout: post 
title: Getting Started With Prosody
location: Manipal
---

Generating a stanza in Prosody is fairly easy. Here is how you do it.

{% highlight lua %}
local st = require "util.stanza";
local reply = st.stanza("message");
reply:tag("body"):text("Enter the valid keys")
        :up()
        :tag("x", {xmlns="jabber:x:oob"})
        :tag("url"):text("some_url"):up():up()  
        :tag("captcha", {xmlns="urn:xmpp:captcha"})
        :up();  
print (reply);
{% endhighlight %}


Debugging Techniques.

Talking to waqas, I got to know several ways to debug my code.

1. Enable mod_admin_telnet in the config. Connect to the telnet, to be greeted by the Prosody ASCII ```telnet localhost 5582``` 

2. Load the required module by doing ```module:load("module_name")``` 

3. Files can also be loaded by doing ```;dofile("filename")``` 


Make sure prosody is not run as a daemon, i.e the posix plugin must be disabled. The
print statements get discarded if prosody is run as a daemon. A few ways to
debug are Using the print command. (It will print to the shell and not the telnet console)


Using ```log("debug", "message")```. (It will be written to the log file, Run a
```tail -f log_file``` to follow the messages)


Using ```;dofile("filename")``` will print the messages to the telnet console,
if you have used the print command.

