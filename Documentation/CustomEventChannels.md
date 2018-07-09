# Custom Event Channels

Custom event channels are a method of logically splitting logs into different sections of the event log and dividing resources. Within any large scale Windows Event Forwarding Deployment, custom event channels will become a necessity. 

## Prerequisites

To get started in createding custom event channels, you will need to download the [Windows SDK for your OS](https://developer.microsoft.com/en-us/windows/downloads/windows-8-1-sdk) in addition to the [template manifest file](CustomEventChannels.man).

## Creation of Custom Manifest

1. Launch `ecmangen.exe`, which can generally be found within `c:\Program Files (x86)\Windows Kits\8.1\bin\x64\ecmangen.exe`.
2. Open the template manifest.
3. Change the manifest to include Names and Symbols to your liking. 
    * Keep in mind that you may only have 8 channels per provider.
4. Save the .man file somewhere on your hard drive.
    * `C:\ECMan` is an easy location to find later.

## Compile the Manifest into an Event Channel DLL

1. Compile the DLL with the steps below:

```
cd\ECMan
"C:\Program Files (x86)\Windows Kits\8.1\bin\x64\mc.exe" C:\ECMan\CustomEventChannels.man
"C:\Program Files (x86)\Windows Kits\8.1\bin\x64\mc.exe" -css CustomEventChannels.DummyEvent C:\ECMan\CustomEventChannels.man
"C:\Program Files (x86)\Windows Kits\8.1\bin\x64\rc.exe" C:\ECMan\CustomEventChannels.rc
"C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe" /win32res:C:\ECMan\CustomEventChannels.res /unsafe /target:library /out:C:\ECMan\CustomEventChannels.dll C:\ECMan\CustomEventChannels.cs
```

2. Next, copy the .man and .dll files and place them into the `c:\Windows\System32` directory on the Event Collector.

## Load the Custom Event Channel DLL

Load the custom event channels by using the following command in an elevated command prompt:

`wevtutil im c:\Windows\system32\CustomEventChannels.man`

If the command is successful, you should expect no output from the command prompt. 

## Unload the Custom Event Channel DLL

If you wish to update your custom event channel DLL, you'll have to unload the DLL.

To do this, stop the Windows Event collector service with `net stop Wecsvc`, then unload the current event channel file with `wevutil um c:\windows\system32\CustomEventChannels.man`.

Remember to set up new size limits and custom paths for your new log channels.

## Relocating Event logs En Masse

Use the command `wevtutil sl /lfn:[new-file-path] /ms:[new-max-size]` to both resize and relocate a log file. 

**Note: ms should be set in bytes**

You can use this in a loop with:

```
$channels = wevtutil el | select-string -pattern "WEF"
foreach ($sub in $channels) {
    wevtutil sl $sub /ms:90000000000 /lfn:"D:\EventLogs\$sub.evtx"
}

**_Resources_**
1. [Creating Custom Windows Event Forwarding Logs](https://blogs.technet.microsoft.com/russellt/2016/05/18/creating-custom-windows-event-forwarding-logs/)