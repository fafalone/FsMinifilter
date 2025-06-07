# FsMinifilter

**FsMinifilter - twinBASIC Minifilter Proof of Concept**\
by Jon Johnson, based on:\
https://github.com/apriorit/Protection-Against-Unauthorized-Launch-Minifilter-Driver

![image](https://github.com/user-attachments/assets/60eb4c77-6750-47d2-a1a0-43608e92f437)
*The file was blocked by a kernel mode minifilter driver.*

>[!IMPORTANT]
>Update to Beta 796 or newer to use this project. Betas 786-795 cannot compile drivers.
                
This is a introduction to writing minifilter drivers in twinBASIC. While hardware drivers and some other types remain impractical, minifilter drivers are quite reasonable to write in tB, and give you powerful kernel mode capabilities for antimalware, system monitoring, access control, and other tasks. This project shows the framework for something actually  useful, as opposed to my initial proof of concept that you could write a driver at all. If you haven't seen that: https://github.com/fafalone/HelloWorldDriver

This first part of my planned series deals with the antimalware aspect. It demonstrates how to block a file from running or being accessed at all based on its name. As a basic proof of concept, there's just 2 names and they're hard coded, and the driver isn't set to start with the system or have protection from unloading. There's also no communication with user mode. These will be addressed in the next version.

There's some additional steps to making a driver in tB, and two additional ones compared to when my HelloWorldDriver was released.

1) Create a new Standard EXE project, then add a .twin Module and delete the form.
2) Remove the reference for OLE Automation, add a reference for the tbKMode package or add
    the files from it to your sources.
3) Change the following settings:\
       - Startup Object: Unchecked.\
       - Project: Native Subsystem: Yes\
       - Project: Override Entry Point: Enabled, set to DriverEntry\
       - Project: Runtime Binding of DLL Declares: No.\
       - Optimizer: Constant Function Folding: Yes\
       - Project: Large Address Aware (LAA): Yes\
       - Project: Strip PE File Relocation Symbols: No.\
       - Project: Enable Address Space Layout Randomization: Yes.
        
**How it works**\
The setup is actually pretty simple here. We register a filter for IRP_MJ_CREATE, the operation sent when a file is created or opened, and in that filter we check if the name matches, and if it does, change the IO status to STATUS_ACCESS_DENIED.

**Using this project**
- Build the driver, always 64bit for 64bit Windows.
- Copy FsMinifilter.sys and FsMinifilter.inf to the same destination folder.
- Right click FsMinifilter.inf and choose 'Install'. The required registry entries will then be made and the driver copied to System32/drivers/
- This driver is not signed at all, so beyond this point Windows will need to have been booted through the Advanced Boot Menu with the 'Disable driver signature enforcement' option. Future updates will demonstrate the use of test signing.
- If no error has occured, you can start the driver.
- Open a command prompt as Administrator and enter `sc start FsMinifilter`. 
- You can now try, hopefully unsuccessfully, to access files named evilfile.txt or run a program named evilprogram.exe. (The .txt file is blocked from being opened at all; the .exe can be opened for read but not executed.)
- You can view messages about that and other status info with DebugView:\
https://learn.microsoft.com/en-us/sysinternals/downloads/debugview

- To stop the driver, use `sc stop FsMinifilter` in the admin prompt.

- To uninstall the driver, `pnputil /delete-driver "FsMinifilter.inf" /uninstall /force`

**Known issues**\
There's currently several bugs in twinBASIC impacting this project: 
- There's an issue with StrPtr to constant strings, so this project continues to use byte arrays for initializing UNICODE_STRINGs.
- There's an issue with VarPtr to 'top level' arrays, so they're currently placed inside a UDT. See the FLT_OPERATION_REGISTRATION declares below.
- Short circuiting operators AndAlso/OrElse currently generated a background call to the API VariantChangeTypeEx, so cannot be used in kernel mode.
    
These are minor issues compared to the amazing ability to make working drivers 😄
