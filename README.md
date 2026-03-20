
# Global Locks For Low-Level Hardware Access

## SMBus (System Management Bus)

Argus Monitor SMBus access is needed for memory temperatures, mainboard RGB and memory modules RGB.

Argus Monitor disables its SMBus access if it detects some other application which is known to access the SMBus without using the proper locking mechanism for SMBus shared access.

Unsynchronized SMBus access is dangerous and might cause weird issues like system shutdowns, BSOD and in rare cases may brick the RAM SPD chip as worst case.

Tools such as Aida64, HWiNFO, SIV, Argus Monitor all share a specific [lock (or mutex)](https://en.wikipedia.org/wiki/Lock_(computer_science)) to ensure secure access to the SMBus, even when multiple programs are running simultaneously.

This mutex is named "Global\Access_SMBUS.HTP.Method".

Unfortunately, some vendors like Asus or Corsair refuse to use this mutex and behave like they are the only one to access the SMBus.

Argus preemptively avoids potential issues for its users by automatically disabling its own SMBus access in such cases.
If you need SMBus access in Argus Monitor (for memory temperatures and RGB) we recommend to stop using tools from Asus and Corsair.


## Known mutex usage of various vendors of low-level hardware access software

| Vendor                                            | SMBus Mutex           | ISABus Mutex          | Remarks |
|---|---|---|---|
| [Aida64](https://www.aida64.com)                  | :white_check_mark:    | :white_check_mark:    | - |
| [Argus Monitor](https://www.argusmonitor.com)     | :white_check_mark:    | :white_check_mark:    | - |
| [HWiNFO](https://www.hwinfo.com)                  | :white_check_mark:    | :white_check_mark:    | - |
| [SIV](http://rh-software.com)                     | :white_check_mark:    | :white_check_mark:    | - |
| [Libre Hardware Monitor](https://github.com/LibreHardwaRemonitor/LibreHardwareMonitor) | :grey_question: | :grey_question: | - |
| [OpenRGB](https://openrgb.org)                    | :grey_question:       | :grey_question:       | - |
| [SignalRGB](https://signalrgb.com)                | :grey_question:       | :grey_question:       | - |
| Corsair iCUE                                      | :x:                   | :x:                   | - |
| Asus: AI Suite, Armoury Crate etc.                | :x:                   | :x:                   | - |

<br>

| Mutex         | Global Name                           | Used for |
|---|---|---|
| SMBus Mutex   | Global\Access_SMBUS.HTP.Method        | memory temperatures, mainboard RGB, memory modules RGB, power chips connected to SMBus |
| ISABus Mutex  | Global\Access_ISABUS.HTP.Method       | SuperIO chips (mainboard temperatures, mainboard fans) |


## Links and Quotations

Martin, author of HWiNFO:

"Well, yes there's a problem with such software as many vendors refuse to play fair and implement synchronization with other applications."
https://www.hwinfo.com/forum/threads/hwinfo64-causes-ram-lighting-issues-with-patriot-viper-rgb-3600mhz.6925/

"Then that's a different conflict on SMBus. Unfortunately Corsair is known for not playing nice with anything else and no will to fix that."
"No way to fix this without Corsair's cooperation, but there's doesn't seem to be any interest from their side."
https://www.hwinfo.com/forum/threads/7-42-update-causes-icue-fans-to-flash-off-on-every-2-4-seconds.8776/page-5


Ray, author of SIV (System Information Viewer):

"When there is a hardware access contention/race issue this will only happen from time-to-time. The fundamental problem is that some programs fail to use global locks and assume they are the only program accessing the hardware."
https://forum.corsair.com/forums/topic/107784-corsair-link-displaying-00-for-devices/page/2/

"If you have ASUS AI Suite active you should not use any other sensor monitoring programs active as it is poorly engineered an fails to use the Global\Access_ISABUS.HTP.Method + Global\Access_SMBUS.HTP.Method + Global\Access_EC + etc. named mutexes to interlock access to the sensors. If you were doing this it could be the root cause of the system reboots.
That said I agree that Corsair Link is not of an acceptable quality."
https://forum.corsair.com/forums/topic/126100-corsair-link-the-longest-running-joke-in-pc-history/


## License

**Creative Commons BY-NC-SA**<br>
Give Credit, NonCommercial, ShareAlike

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
