Day 2 & Day 3 & — Linux routing: How does a packet find its destination
Day 4 — Linux networking checks for DevOps
==============================================================================

Day -2 Commands:

    1. ip route: it will show How many routes our linux mechine will know.
    2. ip route get <IP>: which route Linux will choose to reach the destination. 
    3. ip rule: it tells Linux which routing table to consult based on policy rules when deciding where to send packets.

Day: 3 Commands:

    4. ping — Check ICMP reachability
    5. ip neigh — Inspect the neighbor table
    6. tcpdump — Inspect network packets

Day: 4 Commands:

    7. sudo ss -lntp - Is the application listening?
    8. curl -v http://127.0.0.1:8080/  - Does it respond locally?
    9. If local access works, can remote users reach it?
       Check the application's bind address, EC2 security group, network ACLs, routing, and any host firewall.
    10. dig your-app.example.com - If using a hostname, does DNS resolve correctly?

* Today, we’ll learn how Linux decides where to send a packet. We’ll build on what you already know about Pod IPs, network namespaces, and AWS VPC.

* Imagine Pod A wants to send a request to Pod B.

             Pod A
      Destination: 10.0.2.35
      
         Linux routing table
      Which route should I use for this destination?
      
      Next hop / outgoing interface
      Linux follows the selected route.

What is a routing table?

    A routing table is a list of rules that helps Linux decide where packets should go.

For example, imagine a simplified routing table:

    Destination	          Meaning
    10.0.1.0/24	          Send traffic for this network through a local interface.
    10.0.2.0/24	          Use the configured route for that network.
    0.0.0.0/0	            Default route, used when no more specific route matches.

    These are illustrative routes, not actual routes from your EC2 instance.


Important: Linux generally selects the most specific matching route. The default route is the fallback when no more specific route matches.

2. Practical — 10 minutes
   
Let's inspect routing without changing anything.

* Command 1 — View the routing table: ip route ------> Shows the routes Linux currently knows about.




* Command 2 — Check the route to a destination: ip route get <IP>

       This displays rules that determine which routing table Linux consults.
       Most basic setups use the main routing table, but advanced networking can use multiple tables and rules.

        default via 10.0.1.1 dev eth0
        10.0.1.0/24 dev eth0 proto kernel scope link
  
        Here's how to understand it:
  
        default via 10.0.1.1 dev eth0
        - default — used when no more specific route matches.
        - via 10.0.1.1 — send traffic through this next-hop gateway.
        - dev eth0 — use the eth0 network interface.
  
          Destination: Where the parcel needs to go.
          Gateway: The next delivery point.
          Interface: The road your parcel leaves through.
  
        10.0.1.0/24 dev eth0
        This route covers the 10.0.1.x subnet. Linux can send traffic for that directly connected network through eth0, without using the default gateway.


Shows the route Linux would use for this destination, based on the current routing rules and routing context. The example IP may not exist in your network.

* Command 3 — View routing policy rules: ip rule ---> Shows rules that can influence which routing table Linux consults.

3. Troubleshooting scenario — 10 minutes
   
Imagine an application in Pod A sends a request to Pod B, but the request fails.

You run:

    ip route get 10.0.2.35


* reports that it cannot find a usable route to the destination in the current routing context.
* That gives you a useful clue: investigate the routing configuration first. Then verify the route and connectivity from the relevant Pod or node.
* A missing route in one context doesn't necessarily mean the route is missing everywhere.
* Also remember: a valid route doesn't prove the packet will arrive. Security rules, network configuration, or the destination application may still cause a failure.


Day -3:
-------


Three tools, three purposes

Command: ping — Does the destination respond?

    Sends ICMP Echo Requests and checks for Echo Replies.
    Example: Can this machine get a response from another IP address?

Command: ip neigh — What does Linux know about nearby network devices?

    Displays the neighbor table, which commonly maps IPv4 addresses to MAC addresses on a directly connected network.

Command: tcpdump — What packets are actually visible?

    Captures network packets so you can inspect requests and responses.

Important: A failed ping does not automatically mean the destination is down. ICMP may be blocked even when the application works.


Practical commands — 10 minutes
-------------------------------

