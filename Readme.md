# IR Notes

## Windows ##
### Windows Event IDs and Lateral Movements ###

| Scheduled Tasks Log  | XP: %SystemRoot%\SchedLgu.txt    7: %SystemRoot%\Tasks\SchedLgu.txt |             |                                                   |
|----------------------|---------------------------------------------------------------------|-------------|---------------------------------------------------|
|                      | 106                                                                 |             | Task Scheduled                                    |
|                      | 200                                                                 |             | Task Executed                                     |
|                      | 201                                                                 |             | Task Completed                                    |
|                      | 141                                                                 |             | Task Removed                                      |
| Logon Events         | 528                                                                 | 4624        | Successful Logon                                  |
|                      | 529                                                                 | 4625        | Failed Logon                                      |
|                      | 538                                                                 | 4647 / 4634 | Successful Logoff                                 |
|                      | 540                                                                 | 4624        | Successful Network Logon                          |
|                      |                                                                     | 4672        | Successful Network Logon as Admin                 |
| RDP                  | 21                                                                  |             | RDP logon success                                 |
|                      | 24                                                                  |             | RDP user disconnected                             |
|                      | 24                                                                  |             | RDP user reconnected                              |
|                      |                                                                     | 1149        | RDP user authenticated                            |
| Account Logon Events | 680                                                                 | 4776        | Successful / Failed account authentication        |
|                      | 672                                                                 | 4768        | TGT was issued (successful logon)                 |
|                      | 675                                                                 | 4771        | Pre-authentication failed (failed logon)          |
| Rogue Local Account  | 680                                                                 | 4776        | An account successfully authenticated             |
|                      | 540                                                                 | 4624        | Successful Network Logon immediately following    |
| Share                |                                                                     | 5140        | Share mount                                       |
| Suspicious Services  |                                                                     | 7034        | Service crashed unexpectedly                      |
|                      |                                                                     | 7035        | Service sent a Start/Stop control                 |
|                      |                                                                     | 7036        | Service sent a started or stoped                  |
|                      |                                                                     | 7040        | Start type changed (Boot | On request | Disabled) |
| Clearing Event Logs  | 517                                                                 |             |                                                   |

### Normal ###
Reference: http://digital-forensics.sans.org/media/poster_2014_find_evil.pdf

#### System ####
	Image Path: No Image Path
	Parent Process: No Parent Process
	Number of Instances : One 
	User account: Local System
	Start Time: At boot time

smss.exe
Image Path: %SystemRoot%\System32\smss.exe
Parent Process: System
Number of Instances : One master and another child per session exiting after session is created
User account: Local System
Start Time: Within seconds of boot time for the master instance

wininit.exe
Image Path: %SystemRoot%\System32\wininit.exe
Parent Process: Created by an instane of smss.exe that exists (tools usually don't provide the parent process name)
Number of Instances : One
User account: Local System
Start Time: Within seconds of boot time

taskhost.exe
Image Path: %SystemRoot%\System32\taskhost.exe
Parent Process: services.exe
Number of Instances : One or more
User account: Multiple taskhost.exe processes are normal. Logged-on users and/or local services accounts
Start Time: Within seconds of boot time

lsass.exe
Image Path: %SystemRoot%\System32\lsass.exe
Parent Process: wininit.exe
Number of Instances : One
User account: Local System
Start Time: Within seconds of boot time

winlogon.exe
Image Path: %SystemRoot%\System32\winlogon.exe
Parent Process: Created by an instane of smss.exe that exists (tools usually don't provide the parent process name)
Number of Instances : One or more
User account: Local System
Start Time: Within seconds of boot time for the first instance

csrss.exe
Image Path: %SystemRoot%\System32\csrss.exe
Parent Process: Created by an instane of smss.exe that exists (tools usually don't provide the parent process name)
Number of Instances : Two or more
User account: Local System
Start Time: Within seconds of boot time for the first two instances (Session 0 and 1)
Note: cmd.exe history is stored in these processes' memory 

services.exe
Image Path: %SystemRoot%\System32\services.exe
Parent Process: wininit.exe
Number of Instances : One
User account: Local System
Start Time: Within seconds of boot time for the first two instances (Session 0 and 1)

svchost.exe
Image Path: %SystemRoot%\System32\services.exe
Parent Process: services.exe
Number of Instances : Five or more
User account: Depends of the instance : Local System, Network Service or Local Service accounts
Start Time: Within seconds of boot time or later for services launched after boot
Note: On Win7+ all services bin are signed by Microsoft

lsm.exe
Image Path: %SystemRoot%\System32\lsm.exe
Parent Process: wininit.exe
Number of Instances : One
User account: Depends of the instance : Local System
Start Time: Within seconds of boot time
Note: Handled terminal services including RDP and Fast user switching

explorer.exe
Image Path: %SystemRoot%\explorer.exe
Parent Process: userinit.exe that exists (tools usually don't provide the parent process name)
Number of Instances : One per logged-on user
User account: logged-user
Start Time: Starts when the ownser's interactive session logon begins

### Artifacts ###
Reference: http://digital-forensics.sans.org/media/poster_fall_2013_forensics_final.pdf

$STDINFO
| File Rename          | Local File Move      | Volume File Move     | File Copy            | File Access                             | File Modify          | File Creation      | File Deletion        |
|----------------------|----------------------|----------------------|----------------------|-----------------------------------------|----------------------|--------------------|----------------------|
| Modified – No Change | Modified – No Change | Modified – No Change | Modified – No Change | Modified – No Change                    | Modified – Change    | Modified – Change  | Modified – No Change |
| Access – No Change   | Access – No Change   | Access – Change      | Access – Change      | Access – Change No Change on Vista/Win7 | Access – No Change   | Access – Change    | Access – No Change   |
| Creation – No Change | Creation – No Change | Creation – No Change | Creation – Change    | Creation – No Change                    | Creation – No Change | Creation – Change  | Creation – No Change |
| Metadata – Changed   | Metadata – Changed   | Metadata – Changed   | Metadata – Changed   | Metadata – Changed                      | Metadata – Changed   | Metadata – Changed |                      |

$FILENAME
| File Rename          | Local File Move      | Volume File Move   | File Copy          | File Access          | File Modify          | File Creation      | File Deletion        |
|----------------------|----------------------|--------------------|--------------------|----------------------|----------------------|--------------------|----------------------|
| Modified – No Change | Modified – Change    | Modified – Change  | Modified – Change  | Modified – No Change | Access – No Change   | Modified – Change  | Modified – No Change |
| Access – No Change   | Access – No Change   | Access – Change    | Access – Change    | Access – No Change   | Access – No Change   | Access – Change    | Access – No Change   |
| Creation – No Change | Creation – No Change | Creation – Change  | Creation – Change  | Creation – No Change | Creation – No Change | Creation – Change  | Creation – No Change |
| Metadata – No Change | Metadata – Changed   | Metadata – Changed | Metadata – Changed | Access – No Change   | Access – No Change   | Metadata – Changed | Metadata – No Change |

## Procmon filters
Operation is WriteFile
Operation is RegSetValue
Details containts Desired Access: Generic Write

## OS X ##
List process related to port XXXX (bash)
`process=`lsof -n -i4TCP:XXXX | grep -v COMMAND | cut -d' ' -f1` ; for i in $process; do ps aux | grep $i | cut -d' ' -f 39- ; done`

## Linux ##
List process related to port XXXX 
`process=`sudo netstat -anp | egrep ":XXXX\s" | cut -d/ -f 1 | rev | cut -d' ' -f1 | rev` ; for i in $process; do ps aux | grep $i | grep -v grep; done`