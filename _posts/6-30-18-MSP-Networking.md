---
published: false
---
# Networking in a Managed Services World

My current employer is a managed services provider (MSP) in the southeastern USA. Our target market is the SMB space, with most of our clients having employee counts in the 50-500 employee range. My particular role as *the* network engineer in the company is threefold:
- Management of the network infrastructure of our private cloud environment
- SME for 'networking' (a domain as vast as it is nebulous)
- Networking support for our professional service teams

While that first role is certainly the sexier part of the job, and is where most of my automation efforts have been focused, the reality is that I spend a much larger amount of time on the latter two responsibilities.

### The Escalation Escalator

Like many MSPs, our organization is divided into several managed service teams which are primarily responsible for the support of our clients. This requires a lot of knowledge from our line techs, who are expected to know everything for each of their clients, from Windows Server, line of business applications, VoIP, networking, virtualization, across the board. This breadth of knowledge has lead to certain members of our staff, myself included, designated as SMEs for common technology platforms, such as networking, virtualization, Exchange, etc. As such, when one of our clients has a networking issue or a project that the owning tech cannot solve or complete on their own, I will be called in to assist.

### Everything Old is New Again

This focus on escalations means that I do not often have the comfort of working in a network that I am familiar with or that I personally built. Outages in our private cloud are rare and when they do occur are quickly resolved, partially because of it's resilient design, and partially because I am veriy familiar with it's design and the troubleshooting tools at my disposal. When I am brought in to troubleshoot issues with a client network, I am often coming into a network I have never seen before, is under-documented and not well understood by those who manage it. This means a big focus on fundamentals - being able to get a fast overview of a network from an L2/L3 perspective, identifying security boundaries, and distilling a problem to it's base elements. We are investigating tools such as [Auvik](https://www.auvik.com/), an automated network discovery, graphing, and management tool, to simplify this process, but for now I am left with munging MAC address, routing, and LLDP tables.

### Design in a Cost Constrained World

I often read and hear about stories of network design on blogs and podcasts - stories that talk about whole datacenter fabrics, massive campus networks, or huge multi-cloud deployments. It takes soem effort on my part to remind myself that I live in the Rest of the World where a $1500 USD SonicWall firewall and a $1000 HPE Comware switch is a massive expense for a company, one that they will fight tooth and nail. I have to remember that I live in a world where line rate switches are a luxury, vendor support is a waking nightmare, and whitebox is still not cost competitive for our use case. Much of the network projects that I assist with have been designed with the unspoken understanding that resiliency is too expensive - single fiber pulls between campus data frames to save on cabling and optics, single ESXi hosts (using free hypervisor licensing) because multiple hosts with vSAN would be expensive, single ISP circuits for internet connectivity (or what may be worse, multiple circuits from the same service provider for 'redundancy').

The reality of the situation is that most of our clients do not really *need* the same uptime requirements that larger enterprises have. If they really did, there would be some serious navel-gazing going on to find out how to spend for that requirement, and that would probably include their own IT staff instead of hiring an MSP. The very act of hiring a service provider presupposes some level of detachment of IT from the business, and for a lot of SMBs *that's OK*. The world is not going to end if a CPA firm cannot get to the internet for 4 hours. Our role as the service provider is to identify what within that enterprise does have stricter uptime requirements, such as phones at a law firm, industrial equipment at a manufacturing firm, etc.

#### Conclusion

This was a bit of a rambly post - I am still working on an outlining process for these posts, but I wanted to get my thoughts out on networking focus in an MSP environment. Are any of you working for MSPs out there? DO you have specialized staff for these types of escalations? How do you manage design in an environment where cutting quotes by a hundred dollars is a huge win?

