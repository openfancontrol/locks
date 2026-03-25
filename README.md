
## Global Locks For Low-Level Hardware Access


### SMBus (System Management Bus)

Argus Monitor SMBus access is needed for memory temperatures, mainboard RGB and memory modules RGB.

Argus Monitor disables its SMBus access if it detects some other application which is known to access the SMBus without using the proper locking mechanism for SMBus shared access.

Unsynchronized SMBus access is dangerous and might cause weird issues like system shutdowns, BSOD and in rare cases may corrupt the SPD chips on memory modules.

Tools such as Aida64, HWiNFO, SIV, Argus Monitor all share a specific [lock (or mutex)](https://en.wikipedia.org/wiki/Lock_(computer_science)) to ensure **secure access** to the SMBus, even when multiple programs are running simultaneously.

This mutex is named `Global\Access_SMBUS.HTP.Method`.

Unfortunately, some vendors like Asus or Corsair refuse to use this mutex and behave like they are the only one to access the SMBus.

Argus Monitor preemptively avoids potential issues for its users by automatically disabling its own SMBus access in such cases.
If you need SMBus access in Argus Monitor (for memory temperatures and RGB) we recommend to stop using tools from Asus and Corsair.


### SuperIO Chips

For accessing the SuperIO chip on the mainboard (which delivers mainboard temperatures and mainboard fan data), a similar mutex is used, this one is named `Global\Access_ISABUS.HTP.Method`.

Like with the SMBus mutex, for **safe handling**, all programs accessing the SuperIO chip need to use this mutex when multiple programs are running simultaneously.


### Known mutex usage of various vendors of low-level hardware access software (in alphabetical order)

| Vendor                                                        | SMBus Mutex           | ISABus Mutex          | Remarks |
|---|---|---|---|
| [Aida64](https://www.aida64.com)                              | :white_check_mark:    | :white_check_mark:    | - |
| [Argus Monitor](https://www.argusmonitor.com)                 | :white_check_mark:    | :white_check_mark:    | - |
| [Asus](https://www.asus.com) AI Suite, Armoury Crate, etc.    | :x:                   | :x:                   | - |
| [Corsair](https://www.corsair.com) Link, iCUE                 | :x:                   | :x:                   | - |
| [CPU-Z](https://www.cpuid.com/softwares/cpu-z.html)           | :white_check_mark:    | :white_check_mark:    | - |
| [HWiNFO](https://www.hwinfo.com)                              | :white_check_mark:    | :white_check_mark:    | - |
| [HWMonitor](https://www.cpuid.com/softwares/hwmonitor.html)   | :white_check_mark:    | :white_check_mark:    | - |
| [Libre Hardware Monitor](https://github.com/LibreHardwareMonitor/LibreHardwareMonitor) | :x: | :white_check_mark: | Unknown why SMBus mutex is not used  |
| [OpenRGB](https://openrgb.org)                                | :question:            | :question:            | - |
| [SignalRGB](https://signalrgb.com)                            | :question:            | :question:            | Unknown |
| [SIV](http://rh-software.com) (System Information Viewer)     | :white_check_mark:    | :white_check_mark:    | - |
| [SpeedFan](https://www.almico.com/speedfan.php)               | :white_check_mark:    | :white_check_mark:    | - |

<br>

| Mutex         | Global Name                           | Used for |
|---|---|---|
| SMBus Mutex   | Global\Access_SMBUS.HTP.Method        | memory temperatures, mainboard RGB, memory modules RGB, power chips connected to SMBus |
| ISABus Mutex  | Global\Access_ISABUS.HTP.Method       | SuperIO chips (mainboard temperatures, mainboard fans) |


### Links and Quotations

##### Fiery, author of Aida64:

> "The folks at Corsair are telling you the truth.  What they wouldn't tell you is that the ball is rolling in their court.  In their previous software (called Corsair Link) they used to implement the necessary standardized synchronization mutexes that allowed their software to work with all major 3rd party monitoring software out there, like AIDA64, CPU-Z, HWiNFO, HWMonitor, SIV, etc.  What happened is they discontinued using those mutexes and let the collisions happen, and continue to let them happen despite the fact that multiple developers (including us of course) have been pestering them for over a year now.  I don't think it's wise to stick to their position of one and only one software to rule all Corsair hardware (ie. Corsair iCUE), but it's their position and they seem to want to stick to it no matter what."

https://forums.aida64.com/topic/5097-aida64-and-icue-commander-pro/
<br>
<br>

##### Martin, author of HWiNFO:

> "Well, yes there's a problem with such software as many vendors refuse to play fair and implement synchronization with other applications."

https://www.hwinfo.com/forum/threads/hwinfo64-causes-ram-lighting-issues-with-patriot-viper-rgb-3600mhz.6925/
<br>

> "Then that's a different conflict on SMBus. Unfortunately Corsair is known for not playing nice with anything else and no will to fix that."
> "No way to fix this without Corsair's cooperation, but there's doesn't seem to be any interest from their side."

https://www.hwinfo.com/forum/threads/7-42-update-causes-icue-fans-to-flash-off-on-every-2-4-seconds.8776/page-5
<br>
<br>

##### Ray, author of SIV:

> "When there is a hardware access contention/race issue this will only happen from time-to-time. The fundamental problem is that some programs fail to use global locks and assume they are the only program accessing the hardware."

https://forum.corsair.com/forums/topic/107784-corsair-link-displaying-00-for-devices/page/2/
<br>

> "If you have ASUS AI Suite active you should not use any other sensor monitoring programs active as it is poorly engineered an fails to use the Global\Access_ISABUS.HTP.Method + Global\Access_SMBUS.HTP.Method + Global\Access_EC + etc. named mutexes to interlock access to the sensors. If you were doing this it could be the root cause of the system reboots.
> That said I agree that Corsair Link is not of an acceptable quality."

https://forum.corsair.com/forums/topic/126100-corsair-link-the-longest-running-joke-in-pc-history/
<br>
<br>


### Sample Code

C Sample Code for creating the mutex with the proper access rights (by Ray, author of SIV)

```
HANDLE CreateWorldMutex(wchar_t const* const mutex_name)                      // Create/Open a Mutex with
{                                                                             // appropriate protection
    HANDLE                   mhl;                                             // Mutex Handle
    SID*                     sid {};                                          // Security ID
    SECURITY_ATTRIBUTES      sab;                                             // Security Attributes Block
    SECURITY_DESCRIPTOR      sdb;                                             // Security Descriptor Block
    ACL                      acl[1024];                                       // ACL Area
    SID_IDENTIFIER_AUTHORITY swa[1] = { SECURITY_WORLD_SID_AUTHORITY };       // World access

    InitializeSecurityDescriptor(&sdb, SECURITY_DESCRIPTOR_REVISION);         // setup Security Descriptor

    if (AllocateAndInitializeSid(swa,                                         // SID Identifier Authority
                                 1,                                           // Sub Authority count
                                 SECURITY_WORLD_RID,                          // Sub Authority 0
                                 0,                                           // Sub Authority 1
                                 0,                                           // Sub Authority 2
                                 0,                                           // Sub Authority 3
                                 0,                                           // Sub Authority 4
                                 0,                                           // Sub Authority 5
                                 0,                                           // Sub Authority 6
                                 0,                                           // Sub Authority 7
                                 (void**)&sid)
        &&                                                                    // returned SID
        (InitializeAcl(acl,                                                   // ACL setup OK and
                       sizeof(acl),                                           //
                       ACL_REVISION))
        &&                                                                    //
        (AddAccessAllowedAce(acl,                                             // ACE setup OK and
                             ACL_REVISION,                                    //
                             MUTANT_ALL_ACCESS,                               // Access Rights Mask
                             sid))) {
        SetSecurityDescriptorDacl(&sdb, TRUE, acl, FALSE);                    // yes, setup world access
    } else {
        SetSecurityDescriptorDacl(&sdb, TRUE, nullptr, FALSE);                // no, setup with default
    }

    sab.nLength              = sizeof(sdb);                                   // setup Security Attributes Block
    sab.bInheritHandle       = FALSE;                                         //
    sab.lpSecurityDescriptor = &sdb;                                          //

    if (((mhl = CreateMutex(&sab,                                             // Create/Open with Global\ Unprotected or
                            FALSE,                                            //
                            mutex_name))
         != nullptr)
        ||                                                                    //
        ((mhl = OpenMutex(READ_CONTROL | MUTANT_QUERY_STATE | SYNCHRONIZE,    // Open with Global\ Protected (probably Aquasuite)
                          FALSE,                                              //
                          mutex_name))
         != nullptr)) { }                                                     //

    if (sid) {                                                                // need to free the SID ?
        FreeSid(sid);                                                         // yes, free it
    }

    return mhl;                                                               // return the handle
}
```


### License

**Creative Commons BY-NC-SA**<br>
Give Credit, NonCommercial, ShareAlike

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
