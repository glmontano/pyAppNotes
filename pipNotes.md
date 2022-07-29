### Breakdown of PIP Commands

General structure, assuming trusted hosts must be added (working behind firewalls at work)

````
python
	-m pip install 
	--trusted-host files.pythonhosted.org 
	--trusted-host pypi.org 
	--trusted-host pypi.python.org
	PyQt5
````

### Setting these trusted hosts into pip.ini

This allows those hosts to be permanently trusted

`pip config set global.trusted-host "files.pythonhosted.org pypi.org pypi.python.org"`

### Example of installing pypiwin32

`pip install package1 package2 package3`
