*Cobalt Strike*

Raphael Mudge created Cobalt Strike in 2012 to enable threat-representative security tests and it was one of the first public red team command and control frameworks.  
Cobalt Strike is the go-to red team platform for many businesses consulting organisations around the world.
In 2020, Cobalt Strike was acquired by HelpSystems LLC.


Order of events to use Cobalt Strike: 

- It's recommended to run the team server in a tmux session, as this will prevent the process from closing if the SSH connection is lost.
random-machine@ubuntu ~/cobaltstrike> sudo ./teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/customevil.profile
^ ssh into new machine                                                                              ^ malleable C2 profile (customizable)
                                                                      ^ shared password for client
                                                            ^ this is our IP from random-machine

When connecting to a team server for the first time, the client will prompt you to confirm the server's fingerprint.  
This ensures that nobody is intercepting traffic between your client and the server.

Once initialized we can launch the client from the taskbar and connect again

*Listeners* 
There are two main types of listeners: egress and peer-to-peer.
An egress listener is one that allows Beacon to communicate outside of the target network to our team server.  
The default egress listener types in Cobalt Strike are HTTP/S and DNS, where Beacon will encapsulate C2 traffic over these respective protocols.

#HTTP
The HTTP listener allows Beacon to send and receive C2 messages over HTTP GET and/or POST requests.
Click the little + button next to the HTTP Hosts box.  Here, you provide the IP addresses and/or domain names that the Beacon payload will call back to.  
By default, it will auto-populate with the IP address of the Attacker Linux VM, but it's more realistic to use proper DNS names.  
There is a DNS resolver in the lab that you can use to create arbitrary DNS records, which can be accessed via the Apps menu in Snap Labs.

#DNS
The DNS listener allows Beacon to send and receive C2 messages over several lookup/response types including A, AAAA and TXT.  
TXT are used by default because they can hold the most amount of data.  
This requires we create one or more DNS records for a domain that the team server will be authoritative for.  

OPSEC CONSIDERATIONS: Since 0.0.0.0 is the default response (and also rather nonsensical), Cobalt Strike team servers can be fingerprinted in this way.  This can be changed in the Malleable C2 profile.

#Peer-to-peer (P2P)
Peer-to-peer (P2P) listeners differ from egress listeners because they don't communicate with the team server directly.  
Instead, P2P listeners are designed to chain multiple Beacons together in parent/child relationships.  The primary reasons for doing this are:

    To reduce the number of hosts talking out to your team server, as the higher the traffic volume, the more likely it is to get spotted.
    To run Beacon on machines that can't even talk out of the network, e.g. in cases of firewall rules and other network segregations.

The two P2P listener types in Cobalt Strike are Server Message Block (SMB) and raw TCP.  
It's important to understand that these protocols do not leave the target network (i.e. the team server is not listening on port 445 for SMB).  
Instead, a child SMB/TCP Beacon will be linked to an egress HTTP/DNS Beacon, and the traffic from the child is sent to the parent, which in turn sends it to the team server.


#SMB 
SMB listeners are very simple as they only have a single option - the named pipe name.  The default is msagent_## (where ## is random hex), but you can specify anything you want.  
A Beacon SMB payload will start a new named pipe server with this name and listen for an incoming connection.  This named pipe is available both locally and remotely.

OPSEC CONSIDERATIONS: This default pipe name is quite well signatured.  A good strategy is to emulate names known to be used by common applications or Windows itself.  
Use PS C:\> ls \\.\pipe\ to list all currently listening pipes for inspiration.

#TCP 
A Beacon TCP payload will bind and listen on the specified port number.  You may also specify whether it will bind to only the localhost (127.0.0.1), otherwise it will bind to all interfaces (0.0.0.0).



*PAYLOADS*

https://buffered.io/posts/staged-vs-stageless-handlers/

Payloads of different formats can be generated from the Payloads menu.  Here's a breakdown of each option:

HTML Application
Produces a .hta file (typically delivered through a browser by way of social engineering) uses embedded VBScript to run the payload.  Only generates payloads for egress listeners and is limited to x86.

MS Office Macro
Produces a piece of VBA that can be dropped into a macro-enabled MS Word or Excel document.  Only generates payloads for egress listeners but is compatible with both x86 and x64 Office.

Stager Payload Generator
Produces a payload stager in a variety of languages including C, C#, PowerShell, Python, and VBA.  These are useful when building your own custom payloads or exploits.  Only generates payloads for egress listeners, but supports x86 and x64.

Stageless Payload Generator
As above, but generates stageless payloads rather than stagers.  It has slightly fewer output formats, e.g. no PowerShell, but has the added option of specifying an exit function (process or thread).  It can also generate payloads for P2P listeners.

Windows Stager Payload
Produces a pre-compiled stager as an EXE, Service EXE or DLL.

Windows Stageless Payload
Produces a pre-compiled stageless payload as an EXE, Service EXE, DLL, shellcode, as well as PowerShell.  This is also the only means of generating payloads for P2P listeners.

Windows Stageless Generate All Payloads
Pretty much what it says on the tin.  Produces every stageless payload variant, for every listener, in x86 and x64.


*PIVOT LISTENERS*

A pivot listener can only be created on an existing Beacon, and not via the normal Listeners menu.  
These listeners work in the same way as regular TCP listeners, but in reverse.  A standard Beacon TCP payload binds to 127.0.0.1 (or 0.0.0.0) and listens for an incoming connection on the specified port.  
You then initiate a connection to it from an existing Beacon (with the connect command).  
The pivot listener works the other way around by telling the existing Beacon to bind and listen on a port, and the new Beacon TCP payload initiates a connection to it instead.

To create a pivot listener, right-click on a Beacon and select Pivoting > Listener.  This will open a "New Listener" window.


*PERSISTENT TEAM SERVER* 

Running the team server as a service allows it to start automatically when the VM starts up, which obviously saves us having to SSH in each time and start it manually.  This can be done with a systemd unit file.

First, create the file in /etc/systemd/system.

random-machine@ubuntu ~> sudo vim /etc/systemd/system/teamserver.service

[Unit]
Description=Cobalt Strike Team Server
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
WorkingDirectory=/home/attacker/cobaltstrike
ExecStart=/home/attacker/cobaltstrike/teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile

[Install]
WantedBy=multi-user.target


Next, reload the systemd manager and check the status of the service.  It will be inactive/dead.
Start the service and check its status again.

The service should be active/running and you will see the typical console output from the team server.  Now that we know the service is working, we can tell it to start on boot.
We will now be able to connect from the Cobalt Strike client as soon as the VMs boot up.








