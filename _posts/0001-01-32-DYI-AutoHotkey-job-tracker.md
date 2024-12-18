---
layout: post
title: DYI job tracker with AutoHotkey macros
subtitle: "DYI job tracker with AutoHotkey macros"
date: "2024-11-23"
#published: false
---

## Introduction

<img src="../images/0001-01-32/woman-7659866_640.jpg" class="image fit" alt="Woman with working on flowers"/>

Disclaimer: This is only for people using Windows. The frustration of "messing up" in a job search 
is too real. Often it is probably not your fault. Only you have several you want to apply for in 
several job boards "Linked In", "Glassdoor" and "Indeed".
How does one keep track?? 

Excel? Well spreadsheets are something many organizations move a way from but for your 
own job search, why not?

Automation. Sometimes you want to hyperlink your tailored resume/cover letter for each job 
and not have applied for and navigate through less folders to simply do that. 
You could use a Visual Basic macro but what if you want to do this outside of Excel?

AutoHotkey is the answer. An open source scripting language made in C++. Very easy to implement.
As my Software Development teacher once said "if you are repeatedly doing things several times, 
there is probably a better way to do it."

## What are we learning other than building a job tracker?

In short we are learning:

- Automation.
- Debugging.
- Logging.

So let's say we are in a directory saying, "job tracker". And underneath that we have several folders for each company say "Google"
and "Amazon" and then under those we have several "job title" folders. What if we had one and were navigating back and forth to 
that folder to make changes. Wouldn't be good if we could make one less click.

So rather than navigating into Google and then it's only subfolder "Staff Software Engineer" what if it just opens it up automatically?
Well AutoHotkey can listen for that on startup. If you're interested, let's see how.

## Create the spreadsheet (optional)

You can skip this if you want to build your own. Else you can just download my template as provided. I am not going to waste your time showing you how to insert
data in cells. Rather you can do this.
```csv
Date Applied,Job Title,Company,Job URL,Application Status,Interview Date,Notes,Cover Letter Link,Resume Link,Job Description
2024-11-20,Software Engineer,Tech Innovators Inc.,https://www.techinnovators.com/jobs/123,Applied,2024-11-27,Researching company details.,https://link.to/coverletter1,https://link.to/resume1,Developing cutting-edge software solutions.
2024-11-18,Marketing Specialist,Creative Solutions Ltd.,https://www.creativesolutions.com/jobs/456,Interview Scheduled,2024-11-25,Prepare portfolio.,https://link.to/coverletter2,https://link.to/resume2,Creating and executing marketing campaigns.
2024-11-15,Data Analyst,Data Crunchers Corp.,https://www.datacrunchers.com/jobs/789,Rejected,,Follow-up for feedback.,https://link.to/coverletter3,https://link.to/resume3,Analyzing and interpreting complex data sets.
```

Unfortunately you would need to then SAVE AS xlsx format if you want to save any changes made in Excel. This is the path where you could store your docs for example. As you will notice it's in OneDrive.

After you have changed the Spreadsheet format to xlsx, you can then click on the whole row by selecting it's reference letter on the left left which should be number 1. Turn on the Autofilter option, (one of Excel's killer features) by going in the Data Tab and then filter, which should allow you to search for the data you need. 

```
C:\Users\Jordan Nash\OneDrive\Job Tracking Docs
```

Documents like our cover letter and resume could be hyperlinked from this folder layout under our "Job Tracking Docs".

```txt
Job Tracking Docs            
│
├─[Default]
|                            
├─Google                     
│ │                          
│ ├─Junior Software Developer
│ └─Senior Software Developer
│                            
└─Amazon                     
  │                          
  └─Junior Software Developer                         
```

You will notice we could also have a default folder at the top in square brackets if we are
applying to multiple jobs and don't have time to tailor each resume. I put it in square brackets 
so if we have to find it alphabetically, it is easier. 

You will notice I have created my own script to use any shortcuts globally throughout Windows. 
That's the beauty of AutoHotkey v2. 

You don't need to remember the shortcuts of different applications.
Just make your own. Especially useful for job hunting as I notice some companies block the browser's 
autofill feature on their websites in such a way that turning autofill off in the developer console doesn't alway work.

## Installation of AHK tools. 

You can add it to your path in your environment variables. Usually would add it to "C:\Program Files\AutoHotkey\v2\AutoHotkey64.exe".  

