# kodnet


> This project only contains the VisualFoxPro related code. If you want the c# code for interop library go to [https://github.com/voxsoftware/jxshell.dotnet4](https://github.com/voxsoftware/jxshell.dotnet4)

### Go to [Spanish documentation](README.esLA.md)

## Library and helpers for access to all .NET objects/classes/controls from VisualFoxPro 9
Yes made .NET interop easy for VisualFoxPro 9

**kodnet** adds a small library that allows you to use classes/objects and .NET controls within your *VisualFoxPro 9* project. You can call any method, field or property of any .NET class without having to create a COM component. Access the .NET framework components, the large number of free .NET libraries, or your own components from VisualFoxPro 9 without registering or installing anything.

**kodnet** even allows access to enums, structs, classes and generic methods, which is something you can't achieve by generating a .COM component yourself. 
**kodnet** implicitly provides type conversion whenever possible and also provides *wrappers* that allow you to use native .NET types but not native to VisualFoxPro 9. This is very useful for example with methods that accept *byte, float, long, Decimal* among others.



# Want to be a Sponsor?

If you need any specific requirement or want to contribute to this project continue improving you can contact us at contacto@kodhe.com or use the donation link. If you wish to become a sponsor and have your logo/company appear in this README.md, become a sponsor that donates monthly and contact us.

* Donate to paypal [![](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=XTUTKMVWCVQCJ&source=url)




### NET Framework version support 

**kodnet** runs on .NET Framework 4 or higher and any version of Windows that supports this framework and VisualFoxPro 9


# Getting started 

Load the kodnet environment

```foxpro
* Load kodnet library
do fullpath("kodnet.prg")
```

> If you get a message that says **Unable to load CLR** it's probably because Windows blocks files downloaded from the Internet. To do this right-click on the .DLL libraries distributed with *kodnet* and click unlock. 

```foxpro 
* you can now access to kodnet using _screen.kodnet


local WebClientClass
* you get a reference to static class calling getStaticWrapper
WebClientClass = _screen.kodnet.getStaticWrapper("System.Net.WebClient")
m.WebClientClass.DownloadFile("https://codeload.github.com/voxsoftware/kodnet/zip/master", "kodnet.zip")

* load an assembly by file
local customClass, customObject
_screen.kodnet.loadAssemblyFile("customdotnet.dll")
customClass= _screen.kodnet.getStaticWrapper("CustomClass")

* create an instance of type 
customObject= m.customClass.construct()


* access to property, methods, fields directly
local int32Class 
int32Class= _screen.kodnet.getStaticWrapper("System.Int32")
?int32Class.MaxValue
?int32Class.MinValue 
?customObject.customMethod()


```

## Getting started summary


* Access any .NET component even if it is not marked for *interop* [ComVisible].
* You do not need to register or install the component
* Call any method, property directly like any other object *FoxPro*
* Calling the constructor of a class with parameters is possible
* Call any method overload. **kodnet** select the best, according to your parameters
* Support for any non-native .NET type
* Access to static members, including Structs, Enums, Generics, etc.
* Access *.NET arrays* easily using *Get* and *Set* methods
* You can pass an object *FoxPro* and read it from a .NET method using *dynamic*.
* Multithread support
* Include visual .NET controls within your *VisualFoxPro* forms and access your members like any other .NET class.
* Support for adding/deleting .NET event handlers (*delegates*)
* Great performance in method calls, properties, because internally it doesn't use *Reflection* but uses *CallSite* (the methodology it uses internally *dynamic* in C#) 




# How it works

Download the project to see the examples and learn how to use it. It mainly consists of the following

* ClrHost.dll - Win32 dll to load the .NET runtime
* DLLs used for interop - The lib folder
* kodnet.prg - FoxPro prg, frontend to Proxy
* dotnet4.vcx - FoxPro Visual Class, for adding .NET controls to forms

> In a distributed application, it is recommended that you distribute the kodnet components within a folder called *kodnet* in the same location as your executable.


Take a look at an example of code that uses different features of .NET components


```foxpro 
do fullpath("kodnet.prg")
local X509StoreClass, StoreLocation, store, X509OpenFlagsEnum, Certificates, certificate, count
X509StoreClass= _screen.kodnet.getStaticWrapper("System.Security.Cryptography.X509Certificates.X509Store")

* access to enum
StoreLocation= _screen.kodnet.getStaticWrapper("System.Security.Cryptography.X509Certificates.StoreLocation")
store= m.X509StoreClass.construct(StoreLocation.LocalMachine)

* OR use this for certificates only the user
*store= m.X509StoreClass.construct(StoreLocation.CurrentUser)



X509OpenFlagsEnum=  _screen.kodnet.getStaticWrapper("System.Security.Cryptography.X509Certificates.OpenFlags")
m.store.Open(X509OpenFlagsEnum.ReadOnly)

* manage collections  
Certificates= store.Certificates
count= Certificates.Count 

?"Certificates found: " + str(m.count)
FOR i=1 TO m.count 
	* use item for access to collection items
	certificate= m.Certificates.item(m.i - 1)
    if (!ISNULL(m.certificate))
        ? m.certificate.FriendlyName
		? m.certificate.SerialNumber
		? m.certificate.GetName()
    endif 
endfor 
```

## Async and events 


Consider now a more advanced example that calls asynchronous methods and uses .NET events.

```foxpro


* KODNET (contacto@kodhe.com)
* Download File asynchronous example

do fullpath("kodnet.prg")

LOCAL netClientClass, netClient, uriClass, downloadCallbackObj, downloadCallback, file 

* select a file
file= GETFILE()
IF EMPTY(m.file)
	RETURN MESSAGEBOX("Please select a file",64,"")
ENDIF 

uriClass= _screen.kodnet.getStaticWrapper("System.Uri")
netClientClass= _screen.kodnet.getStaticWrapper("System.Net.WebClient")
m.netClient= m.netClientClass.construct()





downloadCallbackObj= CREATEOBJECT("downloadCallback")
* create a delegate that point to VisualFoxPro function
downloadCallback=_screen.kodnetManager.createeventhandler(m.downloadCallbackObj, "finished", "System.ComponentModel.AsyncCompletedEventHandler")
* add the event handler 
m.netClient.add_DownloadFileCompleted(m.downloadCallback)

* this method is a .NET async method, this call will complete without finish the download
m.netClient.DownloadFileAsync(uriClass.construct("http://storage-ns01.d0stream.com/.drive/Projects/ffmpeg.tar.gz"), m.file)

* this executes before finish for demostrate that method is called async
? "Download has started. Running in background"
return 


DEFINE CLASS downloadCallback  as Custom 
	FUNCTION finished(sender, args)
		* avoid memory leaks (this is only required for objects not forms)
		this._event.destroy()
		IF !ISNULL(args.Error)
			MESSAGEBOX("Failed download: " + args.Error.Message, 48, "")
		ELSE 
			? "download finished"
			MESSAGEBOX("Finished download.", 64, "")
		ENDIF 
	ENDFUNC 
ENDDEFINE 


```











