# How to start development python
- recommended development tool: VisualStudioCode (VSC)

## first steps
1. download a python kernel-version
- !!! if the installation ask u for a development path - accept the autosetup otherwise you need to setup your environment variables by yourself
- link: https://www.python.org/

2. download VSC or Pycharm or Anaconda or what ever u want to use as DEV-tool in your prefered version
- link: https://code.visualstudio.com/

3. test your instance
- open a terminal (win: cmd, linux/mac: terminal)
- type-in: python --> and press enter
- if you see your installed python version and a new shell will open inside your terminal everything is fine
- the new shell is not a new window or tab, its inside your opened window like: ">>>"
- if an error apear u need to check your environment variables 
- you can close the shell with typing-in: exit() 

4. create a development folder on your explorer to start programming in your python language

5. in VSC
- if you create a .py or .ipynb file sometimes you need to manually install extensions
- allow VSC to install pycharm and all necessary extensions to perfectly run your codingsetup :) 


## python files
- python files/scripts will have the post-ending *.py
- inside python files you are able to program your instructions
- for rapid prototyping method we are using jupyter-notebook extension
- to install jupyter u need to download the jupyter module
- command: pip install jupyter
- post-ending of jupyter-files is *.ipynb
- jupyter notebook files are interactive python notebooks with cells, each cell will handle like a complete own py-script

## pip and virtual environments
- pip is pythons built-in package manager (comparable to npm in JavaScript/Node.js, maven/gradle in Java, cargo in Rust, NuGet in C#)
- with pip we are able to install external modules/packages from the official package index (PyPI): https://pypi.org/
- command to install a package: pip install <packagename>
- command to install a specific version: pip install <packagename>==<version>
- command to list all installed packages: pip list
- command to save your project dependencies: pip freeze > requirements.txt
- command to install all dependencies of a project: pip install -r requirements.txt
- !!! important: if you install packages globally, different projects can conflict with each other because they need different versions of the same package
- to isolate a project from your global python installation we use virtual environments (venv)
- a virtual environment is a separate, clean python installation copy just for one project/folder
- command to create a venv: python -m venv .venv
- command to activate the venv:
  - windows: .venv\\Scripts\\activate
  - linux/mac: source .venv/bin/activate
- while a venv is activated, all pip install commands only affect this local environment, not your global python installation
- command to leave/deactivate the venv: deactivate
- comparison: a venv is conceptually similar to a node_modules folder + package.json in a JavaScript project, or a dedicated project SDK in Java/.NET - it keeps dependencies local and reproducible
- later lessons (working with databases, working with web informations) require external packages, always work inside an activated venv for those
