# SCEPTRE Field Device User Guide

## **Connecting to Brash Shell**

On the “system tools” section of the engineer-workstation desktop,

double-click on “putty.exe”.

![](img/manual/UpdatesApril2019/EngineeringWorkstation.JPG)

Once PuTTY is open, double click on the field device you would like to connect
to. Ensure that you are connecting via telnet with port 1337.

![](img/manual/UpdatesApril2019/putty.JPG)

Once you’re connected to the field device, login to the brash shell with the
following credentials:

**Username: `sceptre`**

**Password: `sceptre`**

![](img/manual/UpdatesApril2019/brashStartUp.JPG)

## **Shell Commands**

**`help`:** Displays all supported commands with respective descriptions.

![](img/manual/UpdatesApril2019/helpMenu.JPG)

**`arpShow`:** Displays the system ARP table entries.

![](img/manual/UpdatesApril2019/arpShow.JPG)

**`clear`:** Clears the terminal screen.

**`configShow`:** Displays the configuration file using the text editor vi in the terminal

![](img/manual/UpdatesApril2019/ConfigShow1.JPG)

![](img/manual/UpdatesApril2019/ConfigShow2.JPG)

**`exit`:** Exits the shell and returns the user to the Brash login prompt.

**`FieldDeviceRestart`:** Stops and restarts the field device daemon and
displays the PID before and after restarting.

![](img/manual/UpdatesApril2019/fielddevicerestart-good.JPG)

**`fieldDeviceStart`:** Initializes the RTU field device daemon and gives it a
PID. If the field device daemon is not running, PID will be 0 as in the first
image. The second image is the message if the field device is already running
and cannot be started.

![](img/manual/UpdatesApril2019/fielddevicestart.JPG)

![](img/manual/UpdatesApril2019/fielddevicestartAlreadyRunning.JPG)

**`FieldDeviceStop`:** Stops the RTU field device daemon and displays its PID.

![](img/manual/UpdatesApril2019/fielddevicestop.JPG)

**`ifShow`:** Displays system network interfaces.

![](img/manual/UpdatesApril2019/ifShow.JPG)

**`inetstatShow`:** Displays all active IP socket connections.

![](img/manual/UpdatesApril2019/inetstatshow.JPG)

**`logShow`:** Displays the log for the current field device and the firmware
version.

![](img/manual/UpdatesApril2019/logShow.JPG)

**`passwd`:** Change user password.

![](img/manual/UpdatesApril2019/passwd.JPG)

**`routeShow`:** Displays all IP routes.

![](img/manual/UpdatesApril2019/routeshow.png)

**`su`:** Switch to brash root account. (Password: admin)

![](img/manual/UpdatesApril2019/su.JPG)

**`time`:** Displays the system time.

![](img/manual/UpdatesApril2019/time.JPG)

**`updateConfig`:** Updates the field device configuration with a file
uploaded over FTP. In order to perform this action, must first be root. If the
user is not root, the following error message will be displayed

![](img/manual/UpdatesApril2019/updateConfig.JPG)

## **Connecting as Administrator**

**Note**: This method is only accessible from the MGMT network (172.16.0.0/16).

On the “system tools” section of the engineer-workstation desktop,
double-click on “putty.exe”.

Once PuTTY is open, double click on the field device you would like to connect
to. Ensure that you are connecting via ssh with port 22 using MGMT network IP
addresses.

Once you're connected to the field device, login with the following credentials:

**Username: `root`**

**Password: `SiaSd3te`**
