---
published: true
---
## Migration Day



Jun 16, 2018

This past Friday was the execution day for a large project for one of our 
clients – and I mean that in all definitions of the word. The client 
moved their office _and_ their datacenter in a single evening. 
One long night and morning later, I am writing this post to reflect on 
what went well, what didn’t, and what I need to do better.

#### The Good

- Our datacenter migration process went _mostly_ without a 
hitch. We used VMWare Site Recovery Manager to ship the VMs to the new 
datacenter, and I had already planned out and staged as many of the 
networking changes as I could – firewall rules, OSPF configuration, etc.
 The only change I had to make was to update our core Nexus addressing 
to reflect the shipped prefix, and the client’s datacenter frootprint 
was off to the races.  
- I am fortunate that the project came with enough lead time that I was
 able to write a fairly detailed runbook for the process. With firewall 
changes premade and configuration templates already created for the 
switching changes, all that was left was for the managed service teamto 
make their DNS changes, and the client was up. 
- All of the necessary players (application teams, server teams, 
virtualization, phone and SIP providers) had already been pre-wrangled 
so if someone was needed, they were close at hand. This was particularly
 useful when we start to dive into… 

#### The Bad

- The client had placed a good amount of arbitrary restrictions of what
 equipment could be moved, in the name of reducing the possibility of 
catastrophic failure, such as moving backup appliances before powering 
off servers, moving one server at a time, etc. While prudent on the 
face, this added several hours to the migration process. Given the 
secondary controls we have in place regarding redundant copies of server
 backups, some of this time could have been reclaimed. 
- The office move was a nearly unmitigated disaster, of which I am at 
least (or at least feel) partly responsible. As the sole network 
engineer for my company, I have to wear many hats, including datacenter 
networking and networking specific escalations, and my involvement in 
this project had required my input on several parts of the design, from 
switch stacking to storage networking, routing protocol design and 
firewall configuration. Being stretched this thin means details get 
missed, and I neglected to advise the team that jumbo frames would be 
required on the storage switching – an oversight that cost several hours
 of wasteful troubleshooting. 
- We had divided the assigned technicians into two groups, one focused 
on the datacenter migration and one focused on the office move. 
Communication regarding the status of the move was routed through both 
our project manager and the client’s project manager, meaning status 
updates were at best uneven and at worst nonexistent. 

And finally…

#### The Ugly

- The vision for the new campus switching did not get translated into 
actual configuration. I neglected to validate the configurations once 
they had been completed, and this resulted in several hours late into 
the night and through the next morning and early afternoon reconfiguring
 them.
- Blatant oversights in campus switching, like completely missing 
spanning tree configuration, mismatched stacking domain IDs, leaving 
spanning tree enabled (once configured!) in the VLAN facing the WAN 
circuit, causing their equipment to go err-disabled. While attention to 
detail is something that must be consistently cultivated, I can 
definitely, with hindsight, see that I was stretched too thin in the run
 up to this migration.

### Final Thoughts

This migration has put my efforts into automation in stark relief. I 
was very meticulous in how I documented the time that I spent both in 
the preparation for this migration and the migration itself, and 
reviewing that time shows not only the massive amount of labor hours 
that went into the project – hours that could have been severely reduced
 by using automated processes to perform tasks, but more critically, how
 a predictable, reliable automation system could reduce errors by 
applying a state that is configured outside of the sleep-deprived mind 
of an engineer at 3AM.
