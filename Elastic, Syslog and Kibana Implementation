# Choose your server type
For this demonstration, we're using Ubuntu server for the SIEM which will host Elastic, Kibana and Syslog.
It is assumed you've installed and set up the Ubuntu server using the information from the CSOC Prototype Setup written by Mark/Steve.

## Assumed VM settings:
Name: Ubuntu SIEM
ISO Image: The Ubuntu Server ISO you downloaded
Type: Linux
Version: Ubuntu (64-bit)
Memory: 8 GB (to accommodate Elastic Stack requirements)
Hard Disk: 50 GB (or more, depending on expected log volume)
Skip Unattended Installation: Yes (ticked)

Configure Network Adapter: Adapter 1 only
Attached to: Host-Only Adapter
Name: VirtualBox Host-Only Ethernet Adapter, or the LAN adapter.
Leave other settings at their defaults.

## Ubuntu settings after setup

Network Connections: Use the arrow keys to select the Ethernet interface enp0s3 (or similar) and hit ENTER.
From the menu, choose Edit IPv4.
Change the IPv4 method from Automatic (DHCP) to Manual.

Fill in the following details:
Subnet	192.168.100.0/24
Address	192.168.100.200
Gateway	192.168.100.1
Name servers:	192.168.100.1
Search domains:	Leave blank
Hit Save, then Done.

Proxy Configuration: Leave blank.
Ubuntu Archive Mirror: Use default or http://mirror.aarnet.edu.au/pub/ubuntu/archive for a faster mirror.
Filesystem Setup: Choose Use an entire disk (accept defaults).
Storage Configuration: Accept defaults, and choose Continue when prompted to format the disk.

Profile Configuration:

Your name: e.g., Alan Turing
Your server's name: siem
Pick a username: eg. alan
Choose a password: Password1 (our favourite yippee)
Ubuntu Pro Setup: Skip.

SSH Setup: Use SPACE to select Install OpenSSH server, and Done.
Featured Server Snaps: Skip.
Installation Complete: Select Reboot Now and hit ENTER.

ENSURE THAT YOU HAVE A STATIC IP ASSIGNED, THIS IS IMPORTANT.

# Elastic Implementation
Once you've configured Ubuntu SIEM correctly using the above settings, you can start installing Elastic Search.

Before we can install the Elasticseach package, we first need to add the Elastic APT repository to our Ubuntu server.

First, ensure we have the apt-transport-https package installed so we can download packages over HTTPS:
`sudo apt install apt-transport-https -y`

Next, we must import the Elastic public signing key:
`wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg`

And add the Elastic APT repository to our sources list. This will allow us to install the Elasticsearch package using the package manager:
`echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list`

Now that we have added the Elastic APT repository, we can update and install the Elasticsearch package.
`sudo apt update && sudo apt install elasticsearch`

After the installation, you will be presented with Security autoconfiguration information, snd shown the passsword for the elastic user, for example:

The generated password for the elastic built-in superuser is : ZSei6_oQIN_+ZCVQIMUJ
**Make a note of this password, as you will need it to access the Elasticsearch instance.**
If you missed the password, you can reset it by running the following command:
`/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic`

To make it easier to run Elasticsearch commands, we can add the Elasticsearch binaries to the system path. To do this, we need to edit the .bashrc file in the home directory:
`nano ~/.bashrc`

Add the following line to the end of the file:
`export PATH=$PATH:/usr/share/elasticsearch/bin`
Then save and exit the file with CTRL + S, then CTRL + X, and reload the .bashrc file:
`source ~/.bashrc`

This will add the Elasticsearch binaries to the system path, allowing you to run commands like elasticsearch-reset-password from anywhere.

Now that we have installed Elasticsearch, we need to modify a few settings before first launch. When Elasticsearch starts, it will automatically generate configuration files in the /etc/elasticsearch/ directory containing the hostname of the server.
For this project, you are going to use a fully qualified domain name (FQDN) as the hostname of your SIEM server, and use it both to connect to the server and to secure the communication between Elasticsearch and Kibana via SSL certificates.

