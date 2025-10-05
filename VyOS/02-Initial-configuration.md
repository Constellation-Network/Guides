# Initial configuration of vyos
This will configure basic things such as username, password and basic commands to know

## Setting a new hostname
When you set up a new router the hostname will be vyos, this is ugly and you have no clue what it does. Change it to something descriptive.

1. Type `configure`, you can also use `conf`. 

You will notice that the `$` changes to `#`. This means that you are in configuration mode. From here you can change all your settings with `set`, `delete`, etc. Use TAB to autocomplete

The best way to think about VyOS configuration commands is like a folder or drop down menu. For example

2. Type `set system host-name VTT-00`

This configures the host name by going into the system settings. Right now our changes are staged, to see what is going to happen type `compare` or `compare commands` to see the commands that are getting committed. 

VyOS works by staging the commands then *committing* them so they become active then *saving* them.

3. Type `commit-confirm`, this will auto reboot the system after 10 minutes incase you mess it up. To confirm that the changes work after that type `confirm`.

If you are so inclined you can also type `commit` directly however **DO NOT RUN THIS REMOTELY, IF YOU MESS IT UP YOU HAVE TO GO IN PERSON**

4. Finally type `save`, this will write it to the config file

Basically VyOS works by reading everything from a config file and using that. theoretically you can edit it directly but I can't recommend that because you'll probably mess something up and it wont work anymore.

If you want to go back to running mode type `exit`. Once we have applied the hostname we need to reboot the router with `reboot` | `y`