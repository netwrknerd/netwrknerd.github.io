---
title: "Network Programmability and Automation, Chapter 2: Network Automation"
date: 2020-11-24T01:02:00-04:00
categories:
  - blog
tags:
  - network-automation
---

# Network Programmability and Automation, Chapter 2: Network Automation

This chapter is not meant to be a deep technical chapter, but rather an introduction to the ideas and concepts of network automation. It simply lays the foundation and provides context for the chapters that follow.

### Why Network Automation?
Network automation, like most types of automation, is thought of as a means of doing things faster. While doing things more quickly is nice, reducing the time for deployments and configuration changes isn’t always a problem that needs solving for many IT organizations.

Including speed, we’ll take a look at a few of the reasons that IT organizations of all shapes and sizes should be looking at gradually adopting network automation. You should note that the same principles apply to other types of automation as well (application, systems, storage, telephony, etc.).

### Simplified Architectures
Today, most network devices are configured as unique snowflakes (having many one-off, non-standard configurations), and network engineers take pride in solving transport and application issues with one-off network changes that ultimately make the network not only harder to maintain and manage, but also harder to automate.

Instead of network automation and management being treated as a secondary project or an “add-on,” it needs to be included from the outset as new architectures are being created. This includes ensuring there is the proper budget for personnel and/or tooling. Unfortunately, tooling is often the first item that gets cut when there is a shortage of budget.

The end-to-end architecture and associated 'day 2 operations' need to be one and the same. You need to think about the following questions as architectures are created:

* Which features work across vendors?
* Which extensions work across platforms?
* What type of API or automation tooling works with particular network device platforms?
* Is there solid API documentation?
* What libraries exist for a given product?

When these questions get answered early on in the design process, the resulting architecture becomes simpler, repeatable, and easier to maintain /and/ automate, all with fewer vendor-proprietary extensions enabled throughout the network.

Even after the simplified architecture gets deployed with the right management and automation tooling, remember it's still a necessity to minimize one-off changes to ensure the network configurations don't become snowflakes again.

### Deterministic Outcomes
In an enterprise organization, change review meetings take place to review upcoming changes on the network, the impact they have on external systems, and rollback plans. In a world where a human is touching the CLI to make those upcoming changes, the impact of typing the wrong command is catastrophic. Imagine a team with 3, 4, 5, or 50 engineers. Every engineer may have his or her own way of making that particular upcoming change. Moreover, the ability to use a CLI and even a GUI does not eliminate or reduce the chance of error during the control window for the change.

Using proven and tested network automation to make changes helps achieve more predictable behavior than making changes manually, and gives the executive team a better chance at achieving deterministic outcomes, moving one step closer to having the assurance that the task at hand will get done right the first time without human error. This could be any task from a virtual local area network (VLAN) change to on-boarding a new customer that requires several changes throughout the network.

### Business Agility
We know that network automation offers speed and agility for deploying changes, but it does the same for retrieving data from network devices as fast as the business demands, or more practically, as fast as needed to dynamically troubleshoot a network issue.

Since the advent of server virtualization, server and virtualization administrators have had the ability to deploy new applications almost instantaneously. And the faster applications are deployed, the more questions are raised as to why it takes so long to configure network resources such as VLANs, routes, firewall (FW) policies, load-balancing polices, or all of the above, if deploying a new three-tier application.

It should be fairly obvious that by adopting network automation, the network engineering and operations teams can react faster to their IT counterparts for deploying applications, but more importantly, it helps the business be more agile. From an adoption perspective, it’s critical to understand the existing, and often manual, workflows before attempting to adopt automation of any kind, no matter how good your intentions are for making the business more agile.

If you don’t know what you want to automate, it’ll complicate and prolong the process. Our /number one/ recommendation as you start your network automation journey is to always understand existing manual workflows, document them, and understand the impact they have to the business. Then, the process to deploy automation technology and tooling becomes much simpler.

