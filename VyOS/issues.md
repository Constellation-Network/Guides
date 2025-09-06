# Issues
The following is a list of issues we have while setting stuff up

## 1 - When Attempting To Enter Configuration Mode There Is An Error
```vyos
dawaffleman@SUS-00:~$ conf
WARNING: There was a config error on boot: saving the configuration now could overwrite data.
You may want to check and reload the boot config
[edit]
dawaffleman@SUS-00# 
```
### Backup
Before attempting any troubleshooting I decided to back up just in case I messed up the server
1. Typed `configure`
2. Typed `show`
3. Held **Down Arrow** Until I was at the bottom
4. Copied the config

### Reboot
It appeared that a simple reboot fixed the issue, however this can delete any changes you made and did not commit
1. Typed `reboot`
2. Pressed **Y** | **Enter**

