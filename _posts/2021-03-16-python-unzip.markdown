---
layout: post
title: "Python and zip archives"
date: 2021-03-16T20:19:55+01:00
tags: python JupyterLab unzip
---
# Problem

I got a task at work to provide bank statements for the audit to the auditor. Each of the zipped bank statement archives includes **hundreds** of the bank statements (and yet you have a 12 periods to search for).

# Solution

Each bank statement was named with regular pattern- what made things easier for me. I found that the best and the most efficient way to solve it is to:
- get bank statement archive list
- predefine as a list which accounts are to be found in the archive
- extract only those files that matches the bank account pattern in the bank statement file name
- extract it to some sub directory
- get print out of what bank statement files where found and extracted- for quick crosscheck etc.

# Code

This code solved a problem for me for now. It can be run as a python script or through JupyterLab- that's how I use it.

```python

    import zipfile
    from zipfile import ZipFile
    import os.path

    #set home folder and change working directory
    home_folder=os.getenv('HOMEPATH')
    os.chdir(home_folder)

    #create the empty list to store the zip file name
    zip_files = []

    #create the list of the file names that you want to extract from the zip file
    file_pattern = ["account-aa4444","account-bb7777","account-cc9999"]

    #set the base directory to Downloads and fill in list with the file names
    directory = r'Downloads'
    save_directory = os.path.join(directory,'temp')
    for entry in os.scandir(directory):
        if (entry.path.endswith (".zip")
                or entry.path.endswith (".zip")) and entry.is_file():
            zip_files.append(entry.path)
        
    #loop through the each zip file and find out if there 
    #is a file name containing a pattern; if so extract to the temp directory
    for item in zip_files:
        with zipfile.ZipFile(item) as zip_archive:
            for file in zip_archive.namelist():
                for item in file_pattern:
                    result = (file.startswith(item,13,55))
                    if result == True:
                        zip_archive.extract(file, save_directory)
    print("This is the list of exctracted files from the zip files:")
    for item in os.listdir(save_directory):
        if item == 0:
            print("No files found for extraction")
        else:
            print(item)
```

So in short, code will look up in each zip archive for each of the file name pattern and if found extract it, 
i.e. pattern **account-aa4444** where the full name of the bank statement file is i.e. *Bank-Statement-**account-aa4444**-2020-08.pdf*.

Next thing to add is regex; so instead of looking for a pattern in predefined position in the file name, try to find a pattern regardless where it stands.
This will make code more robust for more irregular file names etc.

At the end I am happy to have that written- it saved me and will save me a lot of time in the future !
