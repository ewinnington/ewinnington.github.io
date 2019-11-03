Title: Getting autoincremented version numbers in Visual Studio
Published: 25/08/2014
Tags: [Migrated, VisualStudio] 
---

On C# projects, in the "AssemblyInfo.cs" file, you can set the Assembly Version to contain \*, which sets the version automatically to the build date and time.

If you comment out the AssemblyFileVersion, then you can see it in the Windows explorer. If you don't then Windows Explorer will always report the AssemblyFileVersion which does not support the \* notation.
```
\[assembly: AssemblyVersion("1.0.\*")\]
```