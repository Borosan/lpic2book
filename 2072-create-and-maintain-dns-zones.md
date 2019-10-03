# 207.2. Create and maintain DNS zones

## **207.2 Create and maintain DNS zones**

**Weight:** 3

**Description:** Candidates should be able to create a zone file for a forward or reverse zone and hints for root level servers. This objective includes setting appropriate values for records, adding hosts in zones and adding zones to the DNS. A candidate should also be able to delegate zones to another DNS server.

**Key Knowledge Areas:**

* BIND 9 configuration files, terms and utilities
* Utilities to request information from the DNS server
* Layout, content and file location of the BIND zone files
* Various methods to add a new host in the zone files, including reverse zones

**Terms and Utilities:**

* /var/named/
* zone file syntax
* resource record formats
* named-checkzone
* named-compilezone
* masterfile-format
* dig
* nslookup
* host

## DNS Zones

Configuring BIND DNS server to work as a forwarder or caching DNS server was pretty easy but some times we need to map our host names to IP addresses in our own network. So we need information about zones which are not defined in any other DNS servers in the world, consequently Forwarding or Caching don't work in this scenario.

As we said We have two different types of DNS queries , forward queries and recursive queries, as a result , we need two types of zones for answering both types.

* forward zone
* reverse zone

A forward lookup zone will, when supplied with a domain name \(for example google.com\), return the IP address for the supplied domain. When we access Google, a DNS server is looking in its forward lookup zone for the IP address\(es\) of the site, and then returning them to us. This type of record is an A record, and is the most common. A forward lookup zone can contain other records, such as MX, C-Name etc. An Example of forward lookup zone:

```text
$ORIGIN example.com. 
$TTL 86400 
@    IN    SOA    dns1.example.com.    hostmaster.example.com. (
            2001062501 ; serial                     
            21600      ; refresh after 6 hours                     
            3600       ; retry after 1 hour                     
            604800     ; expire after 1 week                     
            86400 )    ; minimum TTL of 1 day  


    IN    NS    dns1.example.com.       
    IN    NS    dns2.example.com.        


    IN    MX    10    mail.example.com.       
    IN    MX    20    mail2.example.com.        


dns1    IN    A    10.0.1.1
dns2    IN    A    10.0.1.2    


server1    IN    A    10.0.1.5        
server2    IN    A    10.0.1.6


ftp    IN    A    10.0.1.3
    IN    A    10.0.1.4

mail    IN    CNAME    server1
mail2    IN    CNAME    server2


www    IN    CNAME    server1
```

A reverse lookup zone is used to lookup the domain name when supplied with an IP address. We normally supply the IP address backwards and append .in-addr.arpa .The reverse lookup zone contains PTR resource records. A PTR record allows doing a reverse lookup by pointing the IP address to a host/domain name. When doing reverse lookups, these PTR records are used to point to A resource records. An example of Reverse Zone file:

```text
$ORIGIN 1.0.10.in-addr.arpa. 
$TTL 86400 
@    IN    SOA    dns1.example.com.    hostmaster.example.com. (
            2001062501 ; serial                     
            21600      ; refresh after 6 hours                     
            3600       ; retry after 1 hour                     
            604800     ; expire after 1 week                     
            86400 )    ; minimum TTL of 1 day        


1    IN    PTR    dns1.example.com.       
2    IN    PTR    dns2.example.com.

5    IN    PTR    server1.example.com.
6    IN    PTR    server2.example.com.

3    IN    PTR    ftp.example.com.
4    IN    PTR    ftp.example.com.
```

## DNS Zone Syntax

Some syntax explanation:

One important thing to understand here is the $ORIGIN entry, which is used to make all other entires in the zone file a FQDN. FQDN stands for Fully Qualified Domain Name, and it always ends with a dot \(.\). FQDN stands for Fully Qualified Domain Name, and it always ends with a dot \(.\)

**@** shown in the above line is the NAME value for this SOA record. Using @ at this place will replace it with example.com \(as we have mentioned it in **$ORIGIN** \).

The ‘;’ character in the example above indicates that the rest of the line is a comment that should be ignored by the nameserver.

Also note the trailing dot \(‘.’\) after each record referring to a hostname. Without the dot, the nameserver appends the current zone after the record. For example, srv1.example.com would be interpreted as srv1.example.com.in-addr.arpa.

