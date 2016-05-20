## Welcome to the FreeNAS 10 VM templates archive

In this repo, you will find all of the "canned templates" for creating VMs
on FreeNAS 10 - what you see when you use the ```vm template show``` command.

Here's the step-by-step process I used to create the FreeBSD-current (11.0)
template, using [freebsd-current](ftp://ftp.freebsd.org/pub/FreeBSD/snapshots/ISO-IMAGES/11.0/FreeBSD-11.0-CURRENT-amd64-20160518-r300097-disc1.iso) as the
starting image.  I also used bhyve running on FreeBSD 10.3 as the bootstrap
host, though some folks have reported good results with xhyve on the Mac.

* First, obviously, I needed to check out the vm-templates repo and start working in it:
```
git clone https://github.com/freenas/vm-templates.git
cd vm-templates.git
```

* Next, I copied the template that looked the most like my target template.  In my case, since I was targetting another FreeBSD template, it was obvious enough to simply duplicate the existing FreeBSD 10.2-zfs template (a 10.2 install with the ZFS option selected).
```
cp -pr freebsd-10.2-zfs freebsd-11-zfs
```

* Then I grabbed an ISO installation image from ftp.freebsd.org, as linked above, and started the steps to get bhyve ready to boot it:
```
# Note: These initialization steps for bhyve are necessary to do only once.
kldload vmm
ifconfig tap0 create
sysctl net.link.tap.up_on_open=1
ifconfig bridge0 create
ifconfig bridge0 addm igb0 addm tap0	# replace igb0 with your primary NIC
ifconfig bridge0 up

# Now make a 16gb image file for the HD - this is referenced later, too.
truncate -s 16g disk.img

# Now run bhyve's helpful vmrun.sh script to start things off.
sh /usr/share/examples/bhyve/vmrun.sh -c 1 -m 1024M -t tap0 -d disk.img -i -I FreeBSD-11.0-CURRENT-amd64-20160518-r300097-disc1.iso freebsd-current
```

* At this point, FreeBSD's standard installer ran, the appropriate ZFS installation options were chosen, and I exited bhyve by selecting the loader prompt on the next reboot and typing "quit".  This dropped me back to the shell on the host OS, where I was next able to do:

```
mv disk.img os.img
gzip -9 os.img
```

This little rename/compress step was just to conform with the same naming conventions as my source template, at which point I then edited the ```template.json``` file in my new freebsd-11-zfs direcory to correctly reference this new image, I uploaded the os.img.gz file to the location specified in the ```url``` field (which could be any HTTP server you have access to) and filled in the ```sha256``` checksum field by running ```shasum -a 256 os.img.gz``` and pasting in the results.

* Finally, I committed the result to github with a git commit / git push, since as a FreeNAS committer I have write access (non-freenas project members would send us a pull request), and voila!  My FreeNAS 10 CLI now shows:

```
unix::>vm template show
       Name                          Description                
boot2docker           boot2docker Docker host for FreeNAS       
ubuntu-server-14.04   Ubuntu Server 14.04                       
ubuntu-cloud-14.04    Ubuntu Cloud Edition 14.04                
freebsd-10.2-zfs      FreeBSD 10.2 image with root on ZFS       
freebsd-11-zfs        FreeBSD 11-current image with root on ZFS 
```

Demonstrating that the template list is automatically pulled from github.

* Of course, the final proof was to actually create a VM with the new template on my FreeNAS 10 box:

```
unix::>vm create name=bleedingedge template=freebsd-11-zfs enabled=yes volume=tank
unix::>vm bleedingedge start
unix::>vm bleedingedge console
```

Login is a root (no password), tada!  Running FreeBSD-current from this new template.
