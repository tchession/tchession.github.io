### My 'Dev' Process

In my previous post I wrote that I would talk about the dev tools and workflow that I use in my day job. I will stress that I am not a professional developer, I have no formal training in how to do any of this, and am mostly piecing stuff together through observation of people way smarter than me! Like any good story, I will be breaking this post up into three parts.

#### 1) The OS

I'm not ashamed to admit I am a Windows man. I've grown up on the Redmond Borg cube and feel most comfortable with it's eccentricities. I feel that Microsoft has taken giant steps in reconciling it's reputation among the FOSS community, and the Microsoft <3 Linux tagline is not just marketing. 

There are also practicalities involved. My job, like many, issues Windows laptops. While we have a fair amount of latitude in configuring our devices the way we like, we are limited to Windows. All of our tooling for supporting clients is Windows based, we heavily leverage the Office 365 suite for productivity software, and the technicians we hire and users we support are almost universally Windows-trained.

#### 2) The Language

This is a topic known to rankle the feathers of pretty much every developer out there. In much the same way that many playground arguments when I was a kid were started with Nintendo vs Sega, devs today can often be very territorial of the languages they use. My focus, however, is less on a specific language or tooling and more on how to reach a specified outcome - in this case, a more reliable, resilient, automated network. 

For this purpose, Python is far and away the tool of choice, especially for the rookie network developer/NetDevOps engineer/NRE. Python has a supremely rich ecosystem of libraries and packages made by people much smarter than me that can help with the minutiae of network automation, allowing me to focus on the task at hand. That's not to say that Python completely obfuscates the need to actually understand concepts like data structures, REST, transport, etc, only that these third party tools help keep you sane managing it all. I will cover in more detail the Python specific libraries and tooling that I use in my development process in a later post.

I know that many people in the network automation community are keen on using other, more abstracted tools, most notably Ansible. I think Ansible is a great tool, and I believe that I could do everything that I currently do from an automation perspective with Ansible today. I can, however, also see that Ansible is like a theme park - exciting, flashy, and quick to get to something fun, but also extremely guard railed. It is important to understand that when using a tool like Ansible, you are writing in a domain specific language to a tool in which you have very little access to traditional troubleshooting and debugging tools when things go wrong. Python is more like a backpacking trip in the woods - a long, hard slog through more difficult terrain, but you can see everything.

#### 3) = 1 + 2

The one part of the workflow that I do think makes my process a little unique is that I do not use Python for Windows or the Windows versions of any of my tooling, such as the Poetry package manager. The only component of my toolset that I have installed in Windows is a copy of Git so that I can use the Git credential manager to store my repo credentials, as I'm not quite comfortable enough with my knowledge of SSH keys to configure them to commit to my remote repositories.

I instead use Windows Subsystem for Linux (WSL), which provides a seamless system call translation between Linux commands and Windows binaries to run a (mostly) functional Linux distribution semi-natively on Windows. This allows me to use the Linux-native versions of git, Python (and it's associated tools like Poetry, pyenv, etc), and other great Linux tools like dig, curl, and so on. It even allows you to run these Linux binaries on files in your Windows filesystem, which allows me to keep my development folders in my company OneDrive!

WSL comes with two major drawbacks - poor filesystem performance and limited binary compatibility. For basic operations, WSL runs more or less seamlessly, but point any real amount of filesystem I/O at it and you will see it choke. ```poetry add nornir```, a command to add the Nornir automation framework package to my Python project, would normally take a few seconds, maybe a minute on Python for Windows or on an actual Linux machine. In my WSL environment, it took nearly an hour. It's a good thing I have other things to do with my time! The current version of WSL also has limitations with certain Linux tools, like Docker. It is currently not possible to run the Docker daemon natively within WSL(although you can use the Docker tooling in WSL to run containers in Docker for Windows!). A new version of WSL, coming later this year, promises to massively improve filesystem performance and allow full support for Linux binaries, which is very exciting.

#### How do you work?

That about does it for a brief overview of my workflow. I am still much more at home typing ```conf t``` and ```show ip int b``` than I am typing ```from pynetbox import api```, but every day I feel a little more confident. How are you managing your development workflow? Did deciding on diving into network programmability make you reconsider the tools that you use on a day to day basis? Let me know by sending me an email or dropping me a message on LinkedIn! Thanks for stopping by.