## Types of Network Automation
Automation is commonly equated with speed, and considering that some network tasks don’t require speed, it’s easy to see why some IT teams don’t see the value in automation. VLAN configuration is a great example because you may be thinking, “How /fast/ does a VLAN really need to get created? Just how many VLANs are being added on a daily basis? Do /I/ really need automation?” And they are all valid questions.

In this section, we are going to focus on several other tasks where automation makes sense, such as device provisioning, data collection, troubleshooting, reporting, and compliance. But remember, as we stated previously, automation is much more than speed and agility; it also offers you, your team, and your business more predictable and more deterministic outcomes

### Device Provisioning
One of the easiest and fastest ways to get started with network automation is to automate the creation of the device configuration files that are used for initial device provisioning and pushing them to network devices.

If we take this process and break it down into two steps, the first step is creating the configuration file, and the second is pushing the configuration onto the device.

In order to automate the creation of configuration files, we first need to decouple the /inputs/ (configuration parameters) from the underlying vendor-proprietary syntax (CLI) of the configuration. This means we’ll end up with separate files with values for the configuration parameters such as VLANs, domain information, interfaces, routing, and everything else being configured, and then, of course, a configuration template.

For now, think of the configuration template as the equivalent of a standard golden template that’s used for all devices getting deployed. By leveraging a technique called /network configuration templating/, you are quickly able to produce consistent network configuration files specifically for your network. What this also means is you’ll never have to use Notepad ever again, copying and pasting configs from file to file—isn’t it about time for that?

Two tools that streamline using configuration templates with variables (data inputs) are Ansible and Salt. In less than a few seconds, these tools can generate hundreds of configuration files predictably and reliably.

---

Let’s look at an example of taking a current configuration and decomposing it into a template and separate variables (inputs) file to articulate the point we’re making.

Here is an example of a configuration file snippet:
```
hostname leaf1
ip domain-name ntc.com
!
vlan 10
   name web
!
vlan 20
   name app
!
vlan 30
   name db
!
```

If we decouple the data from the CLI commands, this file is transformed into two files: a /template/ and a /data/ (variables) file.

First, let's look at the YAML variables file:

```yaml
---
hostname: leaf1
domain_name: ntc.com
vlans:
  - id: 10
    name: web
  - id: 20
    name: app
  - id: 30
    name: db
```

The resulting template that'll be rendered with the data file looks like this and is given the filename '/leaf.j2':/

```
!
hostname {{ inventory_hostname }}
ip domain-name {{ domain_name }}
!
!
{% for vlan in vlans %}
vlan {{ vlan.id }}
  name {{ vlan.name }}
{% endfor %}
!
```

In this example, the /double curly braces/ denote a Jinja variable. In other words, this is where the data variable gets inserted when a template is rendered against data. Since the double curly braces denote variables, and we see those values are not in the template, they need to be stored somewhere. Again, we stored them in a YAML file. Rather than use flat YAML files, you could also use a script to fetch this type of information from an external system such as a network management system (NMS) or IP address management (IPAM) system.

In this example, if the team that controls VLANs wants to add a VLAN to the network devices, no problem. They just need to change it in the variables file and regenerate a new configuration file using Ansible or the rendering engine of their choice (Salt, pure Python, etc.).

At this point in our example, once the configuration is generated, it needs to be /pushed/ to the network device. The /push/ and /execution/ process is not covered here, as there are plenty of ways to do this, including vendor-proprietary zero touch provisioning solutions.

Aside from building configurations and pushing them to devices, something that is arguably more important is data collection, which happens to be the next topic we cover.

### Data Collection
Monitoring tools typically use the Simple Network Management Protocol (SNMP)—these tools poll certain management information bases (MIBs) and return data to the monitoring tool. Based on the data being returned, it may be more or less than you actually need. What if interface stats are being polled? You may get back every counter that is displayed in a show interface command, but what if you only needed interface resets and not CRC errors, jumbo frames, output errors, etc. Moreover, what if you want to see the interface resets correlated to the interfaces that have CDP/LLDP neighbors on them, and you want to see them now, not on the next polling cycle? How does network automation help with this?