In previous lesson we had a short over view over different types of DNS records, lets explain more and see how syntax look like:

## Start of Authority \(SOA\) record

Start Of Authority \(SOA\) Record: is the first record in a properly configured zone. It contains information about the zone in a string of fields. An SOA record tells the server to be authoritative for the zone. The SOA record takes the format:

```text
<domain.name.> IN SOA <hostname.domain.name.>     <mailbox.domain.name>
                                <serial-number>
                                <refresh>
                                <retry>
                                <expire>
                                <minimum-ttl>
```

Where:

* domain.name:The name of the domain to which the SOA belongs. Instead of writing out the full domain, you can also use ‘@’ in the file to let the nameserver fill this out automatically. Example:
* IN:The class of the DNS record. ‘IN’ is an abbreviated form of ‘Internet’.
* SOA:The type of DNS record, which in this case is ‘Start of Authority’.
* hostname.domain.name: Also known as the ‘hostmaster’ field. It contains the e-mail address of the person responsible for maintaining the zone. No ‘@’ is used in this field. use "." instead. `me@example.com` would be `me.example.com` .
* serial-number: The serial number of the current version of the DNS database for this domain. If a secondary server’s number is lower than the number of the primary server, it indicates that the secondary server’s records are out of date and that it requires a zone transfer from the primary server.
* refresh : This tells a secondary server how often to poll the primary server and check for changes in the serial number field. Measured in seconds.
* retry: If a refresh attempt fails, a secondary server will retry after the interval specified in the retry field. Measured in seconds.
* expire: If the refresh and retry attempts fail, the secondary server will stop serving the zone after the period specified in the expire field. Measured in seconds.
* minimum-ttl: The default TTL \(Time To Live\) for every record in the zone. The default is only used when a particular resource record does not have its own specified TTL value. When changes are being made to a zone, the default is often set at ten minutes or less.

  Example:

```text
example.com.    IN    SOA   srv1.example.com. hostmaster.example.com. (
                              2003080800 ; sn = serial number
                              172800     ; ref = refresh = 2d
                              900        ; ret = update retry = 15m
                              1209600    ; ex = expiry = 2w
                              3600       ; nx = nxdomain ttl = 1h
                              )
```

Another Example for SOA record in reverse Zone:

```text
       28.12.202.in-addr.arpa. IN SOA srv1.example.com.    dns-admin.example.com. ( 
                                   1999040701 ;Serial number 
                                   10800 ;Refresh 
                                   3600 ;Retry 
                                   604800 ;Expire 
                                   86400) ;Minimum TTL
```

## Address Type \(A\) records

Defines a direct ' name to address' translation

```text
<hostname or FQDN> IN A <IP-Address>
```

Example:

`server1 IN A 192.168.10.111`

or it can be given relative to the current domin \( i.e: just 'server1'\)

`server1 IN A 192.168.10.111`

## Canonical Name \(CN\) records

Allows us to define a host with more than one name \(or role\) in our doman

`<alias-name> IN CNAME <real-name>`

Example:

```text
dns IN CNAME server1.example.com.
```

## Nameserver \(NS\) records

Every domain can have one \(or more\) name servers.They are defined as a NS \(name server\) record. The NS record takes the format:

```text
<domain.name.> IN NS <hostname.domain.name.>
```

and the fileds are:

* domain. name :The name of the domain to which the NS belongs. Instead of writing out the full domain, we can also use ‘@’ in the file to let the nameserver fill this out automatically. 
* IN : The class of the DNS record. ‘IN’ is an abbreviated form of ‘Internet’.
* NS: The type of DNS record, which in this case is ‘Name Server’.
* hostname.domain.name: The hostname of an authoritative server.

  Although the master name server is also defined in the SOA record, that server must have a full entry in the NS record definition

Example:

```text
@     IN   NS    server1.example.com.
```

## Mail Exchange \(MX\) records

Sending E-Mail services \(MTA- mail Transfer Agents\) have to be able to figure out which host handles emails for the zone/domain, we do this by creating one or more MX records:

```text
<domain.name.> IN MX <priority> <hostname.domain.name.>
```

Example:

```text
@    IN MX 10 mailserver1.example.com.

@    IN MX 20 mailserver2.example.com.
```