To choose an FQDN, you can use the following format:

siem.<yourinitialshere>.local
For example, if your name is Alan Turing, you would use siem.at.local.

To set your SIEM server's hostname, run the following command:
`sudo hostnamectl set-hostname siem.<initials>.local`

Replacing <initials> with your own initials.
You will also need to add an entry to the /etc/hosts file to map the hostname to the server's IP address so that the server can resolve its own hostname:
`echo "192.168.100.200 siem.<initials>.local" | sudo tee -a /etc/hosts`

To check that the hostname has been set correctly, ping the hostname you have just set:
`ping -c 4 siem.<yourinitialshere>.local`

Finally, reboot the server to ensure that the changes take effect:
`sudo reboot`

Before we can run Elasticsearch, we need to make a few changes to the configuration file.
Because we have only a single Elasticsearch node, we need to configure it to run in a single-node cluster. To do this, we need to add the discovery.type setting to the configuration file, and set it to single-node:
`echo "discovery.type: single-node" | sudo tee -a /etc/elasticsearch/elasticsearch.yml`

Then we need to comment-out the cluster.initial_master_nodes setting, as it is not required for a single-node cluster:
`sed -i '/^cluster.initial_master_nodes:/s/^/#/' /etc/elasticsearch/elasticsearch.yml`

Next, we'll add a static DNS entry for the server's FQDN in OPNSense, and later in Windows Server, to ensure that the server can be accessed by its FQDN from other machines on the network.
To do this, log into OPNSense, and navigate to Services 🞂 Unbound DNS 🞂 Overrides 🞂 Host Overrides. Click Add and fill in the following details:

Option	Value
Host	SIEM
Domain	[initial].local
Type	A
IP Address	192.168.100.200
Description	Optional
Then click Apply.

You should now be able to ping the Elasticsearch server using the FQDN:
`ping -c 4 siem.<initials>.local`
Note: If you are unable to ping the server using the FQDN, ensure that the DNS entry has been added correctly, and that the client machine is using the OPNSense firewall as its DNS server.

Currently, Elasticsearch is installed but not running, nor will it start automatically on boot. To enable and start the Elasticsearch service, run the following commands:
`sudo systemctl enable elasticsearch --now`

You can check the status of the Elasticsearch service to ensure that it is running:
`sudo systemctl status elasticsearch`

Then test that Elasticsearch is running correctly by sending a request to the server:
`curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic:<PASSWORD> https://localhost:9200`
Where <PASSWORD> is the password you received during the installation. It should return a JSON response with information about the Elasticsearch server, including the version number, build number, and other details:

{
    "name": "siem",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "v8FXK93vQLidL4jhUdXSdg",
    "version": {
        "number": "8.16.0",
        "build_flavor": "default",
        "build_type": "deb",
        "build_hash": "12ff76a92922609df4aba61a368e7adf65589749",
        "build_date": "2024-11-08T10:05:56.292914697Z",
        "build_snapshot": false,
        "lucene_version": "9.12.0",
        "minimum_wire_compatibility_version": "7.17.0",
        "minimum_index_compatibility_version": "7.0.0"
    },
    "tagline": "You Know, for Search"
}
If you seee the above output, Elasticsearch is running correctly.

⚠️ Exit Code 137: Depending on your setup, Elasticsearch may fail to start with an Exit Code 137, which is insufficient memory. If this occurs, you may need to increase the amount of memory allocated to the VM, or reduce the memory requirements of Elasticsearch by creating a file in the /etc/elasticsearch/jvm.options.d/ directory with the following contents:

-Xms4g
-Xmx4g
To do this in a single line:
`echo -e "-Xms4g\n-Xmx4g" | sudo tee /etc/elasticsearch/jvm.options.d/jvm_memory.options`

This will limit the amount of memory allocated to Elasticsearch to 4GB, which should be sufficient for a single-node cluster.


