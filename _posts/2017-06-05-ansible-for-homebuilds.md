---
layout: post
title:  "Using Ansible for your home build PCs"
date:   2017-06-05 08:00:00 +0100
categories: blog
---
### Introduction
When you work long enough in IT you will begin to get your friends, family and
maybe even other colleagues identify you as the 'IT guy' and begin asking you
to fix their computer.  This weekend was no exception!

The girlfriends HP Pavillion had finally given up the ghost (HDD failed) and after
unsuccessfully sending it to another 'IT guy' who summarily stated 'Yes it's broken'
after having it for two weeks I was finally requested to have a go at it.  Since
laptop repair really isn't in my day job I thought hey why not and said id give
it a go (not like I could have broken it further).

A quick bit of shopping on the old [ebuyer](https://www.ebuyer.com) and I bought a cheap
1Tb Seagate spin disk to replace the old disk and while I waited for that to arrive
set to writing a touch of my favorite configuration management tool, Ansible.

I asked the better half what she wanted re-installing on it after I put the new
OS on it, 'R and firefox' came back... Well easy! I thought, not really an insane
role but it was good for me to test my capabilities in using Ansible to speed up
jobs like software installs on fresh builds.  My theory is a tool is only as good
as how easy it is to use and if you need to hack around for an hour to get it
working, you may as well have installed the software manually!

### Test machine setup and Ansible code
So after downloading the [Windows 10 ISO image](https://www.microsoft.com/en-gb/software-download/windows10ISO) from
Microsoft website and spinning up a Windows 10 Home VM in Virtual box. I added a
host only network adapter and installed it just as I would be doing with the physical
machine.  Then got to writing some Ansible to download and install all the requisite
softwares.  Below is the full extract of the `tasks/main.yml` for my `base-windows-build` role:

{% highlight yaml %}
---
# tasks file for base-windows-build
- name: Make ansible dir
  win_file:
    dest: '{{site.ansible_dir}}'
    state: directory

- name: Download & Install R
  block:
    - name: Download R
      win_get_url:
        url: 'https://cloud.r-project.org/bin/windows/base/R-3.4.0-win.exe'
        dest: '{{site.ansible_dir}}\R-3.4.0-win.exe'
      args:
        creates: '{{site.ansible_dir}}\R-3.4.0-win.exe'
    - name: Install R
      win_command: '{{site.ansible_dir}}\R-3.4.0-win.exe /VERYSILENT /COMPONENTS="main,x64"'
      args:
        creates: 'C:\Program Files\R'

- name: Download & Install R Studio
  block:
    - name: Download R Studio
      win_get_url:
        url: 'https://download1.rstudio.org/RStudio-1.0.143.exe'
        dest: '{{site.ansible_dir}}\RStudio-1.0.143.exe'
    - name: Install R Studio
      win_command: '{{site.ansible_dir}}\RStudio-1.0.143.exe /S'
      args:
        creates: 'C:\Program Files\RStudio'

- name: Download and install Mozilla Firefox
  block:
    - name: Download Mozilla Firefox
      win_get_url:
        url: 'https://download.mozilla.org/?product=firefox-53.0.3-SSL&os=win64&lang=en-GB'
        dest: '{{site.ansible_dir}}\firefox-53.exe'
    - name: Install firefox-53
      win_command: '{{site.ansible_dir}}\firefox-53.exe -ms'
      args:
        creates: 'C:\Program Files\Mozilla Firefox'

- name: Download and install Adobe Acrobat PDF Reader
  block:
    - name: Download Adobe Reader 11
      win_get_url:
        url: 'ftp://ftp.adobe.com/pub/adobe/reader/win/11.x/11.0.10/en_US/AdbeRdr11010_en_US.exe'
        dest: '{{site.ansible_dir}}\reader-11.exe'
    - name: Install Adobe Reader 11
      win_command: '{{site.ansible_dir}}\reader-11.exe /SILENT'
      args:
        creates: 'C:\Program Files\Adobe'
  when: adobe_acrobat == 'present'
{% endhighlight %}


As you can see, it is very basic and only uses two Ansible modules `win_get_url`
which downloads the installer packages into the ansible_dir (C:\Ansible) and
`win_command` which runs the installer in 'silent' mode (i.e. commandline install).
I used my standard 'ansible-base-framework' and added a small play for testing:

{% highlight yaml %}
---
- name: Apply windows role to machine
  hosts:
    - windoze
  vars:
    ansible_user: xxxxxx
    ansible_password: xxxxxx
    ansible_dir: 'C:\ansible'

  roles:
    - { role: base-windows-build,
      adobe_acrobat: absent }
{% endhighlight %}

Some of the connection details I add to the `windoze.yaml` file to keep from having
to define them in each play, here they are:

{% highlight yaml %}
ansible_port: 5986
ansible_connection: winrm
ansible_winrm_server_cert_validation: ignore
{% endhighlight %}

After this it is simply a case of editing the frameworks `hosts` file with the IP
address of the VM running locally and we can test it out!  For the sake of your
sanity I will not post the entire play execution, rather just the last two lines:

{% highlight bash %}
2017-06-02 09:47:57,918 p=1 u=root |  PLAY RECAP *********************************************************************
2017-06-02 09:47:57,919 p=1 u=root |  192.168.0.21               : ok=8    changed=7    unreachable=0    failed=0   
{% endhighlight %}
We are all good to go!

### The Build
Ok so code tested locally last week and I had a message from the girlfriend to say
that the new hard drive had come so this Saturday just gone was time to see how my
laptop repair skills were!  

After getting everything ready on the table and HPs
disassembly guide open on my other laptop I turned it over to find about 80% of
the screws holding the top board to the case were missing!  Seems like the previous
repair man tried taking the back off, missed a few key hidden ones
(seriously HP, who the F*** puts screws UNDER the rubber feet which means you have
to peel them off to get access!) and then not put them back in once he had
given up and sent it back!  Nonetheless I removed the 4 or so remaining screws and
the laptop came apart in two and after disconnecting the trackpad and keyboard
ribbon connectors pulled the keyboard / top case off the base and then had access
to the 2.5" hard drive that had undergone a pretty spectacular mechanical failure
(We were on Skype at the time of failure and I even heard it grinding the platters
to oblivion over the call!).  After quickly disconnecting it's ribbon from the main
board and replacing it with the new one (tool-less replacement / frame setup) I put
the laptop back together hoping that all of the ribbons were done up properly.  

While I had been doing this I had my other laptop building a USB install stick for
the windows 10 ISO image I had downloaded earlier in the week. When that was ready
it was simply a case of plugging it in, changing the boot order in the laptop UEFI
to USB first and restarting, then following a routine windows 10 install and
visiting github for a powershell script which configures windows machines to enable
the remote management 'WinRM' protocol and executing this.  This is all thats needed
to give Ansible access to the machine.  Once this was complete all I had to do was
get the laptop to join my LAN, make a note of it's IP address and instruct Ansible
(running on another laptop) to go and configure the host.  
In all it took about 5 mins to run through and install all of the software that was needed.

### Conclusion
Ansible is a very powerful and easy to use piece of software, I love the fact it
is agent-less (great for not bloating the host you are configuring), works using
native powershell (for windows) and Python (for linux) meaning you do not need to
install anything additional on a host to get Ansible to work, it's open source
and written in Python too, meaning should the day ever come that I need to write a
new module it shouldn't be too hard to do.  Although I doubt I will ever need to
as I have not met a situation to this day I couldn't accomplish using Ansibles
standard modules.  It is also very flexible and I find it's ability to be used for
configuring single hosts, small clusters and even up to estates in the hundreds
of servers.  Admittedly I have not used Ansible at that kind of scale _yet_
but am confident if I ever had to, it would cope more than acceptably.

In terms of time spent:
Writing the Ansible code took about an hour, testing it probably took another hour
and I know there will be cries of "You could have installed it manually in less
time than that!".  While this is true, the next time someone asks for a Win10
rebuild on their machine I will probably save over half an hour.  Then another the
next time I build a machine.  

Although the use-case in this article is tiny, it did give me the assurance to use
Ansible not only in my day job, but also for making smaller personal builds easier too.  

I hope this is useful to anyone hoping to see a real world usage of Ansible albeit
a very small example, or just what really happens behind the scenes when you send
a laptop into the repair shop for a new hard drive.

Thanks for reading!