The two records define two mail servers, the number indicates the priority \(order\) they should be tried for mail delivery. They can be different numeric priorities for primary and backups OR they can be the same if intended to be used in a loas balanced configuration.

## Pointer \(PTR\) records

This record type is used in reverse lookup zone files, so that the IP address can be translated to the name.Note not all records have to have or will have a reverse lookup PTR record necessarily. The following illustrates the layout of a PTR record:

```text
<last-IP-digit>    IN    PTR    <FQDN-of-system>
```

For the address ‘192.168.10.155’, this would be:

`155.10.168.192.in-addr.arpa. IN PTR svc00.apnic.net.`

or if the SOA record for the zone already states the in-addr.arpa domain in full, we do not need to write this out again. Instead, we can only to write the last dotted quad from the address:

`155 IN PTR svc00.apnic.net.`

## Other Records Types

There are other types, but these are the most common and are the specific types we should be aware of for the lpic2 exam.

Once we have created all the Zones and reverse Zones and relevant PTR records for our in-addr.arpa zone, the next step is to load them into our name servers. but How and where to do that ?

## /var/named

Zone files contain information about a namespace and by default they are stored in the named working directory \(/var/named/\)

But it can be different based on our distro or configuration.

We use ubuntu and fresh BIND installation so Lets go back to /etc/bind to see zone files which were installed during BIND installation:

```text
root@server1:/etc/bind# ls
bind.keys  db.empty    named.conf.default-zones  zones.rfc1918
db.0       db.local    named.conf.local
db.127     db.root     named.conf.options
db.255     named.conf  rndc.key
```

* in named.conf.default-zones all root DNS Servers have been defined here.
* all files which are started with db are zone files.
* named.conf.options contains all the configuration options for the DNS server
* named.conf.local is defining our local host zones

how ever these files and their names might be differentin different distroes, But generally name.conf and other related files if exist are consist of some sections and statements which must be defined to work:

1. Options statement
2. Zones statement

### 1.Options statement

in named.conf.options file we can define system wide options:

```text
root@server1:/etc/bind# cat named.conf.options 
options {
    directory "/var/cache/bind";

    // If there is a firewall between you and nameservers you want
    // to talk to, you may need to fix the firewall to allow multiple
    // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

    // If your ISP provided one or more IP addresses for stable 
    // nameservers, you probably want to use them as forwarders.  
    // Uncomment the following block, and insert the addresses replacing 
    // the all-0's placeholder.

    // forwarders {
    //     0.0.0.0;
    // };

    //========================================================================
    // If BIND logs error messages about the root key being expired,
    // you will need to update your keys.  See https://www.isc.org/bind-keys
    //========================================================================
    dnssec-validation auto;

    auth-nxdomain no;    # conform to RFC1035
    listen-on-v6 { any; };
};
```

There are many options which can be defined here some of them are:

* allow-query : Specifies which hosts are allowed to query this nameserver. By default, all hosts are allowed to query. An access control list, or collection of IP addresses or networks, may be used here to allow only particular hosts to query the nameserver.
* allow-recursion: Similar to allow-query, this option applies to recursive queries. By default, all hosts are allowed to perform recursive queries on the nameserver.
* blackhole :Specifies which hosts are not allowed to query the server.
* directory: Specifies the named working directory if different from the default value, /var/named/.
* forwarders: Specifies a list of valid IP addresses for nameservers where requests should be forwarded for resolution.
* forward: Specifies the forwarding behavior of a forwarders directive. The following options are accepted:
* first — Specifies that the nameservers listed in the forwarders directive be queried before named attempts to resolve the name itself.
* only — Specifies that named does not attempt name resolution itself in the event that queries to nameservers specified in the forwarders directive fail.
* listen-on :Specifies the network interface on which named listens for queries. By default, all interfaces are used.Using this directive on a DNS server which also acts a gateway, BIND can be configured to only answer queries that originate from one of the networks.The following is an example of a listen-on directive:

```text
options {    
    listen-on { 10.0.1.1; }; 
};
```

So , only requests that arrive from the network interface serving the private network \(10.0.1.1\) are accepted.

* notify :Controls whether named notifies the slave servers when a zone is updated. It accepts the following options:
* yes — Notifies slave servers.
* no — Does not notify slave servers.
* explicit — Only notifies slave servers specified in an also-notify list within a zone statement.