Use these on your Linux EC2 instance or an authorized lab machine. The commands below are for inspection; don't capture traffic on systems you aren't authorized to troubleshoot.

Practical A — Test with ping

    ping -c 4 8.8.8.8


    - -c 4 means send four requests.
    - Replies indicate that ICMP communication succeeded.
    - No replies could mean filtering, packet loss, a routing issue, or an unreachable destination.

Practical B — Inspect the neighbor table

    ip neigh


Example output:

    10.0.1.1 dev eth0 lladdr 02:11:22:33:44:55 REACHABLE


Read it like this:

    - 10.0.1.1 — neighbor's IP address.
    - dev eth0 — interface used.
    - lladdr — neighbor's MAC address.
    - REACHABLE — Linux recently confirmed the neighbor's reachability at the link layer.
    You may also see states such as STALE or INCOMPLETE. These are clues, not automatic proof of a network failure.


Practical C — Capture packets with tcpdump

First, check whether it is installed:

    tcpdump --version


Then, if available and you have the necessary permissions, capture ICMP traffic on eth0:

    sudo tcpdump -ni eth0 icmp


In a second terminal, run:

    ping -c 4 8.8.8.8


Stop the capture with Ctrl+C.

You may see Echo Requests going out and Echo Replies coming back. If requests appear but replies don't, the capture tells you that replies aren't visible on that interface; it doesn't, by itself, tell you exactly where they were lost.

If your interface isn't eth0, find the correct one using:

    ip -br addr


Day 4 — Linux networking checks for DevOps
-------------------------------------------

* Today, we'll focus on the commands DevOps engineers commonly use when an application is running but users cannot access it.
* Imagine you deployed an application on an EC2 instance or in Kubernetes. The deployment succeeded, but the application URL isn't working. How do you find the problem?

* We'll investigate three things:
  
        1. Port: Is the application listening?
        2. Connectivity: Can we connect to the application?
        3. DNS: Does the hostname resolve to an IP address?

#Check listening ports with ss — 10 minutes

Run this on a Linux machine:
 
     sudo ss -lntp

What the options mean:

    - -l — show listening sockets.
    - -n — show numeric addresses and ports.
    - -t — show TCP sockets.
    - -p — show process information where permissions allow.
Example output:

    LISTEN 0  128  0.0.0.0:8080  0.0.0.0:*  users:(("java",pid=1234,fd=45))


* This tells us a Java process is listening on TCP port 8080 on all IPv4 interfaces.

Compare that with:

    LISTEN 0  128  127.0.0.1:8080  0.0.0.0:*  users:(("java",pid=1234,fd=45))


Here, the application listens only on the loopback interface. Other machines generally cannot connect to it directly through the instance's network IP.

DevOps takeaway: Before changing security groups or restarting servers, check whether the application is listening on the expected port and address.

#Test the application with curl — 10 minutes

curl tests the actual HTTP or HTTPS connection, unlike ping, which tests ICMP.

Suppose your application should be available on port 8080.

Run this on the same machine:

    curl -v --connect-timeout 5 http://127.0.0.1:8080/


    - -v — displays connection details.
    - --connect-timeout 5 — limits the time spent establishing the connection.
    - 127.0.0.1 — the local machine.
    - 8080 — the example application port.

    Result	                Possible meaning
    HTTP 200	            Request succeeded.
    HTTP 404	            Server responded, but the requested path wasn't found.
    HTTP 500	            Server responded with an internal error.
    Connection refused	    Nothing accepted the connection at that address and port, or a rule actively rejected it.
    Connection timed out	Traffic may be dropped, or the destination may not be responding.
    
An HTTP error such as 404 still proves that an HTTP server responded. It doesn't necessarily mean the application is healthy.

To test a remote API, use its real URL:

    curl -v --connect-timeout 5 https://example.com/

# Check DNS with dig or nslookup — 5 minutes
Applications often connect using hostnames instead of IP addresses.
Run either command:

    dig example.com


or

    nslookup example.com


* These commands help you see whether DNS can resolve a hostname to an IP address.
* If DNS resolution fails, investigate the hostname, configured DNS resolver, and network access to DNS. If DNS works, the application connection can still fail for other reasons.
* example.com is just a demonstration URL. Use your application's actual endpoint when troubleshooting.
