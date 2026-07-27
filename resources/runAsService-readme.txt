NodalConnect Windows Service - README
====================================

This service uses WinSW to run the Spring Boot NodalConnect API as a Windows Service.

Folder Layout
-------------
Example folder layout:

C:\NodalConnect\NodalConnectService\
    nodalConnect.exe
    nodalConnect.xml
    nodalConnect.jar
    jdk-17.0.17+10-jre\

Important:
- The WinSW executable and XML config should be in the same folder.
- The executable and XML should use the same base name.
  Example:
    nodalConnect.exe
    nodalConnect.xml

Current XML
-----------
The service uses an XML config similar to this:

<service>
  <id>nodalConnectService</id>
  <name>NodalConnect</name>
  <description>NodalConnect API Service</description>
  <env name="JAVA_HOME" value="%BASE%\jdk-17.0.17+10-jre" />
  <executable>%JAVA_HOME%\bin\java.exe</executable>
  <arguments>-jar "%BASE%\nodalConnect.jar" --spring.profiles.active=local</arguments>
  <workingdirectory>%BASE%</workingdirectory>
  <log mode="roll-by-size">
    <sizeThreshold>10240</sizeThreshold>
    <keepFiles>8</keepFiles>
  </log>
  <onfailure action="restart" delay="10 sec" />
  <stoptimeout>15 sec</stoptimeout>
</service>

FIRST RUN / INITIAL SETUP
-------------------------
1. Open Command Prompt as Administrator.

2. Change to the service folder:
   cd /d C:\NodalConnect\NodalConnectService

3. Confirm Java works from the bundled JRE:
   jdk-17.0.17+10-jre\bin\java.exe -version

4. Optional but recommended: test the JAR manually before installing the service:
   jdk-17.0.17+10-jre\bin\java.exe -jar nodalConnect.jar --spring.profiles.active=local

5. Install the Windows Service:
   nodalConnect.exe install

6. Start the service:
   nodalConnect.exe start

7. Check service status:
   nodalConnect.exe status

If the status shows:
   Started
then the service is running.

MANUAL SERVICE COMMANDS
-----------------------
Run these from:
   C:\NodalConnect\NodalConnectService

Start the service:
   nodalConnect.exe start

Stop the service:
   nodalConnect.exe stop

Restart the service:
   nodalConnect.exe restart

Check service status:
   nodalConnect.exe status

Uninstall the service:
   nodalConnect.exe uninstall

WHEN TO USE STOP / RESTART
--------------------------
Use STOP when:
- You need to shut the API down completely
- You are replacing the JAR or JRE
- You are editing the XML and want the service offline first

Use RESTART when:
- You changed the JAR
- You changed Spring properties
- You changed the WinSW XML and want the new settings to apply
- The API is running but needs to be recycled

AUTOMATIC START ON SERVER BOOT
------------------------------
After the service is installed, set it to start automatically with Windows.

Option 1 - In Services:
1. Press Win + R
2. Type:
   services.msc
3. Find:
   NodalConnect
4. Open Properties
5. Set Startup type to:
   Automatic

Option 2 - Command line:
   sc config nodalConnectService start= auto

Note:
- There must be a space after "start=" when using sc.

AUTOMATIC RESTARTS AFTER FAILURE
--------------------------------
There are two places that can help with automatic restart:

1. WinSW XML
   This is already configured with:
   <onfailure action="restart" delay="10 sec" />

2. Windows Service Recovery Options
   It is recommended to also configure Windows service recovery.

Recommended command:
   sc failure nodalConnectService reset= 86400 actions= restart/10000/restart/20000/restart/30000

What this does:
- 1st failure: restart after 10 seconds
- 2nd failure: restart after 20 seconds
- 3rd failure: restart after 30 seconds

You can also configure this in the Services UI:
1. Open:
   services.msc
2. Find:
   NodalConnect
3. Open Properties
4. Go to the Recovery tab
5. Set:
   First failure  = Restart the Service
   Second failure = Restart the Service
   Subsequent failures = Restart the Service

UPDATING THE SERVICE
--------------------
If you replace the JAR or update service config:

1. Stop the service:
   nodalConnect.exe stop

2. Replace the files as needed

3. Start the service again:
   nodalConnect.exe start

If you changed the XML only, you can also run:
   nodalConnect.exe refresh

Then restart:
   nodalConnect.exe restart

LOGS / TROUBLESHOOTING
----------------------
If the service does not start or stops unexpectedly:

1. Check WinSW log files in the service folder
2. Check Spring Boot application logs
3. Check Windows Event Viewer:
   eventvwr.msc

Useful checks:
- Confirm nodalConnect.jar exists
- Confirm jdk-17.0.17+10-jre exists
- Confirm the XML and EXE names match
- Confirm the service is running from the correct folder
- Confirm the application port is not already in use

COMMON COMMAND SUMMARY
----------------------
Install:
   nodalConnect.exe install

Start:
   nodalConnect.exe start

Stop:
   nodalConnect.exe stop

Restart:
   nodalConnect.exe restart

Status:
   nodalConnect.exe status

Refresh config:
   nodalConnect.exe refresh

Uninstall:
   nodalConnect.exe uninstall

NOTES
-----
- Use Administrator Command Prompt for install/uninstall and service configuration commands.
- If this is moved to a real server environment, consider changing:
    --spring.profiles.active=local
  to a server or production profile.
- Consider increasing the XML log size threshold later, since 10240 is only about 10 KB.