and many many more sophisticated options.

### 2.Zones statement

usually zone ifnormation are defined in named.conf.local file, lets vim named.conf.local :

```text
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
"named.conf.local" 8L, 165C                                   1,1           All
```

As it says we need to define our local zones here, but unfortunately there is no predefined configuration! How is the syntax?

A zone statement defines the characteristics of a zone, such as the location of its configuration file and zone-specific options. This statement can be used to override the global options statements.  
A zone statement takes the following form:

```text
zone <zone-name> <zone-class> {      
    <zone-options>;      
    [<zone-options>; ...] 
};
```

In this statement, &lt;zone-name&gt; is the name of the zone, &lt;zone-class&gt; is the optional class of the zone, and &lt;zone-options&gt; is a list of options characterizing the zone.

The &lt;zone-name&gt; attribute for the zone statement is particularly important. It is the default value assigned for the $ORIGIN . The named daemon appends the name of the zone to any non-fully qualified domain name listed in the zone file.

#### The most common zone statement options include the following:

* allow-query :Specifies the clients that are allowed to request information about this zone. The default is to allow all query requests.
* allow-transfer :Specifies the slave servers that are allowed to request a transfer of the zone's information. The default is to allow all transfer requests.

## DNS Dynamic updates

While the records could be manually entered as static DNS records, it would be ideal if the process were more "dynamic". When thinking of the security, it will be very, very stupid to allow anybody to update records.Most servers simply don't allow dynamic updates and those who do, don't allow it for all zones, or allow updating a zone from specific subnet or specific IP-addresses.

For the most part, dynamic update functionality is used by programs like DHCP servers that assign IP addresses automatically to computers and then need to register the resulting name-to-address and address-to-name mappings.

It's also possible to create updates manually with the command-line program nsupdate, which is part of the standard BIND distribution.

* allow-update :Specifies the hosts that are allowed to dynamically update information in their zone. The default is to deny all dynamic update requests.Be careful when allowing hosts to update information about their zone. Do not enable this option unless the host specified is completely trusted. In general, it is better to have an administrator manually update the records for a zone and reload the named service.
* file :Specifies the name of the file in the named working directory that contains the zone's configuration data.
* masters :Specifies the IP addresses from which to request authoritative zone information and is used only if the zone is defined as type slave.
* notify :Specifies whether or not named notifies the slave servers when a zone is updated. This directive accepts the following options:
* yes — Notifies slave servers.
* no — Does not notify slave servers.
* explicit — Only notifies slave servers specified in an also-notify list within a zone statement.
* type:Defines the type of zone.Below is a list of valid options:
* delegation-only — Enforces the delegation status of infrastructure zones such as COM, NET, or ORG. Any answer that is received without an explicit or implicit delegation is treated as NXDOMAIN. This option is only applicable in TLDs or root zone files used in recursive or caching implementations.
* forward — Forwards all requests for information about this zone to other nameservers.
* hint — A special type of zone used to point to the root nameservers which resolve queries when a zone is not otherwise known. No configuration beyond the default is necessary with a hint zone.
* master — Designates the nameserver as authoritative for this zone. A zone should be set as the master if the zone's configuration files reside on the system.
* slave — Designates the nameserver as a slave server for this zone. Also specifies the IP address of the master nameserver for the zone.
* zone-statistics :Configures named to keep statistics concerning this zone, writing them to either the default location \(/var/named/named.stats\) or the file listed in the statistics-file option in the server statement

## Implementing Basic BIND DNS Server

With that, Lets create forward and reverse zones for imaginary zone, named "myzone":

```text
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";


zone "myzone" {
    type master;
    file "/etc/bind/zonedbfiles/db.myzone";
        };

zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zonedbfiles/db.10.168.192";
        };
```

as you see we also declared where BIND daemon can find zone db files.To create zone db files we use existing samples:

```text
root@server1:/etc/bind# mkdir zonedbfiles

root@server1:/etc/bind# cp db.local zonedbfiles/db.myzone
root@server1:/etc/bind# cp db.127 zonedbfiles/db.10.168.192

root@server1:/etc/bind# cat zonedbfiles/db.myzone 
;
; BIND data file for local loopback interface
;
$TTL    604800
@    IN    SOA    localhost. root.localhost. (
                  2        ; Serial
             604800        ; Refresh
              86400        ; Retry
            2419200        ; Expire
             604800 )    ; Negative Cache TTL
;
@    IN    NS    localhost.
@    IN    A    127.0.0.1
@    IN    AAAA    ::1

root@server1:/etc/bind# cat zonedbfiles/db.10.168.192 
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@    IN    SOA    localhost. root.localhost. (
                  1        ; Serial
             604800        ; Refresh
              86400        ; Retry
            2419200        ; Expire
             604800 )    ; Negative Cache TTL
;
@    IN    NS    localhost.
1.0.0    IN    PTR    localhost.
```

and now we need to edit these two useful samples, lets start with db.myzone:

```text
root@server1:/etc/bind# cat zonedbfiles/db.myzone 
;
; BIND data file for local loopback interface
;
$TTL    604800
@    IN    SOA    myzone. root.myzone. (
                  4        ; Serial
             604800        ; Refresh
              86400        ; Retry
            2419200        ; Expire
             604800 )    ; Negative Cache TTL
;
@    IN    NS    ns.myzone.
@    IN    A    192.168.10.129
host2    IN    A    192.168.10.151
```

to verify zone file:

```text
root@server1:/etc/bind/zonedbfiles# named-checkzone myzone. db.myzone 
zone myzone/IN: loaded serial 4
OK
```

and next db.10.168.192 .Please note that there is not a one by one relationship between Forward Lookup zone and Reverse Lookup zone, so usually CNAME records are not defined in reverse zone:

```text
root@server1:/etc/bind# cat zonedbfiles/db.10.168.192 
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@    IN    SOA    myzone. root.myzone. (
                  8        ; Serial
             604800        ; Refresh
              86400        ; Retry
            2419200        ; Expire
             604800 )    ; Negative Cache TTL
;
@    IN    NS    ns.myzone.
151    IN    PTR    host2.myzone.
```

to check the reverse zone:

```text
root@server1:/etc/bind/zonedbfiles# named-checkzone 10.168.192.in-addr.arpa db.10.168.192
zone 10.168.192.in-addr.arpa/IN: loaded serial 8
OK
```

also we cloud have used `named-checkconf` to check the configurations:

```text
root@server1:/etc/bind# named-checkconf
root@server1:/etc/bind#
```

and finally we use rndc to reload configurations without losing cache:

```text
root@server1:/etc/bind# rndc reload
server reload successful
```

Finally to check the results:

```text
root@server1:/etc/bind# dig @localhost myzone

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost myzone
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51897
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;myzone.                IN    A

;; ANSWER SECTION:
myzone.            604800    IN    A    192.168.10.129

;; AUTHORITY SECTION:
myzone.            604800    IN    NS    myzone.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Tue Mar 06 03:59:53 PST 2018
;; MSG SIZE  rcvd: 65
```

and

```text
root@server1:/etc/bind# dig @localhost host2.myzone

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost host2.myzone
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46790
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;host2.myzone.            IN    A

;; ANSWER SECTION:
host2.myzone.        604800    IN    A    192.168.10.151

;; AUTHORITY SECTION:
myzone.            604800    IN    NS    myzone.

;; ADDITIONAL SECTION:
myzone.            604800    IN    A    192.168.10.129

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Tue Mar 06 04:00:40 PST 2018
;; MSG SIZE  rcvd: 87
```

and to check reverse lookup:

```text
root@server1:/etc/bind/zonedbfiles# dig  @localhost -x 192.168.10.151

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost -x 192.168.10.151
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17888
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;151.10.168.192.in-addr.arpa.    IN    PTR

;; ANSWER SECTION:
151.10.168.192.in-addr.arpa. 604800 IN    PTR    host2.myzone.

;; AUTHORITY SECTION:
10.168.192.in-addr.arpa. 604800    IN    NS    ns.myzone.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Tue Mar 06 03:55:44 PST 2018
;; MSG SIZE  rcvd: 99
```

## Implementing Master / Slave BIND DNS servers

In previous Lesson we talked about master / slave DNS server and zone transfer, Let implement that with two ubuntu servers.

We have server1\(192.168.10.129\) as a master DNS server and server2\(192.168.10.151\) as a slave

