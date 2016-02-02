### Building BIRT 4.5.0 on Eclipse 4.5


BIRT version 4.5.0 has been tested for Linux on z Systems (RHEL 6 and SLES 11) by the following instructions.

_**General Notes:**_ 	 
_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


## Downloading and Installing BIRT 4.5.0

1. Install vncserver on the Linux on z Systems to provide a GUI for using Eclipse.

   For RHEL 6
   ```source-shell
   sudo yum install tigervnc-server
   ```
   Select a free port and enable it for the incoming connection (using 5900 as an example)
   ```source-shell
   sudo iptables -I INPUT -m state --state NEW -p tcp --destination-port 5900 -j ACCEPT
   ```


   For SLES 11

   vncserver should come with the distro by default


2. Start vncserver
   ```source-shell
   vncserver
   ```

3. Use a local vnc viewer to remotely connect to the Linux on z Systems for GUI access.


4. Download Eclipse 4.5 for s390x on the Linux on z Systems from [[Eclipse download site link here|https://www.eclipse.org/downloads/download.php?file=/eclipse/downloads/drops4/R-4.5-201506032000/eclipse-SDK-4.5-linux-gtk-s390x.tar.gz]] and put it in the `/<source_root>/` folder.
   ```source-shell
   cd /<source_root>/
   tar xvf eclipse-SDK-4.5-linux-gtk-s390x.tar.gz
   cd eclipse
   ``` 

   For RHEL 6, Eclipse is ready to start.

   For SLES 11, starting Eclipse as is will probably encounter the following error: "JVM terminated. Exit code=160". The fix is described at [[http://www-01.ibm.com/support/docview.wss?uid=swg21598554]].  The workaround is to add "-Dorg.eclipse.swt.browser.XULRunnerPath=/usr/lib64/xulrunner-1.9.2.27/xulrunner" at the end of the eclipse.ini file.
   

5. Download all BIRT dependencies (Also available in [[http://download.eclipse.org/birt/downloads/]]), unzip them into `/<source_root>/eclipse/dropins` directory.  Create separate folder for each package, for example, unzip BIRT package into `/<source_root>/eclipse/dropins/birt` folder. 


    + [[DTP 1.12.0|https://www.eclipse.org/downloads/download.php?file=/datatools/downloads/1.12/dtp-sdk_1.12.0.zip]]
    + [[EMF + XSD 2.11|https://www.eclipse.org/downloads/download.php?file=/modeling/emf/emf/downloads/drops/2.11.0/R201506010402/emf-xsd-SDK-2.11.0.zip]]
    + [[GEF 3.10 | https://www.eclipse.org/downloads/download.php?file=/tools/gef/downloads/drops/3.10.1/R201508170204/GEF-ALL-3.10.1.zip]]
    + [[WTP 3.7.0|https://www.eclipse.org/downloads/download.php?file=/webtools/downloads/drops/R3.7.0/R-3.7.0-20150609111814/wtp-repo-R-3.7.0-20150609111814.zip]]
    + [[BIRT updatesite 4.5.0| https://www.eclipse.org/downloads/download.php?file=/birt/downloads/drops/R-R1-4_5_0-201506092134/birt-updatesite-4.5.0-20150609.zip]]


6. Start Eclipse on the Linux on z server, click on "Help" from the GUI, then "Install new software"
   

   Choose to Work with "Mars - http://download.eclipse.org/releases/mars"
   

   Choose "Business Intelligence, Reporting and Charting" to proceed.


   The first installation will probably fail with the following error:
   ```source-shell
   An error occurred during the org.eclipse.equinox.internal.p2.engine.phases.CheckTrust phase.
   session context was:(profile=SDKProfile, phase=org.eclipse.equinox.internal.p2.engine.phases.CheckTrust, operand=, action=). 
   Error reading signed content. error in opening zip file
   ```


   This is a bug observed in Eclipse on Linux on z. A defect is reported at [[Eclipse Bugzilla|https://bugs.eclipse.org/bugs/show_bug.cgi?id=483927]] for the issue. 


   Just close eclipse at this moment.

7. As a workaround, remove the BIRT folder you just unzipped in `/<source_root>/eclipse/dropins` folder
   ```source-shell
   cd /<source_root>/eclipse/dropins
   rm -r birt
   ```

8. Restart eclipse and repeat step 6. The installation process will succeed this time.


##### References:
http://www.eclipse.org/birt/