Title: Making Fortran DLLs to interface with VBA in Excel
Published: 24/05/2012
Tags: [Migrated, Fortran, VBA] 
---

Note to self, Remember:

- When working with Intel visual fortran, always perform a static link (/MT) in the command line options so that you don't dependent on a DLL being present on the target machine.
- DLLs need Kernel32.lib to be added to the dependencies
- ByVal in VBA doesn't always mean ByVal, when calling a DLL it means "Copy then give a Ref", so on the Fortran side, get the address by reference.

On the VB side, the string gets declared as "ByVal stringname as string". You also need to add the string length to the end of the argument list as "ByVal stringlen as Integer". I also changed the numeric args to ByRef, especially as one of them is written. For example:

```VBA
Module Module1
    Declare Sub TestSSdll _
        Lib "C:\Documents and Settings\ . . . \TestSSdll.dll" _
        (ByRef Phi0 As Double, ByRef RLambda0 As Double, _
        ByRef NWinTimStps As Long, ByVal gridfile As String, _
        ByVal len_gridfile As Integer)
End Module
```


You will need to pass the length of the string explicitly using this arrangement.

On the Fortran side, the routine needs to have the STDCALL and REFERENCE attributes, the string argument should not have the REFERENCE attribute, assuming you want the length passed. For example:
```Fortran
subroutine TestSSdll (phi0, rlambda0, lgridfile, nWinTimStps, &
    gridfile)
implicit none

!DEC$ ATTRIBUTES DLLEXPORT, STDCALL, REFERENCE :: TestSSdll
!DEC$ ATTRIBUTES ALIAS: "TestSSdll" :: TestSSdll

    real(8), intent(in) :: phi0, rlambda0
    integer(8) :: nWinTimStps
    character(*), intent(in) :: gridfile
```

Note that the string length is not explicitly specified in the Fortran, as the compiler will look for it, passed by value.