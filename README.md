# Intro

This is a comparison of packaging with python vs with npm.

# TLDR
Python packaging is 3x as complex as javascript packaging.  See the [conclusion](#conclusion) for more detail (but less than the whole document).

# Categories examined:
* [script creation](#script-creation)
* [conversion to a package](#conversion-to-a-package)
* [user setup](#user-setup)
* [first upload](#first-upload)
* [versioning](#versioning)
* [subsequent uploads](#subsequent-uploads)
* [file system cruft](#file-system-cruft)

<a name='script-creation' />
# Create your script
First, let's create a simple script that you can just run locally.

## python
```
> echo "print('hello world')" > hello_world.py
> python hello_world.py
hello world
>
```

## javascript
```
> echo "console.log('hello world')" > hello_world.js
> node hello_world.js
hello world
>
```

## comparison
Tie.  The complexity of creating a simple script is about the same for both languages.  Pretty simple.  So far, so good.

<a name='conversion-to-a-package' />
# Convert to package
Next, we decided this is an insanely useful script that we should share with the world.  Let's package it up so we can ship it!

For this example, I will assume that you are also sharing this thing on github, and that you've set up your repo for each language separately.

## python
Let's just do the things first, then I'll explain it after each section.  From your local repo:
```
> touch setup.py
> touch README.rst
> mkdir hello_world
> mv ../hello_world.py hello_world/__init__.py
```
* setup.py is the file python uses to track most package information.
* README.rst is the README.  It's not strictly necessary, but it's good practice to provide this.  Pypi only renders ReStructuredText, so if you want to have a nice-looking README on pypi, this is the format to use.
* hello_world is the directory your actual package files for a "hello_world" package need to be in.
* __init__.py is the main package file.  If you don't have this, you don't have a package.

You need to fill in `setup.py` by hand.  Edit it to look like this:
```python

# modules you need
from setuptools import setup, find_packages

# The setup function is where you specify your project's attributes
setup(
    # package name
    name='hello_world',
    # package version
    version='0.1.0',
    # short description
    description='A script that prints "hello world"',
    # the documentation displayed on pypi
    long_description=open('README.rst').read(),
    # not strictly necessary, but it's a good practice to provide a url if you have one (and you should)
    url='https://github.com/<user>/hello_world',
    # you.  you want attribution, don't you?
    author='<user>',
    # your email.  Or, *an* email.  If you supply an 'author', pypi requires you supply an email.
    author_email='<user>@<email service>',
    # a license
    license='MIT',
    # "classifiers", for reasons.  Below is adapted from the official docs at https://packaging.python.org/en/latest/distributing.html#classifiers
    classifiers=[
            # How mature is this project? Common values are
            #   3 - Alpha
            #   4 - Beta
            #   5 - Production/Stable
            'Development Status :: 4 - Beta',

            # Indicate who your project is intended for
            'Intended Audience :: Developers',

            # Pick your license as you wish (should match "license" above)
             'License :: OSI Approved :: MIT License',

            # Specify the Python versions you support here. In particular, ensure
            # that you indicate whether you support Python 2, Python 3 or both.
            'Programming Language :: Python :: 2',
            'Programming Language :: Python :: 2.7',
            'Programming Language :: Python :: 3',
            'Programming Language :: Python :: 3.4',
    ],
    # keywords.  because classifiers are for serious metadata only?
    keywords="hello world insanely useful",
    # what packages are included.  'find_packages' will automatically find your packages, but you can list them manually ('hello_world') if you want.
    packages=find_packages(),
    # minimum requirements for installation (third-party packages your package uses)
    install_requires=[],
    # give your package an executable.
    entry_points={
        'console_scripts': [
            # <name>=<package>:<function>
            # when the user calls 'hello_world' on the cli, 'hello_world/__init__.py::_cli()" will be called.
            'hello_world=hello_world:_cli',
        ],
    }
)
```
edit `hello_world/__init__.py` to look like this:
```python
def _cli():
  print('hello world')
```
This puts the code you want to run into a function that can be called from the executable script which setup.py will install for you.

edit `README.rst` to look like this:
```restructuredtext
===========
Hello World
===========

Prints "hello world" to the console!
```
Add some documentation.

## javascript
As with python, we'll do, and discuss as we go. From your local repo:
```
> touch README.md
```
You'll want a README.  NPM uses markdown.

```
> npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sane defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg> --save` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
name: (hello_world)
version: (1.0.0) 0.1.0
description: prints "hello world" to the console
entry point: (hello_world.js)
test command:
git repository: (https://github.com/<user>/hello_world.git)
keywords: hello world insanely useful
author: <user>
license: (ISC) MIT
About to write to /<project dir>/hello_world/package.json:

{
  "name": "hello-world-js",
  "version": "0.1.0",
  "description": "prints \"hello world\" to the console",
  "main": "hello_world.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/<user>/hello_world.git"
  },
  "keywords": [
    "hello",
    "world",
    "insanely",
    "useful"
  ],
  "author": "<user>",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/<user>/hello_world/issues"
  },
  "homepage": "https://github.com/<user>/hello_world"
}


Is this ok? (yes)
```
`npm init` will ask you a series of questions, build a `package.json` file, ask you to confirm the contents, and save it for you.
This file contains the data that npm uses to track your package.  It's roughly the equivalent of setup.py.

edit `package.json` to add:
```
  "bin": { "hello_world":"hello_world.js" }
```
after the "homepage" line.  Your entire `package.json` looks like:
```
{
  "name": "hello-world-js",
  "version": "0.1.0",
  "description": "prints \"hello world\" to the console",
  "main": "hello_world.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/<user>/hello_world.git"
  },
  "keywords": [
    "hello",
    "world",
    "insanely",
    "useful"
  ],
  "author": "<user>",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/<user>/hello_world/issues"
  },
  "homepage": "https://github.com/<user>/hello_world",
  "bin": { "hello_world":"hello_world.js" }
}
```

edit `hello_world.js` to look like this:
```javascript
#! /usr/bin/env node

console.log('hello world');
```
NPM will link to this file and try to run it.  You need the first line to tell the OS to use node to run it.

edit `README.md` to look like this:
```markdown
# Hello World

Prints "hello world" to the console!
```
Add some documentation.

## comparison
NPM wins.  It:
* builds most of the boilerplate for you
* auto-detects various files to provide sensible defaults (such as git repo, package name, and main script)
* doesn't force any unecessary hierarchy or naming.

Pypi, on the other hand:
* requires you to hand-write or copy/paste/modify a bunch of boilerplate
* requires four different kinds of description:
  * description
  * long_description
  * classifiers (which have valid and invalid values you have to look up)
  * keywords
* requires you to name your main file __init__.py and put it in a subdirectory.

The real advantage for NPM is the auto-discovery and setting of good defaults for the boilerplate, and an interactive dialog to set most of the rest.

<a name='user-setup' />
# Create your user

## python

go to https://pypi.python.org/pypi?%3Aaction=register_form and fill in the fields (4, at the time of this writing)

It has been mentioned to me that you can use `python setup.py register` to perform this step in conjunction with registering your project, and that this way you can skip the web-form.  This is not-explicitly-enough warned against in the [official docs](https://packaging.python.org/en/latest/distributing.html#create-an-account):

> First, you need a PyPI user account. There are two options:
>
> 1. Create an account manually using the form on the PyPI website.
> 1. Have an account created as part of registering your first project (see option #2 below).

Where 'below' is the registration section:
> ...you need to register your project. There are two ways to do this:
>
> 1. (Recommended): Use the form on the PyPI website. Although the form is cumbersome, it’s a secure option over using #2 below, which passes your credentials over plaintext.
> 1. Run python setup.py register. If you don’t have a user account already, a wizard will create one for you.

The relevant bit there is not nearly explicit enough, so I'll repeat it here with emPHAsis:  **#2 ... passes your credentials over plaintext**


## javascript
```
> npm adduser
Username: <user>
Password:
Email: (this IS public) <user>@<email service>
```
This puts a user file at `~/.npmrc` which will be used to log you into npm for publishing.

# comparison
NPM wins.  Again, they walk you through prompts, then generate the a file that makes your life easier later.

Python's webform is not complex or difficult to use, but it does require you to leave the CLI, and does not build or set any config file to help you use the package index in the future.

<a name='first-upload' />
# First upload
## python
```
> pip install twine
> pip install wheel
```

`twine` is required to securely upload your package.

`wheel` is required to correctly build your package in the currently preferred formats.

```
> python setup.py sdist
> python setup.py bdist_wheel --universal
```

Now go to https://pypi.python.org/pypi?%3Aaction=submit_form

To get there, you need to fill in your user credentials.  The easiest way to init your package is to upload your PKG-INFO file, rather than filling out the form manually.

It has been mentioned to me that you can use `python setup.py register` to perform this step more simply (and without going to the web-form).  This is a "bad idea".  See my earlier comments in [user setup](user-setup).

```
> twine upload dist/*
Uploading distributions to https://pypi.python.org/pypi
Enter your username: <username>
Enter your password: <pw>
>
```

The [official word](https://packaging.python.org/en/latest/distributing.html#packaging-your-project) from python is that you are supposed to build a source-distribution ('sdist') and, if you can, a universal wheel.  The 'universal' means it runs on python2 and python3. 

The `twine` call uploads your package files to pypi.

## javascript
```
> npm publish
```

NPM packages up your files, uploads them using the stored credentials, and you're done.

## comparison
NPM is the clear winner here.  A short one-liner gets the job done.

By contrast, python has you:
* install two new packages
* issue two packaging commands
* go to a website and enter your credentials
* upload a file
* return to your cli and issue an upload command
* finally, re-enter your user credentials

# Subsequent uploads
<a name='versioning' />
## Updating your version
### Python
Edit `setup.py` to set the new version.

### javascript
```
> npm version minor
```
...or 'major', or 'patch', according to semver.  If you have a git repo set up, this will also automatically perform a version-bump commit for you.

### comparison
NPM again.  Python goes the manual route, NPM goes the automated route, and throws in a helpful commit.

<a name='subsequent-uploads' />
## Uploading your package
### python

```
python setup.py sdist
python setup.py bdist_wheel --universal
twine upload dist/hello_world-0.2.0*
```

if you 'twine upload dist/*' like before, it tries to upload all versions, including the prior versions, which then gives you an error about how those versions already exist.  
### javascript

```
npm publish
```

### comparison
Again, NPM by a clear margin.  Python makes you run a bunch of commands with specific arguments, even referencing specific build-system-generated files manually.  If you get it wrong, you get odd errors.

NPM just asks you to repeat the same short one-liner, and automatically does the right thing.

# file system cruft <a name='file system cruft' />
## Python

Python builds four build-centric directories that you need to ignore:
* \__pycache__/
* build/
* dist/
* hello_world.egg-info/

## javascript

NPM doesn't add any directories

## comparison
NPM is the clear winner, with no filesystem clutter.

<a name="conclusion"></a>
# Conclusion
Python is very manual and boiler-platey.  NPM is quite a bit more automated, and wins in every comparison after the initial script creation (which it tied for).

The whole procedure can be summarized with some stats:

## Python
* 15 CLI actions
* 32 lines of extra code written manually
* 2 arbitrary file names, including renaming your original script
* 1 unnecessary subdirectory
* 4 clutter subdirectories
* 4 files to edit
* 2 web forms to fill out
* 4 occassions in which you need to enter your credentials
* 2 extra packages to install

## Javascript
* 9 CLI actions
* 4 lines of extra code written manually
* 1 arbitrary file name (README)
* 0 unnecessary subdirectories
* 0 clutter subdirectories
* 3 files to edit
* 0 web forms to fill out
* 1 occassion in which you need to enter your credentials
* 0 extra packages to install

## comparison
This is stats golf, and NPM wins with 18, vs python's 66.  

Assuming each task is equally weighted for difficulty/time/effort (which is a big assumption), python packaging is more than 3x as difficult/time-consuming as javascript packaging.  That sucks.  Even setting aside that assumption, javascript packaging is simpler than python in every examined category, making NPM/javascript the clear winner in the python-vs-javascript packaging comparison.
