# Code will be online after our BlackHat Asia talk (March 28)

# Evil Clippy
A cross-platform assistant for creating malicious MS Office documents. Can hide VBA macros, stomp VBA code (via P-Code) and confuse macro analysis tools. Runs on Linux, OSX and Windows.

## Current features
* Hide VBA macros from the GUI editor
* VBA stomping (P-code abuse)
* Fool analyst tools
* Serve VBA stomped templates via HTTP

If you have no idea what all of this is, check out the following resources first:
* [Our MS Office Magic Show presentation at Derbycon 2018](https://outflank.nl/blog/2018/10/28/recordings-of-our-derbycon-and-brucon-presentations/)
* [VBA stomping resources by the Walmart security team](https://vbastomp.com/)
* [Pcodedmp by Dr. Bontchev](https://github.com/bontchev/pcodedmp)

## How effective is this?
At the time of writing, this tool is capable of getting a default Cobalt Strike macro to bypass all major antivirus products and most maldoc analysis tools (by using VBA stomping in combination with random module names).

## Technology
Evil Clippy uses the [OpenMCDF library](https://github.com/ironfede/openmcdf/) to manipulate MS Office Compound File Binary Format (CFBF) files, and hereto abuses [MS-OVBA specifications](https://docs.microsoft.com/en-us/openspecs/office_file_formats/ms-ovba/) and features. It reuses code from [Kavod.VBA.Compression](https://github.com/rossknudsen/Kavod.Vba.Compression) to implement the compression algorithm that is used in dir and module streams (see MS-OVBA for relevant specifications).

Evil Clippy compiles perfectly fine with the Mono C# compiler and has been tested on Linux, OSX and Windows.

## Compilation

**OSX and Linux**
Make sure you have Mono installed. Then execute the following command from the command line:

`mcs /reference:OpenMcdf.dll /out:EvilClippy.exe *.cs`

Now run Evil Clippy from the command line:

`mono EvilClippy.exe -h`

**Windows**
Make sure you have Visual Studio installed. Then execute the following command from a Visual Studio developer command prompt:

`csc /reference:OpenMcdf.dll /out:EvilClippy.exe *.cs`

Now run Evil Clippy from the command line:

`EvilClippy.exe -h`

## Usage examples

**Print help**

`EvilClippy.exe -h`

**Hide macros from GUI**

Hide all macro modules (except the default "ThisDocument" module) from the VBA GUI editor. This is achieved by removing module lines from the project stream [MS-OVBA 2.3.1].

`EvilClippy.exe -g macrofile.doc`

**Stomp VBA (abuse P-code)**

Put fake VBA code from text file *fakecode.vba* in all modules, while leaving P-code intact. This abuses an undocumented feature of module streams [MS-OVBA 2.3.4.3]. Note that the VBA project version must match the host program in order for the P-code to be executed (see next example for version matching).

`EvilClippy.exe -s fakecode.vba macrofile.doc`

**Set target Office version for VBA stomping**

Same as the above, but now explicitly targeting Word 2016 on x86. This means that Word 2016 on x86 will execute the P-code, while other versions of Word wil execute the code from *fakecode.vba* instead. Achieved by setting the appropriate version bytes in the _VBA_PROJECT stream [MS-OVBA 2.3.4.1].

`EvilClippy.exe -s fakecode.vba -t 2016x86 macrofile.doc`

**Set random module names (fool analyst tools)**

Set random ASCII module names in the dir stream [MS-OVBA 2.3.4.2]. This abuses ambiguity in the MODULESTREAMNAME records [MS-OVBA 2.3.4.2.3.2.3] - most analyst tools use the ASCII module names specified here, while MS Office used the Unicode variant. By setting a random ASCII module name most P-code and VBA analysis tools crash, while the actual P-code and VBA still runs fine in Word and Excel.

`EvilClippy.exe -r macrofile.doc`

**Serve a VBA stomped template via HTTP**

Service *macrofile.dot* via HTTP port 8080 after performing VBA stomping. If this file is retrieved, it automatically matches the target's Office version (using its HTTP headers and then setting the _VBA_PROJECT bytes accordingly).

`EvilClippy.exe -s fakecode.vba -w 8080 macrofile.dot`

Note: The file you are serving must be a template (.dot instead of .doc). Also, fakecode.vba must have the proper attributes set for a macro from a template (contact me for details). You can set a template via a URL (.dot is not required!) from the developer toolbar in Word.

## Limitations

The current version only works with the CFBF-based Office 97-2003 format for Word (the famous .doc files). The newer XML-based format (.docm) is not yet supported. Neither is Excel (.xls and .xlsm). 

All of these lacking features might be added in future releases.

## Authors
Stan Hegt ([@StanHacked](https://twitter.com/StanHacked)) / [Outflank](https://www.outflank.nl)

Special thanks to Nick Landers ([@monoxgas](https://twitter.com/monoxgas)) for pointing me towards OpenMCDF.
