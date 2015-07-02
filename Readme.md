# IR Notes

## Windows 
### Windows Event IDs and Lateral Movements 

| Scheduled Tasks Log  | XP: %SystemRoot%\SchedLgu.txt - 7: %SystemRoot%\Tasks\SchedLgu.txt  |             |                                                   |
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

### Normal 

#### System 
	Image Path: No Image Path
	Parent Process: No Parent Process
	Number of Instances : One 
	User account: Local System
	Start Time: At boot time

#### smss.exe 
	Image Path: %SystemRoot%\System32\smss.exe
	Parent Process: System
	Number of Instances : One master and another child per session exiting after session is created
	User account: Local System
	Start Time: Within seconds of boot time for the master instance

#### wininit.exe
	Image Path: %SystemRoot%\System32\wininit.exe
	Parent Process: Created by an instane of smss.exe that exists (tools usually don't provide the parent process name)
	Number of Instances : One
	User account: Local System
	Start Time: Within seconds of boot time

#### taskhost.exe
	Image Path: %SystemRoot%\System32\taskhost.exe
	Parent Process: services.exe
	Number of Instances : One or more
	User account: Multiple taskhost.exe processes are normal. Logged-on users and/or local services accounts
	Start Time: Within seconds of boot time

#### lsass.exe
	Image Path: %SystemRoot%\System32\lsass.exe
	Parent Process: wininit.exe
	Number of Instances : One
	User account: Local System
	Start Time: Within seconds of boot time

#### winlogon.exe
	Image Path: %SystemRoot%\System32\winlogon.exe
	Parent Process: Created by an instane of smss.exe that exists (tools usually don't provide the parent process name)
	Number of Instances : One or more
	User account: Local System
	Start Time: Within seconds of boot time for the first instance

#### csrss.exe
	Image Path: %SystemRoot%\System32\csrss.exe
	Parent Process: Created by an instane of smss.exe that exists (tools usually don't provide the parent process name)
	Number of Instances : Two or more
	User account: Local System
	Start Time: Within seconds of boot time for the first two instances (Session 0 and 1)
	Note: cmd.exe history is stored in these processes' memory 

#### services.exe
	Image Path: %SystemRoot%\System32\services.exe
	Parent Process: wininit.exe
	Number of Instances : One
	User account: Local System
	Start Time: Within seconds of boot time for the first two instances (Session 0 and 1)

#### svchost.exe
	Image Path: %SystemRoot%\System32\services.exe
	Parent Process: services.exe
	Number of Instances : Five or more
	User account: Depends of the instance : Local System, Network Service or Local Service accounts
	Start Time: Within seconds of boot time or later for services launched after boot
	Note: On Win7+ all services bin are signed by Microsoft

#### lsm.exe
	Image Path: %SystemRoot%\System32\lsm.exe
	Parent Process: wininit.exe
	Number of Instances : One
	User account: Depends of the instance : Local System
	Start Time: Within seconds of boot time
	Note: Handled terminal services including RDP and Fast user switching

#### explorer.exe
	Image Path: %SystemRoot%\explorer.exe
	Parent Process: userinit.exe that exists (tools usually don't provide the parent process name)
	Number of Instances : One per logged-on user
	User account: logged-user
	Start Time: Starts when the ownser's interactive session logon begins

#### Reference
http://digital-forensics.sans.org/media/poster_2014_find_evil.pdf

### Artifacts 
To-Do

| File Download | Open/Save MRU                                                                                                                                                                                                   | E-mail Attachments                                                                                                                                                                                      | Skype History                                                                           | Index.dat/ Places.sqlite                                                                                                                                                                                                              | Downloads.sqlite                                                                                                                                                        |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|               | This key tracks files that have been opened or saved within a Windows shell dialog box                                                                                                                          | E-mail Attachments                                                                                                                                                                                      | Skype history                                                                           | Not directly related to “File Download”. Details stored for each local user account. Records number of times visited (frequency)                                                                                                      | Firefox has a built-in download manager application which keeps a history of every file downloaded by the user                                                          |
|               | XP: NTUSER.DAT\Software\Microsoft\Windows\ CurrentVersion\Explorer\ComDlg32\OpenSaveMRU                                                                                                                         | XP: %USERPROFILE%\Local Settings\Application Data\ Microsoft\Outlook                                                                                                                                    | XP: C:\Documents and Settings\<username>\Application\Skype\<skypeusername>              | XP: IE: %userprofile%\Local Settings\History\ History.IE5 FF: %userprofile%\Application Data\Mozilla\ Firefox\ Profiles\.default\places.sqlite                                                                                        | XP: %userprofile%\Application Data\Mozilla\ Firefox\ Profiles\.default\downloads.sqlite                                                                                 |
|               | Win7: NTUSER.DAT\Software\Microsoft\Windows\,CurrentVersion\Explorer\ComDlg32\ OpenSavePIDlMRU                                                                                                                  | Win7: %USERPROFILE%\AppData\Local\Microsoft\ Outlook                                                                                                                                                    | Win7: C:\Users\<username>\AppData\Roaming\ Skype\<skypeusername>                        | Win7: IE: %userprofile%\AppData\Local\Microsoft\Windows\History\History.IE5 %userprofile%\AppData\Local\Microsoft\Windows\History\Low\History.IE5 FF:  %userprofile%\AppData\Roaming\Mozilla\ Firefox\Profiles\.default\places.sqlite | Win7: %userprofile%\AppData\Roaming\Mozilla\ Firefox\ Profiles\.default\downloads.sqlite                                                                                |
|               | The “*” key – This subkey tracks the most recent files of any extension input in an OpenSave dialog .??? (Three letter extension) – This subkey stores file info from the OpenSave dialog by specific extension | MS Outlook data files found in these locations include OST and PST files. One should also check the OLK and Content.Outlook folder, which might roam depending,on the specific version of Outlook used. | Each entry will have a date/time value and a Skype username associated with the action. | Many sites in history will list the files that were opened from remote sites and downloaded to the local system. History will record the access to the file on the website,that was accessed via a link.                              | Downloads sqlite will include:  Filename, Size, and Type Download from and Referring Page File Save Location Application Used to Open File Download Start and End Times |

| Program Execution | UserAssist                                                                                    | LastVisited MRU                                                                                                                                                                                                                | RunMRU Start->Run                                                                                                                                     | AppCompact Cache                                                                                                                            | Win7 Jump Lists                                                                                                        | Prefetch                                                                                                                                                                                                                                                                                                                                                                       | Service Events                       |
|-------------------|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
|                   | GUI-based programs launched from the desktop are tracked in the launcher on a Windows System. | Tracks the specific executable used by an application to open the files documented in the OpenSaveMRU key. In addition, each value also tracks the directory location for the last file that was accessed by that application. | Whenever someone does a Start -> Run command, it will log the entry for the command they executed.                                                    | Windows Application Compatibility database. Tracks the executable file names, file size, last modified time and in XP the last update time  |                                                                                                                        | Utilized to know an application was executed on a system.                                                                                                                                                                                                                                                                                                                      | Analyze logs for suspicious services |
|                   | NTUSER.DAT\Software\Microsoft\Windows\ Currentversion\Explorer\UserAssist\{GUID}\Count        | XP: NTUSER.DAT\Software\Microsoft\Windows\ CurrentVersion\Explorer\ComDlg32\ LastVisitedMRU Win7: NTUSER.DAT\Software\Microsoft\Windows\ CurrentVersion\Explorer\ComDlg32\,LastVisitedPidlMRU                                  | NTUSER.DAT\Software\Microsoft\Windows\ CurrentVersion\Explorer\RunMRU                                                                                 | XP: SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatibility\ Win7: SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache | Win 7: C:\Users\username\AppData\Roaming\Microsoft\Windows\Recent\ AutomaticDestinations                               | C:\Windows\Prefetch                                                                                                                                                                                                                                                                                                                                                            |                                      |
|                   |                                                                                               | Tracks the application executables used to open files in OpenSaveMRU and the last file path used.                                                                                                                              | The order in which the commands are executed is listed in the RunMRU list value. The letters represent the order in which the commands were executed. | Tool: MANDIANT’s ShimCacheParser                                                                                                            | Creation Time = First time item added to the AppID file. Modification Time = Last time item added to the AppID file.   | Each .pf will include last time of execution, number of times run, and device and file handles used by the program   Date/Time file by that name and path was first executed - Creation Date of .pf file (-10 seconds),  Date/Time file by that name and path was last executed  - Embedded last execution time of .pf file - Last modification date of .pf file (-10 seconds) |                                      |


#### Reference
http://digital-forensics.sans.org/media/poster_fall_2013_forensics_final.pdf

### Windows Time Rules

#### $STDINFO

| File Rename          | Local File Move      | Volume File Move     | File Copy            | File Access                             | File Modify          | File Creation      | File Deletion        |
|----------------------|----------------------|----------------------|----------------------|-----------------------------------------|----------------------|--------------------|----------------------|
| Modified – No Change | Modified – No Change | Modified – No Change | Modified – No Change | Modified – No Change                    | <b>Modified – Change</b>    | <b>Modified – Change</b>  | Modified – No Change |
| Access – No Change   | Access – No Change   | <b>Access – Change</b>      | <b>Access – Change</b>      | <b>Access – Change</b> No Change on Vista/Win7 | Access – No Change   | <b>Access – Change</b>    | Access – No Change   |
| Creation – No Change | Creation – No Change | Creation – No Change | <b>Creation – Change</b>    | Creation – No Change                    | Creation – No Change | <b>Creation – Change</b>  | Creation – No Change |
| <b>Metadata – Changed</b>   | <b>Metadata – Changed</b>   | <b>Metadata – Changed</b>   | <b>Metadata – Changed</b>   | <b>Metadata – Changed</b>                      | <b>Metadata – Changed</b>   | <b>Metadata – Changed</b> | Metadata – No Change |

#### $FILENAME

| File Rename          | Local File Move      | Volume File Move   | File Copy          | File Access          | File Modify          | File Creation      | File Deletion        |
|----------------------|----------------------|--------------------|--------------------|----------------------|----------------------|--------------------|----------------------|
| Modified – No Change | <b>Modified – Change</b>    | <b>Modified – Change</b>  | <b>Modified – Change</b>  | Modified – No Change | Access – No Change   | <b>Modified – Change</b>  | Modified – No Change |
| Access – No Change   | Access – No Change   | <b>Access – Change</b>    | <b>Access – Change</b>    | Access – No Change   | Access – No Change   | <b>Access – Change</b>    | Access – No Change   |
| Creation – No Change | Creation – No Change | <b>Creation – Change</b>  | <b>Creation – Change</b>  | Creation – No Change | Creation – No Change | <b>Creation – Change</b>  | Creation – No Change |
| Metadata – No Change | <b>Metadata – Changed</b>   | <b>Metadata – Changed</b> | <b>Metadata – Changed</b> | Metadata – No Change   | Metadata – No Change   | <b>Metadata – Changed</b> | Metadata – No Change |


### Mass Registry analysis with RegRipper

	$ find path_to_the_files/ -type f -exec ./wrapper.sh {} \; 

	wrapper.sh
	./rip.exe -r "$1" -p user_run >> results.txt


### Procmon filters
	Operation is WriteFile
	Operation is RegSetValue
	Details containts Desired Access: Generic Write

### Domain users' SIDs

vol.py -f memdump.mem --profile=Win7SP1x64 getsids > getsids_output.txt
grep 'S-1-5-21-XXXXXXXXX-XXXXXXXXX-XXXXXXXXX-' getsids_output.txt | egrep '[A-Z][0-9]{6}' -o | sort -u

## OS X ##
List process related to port XXXX (bash)
	
	$ process=`lsof -n -i4TCP:XXXX | grep -v COMMAND | cut -d' ' -f1` ; for i in $process; do ps aux | grep $i | cut -d' ' -f 39- ; done

## Linux ##
List process related to port XXXX 
	
	$ process=`sudo netstat -anp | egrep ":XXXX\s" | cut -d/ -f 1 | rev | cut -d' ' -f1 | rev` ; for i in $process; do ps aux | grep $i | grep -v grep; done