```text
root@server1:~# hostname
server1
root@server1:~# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:03:64:0d brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.129/24 brd 192.168.10.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe03:640d/64 scope link 
       valid_lft forever preferred_lft forever
```

Lets start modifying name.conf.local to allow zone transfer in master server by adding `allow-transfer` :

```text
root@server1:/etc/bind# cat named.conf.local 
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "myzone" {
    type master;
    file "/etc/bind/zonedbfiles/db.myzone";
    allow-transfer    {192.168.10.151; };
};

zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zonedbfiles/db.10.168.192";
};
```

To check bind configuration for errors:

```text
root@server1:/etc/bind# named-checkconf named.conf.local
```

also we need to define both Name servers in Zone cofiguration file:

```text
root@server1:/etc/bind# cd zonedbfiles/
root@server1:/etc/bind/zonedbfiles# cat db.myzone 
;
; BIND data file for local loopback interface
;
$TTL    604800
@    IN    SOA    myzone. root.myzone. (
                 48        ; Serial
             604800        ; Refresh
              86400        ; Retry
            2419200        ; Expire
             604800 )    ; Negative Cache TTL
;
@    IN    NS    ns1.myzone.
@    IN    NS    ns2.myzone.
@    IN    A    192.168.10.129
host2    IN    A    192.168.10.151
ns1    IN    A    192.168.10.129
ns2    IN    A    192.168.10.151
```

Okey lets go to the slave side:

```text
root@server2:~# apt install bind9
root@server2:~# cd /etc/bind
root@server2:/etc/bind# ls
bind.keys  db.empty    named.conf.default-zones  zones.rfc1918
db.0       db.local    named.conf.local
db.127     db.root     named.conf.options
db.255     named.conf  rndc.key

root@server2:/etc/bind# vim named.conf.local 
root@server2:/etc/bind# cat named.conf.local 
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "myzone" {
    type slave;
    masters {192.168.10.129; };
    file "db.myzone";
    };
```

and check for bind service if its not running :

```text
root@server2:/etc/bind# systemctl start bind9.service 
root@server2:/etc/bind# systemctl status bind9.service 
● bind9.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/bind9.service; enabled; vendor preset: en
  Drop-In: /run/systemd/generator/bind9.service.d
           └─50-insserv.conf-$named.conf
   Active: active (running) since Wed 2018-03-07 00:32:19 PST; 1s ago
     Docs: man:named(8)
 Main PID: 5793 (named)
   CGroup: /system.slice/bind9.service
           └─5793 /usr/sbin/named -f -u bind
```

But hey stop! bind can tranfer zones just if required ports are opened in the firewall.

## BIND9 Ports

UDP 53 and TCP 53 ports need to be open for the Domain Name Service \(DNS BIND server\) to function properly:

* UDP port 53 – This is primarily used by clients to make dns queries which are less than or equal to 512 byes. If the DNS server response data exceeds 512 bytes, the UDP query will fail and client will retry using TCP port 53.
* TCP port 53 – This is used to get when response data exceeds 512 bytes. The zone trasfer between master and slave is also done using TCP port 53.

to open tcp and udp port 53 on both side:

```text
root@server1:/etc/bind/zonedbfiles# iptables -I INPUT -p udp --dport 53 -m state --state NEW -j ACCEPT
root@server1:/etc/bind/zonedbfiles# iptables -I INPUT -p tcp --dport 53 -m state --state NEW -j ACCEPT
root@server1:/etc/bind/zonedbfiles# systemctl restart ufw.service
```

and on slave server:

```text
root@server2:/etc/bind# iptables -I INPUT -p tcp --dport 53 -m state --state NEW -j ACCEPT
root@server2:/etc/bind# iptables -I INPUT -p udp --dport 53 -m state --state NEW -j ACCEPT
root@server2:/etc/bind# systemctl restart ufw.service
```

to check weather port 53 is open and ready, telnet each side from other side:

```text
root@server1:/etc/bind/zonedbfiles# telnet 192.168.10.151 53
Trying 192.168.10.151...
Connected to 192.168.10.151.
Escape character is '^]'.
```

and the other side:

```text
root@server2:/etc/bind# telnet 192.168.10.129 53
Trying 192.168.10.129...
Connected to 192.168.10.129.
Escape character is '^]'.
```

let cat /var/log/syslog to see what has happened:

```text
Mar  7 00:52:28 server2 named[5793]: client 192.168.10.129#50324: received notify for zone 'myzone'
Mar  7 00:52:28 server2 named[5793]: zone myzone/IN: notify from 192.168.10.129#50324: serial 48
Mar  7 00:52:28 server2 named[5793]: zone myzone/IN: Transfer started.
Mar  7 00:52:28 server2 named[5793]: transfer of 'myzone/IN' from 192.168.10.129#53: connected using 192.168.10.151#43119
Mar  7 00:52:28 server2 named[5793]: zone myzone/IN: transferred serial 48
Mar  7 00:52:28 server2 named[5793]: transfer of 'myzone/IN' from 192.168.10.129#53: Transfer status: success
Mar  7 00:52:28 server2 named[5793]: transfer of 'myzone/IN' from 192.168.10.129#53: Transfer completed: 1 messages, 8 records, 207 bytes, 0.002 secs (103500 bytes/sec)
Mar  7 00:52:28 server2 named[5793]: zone myzone/IN: sending notifies (serial 48)
Mar  7 00:52:34 server2 named[5793]: received control channel command 'reload'
Mar  7 00:52:34 server2 named[5793]: loading configuration from '/etc/bind/named.conf'
Mar  7 00:52:34 server2 named[5793]: reading built-in trusted keys from file '/etc/bind/bind.keys'
```

To verify if our slave is working properly or not:

```text
root@server2:/var/log# dig @localhost host2.myzone

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost host2.myzone
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40457
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;host2.myzone.            IN    A

;; ANSWER SECTION:
host2.myzone.        604800    IN    A    192.168.10.151

;; AUTHORITY SECTION:
myzone.            604800    IN    NS    ns2.myzone.
myzone.            604800    IN    NS    ns1.myzone.

;; ADDITIONAL SECTION:
ns1.myzone.        604800    IN    A    192.168.10.129
ns2.myzone.        604800    IN    A    192.168.10.151

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Mar 07 01:03:59 PST 2018
;; MSG SIZE  rcvd: 125
```

and we are done. Now we can dig slave server even if the master goes off.

```text
root@server1:/etc/bind# systemctl stop bind9.service
```

we still can query slave server:

```text
root@server2:/etc/bind# dig @localhost host2.myzone

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost host2.myzone
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46872
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;host2.myzone.            IN    A

;; ANSWER SECTION:
host2.myzone.        604800    IN    A    192.168.10.151

;; AUTHORITY SECTION:
myzone.            604800    IN    NS    ns1.myzone.
myzone.            604800    IN    NS    ns2.myzone.

;; ADDITIONAL SECTION:
ns1.myzone.        604800    IN    A    192.168.10.129
ns2.myzone.        604800    IN    A    192.168.10.151

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Mar 07 02:08:55 PST 2018
;; MSG SIZE  rcvd: 125
```

lets see where slave server has made our zone file:

```text
root@server2:/# updatedb
root@server2:/# locate db.myzone
/var/cache/bind/db.myzone
```

As we said slave server information is some how read only, let check if we can edit zone file or not:

```text
root@server2:/# cat /var/cache/bind/db.myzone
Z��LG    :�myzone)myzonerootmyzone0    :�Q�$�    :�"    :�myzone��
�8    :�myzone
                 ns1myzone
                            ns2myzone(    :�host2myzone��
�&    :�
           ns1myzone��
�&    :�
           ns2myzone��
```

Well. That is impossibel, the slave zone files are now saved in a default raw binary format. This was done to improve performance, but at the sacrifice of being able to easily view the contents of the files. it can make debugging more difficult. In order to view the raw binary content, it must be converted to text first.There is command `named-compilezone` which help us to convert raw bind zone data to text and visa versa.\(Its useful if we have challenge between different BIND servers versions also \) :

