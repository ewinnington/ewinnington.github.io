Title: Adding windows printers with a VBScript
Published: 22/04/2012
Tags: [Migrated, WinXP, VBScript, CommandLine]
---

I had to add four network printers to a set of thirty laptops, so to make that faster, I used this small VBScript.

```
On Error Resume Next 
'SETS 'LOAD DRIVER' PRIVILEGE. 
    Set objWMIService = GetObject("Winmgmts:") 
    objWMIService.Security\_.Privileges.AddAsString "SeLoadDriverPrivilege", True
'SETS PRINTER PORT. 
    Set objNewPort = objWMIService.Get \_
      ("Win32\_TCPIPPrinterPort").SpawnInstance\_
    objNewPort.Name = "IP\_192.168.1.5" 
    objNewPort.Protocol = 1 
    objNewPort.HostAddress = "192.168.1.5" 
    objNewPort.PortNumber = "9100" 
    objNewPort.SNMPEnabled = False 
    objNewPort.Put\_
'SETS PRINTER TO PORT. 
    Set objPrinter = objWMIService.Get \_ 
        ("Win32\_Printer").SpawnInstance\_ 
    objPrinter.DriverName = "HP LaserJet 2100" 
    objPrinter.PortName   = "IP\_192.168.1.5" 
    objPrinter.DeviceID   = "NB1" 
    'objPrinter.Location = "Front Office" 
    objPrinter.Network = True 
    objPrinter.Shared = False 
    objPrinter.Put\_
'SETS PRINTER AS DEFAULT. 
    Set colInstalledPrinters =  objWMIService.ExecQuery \_ 
        ("Select \* from Win32\_Printer Where Name = 'NB1'") 
    For Each objPrinter in colInstalledPrinters 
        objPrinter.SetDefaultPrinter() 
    next
```

I adapted scripts found on [gallery.technet.microsoft.com](http://gallery.technet.microsoft.com) (for example [http://gallery.technet.microsoft.com/scriptcenter/710bb2ad-9a8d-42cb-b142-cda2c1452548](http://gallery.technet.microsoft.com/scriptcenter/710bb2ad-9a8d-42cb-b142-cda2c1452548)).

I added four different IP addresses, one for each printer, port was 9100, standard for hp printing, named the script InstallAll.vbs and then applied it on each computer. A normal double script on the script should launch it.

The one problem I found was that having installed all the printers under the administrator account, the default printer was not set for the user accounts, even if the printers were present.