Given that our focus is giving you more power and control, you can leverage open source tools and technology to customize exactly what you get, when you get it, how it's formatted, and how the data is used after it's collected, ensuring you get the most value from the data.

Here is a *very* basic example of collecting data from an IOS device using the Python library **netmiko**

```python
from netmiko import ConnectHandler

device = ConnectHandler(device_type='cisco_ios', ip='csr1',username='ntc',password='ntc123')
output = device.send_command('show version')

print(output)
```

The great part is that *output* contains the *show version* response and you have the ability to parse it as you see fit based on your requirements.

> In the example given, we are describing *pulling* the data off of the devices, which may not be ideal for all environments, but still suitable for many. Be aware that newer devices are starting to support a *push* model, often referred to as streaming telemetry, where the device itself streams real-time data such as interface stats to an application server of your choice.

Of course, any of this may require some up-front custom work but is totally worth it in the end, because the data being gathered is what you need, not what a given tool or vendor is providing you. Plus, isn’t that why you’re reading this book?

Network devices have an enormous amount of static and ephemeral data buried inside, and using open source tools or building your own gets you access to this data. Examples of this type of data include active entries in the BGP table, OSPF adjacencies, active neighbors, interface statistics, specific counters and resets, and even counters from application-specific integrated circuits (ASICs) themselves on newer platforms. Additionally, there are more general facts and characteristics of devices that can be collected too, such as serial number, hostname, uptime, OS version, and hardware platform, just to name a few. The list is endless.

### Migrations
Migrating from one platform to the next is never an easy task. This may involve platforms from the same vendor or from different vendors. Vendors may offer a script or a tool to help with migrations to their platform, but various forms of automation can be used to build out configuration templates, just like our example earlier, for all types of network devices and operating systems in such a way that you could generate a configuration file for all vendors given a defined and common set of inputs (common data model).

Of course, if there are vendor-proprietary extensions, they’ll need to be accounted for too. The beautiful thing is that a migration tool such as this is much simpler to build on your own than have a vendor do it because the vendor needs to account for all features the device supports as compared to an individual organization that only needs a finite number of features. In reality, this is something vendors don’t care much about; they are concerned with their equipment, not making it easier for you, the network operator, to manage a multi-vendor environment.

Having this type of flexibility helps with not only migrations, but also disaster recovery (DR), as it’s very common to have different switch models in the production and DR data centers, and even different vendors. If a device fails for any reason and its replacement has to be a different platform, you’d be able to quickly leverage your common data model (think parameter inputs) and generate a new configuration immediately. 

Thus, if you are performing a migration, think about it at a more abstract level and think through the tasks necessary to go from one platform to the next. Then, see what can be done to automate those tasks, because only you, not the large networking vendors, have the motivation to make multi-vendor automation a reality. For example, think about adding a VLAN as an abstract step—then you can worry about the lower-level commands per platform. The point is, as you start adopting automation, it’s extremely important to think about tasks and document them in human-readable format that is vendor-neutral, before putting hands to keyboard typing in CLI commands or writing code (per platform).

### Configuration Management
As stated, configuration management is the most common type of automation, so we aren’t going to spend too much time on it here. You should be aware that when we mention configuration management we are referring to deploying, pushing, and managing the configuration state of the device. This includes anything as basic as VLAN provisioning to more complex workflows that configure top-of-rack switches, firewalls, load balancers, and advanced security infrastructure, to deploy three-tier applications.

As you can see already, through the different forms of automation that are *read-only*, you do not need to start your automation journey by pushing configurations. That said, if you are spending countless hours pushing the same change across a given number of routers or switches, you may want to!

The reality is that there are so many ways to start a network automation journey, but when you start automating configuration management, remember, with great power comes great responsibility. More importantly, don’t forget to test before rolling out new automation tools into production environments.

The next few types of network automation we cover stem from automating the process of data collection. We’ve broken a few of them out to provide more context, and first up is automating compliance checks.

