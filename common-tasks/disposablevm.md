---
layout: doc
title: DisposableVMs
permalink: /doc/disposablevm/
redirect_from:
- /doc/dispvm/
- /en/doc/dispvm/
- /doc/DisposableVms/
- /wiki/DisposableVMs/
---

# DisposableVMs #

A DisposableVM (previously known as a "DispVM") is a lightweight VM that can be created quickly and will disappear when closed.
DisposableVMs are usually created in order to host a single application, like a viewer, editor, or web browser.

From inside an AppVM, choosing the `Open in DisposableVM` option on a file will launch a DisposableVM for just that file.
Changes made to a file opened in a DisposableVM are passed back to the originating VM.
This means that you can safely work with untrusted files without risk of compromising your other VMs.
DisposableVMs can be launched either directly from dom0's Start Menu or terminal window, or from within AppVMs.
While running, DisposableVMs will appear in Qubes VM Manager with the name `disp####`.

See [this article](https://blog.invisiblethings.org/2010/06/01/disposable-vms.html) for more on why one would want to use a DisposableVM.


## Security ##

If a [DVM Template] becomes compromised, then any DisposableVM based on that DVM Template could be compromised.
In particular, the *default* DVM Template is important because it is used by the "Open in DisposableVM" feature.
This means that it will have access to everything that you open with this feature.
For this reason, it is strongly recommended that you base the default DVM Template on a trusted TemplateVM.

### DisposableVMs and Local Forensics ###

At this time, DisposableVMs should not be relied upon to circumvent local forensics, as they do not run entirely in RAM. 
For details, see [this thread](https://groups.google.com/d/topic/qubes-devel/QwL5PjqPs-4/discussion).

When it is essential to avoid leaving any trace, consider using [Tails](https://tails.boum.org/).


## Qubes 4.0 ##


### DisposableVMs and Networking ###

Similarly to how AppVMs are based on their underlying [TemplateVM](https://www.qubes-os.org/doc/glossary/#templatevm), DisposableVMs are based on their underlying [DVM Template](https://www.qubes-os.org/doc/glossary/#dvm-template).
R4.0 introduces the concept of multiple DVM Templates, whereas R3.2 was limited to only one.

On a fresh installation of Qubes, the default DVM Template is called `fedora-XX-dvm` (where `XX` is the Fedora version of the default TemplateVM).
If you have included the Whonix option in your install, there will also be a `whonix-ws-dvm` DVM Template available for your use.

You can set any AppVM to have the ability to act as a DVM Template with:

    qvm-prefs <vmname> template_for_dispvms True

The default system wide DVM Template can be changed with `qubes-prefs default_dispvm`.
By combining the two, choosing `Open in DisposableVM` from inside an AppVM will open the document in a DisposableVM based on the default DVM Template you specified.

You can change this behaviour for individual VMs: in the Application Menu, open Qube Settings for the VM in question and go to the "Advanced" tab. 
Here you can edit the "Default DisposableVM" setting to specify which DVM Template will be used to launch DisposableVMs from that VM.
This can also be changed from the command line with:

    qvm-prefs <vmname> default_dispvm <dvmtemplatename>

For example, `anon-whonix` has been set to use `whonix-ws-dvm` as its `default_dispvm`, instead of the system default.
You can even set an AppVM that has also been configured as a DVM Template to use itself, so DisposableVMs launched from within the AppVM/DVM Template would inherit the same settings.

NetVM and firewall rules for DVM Templates can be set as they can for a normal VM. 
By default a DisposableVM will inherit the NetVM and firewall settings of the DVM Template on which it is based.
This is a change in behaviour from R3.2, where DisposableVMs would inherit the settings of the AppVM from which they were launched.
Therefore, launching a DisposableVM from an AppVM will result in it using the network/firewall settings of the DVM Template on which it is based.
For example, if an AppVM uses sys-net as its NetVM, but the default system DisposableVM uses sys-whonix, any DisposableVM launched from this AppVM will have sys-whonix as its NetVM.

**Warning:** The opposite is also true. This means if you have changed anon-whonix's `default_dispvm` to use the system default, and the system default DisposableVM uses sys-net, launching a DisposableVM from inside anon-whonix will result in the DisposableVM using sys-net.

A DisposableVM launched from the Start Menu inherits the NetVM and firewall settings of the DVM Template on which it is based.
Note that changing the "NetVM" setting for the system default DVM Template *does* affect the NetVM of DisposableVMs launched from the Start Menu.
Different DVM Templates with individual NetVM settings can be added to the Start Menu. 

**Important Notes:**
Some DVM Templates will automatically create a menu item to launch a DVM, if you do not see an entry and want to add one please use the command:

    qvm-features deb-dvm appmenus-dispvm 1

To launch a DVM from the command line, in dom0 please type the following:
    
    qvm-run --dispvm=NameOfDVM --service qubes.StartApp+NameOfApp


### Opening a file in a DisposableVM via GUI ###

In an AppVM's file manager, right click on the file you wish to open in a DisposableVM, then choose "Open in DisposableVM". 
Wait a few seconds and the default application for this file type should appear displaying the file content. 
This app is running in its own dedicated VM -- a DisposableVM created for the purpose of viewing or editing this very file. 
Once you close the viewing application the whole DisposableVM will be destroyed. 
If you have edited the file and saved the changes, the changed file will be saved back to the original AppVM, overwriting the original.

![r1-open-in-dispvm-1.png](/attachment/wiki/DisposableVms/r1-open-in-dispvm-1.png) ![r1-open-in-dispvm-2.png](/attachment/wiki/DisposableVms/r1-open-in-dispvm-2.png)


### Opening a fresh web browser instance in a new DisposableVM ###

Sometimes it is desirable to open an instance of Firefox within a new fresh DisposableVM. 
This can be done easily using the Start Menu: just go to **Application Menu -\> DisposableVM -\> DisposableVM:Firefox web browser**. 
Wait a few seconds until a web browser starts. 
Once you close the viewing application the whole DisposableVM will be destroyed. 

![r1-open-in-dispvm-3.png](/attachment/wiki/DisposableVms/r1-open-in-dispvm-3.png)


### Opening a file in a DisposableVM via command line (from AppVM) ###

Use the `qvm-open-in-dvm` command from a terminal in your AppVM:

~~~
[user@work-pub ~]$ qvm-open-in-dvm Downloads/apple-sandbox.pdf
~~~

Note that the `qvm-open-in-dvm` process will not exit until you close the application in the DisposableVM.


### Starting an arbitrary program in a DisposableVM from an AppVM ###

Sometimes it can be useful to start an arbitrary program in a DisposableVM. This can be done from an AppVM by running

~~~
[user@vault ~]$ qvm-run '$dispvm' xterm
~~~

The created DisposableVM can be accessed via other tools (such as `qvm-copy-to-vm`) using its `disp####` name as shown in the Qubes Manager or `qvm-ls`.


### Starting an arbitrary application in a DisposableVM via command line from dom0 ###

The Application Launcher has shortcuts for opening a terminal and a web browser in dedicated DisposableVMs, since these are very common tasks.
However, it is possible to start an arbitrary application in a DisposableVM directly from dom0 by running:

~~~
$ qvm-run --dispvm=dvm-template --service qubes.StartApp+xterm
~~~

The label color will be inherited from the `dvm-template`.
(The DisposableVM Application Launcher shortcut used for starting programs runs a very similar command to the one above.)


### Customizing DisposableVMs ###

You can change the template used to generate the DisposableVMs, and change settings used in the DisposableVM savefile. 
These changes will be reflected in every new DisposableVM based on that template. 
Full instructions can be found [here](/doc/disposablevm-customization/).


## Qubes 3.2 ##


### DisposableVMs and Networking ###

NetVM and firewall rules for DisposableVMs can be set as they can for a normal VM. 
By default a DisposableVM will inherit the NetVM and firewall settings of the VM from which it is launched. 
Thus if an AppVM uses sys-net as its NetVM, any DisposableVM launched from this AppVM will also have sys-net as its NetVM. 
You can change this behaviour for individual VMs: in Qubes VM Manager open VM Settings for the VM in question and go to the "Advanced" tab. 
Here you can edit the "NetVM for DisposableVM" setting to change the NetVM of any DisposableVM launched from that VM.

A DisposableVM launched from the Start Menu inherits the NetVM of the [DVM Template](/doc/glossary/#dvm-template). 
By default the DVM template is called `fedora-XX-dvm` (where `XX` is the Fedora version of the default TemplateVM). 
As an "internal" VM it is hidden in Qubes VM Manager, but can be shown by selecting "Show/Hide internal VMs". 
Note that changing the "NetVM for DisposableVM" setting for the DVM Template does *not* affect the NetVM of DisposableVMs launched from the Start Menu; only changing the DVM Template's own NetVM does.


### Opening a file in a DisposableVM via GUI ###

In an AppVM's file manager, right click on the file you wish to open in a DisposableVM, then choose "Open in DisposableVM". 
Wait a few seconds and the default application for this file type should appear displaying the file content. 
This app is running in its own dedicated VM -- a DisposableVM created for the purpose of viewing or editing this very file. 
Once you close the viewing application the whole DisposableVM will be destroyed. 
If you have edited the file and saved the changes, the changed file will be saved back to the original AppVM, overwriting the original.

![r1-open-in-dispvm-1.png](/attachment/wiki/DisposableVms/r1-open-in-dispvm-1.png) ![r1-open-in-dispvm-2.png](/attachment/wiki/DisposableVms/r1-open-in-dispvm-2.png)


### Opening a fresh web browser instance in a new DisposableVM ###

Sometimes it is desirable to open an instance of Firefox within a new fresh DisposableVM. 
This can be done easily using the Start Menu: just go to **Application Menu -\> DisposableVM -\> DisposableVM:Firefox web browser**. 
Wait a few seconds until a web browser starts. 
Once you close the viewing application the whole DisposableVM will be destroyed. 

![r1-open-in-dispvm-3.png](/attachment/wiki/DisposableVms/r1-open-in-dispvm-3.png)


### Opening a file in a DisposableVM via command line (from AppVM) ###

Use the `qvm-open-in-dvm` command from a terminal in your AppVM:

~~~
[user@work-pub ~]$ qvm-open-in-dvm Downloads/apple-sandbox.pdf
~~~

Note that the `qvm-open-in-dvm` process will not exit until you close the application in the DisposableVM.


### Starting an arbitrary program in a DisposableVM from an AppVM ###

Sometimes it can be useful to start an arbitrary program in a DisposableVM. This can be done from an AppVM by running

~~~
[user@vault ~]$ qvm-run '$dispvm' xterm
~~~

The created DisposableVM can be accessed via other tools (such as `qvm-copy-to-vm`) using its `disp####` name as shown in the Qubes Manager or `qvm-ls`.


### Starting an arbitrary application in a DisposableVM via command line (from Dom0) ###

The Start Menu has shortcuts for opening a terminal and a web browser in dedicated DisposableVMs, since these are very common tasks.
However, it is possible to start an arbitrary application in a DisposableVM directly from Dom0 by running

R4.0 (border colour will be inherited from that set in the `dispvm-template`)
~~~
[joanna@dom0 ~]$ qvm-run --dispvm=dispvm-template --service qubes.StartApp+xterm
~~~

R3.2 (border colour can be specified in the command)
~~~
[joanna@dom0 ~]$ echo xterm | /usr/lib/qubes/qfile-daemon-dvm qubes.VMShell dom0 DEFAULT red
~~~

(The DisposableVM appmenu used for starting Firefox runs a very similar command to the one above.)


### Customizing DisposableVMs ###

You can change the template used to generate the DisposableVMs, and change settings used in the DisposableVM savefile. 
These changes will be reflected in every new DisposableVM based on that template. 
Full instructions can be found [here](/doc/disposablevm-customization/).


[DVM Template]: /doc/glossary/#dvm-template

