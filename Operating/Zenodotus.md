# Zenodotus

## What is it?

If you've ever wanted to host a network service on a Seattle node, such as a diagnostic web page or similar, you'll no doubt be familiar with their potential instability. If you didn't watch your nodes like a hawk, You might lose one and not know about it for hours, perhaps even a day or more. And even if you can deal with that, your users still have to punch in an IP Address and port combination to navigate to your service! This is a very clumsy way to do things, so zenodotus was built to make the process cleaner.

By adding a few lines of code to the REPY application running in your node, you can register your service to a domain name through Seattle's open DHT server, allowing for users to navigate to your service with a domain name and a port, rather than having to remember a specific node's IP address. What's more, you can register multiple nodes to a single hostname, meaning that even if one node fails, your service will still be available, allowing you to have uninterrupted uptime without constant vigilance.



## How do I use it?

General instructions for use are as follows:

The following code snippet is an example of how to implement zenodotus hosting in a REPY node. This code runs in each node hosting the service, so each one autonomously advertises its own existence, meaning that nodes which fail for whatever reason will simply be omitted from the DHT after their advertise expires.


```python
include	advertise.repy




# Refresh every	600 seconds (10	minutes)
_REFRESH_FREQUENCY = 600

# Put the address here,	since it's a constant.
_ADDRESS_NAME = "hello.zenodotus.poly.edu"




def _rehost_callback():
  # Refresh the association.
  advertise_announce(_ADDRESS_NAME, getmyip(), _REFRESH_FREQUENCY)

  # Set our timer again.
  settimer(_REFRESH_FREQUENCY - 5, _rehost_callback, [])




if callfunc == 'initialize':
  # For	example	purposes, let's suppose	you're running a service called
  # helloworld that displays "Hello, World!" In	a web page when	the user
  # connects to	a REPY node you control	via web	browser. It makes sense
  # to use the hostname	"hello.zenodotus.poly.edu", so that's
  # what we'll advertise.

  # Associates "hello.zenodotus.poly.edu" : <Node's IP> for
  # _REFRESH_FREQUENCY seconds.
  #
  # ########
  # Note that this line is deprecated. To register a 'A' query in 
  # zenodotus, the _ADDRESS_NAME should be preceded with the 
  # character 'A' and a delimiting space. The format used here 
  # will still work, since it makes sense conventionally to 
  # maintain it, but it is not preferred.
  # ########
  #
  advertise_announce(_ADDRESS_NAME, getmyip(), _REFRESH_FREQUENCY)

  # Set a timer	for a callback function to refresh the DHT association.
  # We should check this every _checking_frequency seconds, to save system 
  # processing resources without sacrificing potential uptime. No args are needed.
  settimer(_REFRESH_FREQUENCY - 5, _rehost_callback, [])

  # You need to	have your service code in here as well.	This code 
  # alone will terminate before it refreshes even once.

```


**To run this code on a Seattle node, you will need to pre-process the code using [wiki:SeattleLib/repypp.py repypp.py]**


When running in a node, this will associate helloworld.zenodotus.poly.edu with your node's IP Address for 600 seconds. This number is arbitrary, but it is worth noting that you should not simply place a very large number here, since it might cause confusion for your users if old associations linger after the service has been taken down.

Note that there is a bug in the above code. If the node this is running in has its computer clock reset during the runtime, getruntime() will fail to provide increasing values, and the node will fail to update for a time, which will give the illusion that it has been terminated. There doesn't seem to be a good way to solve this.

While this is running in at least one node, your users will no longer have to type a URL of the form <IP Address>:<Port> in to their browser to access your service. Instead, they can type in something more friendly, as in the example above: 'hello.zenodotus.poly.edu:631XX'. It is important to remember that this hosting service can't solve the port issue, since REPY VMs can only use GENIports for networking operations. As such, your users will still have to append the GENIport to their URLs.



**Zenodotus supports DNS chaining as well**.
If you want to use your own DNS service, you can register an advertise value of "CNAME name.zenodotus.poly.edu" : Hostname to indicate that a subserver exists, and RRs known by that server should follow the form "mypage.name.zenodotus.poly.edu", where 'name' is the name of your service.



## Debugging

It is important to note that we currently only support simple A lookup queries. A functionality expansion is being worked on, and can be expected fairly soon. For now, however, just remember that there is no support for other query types.