### Compliance
As with many forms of automation, making configuration changes with any type of automation tool is seen as a risk. While making manual changes could arguably be riskier, as you’ve read and may have experienced firsthand, you have the option to start with data collection, monitoring, and configuration building, which are all *read-only* and *low-risk* actions. One low-risk use case that uses the data being gathered is configuration compliance checks and configuration validation. Does the deployed configuration meet security requirements? Are the required networks configured? Is protocol XYZ disabled? When you have control over the tools being deployed, it is more than possible to verify if something is **True** or **False**. It’s easy enough to start small with one compliance check and then gradually add more as needed.

Based on the compliance of what you are checking, it’s up to you to determine what happens next—maybe it just gets logged, or maybe a complex operation is performed, making your application capable of auto-remediation. These are forms of event-driven automation that we also touch upon when we cover StackStorm and Salt in Chapter 9. Our recommendation is that it’s always best to start simple with network automation, but being aware of what’s possible adds significant value as well. For example, if you just log or print messages to see what an interface maximum transmission unit (MTU) is, you’re already prepared should you want to automatically reconfigure it to the right value if it is not the desired MTU. You’d just have to have a few more lines underneath your existing log/print messages. Again, the point is to start small, but think through what else you may need in the future.

### Reporting
Once you start automating the collection of data, you may want to start building out custom and dynamic reports too. Maybe the data being returned becomes input to other configuration management tasks (event-driven again or more basic conditional configuration), or maybe you just want to create reports.

Given that reports can also be easily generated from templates combined with the actual ephemeral data from the device that’ll be inserted into the template, the process to create and use reporting templates is the same process used to create configuration templates that we touched upon earlier in the chapter.

Because of the simple nature of using text-based templates, it is possible to produce reports in any format you wish, including but not limited to:
* Simple text files
* Markdown files that can be easily viewed on GitHub, or some other Markdown viewer
* HTML reports that are deployed to a web server for easy viewing

It all depends on your requirements. The great thing is that the *network automator* has the power to create the exact type of report they need. In fact, you can use one set of data to generate different types of reports, maybe some technical and some higher-level for management.

### Troubleshooting
Who enjoys getting consistently pulled into break/fix problems, especially when you should be sleeping or focused on other things?

Once you have access to real-time data and don’t need to do any manual parsing on that data, automated troubleshooting becomes a reality.

Think about how you troubleshoot. Do you have a personal methodology? Is that methodology consistent across all members on your team? Does everyone check Layer 2 before troubleshooting Layer 3? What steps do you take to troubleshoot a given problem?

Let’s take troubleshooting OSPF as an example:
* Do you know what it takes to form an OSPF adjacency between two devices?
* Can you rattle off the same answers at 2 a.m. or while on vacation at the beach?
* Maybe you remember some like devices need to be on the same subnet, have the same MTU, and have consistent timers, but forget they need to be the same OSPF network type.
* Do we *really* need to remember *all* of this and the associated commands to run on the CLI to get back each piece of data?

In any given environment, these types of compatibility checks need to be performed. Can you fathom running a script or using a tool for OSPF neighbor validation versus performing that process manually? Which would you prefer?

Again, OSPF is only the tip of the iceberg. Think about these other questions still being the tip:
* Can you correlate particular log messages to known conditions on the network?
* What about BGP neighbor adjacencies? How is a neighbor formed?
* Are you seeing all of the routes you think you should in the routing table?
* What about VPC and MLAG configuration?
* What about port-channels? Are there any inconsistencies?
* Do neighbors match the port-channel configuration (going down to the vSwitch)?
* What about cabling? Are all of the cables plugged in properly?

Even with these questions, we are just scratching the surface with that is possible when it comes to automated diagnostics and troubleshooting.

< As you start to consider all of the types of automation possible, start to imagine a closed-loop system such that data is collected in an automated fashion, the data is then processed and analyzed in an automated fashion, and then you use advanced analytics to troubleshoot in an automated fashion. As these start to happen together in a uniform fashion, this becomes a closed-loop, fully changing the way operations are managed within an organization.