Now that Elasticsearch is installed and running, we can install Kibana. Kibana is a web interface that allows you to visualise and interact with the data stored in Elasticsearch.
To install Kibana, run the following command:
`sudo apt install kibana`

We will also need to configure several options in the Kibana configuration file. To do this, open the Kibana configuration file in a text editor:
`sudo nano /etc/kibana/kibana.yml`

And modify the following settings in the file:

server.host: "siem.<initials>.local"
server.publicBaseUrl: "https://siem.<initials>.local:5601"
Replace <initials> with your own initials.

You can also do this using sed:
`sed -i 's/#server.host: "localhost"/server.host: "siem.<initials>.local"/' /etc/kibana/kibana.yml`
`sed -i 's|#\s*server.publicBaseUrl:.*|server.publicBaseUrl: "https://siem.<initials>.local:5601"|' /etc/kibana/kibana.yml`
Again, replace <initials> with your own initials. Then save and exit the file with CTRL + S, then CTRL + X.

Create Enrollment Token
Once installed, and with the configuration file updated, we need to generate an enrollment token to allow Kibana to connect to Elasticsearch securely.This token is generated by Elasticsearch and can be retrieved using the following command:
`/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana`

or, if you have added the Elasticsearch binaries to your path:
`elasticsearch-create-enrollment-token -s kibana`

This will output a token that you can use to connect Kibana to Elasticsearch. Make a note of this token.

# Kibana Implementation

Now, we can start and enable the Kibana service to ensure that it starts automatically on boot, as we did with Elasticsearch:
`sudo systemctl enable kibana --now`

And check the status of the Kibana service:
`sudo systemctl status kibana`

If Kibana is running correctly, you should see a message indicating that the service is active and running. If you see an error message, use journalctl to view the logs and diagnose the issue, and check the configuration file for any errors.

Open a web browser and navigate to http://siem.<initials>.local:5601. You should see the Kibana login page, where you must paste the enrollment token you generated earlier, and click Configure Elastic.
You will then be prompted to input a verification code. To generate one that you can use, run the following command:
`sudo /usr/share/kibana/bin/kibana-verification-code`

Then copy the code and paste it into the verification code field in the browser, and click Verify.
After the setup is complete, you can now log in as the elastic user with the password you received during the Elasticsearch installation. If you have forgotten the password, you can reset it using the following command:
`/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic`

To secure communication between Elasticsearch and Kibana, we generate self-signed certificates.
To do this, run the following commands one at a time:

Set the FQDN, replacing <initials> with your own initials
`export FQDN='siem.<initials>.local'`

Generate the CA certificate
`sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca`

Generate the Kibana certificate and key. Make sure FQDN is set above.
`sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert --pem --ca elastic-stack-ca.p12 --dns $FQDN,elastic.${FQDN#*.},${FQDN%%.*},192.168.100.200 --out kibana-server.zip`

Unzip the certificates and move them to the correct location
`sudo unzip /usr/share/elasticsearch/kibana-server.zip -d /etc/kibana/`
`sudo mv /etc/kibana/instance/instance.crt /etc/kibana/kibana-server.crt`
`sudo mv /etc/kibana/instance/instance.key /etc/kibana/kibana-server.key`
`sudo rm -r /etc/kibana/instance`

Change the ownership and permissions of the certificates
`sudo chown root:kibana /etc/kibana/kibana-server.crt /etc/kibana/kibana-server.key`
`sudo chmod 660 /etc/kibana/kibana-server.crt /etc/kibana/kibana-server.key`

Next, we need to configure Kibana to use the certificates we have generated. Edit the Kibana configuration file at /etc/kibana/kibana.yml, and add the following settings:

server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/kibana-server.crt
server.ssl.key: /etc/kibana/kibana-server.key

