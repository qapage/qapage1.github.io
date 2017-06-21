---
layout: post
title: Windows Desktop Automation
categories: [test automation, windows desktop automation]
tags: [test automation, windows desktop automation]
bigimg: /img/smyrna.jpg
---

Testing a windows desktop application just got easier (at least it did for me). Here’s a no license, no nonsense solution to testing your windows applications. And its all in Python.

Tools we’ll use today,

[PyWinAuto](http://pywinauto.github.io/) -  a Windows GUI automation library written in pure Python

[SWAPY](https://github.com/pywinauto/SWAPY) - a tool thats part of PyWinAuto and helps with Object identification

[pytest](http://doc.pytest.org/en/latest/) - the test runner that I like to use


#### Steps to tackle this task,

* Install python and then pywinauto + pytest in your python environment (preferably in your own virtualenv)
* Launch your desktop app
* Launch SWAPY and look at the list of apps it displays, watch for your app in that list
* Click on your app and you should the different objects on your application that are available for you to interact with. Once you click on an item and select the operation you want to perform against it, you’ll see python code being auto generated for you in the tab on the right in SWAPY.
* Save that code in a file, save as something.py and you can now run this script from the command line as python something.py and you should the script perform the operation you did. Basically you recorded the operation you performed earlier and are able to replay it with any changes you want to make to it.


#### Now, lets try a real working example, 

Lets try to automate interaction with notepad. Easy enough right?

* Launch notepad
![placeholder](/assets/notepad_open.png)

* Launch SWAPY from the executable you downloaded via https://github.com/pywinauto/SWAPY
![placeholder](/assets/SWAPY_launched.png)

* If you look closely, you can see that the notepad application already shows up in the ‘Objects browser’ tab. Lets click on it. Lets say I just want to open notepad.exe, hit the file menu and then hit quit.
![placeholder](/assets/SWAPY_menu.png)

* So I drill down in the object browser menu until I am at the element I want to interact with.
![placeholder](/assets/SWAPY_drilled_down.png)

* I right click on that element and select the operation I want to perform. Which is click in this case.
![placeholder](/assets/SWAPY_click.png)

* After you select ‘Click’ from the right click menu, you can see that the tab named ‘Editor’ gets automatically populated with python code for you to use to perform this action. Cool right?? Here’s the code as seen on SWAPY.
![placeholder](/assets/SWAPY_click_codegen.png)


* You save this into something.py and run it from the command line to see the Notepad.exe being launched and exited via using the File>Exit menu button. There’s your first automated script for windows!

To see details about how you can write a test with asserts + pytest thrown in, look at this little repo of tests I put up on [github](https://github.com/dbhaskaran1/win_logon)