If you are the rock star network engineer on your team, you may want to think about partnering up with a developer, or at the very least, start documenting your workflows, so it’s easier to share the knowledge you possess and it becomes easier to *codify*. Better yet, start your own personal automation journey so you can sleep in every so often and empower everyone else to troubleshoot using some of your automated diagnostic workflows.

As you can see, network automation is much more than deploying configurations faster. After looking at several different types of automation, we are going to shift topics now and look at a few different ways automation tools and applications communicate with network devices, starting with SSH and ending with NETCONF and HTTP-based RESTful APIs.

### Evolving the Management Plane from SNMP to Device APIs

If you want to improve the way networks are managed and operated day-to-day, improvements must begin with how you interface with the underlying devices being managed. This interface is how you and, more importantly, automation tools communicate with devices to perform the various types of network automation, such as data collection and configuration management.

In this section, we provide an overview of the different methods available to connect to the management plane of network devices starting with SNMP and then move on to more modern ways such as NETCONF and RESTful APIs. We then look at the impact of the open networking movement as it pertains to network operations and automation.

### Application Programming Interfaces (APIs)
As a network engineer, you need to embrace APIs going forward, and not fear them. Remember that an API is just a mechanism that is used for computer software on one device to talk to computer software on another device. APIs are used nearly everywhere on the internet today—they just happen to finally be getting the focus they deserve from the network vendors. We’ll soon see that APIs will become the primary means of managing network devices.

While we cover specific network APIs in more detail in Chapter 7, this section provides a high-level overview of a few different types of APIs that you’ll find on network devices today.

### SNMP
SNMP has been widely deployed for over 20 years on network devices. It shouldn’t be new to anyone reading this book, but SNMP is a protocol that is used quite commonly for polling network devices for information such as up/down status and CPU, memory, and interface utilization.

In order to use SNMP, there must be an SNMP agent on a managed device and a network management station (NMS), which is the device that functions as a *server* that monitors and/or controls the managed devices.

Each network device being managed exposes a set of data that can be collected and configured via the SNMP agent. This set of data that is managed through SNMP is described and modeled through management information bases, or MIBs. Only if there is a MIB exposing a certain feature can it be monitored or managed. This includes making configuration changes through SNMP. Often overlooked, SNMP not only supports **GetRequests** for monitoring, but also supports **SetRequests** for manipulating objects and variables exposed through MIBs. The issue is that not many vendors offer full support for configuration management via SNMP; when they do, they often use custom MIBs, slowing down the integration process to network management platforms.

As mentioned, SNMP has been around for decades, but it was not built to be a real-time programmatic interface to network devices. We are already seeing vendors claim the gradual death of SNMP as it pertains to next-generation management and automation tooling. That said, SNMP does exist on nearly every network device, and Python libraries for SNMP also exist—so, if you need to collect basic information from a vast amount of device types, it may still make sense to use SNMP.

Just like SNMP has been used for years to perform network monitoring, SSH/Telnet and the CLI has been used for configuration management. Let’s take a look now at SSH/Telnet and the CLI.

### SSH/TELNET and the CLI
If you have ever managed a network device, you’ve definitely used the CLI to issue commands to perform some action on a device. You probably entered commands through the console and over Telnet and SSH sessions. As we stated in Chapter 1, the reality is that the migration from Telnet to SSH is arguably the biggest shift we’ve had in network operations over the past decade, and that shift wasn’t about operations; it was about security ensuring that communications to network devices were encrypted.

The most important thing to realize as it pertains to managing devices via the CLI is that the CLI was built for humans. It was put on devices to improve usability for human operators. The CLI was *not* meant to be used for machine-to-machine communication (i.e., network scripting and automation).