Alternatively, you can use sed to add these settings to the configuration file:
`sudo sed -i 's/^#\?server\.ssl\.enabled:.*/server.ssl.enabled: true/' /etc/kibana/kibana.yml`
`sudo sed -i 's#^\s*\#\?server\.ssl\.certificate:.*#server.ssl.certificate: /etc/kibana/kibana-server.crt#' /etc/kibana/kibana.yml`
`sudo sed -i 's#^\s*\#\?server\.ssl\.key:.*#server.ssl.key: /etc/kibana/kibana-server.key#' /etc/kibana/kibana.yml`

Next, we will generate keys to encrypt the dashboards and visualisations, reports and session information. Run the following command once:
`sudo /usr/share/kibana/bin/kibana-encryption-keys generate --quiet >> /etc/kibana/kibana.yml`

Finally, restart the Kibana service to apply the changes:
`sudo systemctl restart kibana`

After restarting Kibana, you will want to create an account for yourself, and set a password. To do this:

Open a web browser and navigate to https://siem.<initials>.local:5601.
Log in using the elastic user and the password you received during the Elasticsearch installation.
Click on the top left menu, then Management 🞂 Stack Management 🞂 Users 🞂 Create User.

Username	Your username
Full Name	Your full name
Email Address	Your email address
Password	Password1
Privileges	superuser
Click Create to create the user.

# Netflow Implementation

1. Log into your Kibana
2. Open the Fleet section under the main menu
3. Open the "Agent policies" section
4. Press "Create agent policy" button to create a new Agent Policy
5. Give the policy a name, for example, "NetFlow policy", adjust advance settings if required, create the policy
Open your newly created policy by clicking on it's name
6. Press "Add integration"
7. Search for "NetfFlow Records" and press "Add NetFlow Records"
Adjust configuration, make sure:
- Specify "UDP host to listen on" to the IP address of your server that is going to run the NetFlow Records integration , in this example the address should be "10.0.0.2"
8. Save the integration
9. Press the "Add Elastic Agent to your host" button
10. Follow the instructions on how to add Elastic Agent to your host
11. Make sure you have opened the NetFlow port on your host and elsewhere in the path from your RouterOS device (10.0.0.1), the default destination port is 2055/UDP.
Your Elastic Agent is now ready to receive NetFlow data!


RouterOS
(optional) Create an Interface list (for example, "NetFlow_interfaces") and add interface that need NetFlow data analysis

/interface list
add name=NetFlow_interfaces
/interface list member
add interface=VLAN3000 list=NetFlow_interfaces
Configure Traffic-flow to send NetFlow data to your Elastic Agent (10.0.0.2)

REQUIRED
/ip traffic-flow
enabled=yes interfaces=NetFlow_interfaces
/ip traffic-flow target
add dst-address=10.0.0.2
You should now start to see NetFlow data being ingested!
Continue the guide to start using Kibana

Using Kibana

Log into your Kibana
Open the "Dashboards" menu in the main menu
Search the Dashboards and find "NetFlow"
You should now see multiple NetFlow Dashboards. For example, try opening the "[Logs Netflow] Overview". If your NetFlow data is properly ingested, then you should now see graphs that summarizes your traffic.

Another useful Dashboard is the "[Logs Netflow] Flow records", which shows you exact NetFlow records. A very useful feature is the filtering option (the + button on top), that allows you add filters to NetFlow data, for example, you can filter the records to show only a single IP address:

# Using Kibana

Log into your Kibana
Open "Discover" from the main menu
Add a filter, use the following parameters:

Select a field: data_stream.dataset
Select operator: IS
Select a value: routeros

For simplicity we recommend searching for fields in the  Discover menu and searching for "message", then adding the field to the view
We also recommend searching for "log.syslog.hostname" field and adding to the view as well.
Consider saving the search for easier access later
Done!

NOTE: While searching for logs can be useful, you are more likely looking for a way to create alerts for certain activities and create connectors to send alerts e-mail, webhooks, chats and other options. Consider enabling the Spike in failed logon events rule to alert for excessive failed login attempts. You can also create a threshold rule and set it to alert after fixed amount of failed logins.


