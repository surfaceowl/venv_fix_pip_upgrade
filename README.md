## Upgrading Python Pip in Python Virtual Environments (venv in particular)

Usually upgrading pip is trivial with `pip install --upgrade pip` from the terminal.  Virtual environments are great for development, they keep our project-specific dependencies separate to avoid dependency conflicts between projects, and has the added bonus of also protecting our system installation of python from pollution with project-specific modules.   Let's presume you are using a virtual environment that was setup with pip version 10.0.1, and a newer pip version is available (e.g. 18.1 which is current as of this writing).

## However - on Windows...
However, on some platforms (e.g. windows), the OS does not let you upgrade system files while they are in use.  When you activate a python venv (virtual environment), pip is used and Windows prevents the user from using the pip upgrade command we just mentioned.
Most of the community comments are linux specific, so common suggestions you'll find from the community include the following terminal commands:

+ `pip install --upgrade pip`
+ `python -m pip install --upgrade pip`
+ ...some variation of "use easy_install"
+ ...or more rarely:  `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`

###These work perfectly outside of virtual environments, but in some environments they do not - and there are some traps for the unwary.  Spoiler alert - the last suggestion is our best hope

1. Old faithful:  `pip install --upgrade pip` fails with:
```ERROR: To modify pip, please run the following command:
..\my path to my venv\Scripts\python.exe -m pip install --upgrade pip
You are using pip version 10.0.1, however version 18.1 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.
```

2. The venv Python environment helpfully suggested `python -m pip install --upgrade pip`, but that fails with:
```
Collecting pip
  Using cached https://files.pythonhosted.org/packages/c2/d7/90f34cb0d83a6c5631cf71dfe64cc1054598c843a92b400e55675cc2ac37/pip-18.1-py2.py3-none-any.whl
Installing collected packages: pip
  Found existing installation: pip 10.0.1
    Uninstalling pip-10.0.1:
      Successfully uninstalled pip-10.0.1
  Rolling back uninstall of pip
Exception:
Traceback (most recent call last):
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\basecommand.py", line 228, in main
    status = self.run(options, args)
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\commands\install.py", line 335, in run
    use_user_site=options.use_user_site,
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\req\__init__.py", line 49, in install_given_reqs
    **kwargs
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\req\req_install.py", line 748, in install
    use_user_site=use_user_site, pycompile=pycompile,
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\req\req_install.py", line 961, in move_wheel_files
    warn_script_location=warn_script_location,
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_internal\wheel.py", line 431, in move_wheel_files
    generated.extend(maker.make(spec))
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 403, in make
    self._make_script(entry, filenames, options=options)
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 307, in _make_script
    self._write_script(scriptnames, shebang, script, filenames, ext)
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 243, in _write_script
    launcher = self._get_launcher('t')
  File "..\my path to my venv\lib\site-packages\pip-10.0.1-py3.6.egg\pip\_vendor\distlib\scripts.py", line 382, in _get_launcher
    result = finder(distlib_package).find(name).bytes
AttributeError: 'NoneType' object has no attribute 'bytes'

```

3. easy_install - I did not look at this seriously since easy_install is an [old library](https://packaging.python.org/discussions/pip-vs-easy-install/) and there is no need for yet another library to manage packages - stick with pip.


4. So we stumble around some more, and find this fourth option `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`, but we may be blocked by this fail:

Windows Powershell cannot execute the curl command 
```(venv) > curl https://bootstrap.pypa.io/get-pip.py
curl : The request was aborted: Could not create SSL/TLS secure channel.
At line:1 char:1
+ curl https://bootstrap.pypa.io/get-pip.py
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-WebRequest], WebException
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand
```

Drats!  Sounds scary, but luckily, [this post](https://powershell.org/forums/topic/is-it-possible-to-enable-tls-1-2-as-default-in-powershell/) shows us this is merely an issue with old windows TLS settings not being secure enough for what the Python Software Foundation is using... this is an easy fix.
1- There is a PowerShell command we can use to temporarily upgrade the TLS version that PowerShell is using behinds the scenes.  Paste the command into your PowerShell terminal and hit enter:  [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;

2- Nothing visible happens except PowerShell moves down one command prompt.  Next:

3- Paste the first part of the curl command into Powershell:  ` curl https://bootstrap.pypa.io/get-pip.py` (everything before the "-o")

4- PowerShell with download the get-pip.py file

5- From PowerShell, run get-pip.py with the version number you want (e.g. 18.1):  python get-pip.py pip==18.1
...and you should see
```
(venv) > python get-pip.py pip==18.1
Collecting pip==18.1
  Using cached https://files.pythonhosted.org/packages/c2/d7/90f34cb0d83a6c5631cf71dfe64cc1054598c843a92b400e55675cc2ac37/pip-18.1-py2.py3-none-any.whl
Collecting wheel
  Using cached https://files.pythonhosted.org/packages/5a/9b/6aebe9e2636d35d1a93772fa644c828303e1d5d124e8a88f156f42ac4b87/wheel-0.32.2-py2.py3-none-any.whl
Installing collected packages: pip, wheel
  Found existing installation: pip 18.1
    Uninstalling pip-18.1:
      Successfully uninstalled pip-18.1
Successfully installed pip-18.1 wheel-0.32.2
(venv) >

```  
6- To make sure it's working, type "pip -V", and you should see something like below, showing the upgraded version number.
```
(venv) > pip -V
pip 18.1 from ..\my path to my venv\lib\site-packages\pip (python 3.6)
(venv) >

```

Suggestions, comments and additions welcome

