# DynamicApplicationChooser

In the evironment that I manage, we have a distribution point at the Dell Factory so that we can deploy customized task sequences and application builds to brand new machines.

To facilitate this, we have built a custom Provisioning Portal, which allows us to provision the device with hostname, OU to put it in in AD, Task Sequence to run and Application build to use for that task sequence. This gives us the flexibility to image a machine at the factory with any software load we want, all dynamic.

To accoomplish this, we designed and stood up a portal to stage those devices. This is not tied to MEMCM so we don't have to import devices into MEMCM for the process to work flawlessly. When the devices are ordered through our supplier, once filled at the factory, we get a list of serial numbers per order, which allows us to provision the devices. The devices are then sent to Dell's Config Center, where it gets our build. At the Config Center, we have an ASA, Server (DP,MP) and switch. 

To leverage the provisioning portal, I put together a few things to make it seamless. One, I built API endpoints using Universal Dashboard Community that retrieve information from the Provisioning Portal as well as perform other functions. Along side of the APIs, I wrote a custom powershell startup script that replaces out the default boot image wizard. That startup script performs the bulk of the work, including all automation if the device has already been staged in the provisioning portal.

If it has been provisioned, then it is all automated. The only thing a tech has to do at that point is PXE boot and then accept the PXE, so on Dell machines, F12, choose network PXE, and then enter to kick off the PXE boot. Once the boot image starts loading, then all pre-provisioned devices run automatically, no other work needs to be done until it is finished imaging.

If the device is not found in the provisioning portal, then it kicks off a manaul imaging process where it will as for credentials so that it can verify that you have rights to deploy images, and then it will proceed to display a custom XAML form for entering needed information prior to starting the task sequence. Once it starts the task sequence, we have a process where our techs can dynamically choose applications that they would want to install on the device. Previously, we used UI++ to handle that process, however, in the world of automation, we wanted to find a way to minimize the amount of work that we would need to do and maintain. With UI++, we had to modify the XML every time we updated an application or added an application and this has been a stumbling block for maintaining it. 

This project is the replacement for UI++ for our application deployments for those manual builds. It leverages our Provisioning Portal to select the base apps based on the profile you select, and then allows you to add or remove applications from that list. Albeit, not super cleanly, I have added the ability to leverage Administrative categories in MEMCM to group the Applications to make it easier to find. In my environment, I have over 650 applications to manage and in this script, it displays ALL of them, allowing much more flexibility for the Technician to build a computer the way they need to, first time out.  

This is customized with our branding, but it gives you an idea as to what and how it can be done.

I updated this while at MMS MIAMI and now it is working well... Please change the following lines prior to leveraging:
Line 5: Change to the base64 encoded value of the logo that you would want to use
Line 12: Change to the hex value of your background color
Line 13: Change to the hex value of your foreground color
Line 81: Change from $True to $False
Line 82: Change to point to the server that you use for calling your APIs

Along those lines, you will need to add an API to your system: /AllCategories

The syntax for the endpoint will be the following:

$Password = ConvertTo-SecureString “SQL PASSWORD” -AsPlainText -Force
    $Credential = New-Object System.Management.Automation.PSCredential (“SQL USERNAME”, $Password)
Invoke-Sqlcmd -ServerInstance <SCCM SITE SERVER> -Database CM_<SITECODE> -Credential $Credential -Query "SELECT App.DisplayName,stuff((select '; '+ LC.CategoryInstanceName	from v_LocalizedCategories LC, vAdminCategoryMemberships ACM where ACM.CategoryInstanceID = LC.CategoryInstanceID AND App.CI_UniqueID = ACM.ObjectKey for xml path(''), type).value('.', 'varchar(max)'), 1, 1, '') As 'Categories' FROM fn_ListLatestApplicationCIs(1033) App ORDER BY App.DisplayName" | ConvertTo-Json
  
You may have to modify the connection as you may not need to have the authentication.
