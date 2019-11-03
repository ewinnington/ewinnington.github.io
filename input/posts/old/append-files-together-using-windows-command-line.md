Title: Append files together using windows command line
Published: 26/06/2015
Tags: [Migrated, CommandLine, FileProcessing]
---


In the command prompt, an easy way to append files together is to use the "type" command and pipe it to an output file.

```type Results\_\* > Results\_combined.txt```

To call this from a piece of C# code, I have been using the command line with the /c parameter, which makes it execute the attached command.

```cmd /c type Results\_\* > Results\_combined.txt```

C# Example usage:
```CSharp
using (Process RunProc = new Process()) {
    string s = "cmd.exe";  
    string p = "/c type Results\_\* > Results\_combined.txt";
    RunProc.StartInfo = new ProcessStartInfo(s, p);
    RunProc.Start();    
    RunProc.WaitForExit();
}
```
