Version 0.9, 03 Oct 2024 	

[1 Requisites](#requisites)

[2 Code repositories1](#code-repositories)

[3 Installation](#installation)

[3.1 Test LIMS](#test-lims)

[3.2 Hardware Requirements](#hardware-requirements)

[3.3 Senaite installation](#senaite-installation)

[3.4 Production LIMS](#production-lims)

[4 Docker](#docker)

[5 Maintenance](#maintenance)

[6 Coding](#coding)

[6.1 Convention](#convention)

[6.2 Code Entry Points](#code-entry-points)

[7 API](#api)

[8 Tech Support](#tech-support)

1. # Requisites

   Installing Senaite requires sound Linux systems administration and [Git](https://git-scm.com/) version control skills. For coding, good Python and Javascript skills and patience with the Plone learning curve. [Plone](https://plone.org/) is the CMS that Senaite LIMS is built on.

2. # Code repositories 

   [Senaite](https://www.senaite.com/) evolved from [Bika LIMS](https://www.bikalims.org/). Bika Lab Systems (project founders) codes on Senaite, and develops and maintains Senaite add-ons, some non core functions under the Bika banner.

   Main code and add-ons live in the Senaite repositories, [https://github.com/senaite](https://github.com/senaite).

   Core 

   [senaite.core](https://github.com/senaite/senaite.core) 

   [senaite.app.listing](https://github.com/senaite/senaite.app.listing) ReactJS listing component 

   [senaite.lims](https://github.com/senaite/senaite.lims) Meta package installing dependencies

   Important add-ons

   [senaite.impress](https://github.com/senaite/senaite.impress) for rendering COAs to PDF

   Popular add-ons

   [senaite.instruments](https://github.com/bikalims/senaite.instruments) Collection of instrument interfaces. (NB from the Bika repos \- the Senaite one was last added to in 2021\) 

   [bika.coa](https://github.com/bikalims/bika.coa) Collection of differentiated COAs

   [senaite.sampleimporter](https://github.com/bikalims/senaite.sampleimporter) Import bulk Samples from spreadsheets

   [senaite.storage](https://github.com/senaite/senaite.storage) Sample storage add-on 

   [senaite.batch.invoices](https://github.com/bikalims/senaite.batch.invoices) Issue invoices per batch

   [senaite.patient](https://github.com/senaite/senaite.patient) Add patients for health care labs

   [bika.ui](https://github.com/bikalims/bika.ui) Makes branding easier and restores classic Bika iconography to replace default black and white scheme

   [senaite.app.spotlight](https://github.com/senaite/senaite.app.spotlight) Easy search tool

3. # Installation 
   1. # Test LIMS 

   We recommend both Production and Test/Training LIMS installations. 

   The secondary LIMS is used for acceptance testing, thereafter as an e-learning sandbox for users to uninhibitedly test real life scenarios without interrupting production.

   2. # Hardware Requirements

   Plone requires a lot of memory and for best performance, multi core processors are also recommended. Moderate disk space is needed.

   If possible, please see if you can install Linux with an ZFS based filesystem to eliminate running out of inodes that sometimes (rare) happens on conventional file systems.  When backups are not properly configured, many small blob files are created and while there might be enough disk space left there could be no more inodes which are used to index the files.

   To make use of multiple processor cores, ZEO (Zope Enterprise Objects) must be installed and configured to run a thread per ZEO client.

   We recommend starting on a VM to which resources can be dynamically added, on say a 4 core processor, 32 GB RAM and 250 GB disk. Smaller labs will require less, e.g. 2 cores and 16 GB RAM.

   3. ## Senaite installation

   You can use docker to install senaite if you are familiar with it, see [Section 4](#docker) below, this section covers installing senaite manually.  The [installation section of the Senaite documentation](https://www.senaite.com/docs/installation) is outdated so here is how to install the correct versions of Python and Plone to ensure Senaite works correctly. The Senaite install itself uses [Buildout](http://www.buildout.org/en/latest/) for installation automation. Plone, Senaite and the add-ons are added as eggs .

   # Installation on Ubuntu 24.04

* Ensure all system dependencies are installed:

		sudo apt install build-essential  libxml2 libxml2-dev libxslt1.1 libxslt1-dev  libffi-dev libcairo2 libpango-1.0-0 libgdk-pixbuf2.0-0 libpangocairo-1.0-0 libgdk-pixbuf2.0-0 zlib1g zlib1g-dev git vim
* Create new user called `zope` and give it sudo access

		sudo adduser zope
  		sudo usermod -aG sudo zope
  		su - zope
  		mkdir instances dist src

* To allow us to install python 2.7, install pyenv using the [pyenv-installer](https://github.com/pyenv/pyenv-installer):

  		curl https://pyenv.run | bash

	If you experience issues accessing the git repository, try:

  		git clone https://github.com/pyenv/pyenv.git \~/.pyenv

	Extend .bashrc with the following commands

		echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
  		echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
		echo -e 'if command -v pyenv 1>/dev/null 2>&1; then  eval* "$(pyenv init --path)" fi' >> ~/.bashrc

	Refresh the terminal

		source ~/.bashrc

	Test pyenv is installed

		pyenv —version	

* Install the latest version of python 2.7

		pyenv install 2.7.18

* Download and extract [unified installer](https://github.com/plone/Installers-UnifiedInstaller) into \~/src and install plone

		~/src/Plone-5.2.xx-UnifiedInstaller-xx/install –with-python=/home/zope/.pyenv/shims/python2.7.18  --target=/home/zope/instances/staging --password=local zeo


	If the installer does not complete, then copy pre-installed version of buildout-cache

		cd /home/zope/instances/staging
  		curl https://drive.google.com/file/d/1deBLvzffX12MmMryUwgfMyhulkBNPoTZ/view?usp=drive_link
  		rm -rf buildout-cache
  		tar -xf buildout-cache.tgz
  
  	We then need to put a senaite version of buildout in place and rebuild
  
  		cd zeocluster
  		curl https://github.com/bikalims/bika.documentation/blob/main/docs/buildout.cfg
  		./bin/buildout
  	
	To test

		cd /home/zope/instances/staging/zeocluster
		./bin/zeoserver start
		./bin/client1 fg

	If the client1 starts successfully on port, you’re good to go. Stop the instance before contining

 		CTRL-D
  		./bin/zeoserver stop

* To control the starting and stopping of the instance you can install [supervisord](https://supervisord.org)
  
  		sudo apt install supervisor
  
  	Copy the staging conf file into conf.d folder
  
  		sudo curl -o /etc/supervisor/conf.d/staging.supervisord.conf https://github.com/bikalims/bika.documentation/blob/main/docs/staging.supervisord.conf

  	Update supervisor (which also starts the LIMS)
  
  		sudo supervisorctl update

  	To see if the LIMS instance started
  
  		supervisorctl status

* To be able to access the instance from another computer on the LAN, install [nginx](https://nginx.org)
  
  		sudo apt install nginx

 	Copy the staging conf file into conf.d folder
  
  		sudo curl -o /etc/nginx/conf.d/staging.nginx.conf https://github.com/bikalims/bika.documentation/blob/main/docs/staging.nginx.conf
  
	Reload nginx

		sudo nginx -s reload			



Installation does not always work first time, often because of incorrect version dependencies. Please search this comprehensive Senaite installation thread: [Complete setup guide, step-by-step](https://community.senaite.org/t/complete-setup-guide-step-by-step/137) for the errors you get as first stop, then the Internet.

4. ## Production LIMS

   For a Production LIMS it is best to install a performance optimised and load balanced stack, all available as Open source and free. Use the equivalent of:

   Essential

   Plone

   Senaite LIMS and add-ons

   [nginX](http://nginx.org/en/docs/http/load_balancing.html) load balancing web server

   [Supervisor](http://supervisord.org/) Process control

   Firewall, ssl, https access and security certificate

   Mail server

   DNS

   Back-up and Restore procedures 

   Logs rotation 

4. # Docker

   Docker images are also available, e.g. [senaite.docker](https://github.com/senaite/senaite.docker) that can be modified to fit. See  [Readme](https://github.com/senaite/senaite.docker/blob/master/README.md).

5. # Maintenance

   Senaite and the add-ons are regularly updated with official releases every few months, it is recommended that the upgrades are done on the Test LIMS when they become available, and thoroughly tested before upgrading to Production.

   The releases are subjected to automated unit tests, however If you are running your own customised add-ons, test those particularly well as changes in the core might affect them.

   Upgrade scripts may include data conversions and those take longer to complete. 

   Formal upgrades can be run in the LIMS UI by admin users when prompted (after the code base was upgraded).

6. # Coding

   [The Anatomy of Plone](https://training.plone.org/mastering-plone-5/anatomy.html#the-anatomy-of-plone). Zope object-oriented database, [ZODB](https://zodb.org/en/latest/).

   [Plone Development and Customization intro](https://2022.training.plone.org/index.html)

   [Developing for Plone](https://5.docs.plone.org/develop/index.html)

   1. ## Convention {#convention}

   See the [Senaite Contribution guide](https://github.com/senaite/senaite.core/blob/2.x/CONTRIBUTING.md). For your code:

* Include tests

* Follow [Plone's Python style guide](https://5.docs.plone.org/develop/styleguide/index.html)

* Write a descriptive and detailed summary

* Include whitespace and formatting changes in discrete commits

* Add a changelog entry in CHANGES.rst

  2. ## Code Entry Points {#code-entry-points}

  Coding COAs is a good entry point. The COA's are page templates which are HTML files with some additional information, written in TAL, METAL and TALES.

     TAL (Template Attribute Language) is used for templating HTML using DB attributes and to include expressions. TALES (TAL Expression Syntax) defines the syntax and semantics of the expressions.

     METAL (Macro Expansion for TAL) enables the combination, re-use and nesting of templates.

     [Plone examples and training documentation](https://training.plone.org/mastering-plone-5/zpt.html). See a [Bika COA code example](#heading=h.tjhigkfofvq6) below

  As a next step, coding instrument interfaces are recommended as many of them exist and can be used as templates. See [Interface code example](#heading=h.uwuqb5qv5tck)

7. # API

   Robust RESTful JSON API, [senaite.jsonapi](https://senaitejsonapi.readthedocs.io/). Create, Read and Update (CRU operations) through http GET/POST requests

8. # Tech Support

   The public Bika Slack channels are good for general gratis user and technical support. Prioritised professional report is billable and managed in private channels, ditto for issues in the [Bika Jira tracker](https://bika.atlassian.net/jira/dashboards/10000).  [Bika Issue Tracking](https://www.bikalims.org/manual/issue-tracking)