If you issue a **show** command on the CLI of a device, you get raw text back. There is no structure to it. The best options to *parse* the response are to use the *pipe* (|) and keywords such as **grep**, **include**, and **begin** to look for particular lines of configuration. An example of that would be to check the description of an interface with the command `show interface Eth1 | include description`. This means if you needed to know how many CRC errors were on an interface after issuing a show interface in a script, you’d be forced to use some type of regular expression or manual parsing to figure it out. This is unacceptable.

However, when all we have is the CLI, CLI is what gets used. This is why there are plenty of network management platforms and custom scripts that have been built over the past two decades that perform management and automated operations using the CLI over SSH dealing with expect scripts and manual parsing. It’s not that SSH/CLI makes it impossible to automate; rather, it makes automation extremely error prone and tedious.

The network vendors started to realize this, and now most newer device platforms have some type of API that simplifies machine-to-machine communication (many are incomplete, so be sure to test your favorite device’s API), yielding a much simpler approach to automation that is also more in line with general software development principles.

After a brief look at common protocols such as SSH and SNMP, we’ll look at NETCONF, an API that is becoming quite popular as it pertains to network automation.

### NETCONF
NETCONF is a network management layer protocol. At the highest level, it can be compared to SNMP, as they are both protocols used to make configuration changes and retrieve data from networking devices.

The differences come in the details, of course.

* NETCONF is a connection-oriented protocol and commonly leverages SSH as its transport.

* Data sent between a NETCONF client (automation tool/script) and NETCONF server (network device) is encoded in XML. Don’t worry if you aren’t familiar with XML; we cover it in Chapter 5.

* Remote procedure calls (RPCs) are encoded in the XML document sent to the device and the device processes these RPCs. The <rpc> element is used to enclose a NETCONF request sent from the client to the server. In this context, think of these remote procedure calls as performing a prearranged operation on the device. RPCs are a way for a client to communicate to the server what structure and what type of request is being made.

* Supported RPCs map directly to supported NETCONF *operations* and *capabilities* for particular devices. For example, if you are making a change on a device you use the **edit-config** operation. If you are retrieving configuration data, you use the **get** or **get-config** operation. These operations are wrapped inside the XML document within the <rpc> element sent to the device.

Additionally, NETCONF offers value in that it supports transaction-based changes. This means that if you are making more than one change in a given NETCONF session, or single XML document, and one of those changes fails, the complete change is *not* applied to the device (of course, these types of settings can usually be overridden too). This is in contrast to sending CLI commands sequentially and ending up with a partial configuration due to a typo or invalid command.

> It’s worth pointing out that just because two different device platforms support NETCONF (or any common transport method) does not mean they are compatible from a tooling and developer’s perspective. Even with the assumption that both devices support the same NETCONF features and capabilities, how the data is modeled is, more often than not, vendor specific. Data modeling is how the device represents state and configuration data. We’ll learn more about data representation in JSON and XML and YANG, a common data modeling language, in Chapter 5.

### RESTful APIs
REST stands for REpresentational State Transfer and is a style used to design and develop networked applications. Thus, systems that implement and adhere to a REST-based architecture are said to be RESTful.

Keeping this in context from a network perspective, the most common devices that expose APIs and adhere to the architectural style of REST are network controllers. That said, there are network devices that expose RESTful and general HTTP-based APIs too.

While the terms REST and RESTful APIs are new from a network standpoint, you’re already interacting with many RESTful systems on a daily basis as you browse the internet using a web browser. We said that REST is a style used to develop networked applications. That style relies on a stateless client-server model in which the client keeps track of the session and no client state or context is held on the server. And best yet, the underlying transport protocol used is most commonly HTTP. Doesn’t this sound like most systems found on the internet?

This means that RESTful APIs operate just like HTTP-based systems. First, you need a web server accessible via a URL (i.e., *SDN controller* or *network device* to communicate with), and second, you need to send the associated HTTP request to that URL.

For example, if you need to retrieve a list of devices from an SDN controller, you just need to send an **HTTP GET** to the given URL of the device, which could look something like this: http://1.1.1.1/v1/devices. The response that comes back would be some type of structured data like XML or JSON (which we cover in Chapter 5).

