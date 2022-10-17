# Venus_rgpio
Remote Relay for Cerbo GX
Adding external Relays to Cerbo GX

As I wanted the relays to not rely on an external Ethernet network, I went with a Modbus/RTU based relai from Dingtian.
This box provide 8x relays, that can be controlled with Modbus Serial, but also over IP and with various additional protocols: https://fr.aliexpress.com/item/4000999069820.html They have variants of 4 or 8 relays.
A USB to RS 485 adapter will be required. I selected this one as this is coming with a USB cable and can fit nicely on the Cerbo GX: https://fr.aliexpress.com/item/1005004778767986.html

Now we need to do few teaks so the 4x first relays are completely managed by the Cerbo GX GUI.
Only 4x relays can be added at bus level so the Victron’s Node-Red flow and Cerbo GX Gui can control it.
The remaining relays will be managed directly from Node-Red Modbus flow


The below steps requires to ssh to the Cerbo GX with root access.
I suggest you download the configuration files in /data/rgpio so it remains there even after SW upgrades.
Any suggestion to execute the below steps in a better appropriate way (may be also adding automatic adding it from /etc/rcS.d) are welcome.
Repository is here:
https://github.com/Lucifer06/Venus_rgpio

cd /data/
wget github.com/Lucifer06/Venus_rgpio/releases/download/Latest/rgpio.tar.gz
tar -xvzf rgpio.tar.gz
rm rgpio.tar.gz


1/ Creating /dev/gpio links at boot so the bus services are automatically created

cd /etc/rcS.d
ln -s /data/rgpio/conf/S90rgpio_pins.sh /etc/rcS.d/S90rgpio_pins.sh



2/ Modify Relaystate Python script

mv /opt/victronenergy/dbus-systemcalc-py/delegates/relaystate.py /opt/victronenergy/dbus-systemcalc-py/delegates/relaystate.py.ori
cp /data/rgpio/conf/relaystate.py /opt/victronenergy/dbus-systemcalc-py/delegates/relaystate.py


3/ Need to add Relays 3, 4, 5 and 6 in /etc/venus/gpio_list so they can be configured on the GUI

mv /etc/venus/gpio_list /etc/venus/gpio_list.ori
cp /data/rgpio/conf/gpio_list  /etc/venus/gpio_list


4/ Need to update Node-Red service for adding the 4x relays

mv /usr/lib/node_modules/@victronenergy/node-red-contrib-victron/src/services/services.json /usr/lib/node_modules/@victronenergy/node-red-contrib-victron/src/services/services.json.ORI
cp /data/rgpio/Node-Red/services.json /usr/lib/node_modules/@victronenergy/node-red-contrib-victron/src/services/services.json


5/ Reboot or restart the services

svc -d /service/dbus-systemcalc-py/ ; svc -u /service/dbus-systemcalc-py/
svc -d /service/gui ; svc -u /service/gui
svc -d /service/node-red-venus ; svc -u /service/node-red-venus


6/ Display and configure the additional 4x relais in the Venus GUI

Name and display the additional relays from Settings / Relays in the GUI.
You can swipe to a specific page with the 6x relays, but I believe this is provided by the excellent GuiMods add-on from Kwinderm.

![6x relays GUI](https://user-images.githubusercontent.com/10178879/196140950-01c7880d-6900-4fce-ae18-c9d671c7e0e8.png)


7/ Test with command lines

List of all dbus relays and their status:
dbus -y com.victronenergy.system /Relay GetValue

For some reasons I can’t control the relays. If someone knows the correct command line?
Here is what I tried:
dbus -y com.victronenergy.system /Relay/3/State SetValue 1


8/ Using Node-Red for controlling the 8x additional relays

The Relays 3, 4, 5 and 6 are normally controlled with Victron’s Relay Nodes, and their status are correctly reported on the Victron GUI
Use flow Relays3456.json as example

The additional 4x relais 7, 8, 9 and 10 are exposed only through Node-Red Dashboard.
They all require the flow Dingtian_Relays.json for sensing the status and controlling the remote Relay from the Dingtian box, attached via RS422 (ModBus RTU protocol).


9/ Additional Digital Inputs

The Dingtian relay box also offer additional Digital inputs.
Here is the flow for sensing the values of the Digital Inputs

