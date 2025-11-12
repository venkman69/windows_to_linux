
My personal windows to linux migration

Web is filled with comparisons for users to use this or that linux that is "MOST" like the windows we use - I felt most focused on UI aspects and how closely it mimicked windows. Which is, to be fair, definitely useful, but for me it does not deliver a full-on solution. I have to emphasize "me" in this as this was a personal journey with personal requirements that many others might have no care for - but I suspect there are those like me who would love to know how it went (and perhaps some who would say 'you did all of it wrong' 😊 ).

So, in my particular case I have a perfectly running windows 11 laptop - which is my daily driver, and I use wsl to develop on it and the workflow and the ecosystem is nice. My changes are pushed via git and I use a 10yo (i7 4770 with 32GB) as my server and use that as my 'server'.<br>
So with the recent Win 10 end of life and no path to upgrade to windows 11 (double whammy on the lack of TPM and unsupported CPU) I went in headlong into a migration effort.

# TL;DR

Summary is that my windows setup while glitchy in cases scored a 68, and moving to linux improved it to 76. So, it is better but not by a whole lot. In some sense I would have stuck with Windows if upgrade path was clear and available and avoided the hassle. But I love hassle 😂

| Index | Feature | Windows Score | Linux Score | 
 | ----- | ----- | ----- | ----- | 
| 1a | Remote Desktop (UI access) | 5 | 2 | 
| 1b | Startup with UI and Services | 2 | 5 | 
| 1c | NAS Mounts | 2 | 5 | 
| 1d | UPS Shutdown (via NUT) | 5 | 4 | 
| 1e | Wake on LAN (WoL) | 3 | 5 | 
| 1f | Port Forwarding (WSL/Services) | 2 | 5 | 
| 2 | Handbrake UI, MakeMKV | 5 | 5 | 
| 3 | Synology Active Backup/Baremetal Recovery | 5 | 2 | 
| 4 | PRTG Monitor (or alternative) | 5 | 3 | 
| 5 | Jellyfin | 5 | 5 | 
| 6 | Media Management Helpers (Python/Streamlit) | 4 | 5 | 
| 7a | Development Tools: WSL/Environment Stability | 4 | 5 | 
| 7b | Development Tools: VSCode | 5 | 5 | 
| 7c | Development Tools: uv/Python | 5 | 5 | 
| 7e | Development Tools: Podman | 5 | 5 | 
| 8 | Overall Stability | 3 | 5 | 
| 9 | Overall Repeatability | 3 | 5 | 
| **TOTAL SCORE** | (Out of 80) | **68** | **76** |

# My thoughts on of each of the items above
My laundry list that I needed to migrate:

1. Windows server related<br>
   a. Remote desktop:
      Because I still want a UI to run handbrake and MakeMKV etc.
      - **Windows: 5**<br>
        This is absolutely trivial to setup in windows and works fast and flawlessly. I use parallels free client on my IPad, laptop, android and have enjoyed the simplicity of connecting remotely.
      - **Linux: 2**<br>
        This was oddly very tricky. After many hours of struggling I found that if you are logged in directly on the server, then remoting in gets all confused and does not work. Later I accidently sourced a non-existant path in my home .profile - which has no issue during login but xrdp would not launch. Took me a while to troubleshoot that - why make something break due to a user's .profile? There must be some reason - but this effort altogether took way too much time and effort. Now I have gone and commented out xrdp's startup so it does not source user home .profile at all.<br>
        I hit several other snags along the way and each was very hard to troubleshoot, config files in etc, sudo-ing, checking logs, but the .profile one was key.<br>Once this was figured out, xrdp works very well - but I wanted the install to be a no-brainer and it hurt my brain.<br>

   b. Startup with UI and Services
      - **Windows: 2**<br>
        Setting up autologin in windows was quite simple and worked flawlessly.<br>
        Then having taskscheduler launch services was also pretty sane. But once WSL came along I moved a number of items into systemd and cron within WSL. That made it even more solid. <br>
        However, oddly I would have WSL die every now and then. I have never been able to figure out why. When I restart it by hand, everything would just work. To the point I created a task to launch wsl (which never quite worked).<br>
        I have spent an inordinate amount of time chasing intermittent issues.
      - **Linux: 5**<br>
        So remote desktop does not like autologin - gotcha! so don't do that! But.. EVERYTHING else is rock solid with cron/systemd elements starting up predictably.

   c. NAS mount
      - **Windows: 2**<br>
        What has been a really complex setup for mounting a NAS share from my synology. To run services, wsl, task scheduler items you have to have autologon. But there is a "who's on first" type of battle where it is complex to tell a non-logged in windows to mount a NAS share. For one thing after reading a lot there is a fragile setup where you have to muck with registry or group policy etc to mount the Synology drive at-boot. I never quite figured out the entire math on the not-logged-in state vs logged-in state. It finally worked and even if the mounts were marked with a red 'x' they were navigable. But every now and then it would need to be reset - and it would break my Jellyfin server (and all other services using the share) since I stored my media on the NAS.<br> This was a periodic pain that needed attention.
      - **Linux: 5**<br>
        Once the cifs entries were in my /etc/fstab it is done, there is nothing more to do - it just works flawlessly right after boot. It was shockingly simple.

   d. UPS shutdown
        I have my UPS connected to by Raspberry PI which exposes a NUT service.
      - **Windows: 5**<br>
        With win-nut tools it was quite nice to see a UI in the toolbar and see notifications and setup the config to shutdown when on battery etc.
      - **Linux: 4**<br>
        Not quite the same experience when you are mucking around command line and config files and just way too much confusion for a first timer. Now that I have it documented on what is what it is possible I won't spend as much time, but win-nut is awesome.

   e. Wake on Lan
      - **Windows: 3**<br>
        This was an odd experience. My motherboard and whatever settings, there were certain combinations of steps where wake-on-lan would not work. For example if I shut the server down, then it would work. However if power failed and came back on it wouldn't - or maybe I am mixing the two - either way it was not predictable and simple.
      - **Linux: 5**<br>
        Just worked - I haven't tried all the combinations of power faiures yet, but somehow I don't think that is a problem here.

   f. Port forwarding WSL
      - **Windows: 2**<br>
        While you can run all sorts of services in WSL, it needs to be port forwarded using admin `netsh`. This was always painful and was also confusing to keep in sync when starting new things and decommissioning old.
      - **Linux: 5**<br>
        No such issues running services. Only thing I ever had to do was to knock down the ability for non-root to use ports down to 80 (`net.ipv4.ip_unprivileged_port_start=80`). And it is an internal personal server that is not exposed in anyway to internet etc.

