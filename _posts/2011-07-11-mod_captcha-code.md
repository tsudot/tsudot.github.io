--- 
layout: post
title: mod_captcha code
location: Manipal
---
I'll start right away with the [code](http://code.google.com/p/prosody-gsoc/source/browse/mod_captcha.lua).


{% highlight lua %}
module:hook("presence/full", generate_captcha, 20);
{% endhighlight %}


A hook is written to override the priority of `presense/full`, which is an
event generated when someone tries to log into an muc. This hook redirects that
event to the generate_captcha function.



{% highlight lua linenos %}
local function generate_captcha(event)
    local stanza = event.stanza;
    if stanza.attr.captcha_verified and stanza.attr.captcha_verified == "true" then
        return nil
    else
        local options = {}
        options.body = "api_key";
        local request_url = "http://www.google.com/recaptcha/api/noscript";
        http.request(request_url, options, function(...) 
            return generate_captcha_response(event, ...) 
        end);
    end
end
{% endhighlight %}


The function ```generate_captcha``` basically sends a requests via the recaptcha API,
and gets a captcha image. This image is then passed to the function
```generate_captcha_response``` which send the image along with the data
form to the jid. It also checks if the ```attr.captcha_verified``` field in
the stanza is set to "true". If it has been set to "true" this means the user
has already passed the captcha test, and hence the stanza is forwarded to the
```mod_muc``` plugin.


{% highlight lua linenos %}
local function generate_captcha_response(event, result, status, r)
    local url = "http://www.google.com/recaptcha/api/";
    local session, stanza = event.origin, event.stanza;
    local node, host, resource = jid.split(stanza.attr.to);
    local ip = session.ip;
    local img_src = result:match([[\<img .*alt="".* src="([^"]+)"]])
    url = url..img_src
    local challenge_id = img_src:match("=.+")
    local sid = stanza.attr.id;

    if requests[stanza.attr.from] then
        requests[stanza.attr.from][img_src] = true;
    else
        requests[stanza.attr.from] = {};
        requests[stanza.attr.from][img_src] = true;
    end

    -- Store events here for firing later
    events[challenge_id] = event
    stanza.name = "message";
    local reply = st.reply(stanza);
{% endhighlight %}


This function is responsible for generating the data form and sending it to the
client. It also saves maps the ```attr.from``` and the ```image_src``` in a table which is
used in case mutiple requests come from the same client. Also the event is
stored in the events table mapped to the ```challenge_id```.

{% highlight lua %}
module:hook("iq-set/host/urn:xmpp:captcha:captcha", verify_captcha)
{% endhighlight %}

This is a hook which sends the iq-set/host/urn:xmpp:captcha:captcha event to the verify_captcha function.

{% highlight lua linenos %}
local function verify_captcha(event)
    local pri_key = "pri_key";
    local session, stanza = event.origin, event.stanza;
    local captcha_form = stanza.tags[1]:get_child("x", "jabber:x:data");
    local fields = captcha_response_layout:data(captcha_form);
    local url = "http://www.google.com/recaptcha/api/verify";
    if requests[stanza.attr.from] 
            and requests[stanza.attr.from][fields.challenge] 
            and fields.ocr ~= "" then
        requests[stanza.attr.from][fields.challenge] = nil;
        local options = {}
        options.body = "privatekey="..pri_key.."&remoteip="..session.ip..
                "&challenge="..fields.challenge.."&response="..fields.ocr
        http.request(url, options, function(...) 
            return verify_captcha_response(event, fields.challenge, ...) 
        end);
    end
end
{% endhighlight %}

The ```verify_captcha``` function verifies if the client has entered the correct
value for a captcha. I uses the recaptcha API to check the same. From here the
response from the API is redirected to the ```verify_captcha_response``` function.

{% highlight lua linenos %}
local function verify_captcha_response(event, challenge_id, result, status, r)
    local session, stanza = event.origin, event.stanza;
    local node, host, resource = jid.split(stanza.attr.to);
    local entries = {}
    for entry in result:gmatch("[^\n]+") do
        table.insert(entries, entry);
    end

    if entries[1] and entries[1]=="true" then
        session.send(st.iq({type="result", from=host}));
        -- fire the original event again with an added 
        -- field of captcha_verified = 'true'
        local original_event = events.challenge_id;
        original_event.stanza.attr.verifired = "true";
        module:fire_event("presence/full", original_event)
    else
        session.send(st.error_reply(stanza, "cancel", 
            "service-unavailable", "Not a valid input"));
    end
end
{% endhighlight %}


In this function, the response is checked, If it is true then the
```attr.verified``` flag is set to "true" and the event is fired again. If
the response is not correct, then an error reply is sent.
