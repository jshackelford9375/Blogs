July 22, 2019 

 

How to use modules and package providers during OSD with SCCM 

 

For the past few days I've been implementing SnipeIT into our environment to start actually tacking assets as opposed to the archaic spreadsheet we had going. I wanted to add any new machine that was imaged into the database right away without any input from me. That way all I had to do was assign a user to the device in the asset management portal for SnipeIT and I was good to go because it had all the required info already. However this proved to be quite the headache for me. Special thanks to Christopher Kibble and Colin Wilkins for all their support and helping me learn some PS along the way (I'm still a noob). 

 

It's not as simple as it sounds 

 

I've used many powershell scripts during OSD In the past and thanks to the recent changes where you can just paste a script into a task sequence it's become even easier! With this in mind I created a script and tested it on my dev machines and a few others and everything looked great. They ran in the SYSTEM context fine, as an admin fine, all looked dandy. I put it into my OSD and it returned successful but never actually showed up into my asset management system. After trying everything under the sun including placement of the script during OSD, turning it into an application, making it a package, etc etc I just could not get it to work during OSD. I was about ready to give in but was thinking maybe it's an issue with installing the modules and package provider (It was). 

 

How to get those lovely modules and packages during OSD 

 

In my use case I already had the module and package provider I needed installed to my machine.  


You can find modules at $env:ProgramFiles\WindowsPowerShell\Modules 

You can find PackageProviders at  $env:ProgramFiles\PackageManagement\ProviderAssemblies 

I copied the WindowsPowerShell folder and the PacakageManegement folders to a new folder on my SCCM server 

 

In my case my folder structure for the entire package looks like so: 

![Imgur2](https://imgur.com/7ejs9Ro.jpg)
 

Once we have these two folders in a folder on our SCCM we can create a package.  

Now please forgive me for my terrible PS skills I'm working on improving them! 

Create a script in the same folder you put PackageManegemet and WindowsPowerShell that looks like the following 

 

$executingScriptDirectory = Split-Path -Path $MyInvocation.MyCommand.Definition -Parent 

copy-item $executingScriptDirectory\PackageManagement $env:ProgramFiles -force -Recurse 

copy-item $executingScriptDirectory\WindowsPowershell $env:ProgramFiles -force -Recurse 

 

In my case I called this script MoveItems.ps1 

 

In your OSD task Sequence create a run powershell script step and reference the package you made like so 

![Imgur3](https://imgur.com/oOCbxWD.jpg)

 

Next on your original script that's failing to run you'll want to IMPORT those modules or packageproviders as opposed to installing them since you now have the content copied to that local machine. In my case the code looks like so: 

 

Import-Module -Name $env:ProgramFiles\WindowsPowerShell\Modules\SnipeitPS -Verbose 

Import-PackageProvider -Name "Nuget" -Verbose 

 

Add another Run powershell script to your task sequence and reference your script with the newly added Import commands and you should be good to go! Don't forget to update your distribution point for the package you created!  
