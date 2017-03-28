While trying to wrangle computers to do your bidding there are a number of concepts that are universal to all operating systems. Knowing these concepts and the tools that let you inspect their state on the operating system level can give you more insight into when things inevitably go wrong. A good one for networking code is the idea of ports. You have used a port to view this web page, although you may not have known it. If you have pulled down information from an API you used a port.  Having done the tutorial you will have some idea of the following:

* Debugging network code you have written
* Finding out what network traffic is currently happening on your computer
* Important background knowledge when talking about things like firewalls or docker containers

Ports are an important part of networking on all platforms. They allow computers to communicate to multiple different other computers at once, without talking over each other. 

To communicate with a computer you need two things a name or number and a port. The familiar www.google.com is an example of a name and http as a standard talks over port 80 (or port 443 for https). This standard means you do not have to specify it when using many tools. You can think of a port as an extension number for a phone number in an office. Every networking program has to specify which port it is listening to or talking to. A networking program that is listening on a port is sometimes called a listening service. A listening service can still talk, it is just there listening for the equivalent of a telephones ringing to set up the connection.

This tutorial assumes that you are on MacOSX, I'll translate it to Windows and Linux as time allows.

## Talking to yourself

Let us start with a program that listens on a port and displays it to the screen. nc is a swiss army knife of a networking tool, it allows you to listen, talk and also forward messages. Nc is short for netcat, which is not much more illuminating to it's purpose. To make it listen on a port 8080 type.
<pre>
nc -l 8080 
</pre>
The -l parameter means listen, so it just sits there waiting for something to talk to it. Lets talk to it using telnet. telnet a useful command for talking out to a specific host and port. Localhost is a special name for your own computer. A venerable tool to do this is telnet. So to talk to nc, open up another command line window and type.
<pre>
telnet localhost 8080
</pre>
It should just sit there doing nothing. Congratulations! Your computer is now talking to itself.

Type in either command line window and once you hit return you should see the text appear in the other window.


## Talking http

Quit out of telnet by typing ctrl + ] and then typing quit. ^ means ctrl so 'Escape character is ^]' means ctrl + ] will get you out of the connection, then quit will get you out of telnet. Now to get a flavour of http we will use another useful tool called curl. It can download web pages and tell you detailed debugging information about what is going on.

Type:
<pre>
curl http://localhost:8080 
</pre>
You need to specify the port after the name with a colon else it will try and talk to port 80 as standard.

In the nc window you should see something like.

<pre>GET / HTTP/1.1
Host: localhost:8080
User-Agent: curl/7.43.0
Accept: */*</pre>

You can see that it is curl that is connecting by the user agent. Let us see if we can fool google into thinking we are a web browser. To do that we go back to our trusty telnet command and type.
<pre>
telnet www.google.com 80
</pre>
Copy and paste the first line of the request made by curl "GET /" to your telnet window. Type another return character as well, so it knows the request is finished. You should get a response like
<pre>
"HTTP/1.1 302 Found"
</pre>
You have managed to fool google into thinking you know how to speak http. If you want to actually learn to speak HTTP you can look up the [verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) and the [status codes](https://tools.ietf.org/html/rfc7231#section-6) in the RFCs. RFCs are requests for comments and how various networking protocols get defined. They are not the prettiest things in the world, but are the source for how things should (but don't necessarily) work.

## Exploring your computers chatter

Your computer is continually talking to lots of other computers and listening for other computers trying to talk to it. Sometimes you need to find out what is going on. So type
<pre>
telnet localhost 8080 
</pre>
again to set up the connection. nc should still be listening in the background.

Now type
<pre>
netstat -f inet -n
</pre>
This gets you connections in the inet family (which is the standard internet family) and specifies that you want the numeric values of things (rather than names that netstat normally gives you). 

You should see an entries for

<pre>
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
-------------------------------
tcp4       0      0  127.0.0.1.8080         127.0.0.1.XXXXX        ESTABLISHED
tcp4       0      0  127.0.0.1.XXXXX        127.0.0.1.8080         ESTABLISHED
</pre>

Where XXXXX is high number.

127.0.0.1 is the special number that localhost is translated to. You would have seen it when telnet-ing to your local machine. The port is the final number of the 'address'.

This shows you both side of your telnet/nc connection and that outgoing connections get a random port number assigned to them as well.  The state is the state of the connection, it is an established state. There are a few different states it could be in, established means you can send data.

You will probably see lots of other connections open. These will be web browsers connected to servers and other things your computer is doing. Have a look at this [port listing](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) and try and figure out what is going on.  You can also run netstat without the -n parameter to get the names of the servers it is talking to with the names of the ports.


## Tcpdump 

One last very powerful tool is tcpdump. It allows you to see what is going inside a connection without disturbing it. Make sure nc and telnet are still running and in a third window type:
<pre>
tcpdump -A --interface lo0 port  8080
</pre>
With the -A parameter it tries to print out the contents of the communications.  The --interface command tells it which network interface to use (imagine you had physical and a wifi network device, each would have a seperate interface), in this case it is lo0 which is the loopback interface, which is where localhost lives.

Start typing in either the nc or the telnet windows and you should be able to see the content of those messages in the tcpdump window. There is lots of other extraneous detail as well, you do not have to understand all of it.

Tcpdump has a complex query language so you can select only certain connections to spy on. That is what the port 8080 means. You can specify the destination address with "dest <the_address>" Try tcpdump-ing some of the established connections you saw in netstat to see what their chatter looks like. HTTPS traffic is unitelligible because it is encrypted. HTTP traffic is more easily intercepted and anyone on the way to a server can run a tcpdump and find out what is going on. A tutorial for tcpdump can be found [here](https://danielmiessler.com/study/tcpdump/#gs.9bGSpcI).


