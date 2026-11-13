What is the DNS ?
=================

DNS: is the Domine Name System.

* It will act like a PhoneBook to our Internet.
* If i serch for a google.com in the browser then this browser will check the IP address of the Google.con in the DNS.
* If the Ip was foung it will send it back to browser. so the browser will know this is the Ip addrees of the domine. i have to forword this
  request to thet server.

* these above statements are simple but in real world it not that much easy. why because we have lot of websites and we have lot of users over the
  world.
* one dns server will not bare the enire load.
* and another issue every time when we are requesting the website everytime it will always go to DNS and do all the things mentioned above.
* due that we experions a latency.

* To over come this issue we have catche. DNS hava catche, our browser have a catche, our os have a catche.
* Means only the first time when we are accessing the website will do check with DNS server and for rest all the 2, 3, etc access request will
  use these catches.

* one of the issue was resolved with the help of catche. But how we can trust a single server for billions of request.
* we can't trust a single server.
* in real world we have 13 root ns servers to accepts these internet requests.
* these 13 will acts like a load balancer internally they have some many servers.

* if you make a request for a particualr wibsite then the request will to near by root NS.

* Take an eg:
  www.Aravind.dev

  www --> Subdomine
  .dev --> is top level domine

  * we have a separate servers for Top Level domines.
  * For .com we have one server , for .dev we have one server, for .org we have one server.
  * In this case my top level domine is .dev so the Root NS will route the request to .dev server.
  * Then the request will go From Top level server to a Autherity name server like go dady, route53, google domine.
  * and there it will request will resolve.



  * In DNS we have some types of records:
    -------------------------------------
    1. A record  ---> IPV4 Address
    2. AAAA record ---> IPV6 Address
    3. C Name Records
    4. NS Records -- learn in feature
    5. HTTPS Records
       and a lot more

   1. A record: if you add A record in DNS then this A records will forword our request to a backend server based up on the Domine name.
 
      Type : A
      Name: @ --> root domine with out any subdomine
      IPV4 Address: our backedn server IP.

 2. C Name: By using this C Name records we will point our SUB-domines to our root Domine.
    Eg:
    my domine is aravind.dev
    Type: CNAME
    in the C name records will add a subdomine Magic.aravind.dev
    then this subdomine point out to our root domine aravind.dev A Record.
    Another Eg:

    1. I have a domine aravind.com
    2. this domine is managed By Pavani.com
    3. so if i hard coded the Pavni.com  A record IP address like in my domine aravind.com By using A record.
    4. due to some reasons the pavani.com was changed there A record IP addess.
    5. it will causes who are the uses are accessing aravind.com will face down time because still i hava a old record of the Pavani.com.
    6. so to avoid these thing i will use CNAME for this
    7. now in my aravind.com i will use CNAME record to point it out to pavani.com
    8. so in future if they change the IP A record of the pavani with the help of CNAME I WILL access the it.
   
  3. NS records:
     -----------
     1. if i use this it will tell for my domine and my sub domines will be managed by ns record.

     Eg:
     1, i have a domine aravind.com and for thet i have a neameing server NS record like ns.someting.com
     2. i have some  subdomines eg: pav.aravind.com , z.aravind.com, user.aravind.com for my aravind.com
    
     3. in my aravind.com i have a record like:
     4. NS       aravind.com             ns.something.com

     5. if some one try to access the my subdomine pav.aravind.com first it will see the root domine in my sub domine aravind.com
     6. then it will find the NS record in my domine and it will point it out to ns.something.com
     7. Now this ns.something have a A record with an IP called 1.2.3.4 but hear we can't resolve the domine name with this IP of ns.something.com
     8. this ns.something.com have a server which is running on Port 53 with UPD exposed with the help of this server will resolve our domine name and get the IP          address of my sub-dome.
     9. it's just like creating our own DNS server.

    

* if you want to add some DNS records loacally we have to add them in /ect/hosts file.
* these will like a A records in localy
* second file /etc/resolve.conf
* this file will acts like DNS settings who will resolve intenally my domine name.
* you will see your router IP hear.
* you'r router will do domine resolving.
      
      