Ensure you also have Visual Studio code installed. Even it's depreciated extensions will make debugging easier. I have used [Mark Wiemer's](https://marketplace.visualstudio.com/items?itemName=mark-wiemer.vscode-autohotkey-plus-plus) extension so I can set breakpoints to know how my code executes upon compilation, where I screwed up and why.

```txt
#Requires AutoHotkey v2.0

::_jn::Jordan Nash
::_jnA:: my address
::_eo:: my outlook email
::_eg:: my gmail
::_dob:: my date of birth
::_mob:: my mobile number
::_salary:: ;Expected salary for software developer

; Current date and time. Optimized for filename automation.
::_dt::
{    
    ; This ensures that result will always be in english even if user's locale is not.
    currentDateTime := FormatTime(A_Now . ' L0x809', ' yyyy-MM-dd hhmmtt')

	Sendinput currentDateTime
}
```

Ensure you rename the values where applicable. You should be able to run the script with CTRL + F9.

## Create new script to skip company folder

This is to automate some actions in our directory that skips to the single job folder that exists in the company folder. 
Say our current folder is "Amazon" and it's subfolder is "Junior Software Developer". In that case AutoHotkey can auto select that "Junior Software Developer" folder and
navigate there in the file explorer.  

For now create the following file. SkipCompanyFolder.ahk. 

We will start with the following lines:

```txt
#Requires AutoHotkey v2
_logFile := "SkipCompanyFolder_logFile.txt"
targetDir := "C:\Users\<Username>\OneDrive\Documents\Job Tracking Docs"
```

## Implement error handling.

As you can see this is where I have decided to declare the log file and directory where AutoHotkey will scan for. First thing first, error catching. We can log our bugs and see incremental changes to them in our code and if the output is successful. 

## Process the File Explore's selected paths.

```txt
try
{
   currentDateTime := FormatTime(A_Now, ' yyyy/MM/dd hh:mmtt')
    
    ; This ensures that result will always be in English even if user's locale is not.
    currentDateTime := FormatTime(A_Now . ' L0x809', ' yyyy/MM/dd hh:mmtt')
    FileAppend("Script started on " currentDateTime "`n", logFile)
}

