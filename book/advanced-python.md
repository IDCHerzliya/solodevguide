# Advanced Python Bundling

The best way to deploy and run Python code on your Solo is to use the *Solo CLI*, as [described here](concept-dronekit.html#deploying-scripts-to-solo).
This approach uses just two simple commands to package your scripts and their dependencies, uploading the package to Solo, and running the file.

This guide shows the method that was used prior to introduction of *Solo CLI* packaging. It may be of interest/use to some Solo developers.


## Manual bundling

First create and navigate to a new directory on your host computer. This directory will be populated with your own Python scripts and all their dependencies. The entire directory will then be sent to Solo. 

Start by creating a virtual environment on your host computer (Linux/Mac OS X):

<div class="host-code"></div>

```sh
pip install virtualenv
virtualenv env
source ./env/bin/activate
```

We want to configure our environment to not compile any C extensions. We can do this simply in our virtual environment with this command:

<div class="host-code"></div>

```sh
echo 'import sys; import distutils.core; s = distutils.core.setup; distutils.core.setup = (lambda s: (lambda **kwargs: (kwargs.__setitem__("ext_modules", []), s(**kwargs))))(s)' > env/lib/python2.7/site-packages/distutils.pth
```

<aside class="note">
The commands above are for Linux! Windows has slightly different commands (see the [User guide](http://virtualenv.readthedocs.org/en/latest/userguide.html)) so instead do:

<div class="host-code"></div>

```sh
pip install virtualenv
virtualenv env
env\Scripts\activate.bat
echo 'import sys; import distutils.core; s = distutils.core.setup; distutils.core.setup = (lambda s: (lambda **kwargs: (kwargs.__setitem__("ext_modules", []), s(**kwargs))))(s)' > env\Lib\site-packages\distutils.pth
```
</aside>

Now you can install Python packages into the new environment using `pip install`. 

<aside class="caution">
When modules require a C extension, they will fail silently. Test your code!
</aside>

When you're ready to move code over to Solo, create a `requirements.txt` file containing what packages you've installed:

<div class="host-code"></div>

```sh
pip freeze > requirements.txt
```

Next, download these packages on your host computer so they can be moved to Solo along with your code. You can do this by using [pip wheel](https://pip.pypa.io/en/latest/reference/pip_wheel.html) to download them into a new folder (`./wheelhouse`). Run this command:

<div class="host-code"></div>

```sh
pip wheel -r ./requirements.txt --build-option="--plat-name=py27"
```

This installs all the dependencies in `requirements.txt` as Python wheel files, which are source code packages.

Next, you can move this entire directory (except for *env*) over to Solo. The following command shows how to do this using *rsync*:

<div class="host-code"></div>

```sh
rsync -avz --exclude="*.pyc" --exclude="env" ./ root@10.1.1.10:/opt/my_python_code
```

<aside class="tip">
On Windows, you may prefer to copy the folder using a graphical tool like *WinSCP* ([download here]([WinSCP](https://winscp.net/download/winscp576setup.exe)).
</aside>

Install _pip_ on Solo (from the host computer):

<div class="host-code"></div>

```sh
solo install-pip
```

SSH into Solo and install _virtualenv_:


```
pip install virtualenv
```

Finally, navigate to the newly made code directory (`/opt/my_python_code`) and run these commands:

```sh
virtualenv env
source ./env/bin/activate
pip install --no-index ./wheelhouse/* -UI
```

This requires no Internet connection. Instead, it installs from all the downloaded dependencies you transferred from your computer. You can now run your Python scripts with any packages you depended on, without having impacted any of Solo's own Python dependencies.
