--- 
layout: post
title: Putting unique values in a database
location: Manipal
---

I was implementing the POP3 protocol in python for my Project Lab and I'd like to put some interesting tidbits I learned in the process.

A small problem was, whenever you query the mail server using the
[LIST](http://tools.ietf.org/html/rfc1939#page-6) command, it will always
return all the messages which are there in the inbox, so its our job to check
for unique mails and to store only those. A wrote a simple python script to do
just that. Now my msg_id column has an index defined for it. and I'm hashing
all the message id's and storing them in the database for uniformity.


{% highlight python %}
conn = sql.connect("mail")
c = conn.cursor()
m = hashlib.md5()
m.update(msgid)
c.execute('select exists(select * from mail where msg_id=?)',[m.hexdigest()])
for row in c:
    predict = row[0]
if not int(predict) :
    c.execute("insert into mail values(:body, :date, :msgid, :p_from, :sub)",
            {"body":body,"date":date,"msgid":m.hexdigest(),"p_from":p_from, "sub":sub})
{% endhighlight %}