```text
root@server2:/var/cache/bind# ls
db.myzone  managed-keys.bind  managed-keys.bind.jnl
root@server2:/var/cache/bind# named-compilezone -f raw -F text -o myzone.text myzone db.myzone
zone myzone/IN: loaded serial 49
dump zone to myzone.text...done
OK

root@server2:/var/cache/bind# ls
db.myzone  managed-keys.bind  managed-keys.bind.jnl  myzone.text
root@server2:/var/cache/bind# cat myzone.text 
myzone.                          604800 IN    SOA    myzone. root.myzone. 49 604800 86400 2419200 604800
myzone.                          604800 IN    NS    ns1.myzone.
myzone.                          604800 IN    NS    ns2.myzone.
myzone.                          604800 IN    A    192.168.10.129
host2.myzone.                      604800 IN    A    192.168.10.151
ns1.myzone.                      604800 IN    A    192.168.10.129
ns2.myzone.                      604800 IN    A    192.168.10.151
```

Saving salve zone files in default raw binary format is an added layer of complexity, but if we need the microscopic performance boost, that’s the way to go. If we want BIND to use the text file format, we simply update the named.conf files for ourslave zones to include the line `masterfile-format text;` and done! for example:

```text
zone "mydomain.com" in {
                type slave;
                notify no;
                file "data/dbmydomain.com";
                masterfile-format text;
                masters { 10.100.200.10; };
        };
```

okey go back and forget these tricks, lets start master server and let them work:

```text
root@server1:/etc/bind# systemctl start bind9.service
root@server1:/etc/bind# systemctl status bind9.service
● bind9.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/bind9.service; enabled; vendor preset: en
  Drop-In: /run/systemd/generator/bind9.service.d
           └─50-insserv.conf-$named.conf
   Active: active (running) since Wed 2018-03-07 02:22:29 PST; 14s ago
     Docs: man:named(8)
  Process: 3924 ExecStop=/usr/sbin/rndc stop (code=exited, status=0/SUCCESS)
 Main PID: 4022 (named)
   CGroup: /system.slice/bind9.service
           └─4022 /usr/sbin/named -f -u bind

Mar 07 02:22:29 server1 named[4022]: zone myzone/IN: loaded serial 48
Mar 07 02:22:29 server1 named[4022]: zone 255.in-addr.arpa/IN: loaded serial 1
Mar 07 02:22:29 server1 named[4022]: zone 127.in-addr.arpa/IN: loaded serial 1
Mar 07 02:22:29 server1 named[4022]: all zones loaded
Mar 07 02:22:29 server1 named[4022]: running
Mar 07 02:22:29 server1 named[4022]: zone 10.168.192.in-addr.arpa/IN: sending no
Mar 07 02:22:29 server1 named[4022]: zone myzone/IN: sending notifies (serial 48
Mar 07 02:22:29 server1 named[4022]: client 192.168.10.151#50621 (myzone): trans
Mar 07 02:22:29 server1 named[4022]: client 192.168.10.151#50621 (myzone): trans
Mar 07 02:22:29 server1 named[4022]: client 192.168.10.151#34407: received notif
lines 1-21/21 (END)
```

now we can edit zone file in master and consequently slave server will be updated, but do not forget to increase serial number. \(We have edited host2 to host3 with ip address 153\) :

```text
root@server1:/etc/bind/zonedbfiles# cat db.myzone 
;
; BIND data file for local loopback interface
;
$TTL    604800
@    IN    SOA    myzone. root.myzone. (
                 49        ; Serial
             604800        ; Refresh
              86400        ; Retry
            2419200        ; Expire
             604800 )    ; Negative Cache TTL
;
@    IN    NS    ns1.myzone.
@    IN    NS    ns2.myzone.
@    IN    A    192.168.10.129
host3    IN    A    192.168.10.153
ns1    IN    A    192.168.10.129
ns2    IN    A    192.168.10.151

root@server1:/etc/bind/zonedbfiles# systemctl restart bind9.service
```

and dig slave for changes:

```text
root@server2:/# dig @localhost host3.myzone

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @localhost host3.myzone
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2016
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;host3.myzone.            IN    A

;; ANSWER SECTION:
host3.myzone.        604800    IN    A    192.168.10.153

;; AUTHORITY SECTION:
myzone.            604800    IN    NS    ns2.myzone.
myzone.            604800    IN    NS    ns1.myzone.

;; ADDITIONAL SECTION:
ns1.myzone.        604800    IN    A    192.168.10.129
ns2.myzone.        604800    IN    A    192.168.10.151

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Mar 07 02:28:46 PST 2018
;; MSG SIZE  rcvd: 125
```

also we can use dig AXFR @masterdnsserver yourdomain.com to complete zone transfer if its allowed . That is all we are done!

