# AD HomeLab



![](https://i.imgur.com/wEhm3hY.png)


For this lab we need two VMs:
- Domain Controller(Windows Server 19)
- Client (Windows 10)

You can use VMware or Virtualbox to create the VMs.
We will go through the configuration steps and run a Powershell script to create around 1000 sample users in the domain.
The domain controller will route the internal traffic to the Internet, and act as DHCP and DNS server for the clients.
It will have two NICs, one for the outbound connection to the Internet, and a second for connecting with the internal network. 


# Install Windows Server 2019

[Download Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
In VMware assign the domain two NICs:

![|300](https://i.imgur.com/zpcqIhT.png)


![](https://i.imgur.com/GmuImFm.png)


Choose "Custom Install":

![](https://i.imgur.com/RIxd4Cp.png)


![](https://i.imgur.com/99zqrJU.png)

After successful installation, run setup.exe from the VM's DVD drive for a better User experience, like a proper resolution to maximize the screen.

Look for the 2 Network adapters and rename them.
One for the Internet connection I call "Internet," and the second one for the internal connection I name "X_Internal_X".

For the internal one, set the IP address to: `172.16.0.1`

![|400](https://i.imgur.com/XDQ06X4.png)

Rename the PC:

![|500](https://i.imgur.com/JNMUmzs.png)



# Install AD Domain Services

Go to the Sever Manager Dashboard and click on "Add Roles and Features":

![|600](https://i.imgur.com/Hbo6LCP.png)


Go through the installation process. At server roles, select "Active Directory Domain Services".

![](https://i.imgur.com/dJnPi6Y.png)


Promote this server to a domain controller.
![](https://i.imgur.com/IIYi6PQ.png)



# Configuring the domain controller

Create a new domain:

![](https://i.imgur.com/00osq6u.png)

Continue the installation with default settings.

Login into the Domain Admin Account:

![|500](https://i.imgur.com/jqtmerq.png)


Open "Active Directory Users and Computers," which is located in the Start menu.

Create a new "Organizational Unit" in our Domain:

![](https://i.imgur.com/13lZxhD.png)


Create a new User in our Domain:

![](https://i.imgur.com/FoYygcJ.png)


![|400](https://i.imgur.com/tt2szk8.png)

Once the User is created, we add him to the Domain Admin Group.
Right-click on the User and open the "Properties". In the "Member of" tab, you can enter a  new object:

![](https://i.imgur.com/rBegg05.png)


Acknowledge and sign off the current user, then login with the newly created admin account.

![|400](https://i.imgur.com/2eMBlav.png)


The machines inside our network should have Internet access through the domain controller. We have to install RAS/NAT on the domain controller to do that.

Inside the Server Manager Dashboard, click on "Add roles and features" and at server roles, select "Remote Access":

![|500](https://i.imgur.com/VPuqMxB.png)


Then select "Routing":

![|500](https://i.imgur.com/PK7r0rA.png)

After finishing, proceed to Tools and click on "Routing and Remote Access":

![|300](https://i.imgur.com/zmpLo4i.png)


Select "Configure and Enable Routing and Remote Access":

![|500](https://i.imgur.com/TvLvz2r.png)

Then choose NAT, because we have to configure it.

![|400](https://i.imgur.com/pREosnX.png)


Select the public network interface accordingly.

![|500](https://i.imgur.com/n64ShU1.png)


We also have to install DHCP on the Domain Controller, so that the clients get IP addresses. Go to Server Manager Dashboard again and add the "DHCP Server" role:

![](https://i.imgur.com/cFlOpsw.png)


After that, go to Tools and select DHCP. We define here the scope of the IP addresses to be given by the Domain Controller.
Right-click on IPv4 and select "New Scope":

![](https://i.imgur.com/Ah7qIqR.png)

Choose a starting and ending IP for the address pool.

![|400](https://i.imgur.com/oeYnaJM.png)


The lease time is how long a computer can have an IP address, until it has to be refreshed.
It depends on the use case. For our lab, eight days is enough.

![|500](https://i.imgur.com/we63zLN.png)

Select "yes" for configuring DHCP options.

![|500](https://i.imgur.com/biqnJH9.png)

The domain controller also has a forward function. It has to forward traffic from the clients to the Internet. In the next step, we choose the default gateway for the clients, which will be the internal NIC of the domain controller.

![|400](https://i.imgur.com/pNaL8cu.png)

Add the same IP as the DNS server, because DNS is automatically installed with AD on the Domain Controller.

After finishing, right-click on "dc.mydomain.com", select "Authorize" and then "Refresh".

Now we are ready to add Users to the domain.

Download the files from this [repository](https://github.com/joshmadakor1/AD_PS.git).
It contains a file "names.txt "with many random Usernames and a Powershell script for bulk user creation.
The script "1_CREATE_USERS.ps1" reads the Usernames into a variable, then runs a for loop for each user. Inside the for loop, the name is spitted, and a username is created, which consists of the first letter of the first name and the last name.

Example: 
Suzanne Nell --> snell

All accounts are created in the AD with the initial password of "Password1".

Copy the folder on the domain controller and open Powershell ISE as Administrator.
First, we have to enable the execution of scripts on this server.
`Set-ExecutionPolicy Unrestricted`

Navigate then to the folder where the Powershell script is located and run it.
It starts creating the Users from the file in the AD in the "Organizational Unit "\_Users_".

Now our domain controller is configured and the users are ready.


# Install Windows 10 on a client machine

[Download Windows 10](https://www.microsoft.com/en-us/software-download/windows10ISO)

During installation, select the "Windows 10 Pro" version.
In VMware, configure the network adapter on this machine to "Host-only".
The clients will only have Internet access through the domain controller.

Select the Custom Install:

![|500](https://i.imgur.com/ZNzmjvC.png)

After the installation, check your client's machine IP:

![|500](https://i.imgur.com/9N6DeRy.png)


It gets the first IP of the assigned pool, which is 172.16.0.100, and also the gateway is correct.
Now we can ping outside our local network into the Internet, to test if the domain controller routes the traffic.

![|400](https://i.imgur.com/Oq1q8Xi.png)

Great, it works!
Our domain controller not only routes the traffic, but also works as a DNS server because google.com has been resolved.

Let us change the hostname of this machine. Navigate to the system settings and click on "Rename this PC(advanced)":

![|500](https://i.imgur.com/gIz3oWp.png)


Then click on "Change":

![|300](https://i.imgur.com/sUmyxT8.png)

Enter the new Hostname and the domain we like to join, and type in the administrator password.

![|300](https://i.imgur.com/onrFJFj.png)


# Check the joined machine

Now let's go to the domain controller and look at the DHCP settings:

![](https://i.imgur.com/bLRovM0.png)

At "Address Leases," you can see that the CLIENT1 machine got an IP address and joined our domain.

Navigate to "Active Directory Users and Computers":

![](https://i.imgur.com/2xAQ2VO.png)

Here you also see that our Client has been added to the domain.

Next, try to login with a User we created by the Powershell script, to check if he can sign in to the domain:

![|400](https://i.imgur.com/idfV1et.png)
