# CodeFest 2020 IOS XE Events project
### Contributors
####   Don Jones
####   Phil Selker

## **Overview**

An event happens with an interface on an IOS XE device.  The event (an interface goes down) is the trigger to start an EEM (Embedded Event Manager) script.  The script performs the following actions:
- "show tech" and saves the output in a .txt file on the device
- Runs a python script in the guestshell

The python script performs the following actions:
- Creates a ServiceNow ticket with information about the event
- Uploads the .txt file to the ServcieNow ticket

The ServcieNow ticket performs the following action:
- Uses a Webex Teams Bot to send an alert message to a Team space stating a critical failure has occurred, provide url to view the ServiceNow ticket and provides instructions on how to start an instant meeting iwth all users in the space.

Below are instructions on how to enable an environment to execute this demo

---

### **Table of Contents**

- [ServiceNow account and Developer instance](https://https://github.com/pselker2/IOSXE_Events/#servicenow-account-and-developer-instance)
- [Add ServiceNow app from repo to your Dev instance](https://github.com/pselker2/IOSXE_Events/#add-servicenow-app-from-repo-to-your-dev-instance)
- [Create Webex Teams Bot](https://github.com/pselker2/IOSXE_Events/#create-webex-teams-bot)
- [Add Bot to a Webex Teams space](https://github.com/pselker2/IOSXE_Events/#add-bot-to-a-webex-teams-space)
- [Get Webex Teams roomId](https://github.com/pselker2/IOSXE_Events/#get-webex-teams-roomid)
- [Add Bot Access Token and roomId to ServiceNow app](https://github.com/pselker2/IOSXE_Events/#add-bot-access-token-and-roomid-to-servicenow-app)
- [Validate](https://github.com/pselker2/IOSXE_Events/#validate)
- [ServiceNow trouble shooting tips](https://github.com/pselker2/IOSXE_Events/#servicenow-trouble-shooting-tips)
- [Python files](https://github.com/pselker2/IOSXE_Events/#python-files)
- [EEM script](https://github.com/pselker2/IOSXE_Events/#eem-script)
- [Enable guestshell](https://github.com/pselker2/IOSXE_Events/#enable-guestshell)

---

### **ServiceNow account and Developer instance**

1. Create an account on [developer.servicenow.com](https://developer.servicenow.com)  
2. Log in and navigate to Manage > Instance
3. Request Instance
    - There are several versions (Choose the most recent instance if no preference)
4. Start Dev Instance by clicking on URL
    URL looks like:  https://dev12345.service-now.com/
4. Optional: Take the following courses: 
    (developer.servicenow.com menu: Learn > Training):
    - Build My First Application
    - Build the NeedIt Application
    - Scripting in ServiceNow
    
---

### **Add ServiceNow app from repo to your Dev instance**

1. git clone https://https://github.com/pselker2/IOSXE_Events.git to a public repo like GitHub to enable the ServiceNow Dev instance to access it
2. ServiceNow Navigator > Studio
    - A new browser tab opens with the Load Application window
    - Select Import From Source Control
    - Use the information from the **new** repo you cloned in step 1
    - In the left Studio pane you should see files associated with the IOSXE_Events app
    - Studio > Source Control > Create Branch
        - The Branch will capture all your changes
        - Source Control > Commit Changes  to save changes to the new repo 

---

### **Create Webex Teams Bot**

1. Log in on [developer.webex.com](https://https://developer.webex.com) (Sign up if you need an account)
2. Click on the "Start Building Apps" button
3. Click on the "Create a New App" button
4. Click on "Create a Bot" button
5. Fill in the information and click the "Add Bot" button at the bottom of the page
6. Save the "Bot's Access Token" in a safe place

---

### **Add Bot to a Webex Teams space**

1. Create a space for testing the Bot you just created
2. Add your new Bot to the space
    - Click the circle with three dots in the upper right corner of the space
    - Click the People circle
    - Click the Add people circle
    - Type the Bot Username (example@webex.bot)
    
---

### **Get Webex Teams roomId**

1. Log in on [developer.webex.com](https://https://developer.webex.com)
2. Click on the "Documentation" 
3. Click on "API Reference" on the left side
4. Click on "Teams" on the left side
5. Click on the Get Method url with the Description List Teams
6. Click the yellow "Run" button on the right side
    - All the Teams you are in will be listed in the response section
    - Copy the "id" of the Team that contains the space with your Bot
7. Click on "Rooms" on the left side
8. Under "Method" click on the Get url to "List Rooms"
    - On the right under "Query Parameters"
        - Paste the "id" of the Team from step 6 into the "teamId" parameter
        - Click the yellow "Run" button on the right side
        - Copy the "id" of the Room that contains your Bot

---

### **Add Bot Access Token and roomId to ServiceNow app**

1. ServiceNow Studio browser tab
    - Click on "SN Bot for Webex Teams" in the left pane under "Business Rules"
    - Right pane click "Advanced" tab
    - Replace the roomId in line 18 of the script with your roomId you found in the previous section. (roomId in the cloned script looks like 'Y2lzY29...ZTNk')
    - Click the "disk" icon to save the script change
    - Click the "Update" button in upper right
2. ServiceNow Studio browser tab
    - Click on "Cisco Webex Teams Integration" in the left pane under "REST Messages"
    - Click on the "HTTP Request" tab in the right pane
    - Click the "Authorization" link in the right pane
    - In the "value" box replace only the Bot access token that looks like NWViY...e10f with the access token of the Bot you created in the "Create Webex Teams Bot" section
    - Click the "Update" button in the upper right 
    
---

### **Validate**

1. In the Navigator browser tab (not Studio) type "IOSXE-Events"
    - You should see IOSXE-Events listed twice
    - Click the second "IOSXE-Events"
    - Click the blue "New" icon
    - Click "Submit" button in upper right
    - You should see that a record was created in the right pane
    - You should also have a message in your Webex Teams space from the BOT in ServiceNow that provides the information you just entered in the ServiceNow form
    
---

### **ServiceNow trouble shooting tips**

1. In the ServiceNow Navigator
    - Find/Search window 
        - Type System Log
            - View Script Log Statements
            - View Application Logs

---

### **Python files**

1. Download the config.py and create_snow_ticket.py from this repository:  https://github.com/pselker2/Python-files-for-IOSXE_Events
2. Copy the files to the bootflash directory on the device
    
---

### **EEM Script**

```
event manager applet interface_shutdown
  event syslog pattern ".*Interface Loopback0, changed state to administratively down.*" maxrun 60
  action 1.0 cli command "enable"
  action 1.5 cli command "delete /force flash:tech.txt"
  action 2.0 cli command "show tech | redirect flash:tech.txt"
  action 3.0 cli command "guestshell run python /bootflash/create_snow_ticket.py"
  action 4.0 cli command "config t"
  action 5.0 cli command "interface Loopback0"
  action 6.0 cli command "no shutdown"
  action 7.0 cli command "end"
```

See this link for EEM documentation:  https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/eem/configuration/xe-3s/eem-xe-3s-book/eem-overview.html#GUID-A33C0359-B4BE-46FE-8992-0D468A246E4A

---


### **Enable guestshell**

See this link https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/1611/b_1611_programmability_cg/guest_shell.html

When building guestshell do the following first:
```
app-hosting appid guestshell
vnic gateway1 virtualportgroup 0 guest-interface 0
```

Then add:
```
vnic gateway1 virtualportgroup 0 guest-interface 0 guest-ipaddress 192.168.35.2 netmask 255.255.255.0 gateway 192.168.35.1 name-server 8.8.8.8
```

Then run:
```
guestshell enable
```