There are a few other things that we didn’t touch upon, such as authentication, data encoding, and how to send an HTTP request if you’re making a configuration change (HTTP PUT/POST/PATCH). As this section was just a short high-level introduction to REST and RESTful APIs, we cover more of those details in Chapter 7.

Next up is a short look at the impact *open networking* is having on the overall management of network devices.

### Impact of Open Networking
There is a growing trend of all things *open*—open source, open networking, Open APIs, OpenFlow, Open Compute, Open vSwitch, OpenDaylight, OpenConfig, and the list goes on. While the definition of *open* can be debated, there is one thing that is certain: the *open networking* movement is improving what is possible when it comes to network operations and automation.

With this movement, we are seeing drastic changes in network devices, and this is a primary reason for writing this book.

First, many devices now support Python on-box. This means that you are able to drop into the Python Dynamic Interpreter and execute Python scripts locally on each network device. We cover Python in much more detail in Chapter 4, and you’ll see what we mean firsthand.

Second, many devices now support a more robust API other than SNMP and SSH. For example, we just looked at NETCONF and RESTful HTTP-based APIs. One or both of those APIs are supported on many of the newer device operating systems that have emerged in the past 18 to 24 months. Remember, we cover device APIs in more detail in Chapter 7.

Finally, network devices are exposing more of the Linux internals that have been hidden from network operators in the past. You can now drop into a *bash* shell on network devices and issue commands such as **ifconfig**, write bash scripts, and install monitoring and configuration management tools via package managers such as **apt** and **yum**. You’ll learn about all of these things in Chapter 3.

While *open networking* doesn’t always mean interoperability, it is evident that network devices and controllers are opening themselves up to be operated in a much more programmatic manner better suited for enhanced network automation. There are a number of APIs on network devices that didn’t exist a few years ago, ranging from Cisco’s NX-API, Arista’s eAPI, and Cisco’s IOS-XE RESTCONF/NETCONF to any new SDN controllers that have APIs. The net result, for you as operators, is that you can take control of your networks and reduce the number of operational inefficiencies that exist today as you start using these APIs.

## Network Automation in the SDN Era
We’ll now take a look at the continued importance of network automation even when controller solutions are being deployed such as OpenDaylight or even commercial offerings like Cisco ACI or VMware NSX. The operations that the controllers perform on the network, such as acting as the control plane or managing policy and configuration, are irrelevant for this section.

The fact is that controllers are becoming common in next-gen architectures. Vendors such as Cisco, Juniper, VMware, Big Switch, Plexxi, Nuage, Viptela, and many others all offer controller platforms for their next-gen solutions, not to mention open source controllers such as OpenDaylight and OpenContrail.

Almost every controller on the market exposes northbound RESTful APIs, making controllers extremely easy to automate. While controllers themselves inherently simplify management and visibility through a single pane of glass, you can still end up making manual and error-prone changes through the GUI of a controller. If there are several pods or controllers deployed, from the same or different vendors, the problems of manual changes, troubleshooting, and data collection do not go away.

As we start to wrap up this chapter, it’s important to note that even in the new era of SDN architectures and controller-based network solutions, the need for automation, better operations, and more predictable outcomes does not go away.

# Summary
This chapter provided an overview of the value of network automation and various types of network automation; an introduction to common device APIs including SNMP, CLI/SSH, and more importantly NETCONF and RESTful; and a brief mention of YANG, a network modeling language that we’ll cover in more detail in Chapter 5.

The chapter closed with a brief look at the impact that the open networking movement is having on network operations and automation. Finally, we touched on the value of network automation even when SDN controllers are deployed.

In each subsequent chapter, we dive deeper into each technology, providing hands-on practical examples whenever possible, but at the same time reviewing the importance of the people, process, and culture required to adopt comprehensive automation frameworks and pipelines. In fact, we focus significantly on people and culture in Chapter 11.

#network_automation