2. Handbrake UI, MakeMKV
   - **Windows: 5**<br>
     works fine
   - **Linux: 5**<br>
     works fine.
3. Synology Active Backup (and baremetal recovery)
   - **Windows: 5**<br>
     Works amazingly well and really simple setup.
   - **Linux: 2**<br>
     I cannot make Synology backup client work at all.<br> After some effort, I setup `Relax and Recover` (ReaR) which is very good, but I think if Synology backup would just work, I'd use that in a heartbeat. Now, I can confirm both Synology backup and ReaR do a good job with recovering from baremetal back to a fully operational state. I have now used both several times.
     So, I am giving 2 here because of the effort to setup ReaR.
4. PRTG monitor
   - **Windows: 5**<br>
     This was my go-to free monitor and it did SO many things simply out of the box, drive temps, checking SSL certs, email warnings. Setup and running is amazingly trivial. With extensibilty being a bit wonky (I could not understand the schema at first and somehow got it all working in python eventually).
   - **Linux: 3**<br>
     I am testing CheckMK, it is not bad, but it is no PRTG. Visually I feel PRTG dash is much simpler to see. But, I can see getting used to CheckMK and bringing it to par with PRTG.
5. Jellyfin
   - **Windows: 5**<br>
     Simple to install and run, nice little toolbar helper icon, however see 1.c NAS mounts, once they flunk then jellyfin is lost.
   - **Linux: 5**<br>
     I moved it into a podman and with the stability of the cifs mounts so far it has been solid.
6. Several media management helpers I run using Python and streamlit
   - **Windows: 4**<br>
     I was running everything from cron or systemd inside WSL. But the WSL itself sometimes would not wake up. So it was good, but system was not super stable. PRTG would send me alerts and I would have to go in and kick it.
   - **Linux: 5**<br>
     I have moved all into systemd (said bye-bye to cron as well) and it is quite smooth so far.
	
7. Development tools

   a. wsl
      - **Windows: 4**<br>
        Everything is good until it dies for no reason.<br>
        Then there is an odd condition where it creates a TON of files in `%APPDATA%/Local/Temp/DiagOutputDir/RdClientAutoTrace`. Suddenly too. It will be in a few kb one moment and about 100MB in the next moment. I actually had a WSL cron job running checking the 'du' on this folder and blowing everything if it got past 50MB or so. Some other constant glitches here and there, but mostly solid.
      - **Linux: 5**<br>
        No wsl just itself 😊

   b. vscode
      - **Windows: 5**<br>
        Just works.
      - **Linux: 5**<br>
        Just works.

   c. uv/python
      'uv' is just magic! From swapping out python versions to installing libs this thing is amazing. I am moving most of my workflows to 'uv' especially because of the python installation isolation.
      - **Windows (wsl): 5**<br>
        Within wsl, everything works fine.
      - **Linux: 5**<br>
        Works fine. Without `uv` changing python version was scary for me in linux, this is a total game changer. Windows side I could install new python in another folder and I've made that work with little or no trouble but in Linux installing new python without messing up the OS needs is super tricky. `uv` totally rocks.

   e. podman
      - **Windows (wsl): 5**<br>
        Just Works.
      - **Linux: 5**<br>
        Just works.

8. Overall Stability
   - **Windows: 3**<br>
	 Needs periodic maintenance and updates. Things like it will wake up after reboot and get stuck on the screen that wants you to click on sign up for something button. 
   - **Linux: 5**<br>
	 Seems pretty set it and forget it.

9. Overall Repeatability
   - **Windows: 3**<br>
	 Much of the configs are manual. Very little can be put into source due to OS level elements. I tried to move most things into WSL as best as I could but there are just certain things that were too tricky.
   - **Linux: 5**<br>
     Most of the items are in source control. I think I changed the protected ports inside the server to 80 to make it easier to run Caddy for rev proxy.


So, summary - while I moved initially to a Debian Q4OS, I am continuing to move to an ESXi right now as an experiment, next stop will be proxmox.
