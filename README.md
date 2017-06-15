# omrsafetyo.github.io

This is a blog intended to demonstrate how I have solved problems using DevOps approaches.  

## What is DevOps?

DevOps doesn't really have an absolute definition.  For some it simply means applying development practices to IT Operations.  This means doing things like using Puppet to define your infrastructure as code, and keeping it in version control, and applying tests upon changes.  For many, DevOps is simply a tighter collabration between Developers and Operations, which allows for faster, continuous software delivery.  

For me, I don't necessarily use DevOps for software delivery.  Instead, what I'm interested in is delivering services, such as VM builds, VM modifications, DNS services, user account management, etc.  My goal in my DevOps role is to bridge the gap between VMWare architects, Network Engineers, and Operations teams to break down the barriers of dinosaur change management and cumbersome operational runbook routines, and wrap these into a streamlined process that reduces the time from request to delivery.

## My Story

My main tool for years now has been PowerShell.  Prior to being a Windows Administrator, I was an Operations Specialist in a couple different UNIX environments.  In my last job, I was a lowly Computer Operator in a SCO UNIX and Windows 2000 Server environment.  One of my daily tasks was to take a lot of reports from a couple of systems; literally books of reports, and go through manually to make sure the numbers matched up.  Part of this process was going through a series of xls (Excel) file dumps, and opening them one at a time to find reports that met certain criteria - printing these, and then matching them up against the corresponding reports in another document.  I thought there had to be a better way.  So I got to work on an Excel macro in VBA, and before too long, I had a little Excel VBA application that would, at the very least,  go find a selection of reports that met my criteria, and then print them off for me.  In hind sight, this doesn't sound like a lot, but it saved me and others an hour or so of work every day.  From there, I began automating in Bash as well, as my managers learned that I was able to make my position more efficient, they gave me a little free reign.

When I moved on to my current employer, I had scripting in Bash, VBScript, and VBA under my belt.  The environment at the time was purely AIX UNIX with INFORMIX database.  I started using ksh (Korn shell) for my scripting.  Shortly after I joined though, we started installing new products that ran on Windows with SQL Server, and SharePoint.  Having come from a ~50% Windows environment, in a largely UNIX shop, I became the Windows guy.

I rewrote all of our scripts, one at a time in PowerShell.  It certainly wasn't good Powershell, as I was using v1, and didn't have any experience.  It mostly consisted of wrappers of directory services commands (dsadd, etc.), and other Windows batch commands; along with some use of the GNU UnixUtils commands.  So I was very lazy at first about learning PowerShell the correct way, and instead leveraged a lot of built-ins, and different shells where I could.

But eventually, I got past that.  I now have tens of thousands of lines of PowerShell across various scripts.  I get a weekly report about each of the servers in my environment to ensure they are getting patched.  I have a daily report of any of over 10,000 databases that are missing backups.  I get weekly reports to ensure that our systems are in our monitoring software, inventory management, cross-check the VMWare environment against Active Directory to ensure computer accounts that no longer exist in VMWare are can get cleaned up; scripts that check our databases to ensure they are compliant with our DBA's set standards (appropriate growth settings, versions, etc.).

But like everyone else, I'm still learning, icnluding new technologies, and new techniques and standards.  This blog will be a look into my journey of automating the IT world.

## Technologies

The technologies I use most frequently are:

* Powershell
* PowerCLI
* REST
* vRealize Orchestrator
* the occasional C# with MVC and Entity Framework
