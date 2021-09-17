### Preparation

1. Open Trakcare download website:  
!!! note "ISC Download Location Details"
    **URL**: https://trakcaresupport.intersystems.com/Downloads/Patches/TrakCare/2016V2/  
    **Username**: trakhealth\swarh  
    **Password**: Stored in CCIO password vault

2. Download patch zip files from the website and save to

```
\\<SERVER>\D$\trakcare\ausw\<ENV>\patches\2016.2_<DATE>_<TIME>_R<PATCH>_B<BUILD>.zip
```
    For example:
```
\\satrkscrh16\D$\trakcare\\ausw\patches\2016.2\_20160120\_1602\_R7\_B92.zip
```
!!! note
    Move any existing old/other patches into the Archive subfolder.


3. Review any readme or other documentation

### Disable Application (Not required for AdHoc patches)

1. **[LIVE ONLY]** Log into TrakCare application, browse to Interface Workbench, note which interfaces are running and stop all interfaces, then log out.

2. Disable Trak Print Services and Ensemble Print Services for environment being patched by logging on to print servers:

> **LIVE**: SATRKLIVEPR1H16, BHTRKLIVEPR1H16  
> **OTHER**: SATRKTESTPRH16

3. **TPS:** Open Configure Trak Print Services, stop services for appropriate environment

4. **EPS:** Start the Healthshare System Management Portal for the print server, click ‘Ensemble’, and select the namespace you are patching. Navigate to Configure -> Production and click ‘Stop’.

5. **Disable the Application**:
   
   1. RDP to the server hosting the environment you are patching 
   
   2. Open Internet Explorer and navigate to CSP Gateway Management
      
```
http://localhost/csp/bin/Systems/Module.cxw?CSPSYS=0&CSPSYSreferer=_CSP.Portal.Home.zen
```
   
   3. Click Application Access
   
   4. Select the /trakcare/<*environment*>/web from the list box
   
   5. Select the ‘Edit Application’ option
   
   6. Click Submit
   
   7. Change Service Status to Disabled
   
   8. Click Save Configuration.
   
   9. Keep window open for enabling application later.

!!! note
    The system is unavailable to users from this point on.

### Backup Journal Files **[LIVE Only]** (Not required for AdHoc patches)

1. Navigate to
   
   ```
   \\swisilon-cifs\trakbackup$\LIVE-T2016
   ```

2. Open the log file:
   
   ```
   LIVE-TRAK_DAILY.log
   ```

3. Look for the text ‘Journal file switched to:’ and note the file name immediately following:
   
   ```
   Journal file switched to:
   j:\journal\AUSWLIVEDB_20150217.028
   ```

4. Open a Cache terminal session from your local machine to the appropriate Trak server and change to the %SYS namespace:
   
   ```
   ENV-TRAK>zn "%SYS"
   ```

!!! important
    Make sure you have the correct server selected in the Ensemble E popup menu.

5. Switch Caché to a new journal:
   
   ```
   %SYS> do ^JRNSWTCH
   Journal file switched to:
   j:\journal\AUSWLIVEDB_20150218.10
   ```

6. Note the file name of the journal file which was switched to.

7. Open Explorer window open to:
   
   ```
   \\trakcarelive\j$\Journal\tc
   ```

8. Copy the file noted in step 3, the file noted in step 6, and all files created between them to:
   
   ```
   \\swisilon-cifs\trakbackup$\LIVE-T2016\JournalBackup\
   ```

### Patching

1. In the terminal window, change namespace to the environment being patched and run ^SSADMIN:
   
   ```
   TRAKDB:%SYS>zn "<ENV-TRAK>"
   
   TRAKDB:<ENV-TRAK>>d ^SSADMIN
   ```

2. Select:
   
   ```
   1. Patching 
   ```

3. Select:
   
   ```
   5. Run AutoPatch
   ```

4. Answer ‘Y’ to each question
   
```
   ------------------------------------------------------------------------------
   This utility will execute:
   D ##class(websys.AutoPatch).Apply()
   to run the TrakCare auto patching utility.
   ------------------------------------------------------------------------------
   Run TrakCare AutoPatch in this environment : (Y/N) ?
   
   Before applying patches, do you want to validate some of the core TrakCare globals? : (Y/N) ?
   
   .......
   
   Before applying patches, do you want to validate reference map items ? : (Y/N) ? y
   
   .......
   
   After applying patches, do you want to recompile the custom classes 
   (Custom.*;User.U*;User.z*;web.U*;web.z*)? : (Y/N) ? 
   
   After applying patches, do you want to regenerate the questionnaires? : (Y/N) ?
   
   After applying patches, do you want to update security setting by running GrantAllSQLPrivileges? : (Y/N) ?
   
   After applying patches, do you want to run conversion routines? : (Y/N) ?
```

