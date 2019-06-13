---
published: true
---
### Trust, but Verify

I wrote about in a blog post a couple of weeks ago about my failed attempts to implement network automation in the past. A large part of those failures were my attempts to 'boil the ocean', or do too much too fast, vastly exceeding my skillset, getting frustrated, and giving up. This time around, I am being mindful to not creep the scope of my automation efforts too fast beyond my skills, and to aim for low hanging fruit as much as possible. Today, I am going to talk about my first success in this area, but first, some background about why I chose the task I did.

#### Using Automation to Enforce Reliability

Matt Oswalt has been writing and speaking a lot recently about the concept of '*Network Reliability Engineering*', a networking-centric counterpart to the more systems and application oriented idea of Site Reliability Engineering. A critical component to this concept of NRE is that the purpose of automation is not and should not be sold/advocated as speed. I can tell you from personal experience that if you are looking to get into programming and automation for your network in the hopes of saving yourself *time*, you are in the wrong place. The purpose of automation is to perform tasks **correctly and reliably**. Changes to the network, by nature of the reliance that the business places on modern networks, necessarily have a massive blast radius, and NRE focuses on implementing automation that not only performs tasks, but tests that those tasks were performed correctly and that the requested service is being provided to your customer. Focusing on network delivery in this way will, by nature of it's reliable delivery, increase the speed of network service provisioning through the minimization of outages and risks.

#### Eating a Horse, One Bite at a time

In order to identify what tasks I can target for a quick and valuable automation implementation, I have spent the past month or so breaking down the most common network configuration tasks that I perform to as close to their primitive components as possible. I did this using a technique I learned while a system administrator at IBM, called *the Five Whys*. While it is usually a troubleshooting technique, it's purpose is really to help you mentally break down a problem or task into it's components. The idea is that you take any given problem or task, then ask the question 'Why?' Answer that question, then ask 'Why?' again. Repeat five times, and you will typically arrive at your root cause/primitive tasks. In drilling the root cause of each particular problem down until you reach the true cause of the issue, you can better drive problem identification and remediation.

In my case, I have been reducing the tasks I perform, such as firewall NAT provisioning, VLAN creation, switchport configuration, etc. into the device commands and information gathering steps they consist of. Once the problems have been reduced to pseudocode like this, I can then begin working on an semi-automated workflow. This could be as simple as a Jinja2 template that renders out so you can copy-paste the output into the networking device. Once that is implemented, you can continue to iterate on your scripts to reduce the amount of human touch required.

In my particular case, my first target for a real, semi-automated solution was the provisioning of a new firewall NAT policy. In breaking down this process, I identified several primitive steps involved to perform this task, given certain inputs from the requester.

Those inputs consist of:
- What client the NAT policy is for
- What is the real IP the NAT policy should map to