catch Error as err { 
    FileAppend("Error: " err.Message "`n", logFile) 
}
```

The code in our try block is similar to our _dt function. If you wanted to you could inherit the 
_dt function from our previous script as AHK supports the Object Orientated paradigm. 
Which means it's reusable.

But we won't go into that.

We want to continue the script.

## Create method to check our directory:

```txt
CheckDirectory() {
    static hwnd := 0
```

In here we set the WindowsTitle to 0.

To give more context, in Windows 11, `ahk_id HWND` is each window with a unique ID. The ID can be used to keep track of the specific window even if it's text or title were to change. But the ID is not unique to each individual program or window. 

It helps us identify the type of window wether it is the ID of the Windows File Explorer or Firefox. Do not confuse it with ahk_pid which is the process id.

If this is starting to feel very confusing and abstract, do not worry! It will make a lot more sense when you see how Windows Spy works, an amazing AHK tool which comes packaged with the programming language. 

Shows the backend values of the currently selected window. Stops us from having to figure out what is the identification value of the selected file explorer window that we need to search for.

<img src="../images/0001-01-32/WindowsSpyValues.png" class="image fit" alt="Windows spy values"/>

So I have used my mouse to select the file explorer and made sure I am in the target path and that the window is in focus so that I
can grab all the info needed to id it.

```txt
    try 
    {
        ; Find the File Explorer window with the specified title
        hwnd := WinExist("ahk_class CabinetWClass ahk_exe explorer.exe")
        if !hwnd {
            return
        }
        
        for window in ComObject("Shell.Application").Windows {
            if (window.HWND == hwnd) {
                currentDir := StrReplace(window.LocationURL, "file:///", "")
                break
            }
        }             
```

So we are going through a process of elimination. Making sure the Windows Spy values match up. So for the Windows Explorer we can help ensure we choose the right objects where object could be ahkclass and it's value could be CabinetWClass.

You will notice we used StrReplace to ommitt the generic "file:///" which is generated  at runtime. We also have to do a bit more work there. The generic runtime file path is URL-encoded with forward slashes instead off back slashes so Windows won't recognize that. It will also output %20 characters. 

The code below will clean up our in focus file path before AHKv2 attempts the process it so as to avoid errors.

```txt
        currentDir := StrReplace(currentDir, "%20", " ") ; Decode URL-encoded spaces
        currentDir := StrReplace(currentDir, "/", "\") ; Convert to backslashes for consistency
```

## Find the File Explore's in focus path.

Wre checked we are getting the correct object which is the File Explorer.

Now all that is required is to ensure it is scanning for when we have selected the target path and that it matches our currently selected folder in the File Explorer.

```txt
; Check if the current directory starts with the target directory
if (InStr(currentDir, targetDir) = 1) {
    FileAppend("Current directory starts with the target directory" _currentDateTime "`n", _logFile)           
    subfolders := []
    Loop Files, currentDir "\*.*", "D"  ; D = directories only
    {
        subfolders.Push(A_LoopFileFullPath)
    }
```

I will not explain what the first line is doing. If the current directory does contain our target, we will log that to our log file.
Next we are creating an empty array and are using the Loop Files command to [retrieve a list directories.](https://www.autohotkey.com/docs/v2/lib/LoopFiles.htm) inside our current directory. We are doing by:

- Using a blank wild card pattern to search our subfolder via the FilePattern: "\*.*"
- Specifying the D parameter allows our wild card FilePattern to scan our whole directory specifically for folders only.

The loop body command will check every absolute path only in our current directory until it has found 
the one that matches the in focus directory and then write that to our empty subfolder array with the 
string method ["Push(...)".](https://www.autohotkey.com/docs/v2/lib/Array.htm#Push) 

## Process the in focus path.

We have sorted out a lot of the How. Now that the program has found the in focus directory, we need to know a few things:

- Who and what: Is there more than one child folder in our subfolder array?
- Where and why: Where are we redirecting the user to, and why is it feasible? 

Let's see why.

```txt
    if (subfolders.Length = 1) {  ; Check if only one subfolder is present
        folder := subfolders[1]
        ; Navigate to the subfolder within the same window
        for window in ComObject("Shell.Application").Windows {
            if (window.HWND == hwnd) {
                window.Navigate(folder)
                break
            }
        }
        FileAppend("Navigated to: " folder " " _currentDateTime "`n", _logFile)
    } 
    else {
        FileAppend("Manual navigation required: multiple subfolders found" _currentDateTime "`n", _logFile)                              
    }   
};End of first IF in IF nest.
```

The first if statement should be self explanatory. The others show that we are only assigning the one subfolder found  to our path.
Again we are checking to ensure the in focus window will navigate to the next path if the ahk id is the same. Again this saves the user who is job hunting from jumping back and forth through folder paths.

Else of course the else statement will output something like:
```txt
Manual navigation required: multiple subfolders found 2024/12/07 05:17PM
```

You will also notice we can see the output of this in our log file which will help us see a record of our debugging and in realtime.

## Debugging techniques used:

We just have one else and catch statment and our program is complete. 

```txt
        else
        {
            FileAppend("Current directory does not start with target directory" _currentDateTime "`n", _logFile)
        }

    } catch Error as err { 
        FileAppend("Error: " err.Message " " _currentDateTime "`n", _logFile) 
    }
};End off CheckDirectory()
```

It's preety basic. If we are not in the target directory then our else statement would advise us and log the problem to the log file to see with the date and time. If we had a bug in either of our statements for example, a genric AHKv2 error would be thrown and caught by the catch statement, diriving from the Error object [which are thrown by built-in code when a runtime error occurs, and may also be thrown explicitly by the script](https://www.autohotkey.com/docs/v2/lib/Error.htm).  

So for example this could be the generic error thrown to our log file by the Error object.

```txt
Error: Divide by zero.
```

So you cannot divide by 0. Let's make make up some code that will break when we press the F12 key.

```txt
result := 1 / 0
```

Now let's put it in the first try statement we made.

```txt
try {    
    ; Intentional runtime error
    result := 1 / 0
    ; This ensures that result will always be in English even if user's locale is not.
    _currentDateTime := FormatTime(A_Now . ' L0x809', ' yyyy/MM/dd hh:mmtt')
    FileAppend("Script started" _currentDateTime "`n", _logFile)
    SetTimer CheckDirectory, checkInterval
} 
```

But how can we make a proven asumption that the particular line was responsible for our program breaking? What if there's thounds of lines to go through?

Breakpoints. Breakpoints are what break the problem down instead of us mearly staring at it in frustration.

<!-- <iframe width="420" height="315" src="https://youtu.be/oS261hvfsXE" frameborder="0" allowfullscreen></iframe> -->
<iframe width="560" height="315" src="https://www.youtube.com/embed/oS261hvfsXE?si=5WuPWfwDhHKMM2QB" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Make sure you installed the AHK++ extension. It is what I use. Click to the left of the line marked number (you may not have line numbers enabled which shouldn't matter). A breakpoint should appear like this little record button 🔴. Hold `CTRL + ALT + F9` to start debugging and then press F11 which I am doing in the video.   

I haven't shown how the catch statement executes with a breakpoint, in order to keep things simple.

You can now delete this line:

```txt
result := 1 / 0
```

Your code should be ready to easily auto navigate through single folders in the file path defined. I hope you have as much joy with
AHKv2 as I have!

```txt
#Requires AutoHotkey v2
_logFile := "SkipCompanyFolder_logFile.txt"
targetDir := "C:\Users\Jordan Nash\OneDrive\Job Tracking Docs"
checkInterval := 1000 ; Time in milliseconds between checks (1 second)

try {    
    ; This ensures that result will always be in English even if user's locale is not.
    _currentDateTime := FormatTime(A_Now . ' L0x809', ' yyyy/MM/dd hh:mmtt')
    FileAppend("Script started" _currentDateTime "`n", _logFile)
    SetTimer CheckDirectory, checkInterval
} 
catch Error as err { 
    FileAppend("Error: " err.Message "`n", _logFile) 
}

CheckDirectory() {
    static hwnd := 0

    try {
        ; Find the File Explorer window with the specified title
        hwnd := WinExist("ahk_class CabinetWClass ahk_exe explorer.exe")
        if !hwnd {
            return
        }

        for window in ComObject("Shell.Application").Windows {
            if (window.HWND == hwnd) {
                currentDir := StrReplace(window.LocationURL, "file:///", "")
                break
            }
        }             

        currentDir := StrReplace(currentDir, "%20", " ") ; Decode URL-encoded spaces
        currentDir := StrReplace(currentDir, "/", "\") ; Convert to backslashes for consistency

        ; Check if the current directory starts with the target directory
        if (InStr(currentDir, targetDir) = 1) 
        {
            FileAppend("Current directory starts with the target directory" _currentDateTime "`n", _logFile)                
            subfolders := []
            Loop Files, currentDir "\*.*", "D"  ; D = directories only
            {
                subfolders.Push(A_LoopFileFullPath) 
            }
            if (subfolders.Length = 1) {  ; Check if only one subfolder is present
                folder := subfolders[1]
                ; Navigate to the subfolder within the same window
                for window in ComObject("Shell.Application").Windows {
                    if (window.HWND == hwnd) {
                        window.Navigate(folder)
                        break
                    }
                }
                FileAppend("Navigated to: " folder " " _currentDateTime "`n", _logFile)
            } 
            else {
                FileAppend("Manual navigation required: multiple subfolders found" _currentDateTime "`n", _logFile)                              
            }            
        } 
        else
        {
            FileAppend("Current directory does not start with target directory" _currentDateTime "`n", _logFile)
        }

    } catch Error as err { 
        FileAppend("Error: " err.Message " " _currentDateTime "`n", _logFile) 
    }
}
```

## REFERENCES:

Note I have not included all references. Rather I included a few to help you on your journey in using the documentation. Note while I used AI to generate the code, I still had to do a lot of debugging and setting of breakpoints to identify initial errors it generated. Also had to ommit code that provided no functionality. 

Autohotkey.com. (2024). WinTitle & Last Found Window | AutoHotkey v2. \[online] 
Available at: https://www.autohotkey.com/docs/v2/misc/WinTitle.htm#ahk_id \[Accessed 1 Dec. 2024].

Autohotkey.com. (2024). Error Object | AutoHotkey v2. \[online] 
Available at: [Error Object](https://www.autohotkey.com/docs/v2/lib/Error.htm) \[Accessed 15 Dec. 2024].

Autohotkey.com. (2024). Array Object - Methods & Properties | AutoHotkey v2. \[online] 
Available at: https://www.autohotkey.com/docs/v2/lib/Array.htm \[Accessed December 15, 2024].