5. You will then receive a final prompt to update:
   
   ```
   ********************************************************************************
   Patching summary: PLEASE VERIFY BEFORE YOU CONTINUE
   ********************************************************************************
   Instance:            AUSWLIVEDB
   Namespace:            LIVE-TRAK
   Current full build version:    2014
   Patches to be applied:        2014_20150120_1602_R7_B92
   Recompile custom classes:    YES
   Regenerate questionnaires:    YES
   Update security settings:    YES
   ********************************************************************************
   Log file:
   I:\InterSystems\HealthShare\mgr\logs\TrakCareAutoPatch_20150218_0516_LIVE-TRAK.log
   ********************************************************************************
   
   Do you want to proceed and patch this system ?
   
   Final answer: Y
   ```

6. When complete press ‘Q’ to exit the patching utility

7. Press ‘Q’ to exit SSADMIN utility

8. On the server hosting the environment that has just been patched, go to the window that was left open from ‘Disable Application’ step 5.

9. Change Service Status: to Enabled

10. Click Save Configuration

11. Log on to Trak print servers and start TPS & EPS for the appropriate environment

12. Open a web browser and navigate to the TrakCare application

13. **[LIVE ONLY]** Log on to TrakCare, browse to Interface Workbench and enable all interfaces that were running before patching commenced.

14. **[LIVE ONLY]** Open the System Management Portal for the environment being patched and go to the Task Manager: ‘System Operation | Task Manager | Task Schedule’. Check for any extract tasks that have been suspended due to error and restart if necessary.

15. **[LIVE ONLY]** Copy web files to mirror and IIS servers:
    
    ```
    ROBOCOPY \\<PRIMARY MIRROR>\d$\trakcare\ausw\live\web \\<BACKUP MIRROR>\d$\trakcare\ausw\live\web /COPY:DA /XA:SH /S
    ROBOCOPY \\<PRIMARY MIRROR>\d$\trakcare\ausw\live\web \\satrkiis1h16\c$\trakcare\ausw\live\web /COPY:DA /XA:SH /S
    ROBOCOPY \\<PRIMARY MIRROR>\d$\trakcare\ausw\live\web \\bhtrkiis1h16\c$\trakcare\ausw\live\web /COPY:DA /XA:SH /S
    ```

> You can identify which server is primary mirror and which is backup mirror by viewing ‘Mirror Status’ in System Management Portal: http://trakcarelive:57772/csp/sys/op/%25CSP.UI.Portal.Mirror.Monitor.zen?$NAMESPACE=%25SYS

16. **[TEST ONLY]** Copy web files to mirror and IIS servers:
    
    ```
    ROBOCOPY \\<PRIMARY MIRROR>\d$\trakcare\ausw\test\web \\<BACKUP MIRROR>\d$\trakcare\ausw\test\web /COPY:DA /XA:SH /S
    ```

17. Check Zen report print server: If status in ‘Inactive’, restart:
    
    ```
    http:// trakcarelive:57772/csp/sys/mgr/%25CSP.UI.Portal.ZenReportPrintServers.zen?$NAMESPACE=LIVE-TRAK
    ```

18. Check Zen report render server. If status in ‘Inactive’, restart:
    
    ```
    http://trakcarelive:57772/csp/sys/mgr/%25CSP.UI.Portal.ZenReportRenderServers.zen?$NAMESPACE=LIVE-TRAKc
    ```

19. Verify patch version number by logging into the patched environment, go to ‘Tools’, ‘System’, ‘Build History’

20. An entry should have been added showing Build Version and Log No.

21. Notify users that application is available

### TrakCare Emergency Department Contact Details

| **Hospital** | **ED Phone No** | **ED Manager**                            | **Switchboard** |
| ------------ | --------------- | ----------------------------------------- | --------------- |
| Warrnambool  | 31473           | Kate Sloan – 31457                        | 5563 1666       |
| Portland     | 10340           | Linzi Donlan – 10665 or Deb Tozer – 10340 | 5521 0333       |
| Hamilton     | 18216           |                                           | 5551 8222       |
| Terang       | 20229           |                                           | 5592 0222       |
| Timboon      | 86060           |                                           | 5558 6000       |
| Camperdown   | 5593 7312       |                                           | 5593 7300       |
| Colac        | 25169           | Delia Melville – 25172                    | 5232 5100       |

Contingency Plan Procedure
Restore from backup and restore journals

!!! danger 
    WARNING NOT FULLY TESTED! There may be additional responses required!