With that input, I then perform the following tasks:
1. Identify the globally unique IP address to map to the provided real address
2. Assign both that globally unique address and the real IP address provided to the client in their respective prefixes inside of our IPAM tool (in our case, the awesome open source project [Netbox](https://github.com/digitalocean/netbox))
3. Add the globally unique address to an IP group assigned to the client in our Netflow collection tool ([Plixer Scrutinizer](https://www.plixer.com/products/scrutinizer/)), so that we may bill the client at 95th percentile for the bandwidth they use through this address.
4. Finally, create the NAT policy on our edge firewall, in our case FortiGate 800 series firewalls.

It is important at this point to note that I am **not** performing any actual firewalling within this script. This may seem strange, as what good is a NAT policy if I don't assign it to any policies to actually permit any traffic? That step, however, would require much more thought and work to actually ship that feature, and leads down the point of getting frustrated and giving up. It was important to me to develop a script that I can ship and demonstrate value from.

The final feature that I *did* want to implement was an automated test to verify some of the steps. I am currently only testing to verify that the IP address is present in the Netflow IP group, but again, this gets the script out the door and leaves room for additional iteration, such as adding tests to verify 200 OK from the Netbox API, and later on, rules to verify the firewall rule is applied and does what it says it does.

#### The Nitty-Gritty

Time to take a tour through the script. I am well aware that this code is very bad. There is little to no error detection, there are huge sections that could easily be spun out to functions to provide some modularity, and it is in general all over the place. My goal was not to write the sexiest or most Pythonic code, but to write something that I can use to get work done today, while giving me something to iterate upon.

```python
from jinja2 import Environment, FileSystemLoader
from pynetbox import api
from csv import writer
from paramiko import SSHClient, AutoAddPolicy
from time import sleep
from scp import SCPClient
from pymysql import connect
from netaddr import IPNetwork
```

I start with importing a good amount of modules.
- Jinja2 to render the template for the actual firewall VIP
- the pynetbox API for interfacing with netbox
- writer from the Python csv library. Scrutinizer does not have a fully functional API, so the only means I have of programmatically modifying the IP groups is by importing a CSV file
- SSHClient and AutoAddPolicy from paramiko to execute the CSV import commands on the Scrutinizer server
- time for a sleep, as the utility to import the CSV takes a few seconds to initialize
- SCPClient to copy the CSV to the Scrutinizer server
- pymysql to connect to the Scrutinizer database for the configuration test
- IPNetwork from the great netaddr module to simplify formatting for IP addresses while running the test

Not all of these libraries are strictly required, for example netaddr, but they made my life easier and get the task done simpler.

I then set up some tools I will use later in the script - a Jinja2 environment based on a template I have created for a VIP in FortiOS syntax, an API connection object to Netbox, and a dictionary that I will pass to pymysql to connect to the Scrutinizer database.

```python
# create jinja2 environment
j2_env = Environment(loader=FileSystemLoader('Template'))
template = j2_env.get_template('New Firewall VIP.j2')

# create object to load netbox api
nb = api('https://netbox_URL', token = '$my_API_key')
# create kwargs dictionary for netflow
netflow = {'host': 'netflow_server_FQDN_or_IP_address', 'user': '$username', 'passwd': '$password', 'db': 'plixer'}
```

I then begin taking user input. There is a lot of user input in this script, which is obviously not ideal, but again, it exists in the interest of getting to a working state. As it stands, I am the only person who interacts with this script, but the whole purpose is to remove my ability to make mistakes, so further iterations will reduce the amount of user input required.

I get a name for the VIP object, the public prefix that the VIP should be created in, and the client it should be assigned to, real IP information, a description of the real IP device, and the VRF ID as it is in Netbox. Future iterations will make these options menu based, with certain options available based on the datacenter the client is in, and with checks to see if the real IP already exists or not.

```python
# user inputs what prefix to create the VIP in
vip_name_j2 = input('Enter the VIP name: ')
requested_prefix = input('Public prefix to create the VIP in, including netmask: ')

# user inputs what tenant to create the VIP for, then we pull the ID
tenant_name = input('Tenant this IP should be associated to: ')
tenant_id = nb.tenancy.tenants.get(q = tenant_name)

# create the real IP dictionary for its creation later
real_ip = {}

# populate real IP dictionary with address, description, and VRF
real_ip['address'] = input('Real IP the VIP should map to, including netmask: ')
real_ip['description'] = input('Enter a description of the device: ')
real_ip['vrf'] = input('Enter the Netbox VRF name: ')
real_ip['role'] = 10
real_ip['tenant'] = tenant_id.id
```

I use the dictionary I have created from the real IP information to create the real IP in Netbox, as a NAT association within Netbox requires the real IP to exist as a provisioned number before you can map the global address to the inside address. I then query the API for the new real IP's numeric ID and save it for that NAT mapping. Again, there is no error detection at this stage, which is space for iteration.

```python
# create the real IP as defined by user input, and get the ID back to be used in the NAT mapping
created_real_ip = nb.ipam.ip_addresses.create(**real_ip)
real_ip_id = created_real_ip.get('id')
```

I look up the public prefix based on user input and run the available_ips function in pynetbox to automatically detect an available IP within that prefix, assign it to the client, and map it to the real IP. I have some error correction in here if the user mistypes a prefix, but there are more errors I could catch and deal with, as well as looping these inside a while block so the user can correct mistakes.

```python
# try to get the prefix, and catch the error if an invalid prefix is entered
try:
    prefix = nb.ipam.prefixes.get(q = requested_prefix)
except ValueError:
    print('Invalid prefix entered.')

# create a new VIP and reload it as an object
new_mapped_ip_dict = prefix.available_ips.create()
new_mapped_ip_obj = nb.ipam.ip_addresses.get(q = new_mapped_ip_dict.get('address'))

# update the tenant and NAT fields of the VIP and save it
new_mapped_ip_obj.tenant = tenant_id
new_mapped_ip_obj.nat_inside = real_ip_id
new_mapped_ip_obj.save()
```

At this point the IPAM work is complete. Now we are moving on to adding the new global address to the client's IP group in Scrutinizer. To do this I create a CSV file in the working directory of the script, adding in the client name and the IP address.

```python
# create the CSV containing the tenant name and IP to be added to Scrutinizer
with open('ipimport.csv', 'w', newline = '') as csvfile:
    filewriter = writer(csvfile, delimiter = ',')
    ip_group_information = [tenant_name, new_mapped_ip_dict.get('address')]
    filewriter.writerow(ip_group_information)
```

Once the CSV is created, I use paramiko and scp to connect to the server running Scrutinizer, clean up any old files, scp the new CSV over, and execute the commands to import them. Once again there is no error detection or handling here, which is room for improvement.

```python
# SSH to Scrutinizer, SCP the CSV to the server, and run the scrut_util.exe binary to import the file, then clean up
with SSHClient() as ssh:
    ssh.set_missing_host_key_policy(AutoAddPolicy())
    ssh.connect('$Scrutinizer_FQDN_or_IP', username = '$username', password = '$password')
    remote_conn = ssh.invoke_shell()
    remote_conn.send('rm -f /home/plixer/ipimport.csv\n')
    scp = SCPClient(ssh.get_transport())
    scp.put('ipimport.csv', remote_path = '/home/plixer/ipimport.csv')
    remote_conn.send('scrut_util.exe\n')
    sleep(5)
    remote_conn.send('import ipgroups /home/plixer/ipimport.csv\n')
    sleep(1)
    remote_conn.send('quit\n')
    remote_conn.send('rm -f /home/plixer/ipimport.csv')
```

At this point all of the tasks are done except for rendering the Jinja2 template. At this point I want to run my configuration test to verify that the address has actually been added to the IP Group. I have had issues in the past with VIPs being created but not added to IP groups, so our billing is then incorrect at the end of the month, which takes a large amount of time to reconcile. If I can test that this task gets done, several hours every month are saved reconciling tenants to IP groups, and we can trust that our 95th percentile billing is accurate.

Scrutinizer does not have programmatic access to IP groups through any kind of queryable API, so I had to go to the database directly. I connect to the MariaDB instance running on the server and run a query to get the IP group for the client and save it to a variable. This syntax was not immediately obvious to me, but running the execute() method on the cursor does not actually return the result of the query. You have to fetch the information from the cursor object you create using the cursor() method using fetchall(), which returns a tuple of the results of the query.

```python
# Connect to the SQL DB on the Scrutinizer server and load the query response
sqlconn = connect(**netflow)
cursor = sqlconn.cursor()
sql_query = ("SELECT flow_class.name, inet_b2a(ip_groups.network) "
             "FROM flow_class "
             "INNER JOIN ip_groups ON flow_class.fc_id = ip_groups.group_id "
             "WHERE flow_class.name like '{}';"
             )
cursor.execute(sql_query.format(tenant_name))
sql_return = cursor.fetchall()
```

Once I have the data back, it's time to run our check. I am using the INET_B2A procedure in the SQL query to convert the encoded information in the database to an ASCII string. This presents a problem, in that Python 3 strings are by default Unicode, so for example a string of 'Hello World' that is ASCII encoded will show in Python 3 as b'Hello World' to denote it as a byte string. I then have to use a byte string encoded version of the new VIP address to match against the contents of the SQL query. For that, I need the address without the CIDR prefix, as Netbox returns all addresses with their associated mask. I considered a whole host of regex implementations to strip the prefix off of the address, but decided I would go for the quicker implementation of using netaddr to return the address without the mask. Once done, I take that value and check to see if it is in any of the items in my tuple. If it is, I print that the test was successful, then I render the Jinja2 template. If it is not in the tuple, then I print that the test has failed, and I do not render the template, as something has gone wrong in the execution.

```python
# convert the created VIP IP address to a byte string to match against the tuple that returns from SQL
mapped_ip_obj = IPNetwork(new_mapped_ip_dict.get('address'))

# check the converted byte string against the loaded SQL output, if it is complete, then print the rendered template to be sent to the firewall.
if any(str(mapped_ip_obj.ip).encode('utf-8') in ips for ips in sql_return):
    print('SQL test successful - the mapped IP {} is present in the {} IP group.'.format(new_mapped_ip_dict.get('address'), tenant_name))
    rendered_template = template.render(vip_name = vip_name_j2, outside_ip = new_mapped_ip_dict.get('address'), inside_ip = real_ip.get('address'))
    print(rendered_template)
else:
    print('SQL test failed - the mapped ip {} was not found in the {} IP group.'.format(new_mapped_ip_dict.get('address'), tenant_name))
```

And that's it. My first little pinky toe into the bottomless ocean that is reliability automation. I am proud of myself for the accomplishment, while still cognizant that I have a lot of work to do, not only on this script (which I am well aware is terrible) but on my script design in general, software design principles, and of course, other tasks I can give this same style of treatment.

This script took me about 12 hours to develop, and is the first really serious coding/scripting I have done since C++ and Java for non-CS majors when I was in college. I am not concerned about the time investment, because speed is not the goal. The purpose is to more reliably interface with the network, and that is what this feeble attempt at a script, in it's own small way, tries to do.
