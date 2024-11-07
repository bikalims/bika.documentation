Version 0.9, 03 Oct 2024 	

[1 Requisites](#requisites)

[2 Code repositories](#code-repositories)

[3 Installation](#installation)

[3.1 Test LIMS](#test-lims)

[3.2 Hardware Requirements](#hardware-requirements)

[3.3 Senaite installation](#senaite-installation)

[3.4 Production LIMS](#production-lims)

[4 Production](#production)

[5 Docker](#docker)

[6 Maintenance](#maintenance)

[6.1 Backup and Restore](#backup-and-restore)

[6.2 Software Updates](#software-updates)

[7 Coding](#coding)

[7.1 Convention](#convention)

[7.2 Code Entry Points](#code-entry-points)

[8 API](#api)

[9 Tech Support](#tech-support)

[10 Code Examples](#code-examples)

[10.1 COA](#coa)

[10.2 Instrument Interface](#instrument-interface)

1. # Requisites

   Installing Senaite requires sound Linux systems administration and [Git](https://git-scm.com/) version control skills. For coding, good Python and Javascript skills and patience with the Plone learning curve. [Plone](https://plone.org/) is the CMS that Senaite LIMS is built on.

2. # Code repositories 

   [Senaite](https://www.senaite.com/) evolved from [Bika LIMS](https://www.bikalims.org/). Bika Lab Systems (project founders) codes on Senaite, and develops and maintains Senaite add-ons, some non core functions under the Bika banner.

   Main code and add-ons live in the Senaite repositories, [https://github.com/senaite](https://github.com/senaite).

   **Core** 

   [senaite.core](https://github.com/senaite/senaite.core) 

   [senaite.app.listing](https://github.com/senaite/senaite.app.listing) ReactJS listing component 

   [senaite.lims](https://github.com/senaite/senaite.lims) Meta package installing dependencies

   **Important add-ons**

   [senaite.impress](https://github.com/senaite/senaite.impress) for rendering COAs to PDF

   **Popular add-ons**

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

		sudo apt install build-essential  libxml2 libxml2-dev libxslt1.1 libxslt1-dev  libffi-dev libcairo2 libpango-1.0-0 libgdk-pixbuf2.0-0 libpangocairo-1.0-0 libgdk-pixbuf2.0-0 zlib1g zlib1g-dev git vim python3-dev openssl libssl-dev libjpeg-dev libreadline-dev
* Create new user called `zope` and give it sudo access

		sudo adduser zope
  		sudo usermod -aG sudo zope
  		su - zope
  		mkdir instances dist

* To allow us to install python 2.7, install pyenv using the [pyenv-installer](https://github.com/pyenv/pyenv-installer):

  		curl https://pyenv.run | bash

	If you experience issues accessing the git repository, try:

  		git clone https://github.com/pyenv/pyenv.git ~/.pyenv

	Extend .bashrc with the following commands

		echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
  		echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
		echo -e 'if command -v pyenv 1>/dev/null 2>&1; \n then  eval "$(pyenv init --path)" \n fi' >> ~/.bashrc

	Refresh the terminal

		source ~/.bashrc

	Test pyenv is installed

		pyenv --version	

* Install the latest version of python 2.7 and create a virtualenv

		pyenv install 2.7.18
  		pyenv virtualenv 2.7.18 venv2.7.18

* Download and extract [unified installer](https://github.com/plone/Installers-UnifiedInstaller) and install plone
  	
		wget --no-check-certificate https://launchpad.net/plone/5.2/5.2.14/+download/Plone-5.2.14-UnifiedInstaller-1.0.tgz
  
  		tar -xf Plone-5.2.14-UnifiedInstaller-1.0.tgz
  
		~/Plone-5.2.14-UnifiedInstaller-1.0/install.sh --with-python=/home/zope/.pyenv/versions/venv2.7.18/bin/python  --target=/home/zope/instances/staging --password=local zeo


	If the installer does not complete, then copy a pre-installed version of [buildout-cache](https://drive.google.com/file/d/1deBLvzffX12MmMryUwgfMyhulkBNPoTZ/view?usp=drive_link) into your Downloads folder

		cd /home/zope/instances/staging
  		rm -rf buildout-cache
  		mv ~/Downloads/buildout-cache.tgz .
  		tar -xf buildout-cache.tgz
  
  	We then need to put a senaite version of buildout in place and rebuild
  
  		cd zeocluster
  		rm buildout.cfg
  		curl -O https://raw.githubusercontent.com/bikalims/bika.documentation/refs/heads/main/docs/buildout.cfg
  		./bin/buildout
  	
	To test

		cd /home/zope/instances/staging/zeocluster
		./bin/zeoserver start
		./bin/client1 fg

	If the client1 starts successfully on port, you’re good to go. Stop the instance before contining

 		CTRL-C
  		./bin/zeoserver stop

* To control the starting and stopping of the instance you can install [supervisord](https://supervisord.org)
  
  		sudo apt install supervisor
  
  	Copy the staging conf file into conf.d folder (note that on line 8 the server name might need to be changed to the server IP Address)
  
  		sudo curl -o /etc/supervisor/conf.d/staging.supervisord.conf https://raw.githubusercontent.com/bikalims/bika.documentation/refs/heads/main/docs/staging.supervisord.conf

  	Update supervisor (which also starts the LIMS)
  
  		sudo supervisorctl update

  	To see if the LIMS instance started
  
  		supervisorctl status

* To be able to access the instance from another computer on the LAN, install [nginx](https://nginx.org)
  
  		sudo apt install nginx

 	Copy the staging conf file into conf.d folder
  
  		sudo curl -o /etc/nginx/conf.d/staging.nginx.conf https://raw.githubusercontent.com/bikalims/bika.documentation/refs/heads/main/docs/staging.nginx.conf
  
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

5. # Docker

   Docker images are also available, e.g. [senaite.docker](https://github.com/senaite/senaite.docker) that can be modified to fit. See  [Readme](https://github.com/senaite/senaite.docker/blob/master/README.md).

5. # Maintenance
   1. ## Backup and Restore
* In the instance folder structure there are 2 backup folders `var/backups` and `var/blobstoragebackups`
* In the instance `bin` folder there commands `backup` and `restore`
* To backup an instance:


    	cd /home/zope/instances/staging/zeocluster
    	./bin/backup

* The files in backup folders `var/backups` and `var/blobstoragebackups` should them be copies to a backup storage location
* To restore from a previous backup simply copy the backed up folders into folders `var/backups` and `var/blobstoragebackups`, run restore and restart the entire instance
* To restore a backup:

    	cd /home/zope/instances/staging/zeocluster
    	./bin/restore
    	./bin/zeoserver restart
    	./bin/client1 restart
    	./bin/client2 restart
    
    
   2. ## Software Updates

   Senaite and the add-ons are regularly updated with official releases every few months, it is recommended that the upgrades are done on the Test LIMS when they become available, and thoroughly tested before upgrading to Production.

   The releases are subjected to automated unit tests, however If you are running your own customised add-ons, test those particularly well as changes in the core might affect them.

  Upgrade scripts may include data conversions and those take longer to complete.  Formal upgrades can be run in the LIMS UI by admin users when prompted (after the code base was upgraded).


6. # Coding

   [The Anatomy of Plone](https://training.plone.org/mastering-plone-5/anatomy.html#the-anatomy-of-plone). Zope object-oriented database, [ZODB](https://zodb.org/en/latest/).

   [Plone Development and Customization intro](https://2022.training.plone.org/index.html)

   [Developing for Plone](https://5.docs.plone.org/develop/index.html)

   1. ## Convention

   See the [Senaite Contribution guide](https://github.com/senaite/senaite.core/blob/2.x/CONTRIBUTING.md). For your code:

* Include tests

* Follow [Plone's Python style guide](https://5.docs.plone.org/develop/styleguide/index.html)

* Write a descriptive and detailed summary

* Include whitespace and formatting changes in discrete commits

* Add a changelog entry in CHANGES.rst

  2. ## Code Entry Points

  Coding COAs is a good entry point. The COA's are page templates which are HTML files with some additional information, written in TAL, METAL and TALES.

     TAL (Template Attribute Language) is used for templating HTML using DB attributes and to include expressions. TALES (TAL Expression Syntax) defines the syntax and semantics of the expressions.

     METAL (Macro Expansion for TAL) enables the combination, re-use and nesting of templates.

     [Plone examples and training documentation](https://training.plone.org/mastering-plone-5/zpt.html). See a [Bika COA code example](#heading=h.tjhigkfofvq6).

  As a next step, coding instrument interfaces are recommended as many of them exist and can be used as templates. See [Interface code example](#heading=h.uwuqb5qv5tck)

7. # API

   Robust RESTful JSON API, [senaite.jsonapi](https://senaitejsonapi.readthedocs.io/). Create, Read and Update (CRU operations) through http GET/POST requests

8. # Tech Support

   The public Bika Slack channels are good for general gratis user and technical support. Prioritised professional report is billable and managed in private channels, ditto for issues in the [Bika Jira tracker](https://bika.atlassian.net/jira/dashboards/10000).  [Bika Issue Tracking](https://www.bikalims.org/manual/issue-tracking)

9. # Code Examples

   1. ## COA

   From [The Zope Book 1](https://zope.readthedocs.io/en/latest/zopebook/index.html), see [10. Using Zope Page Templates 1](https://zope.readthedocs.io/en/latest/zopebook/ZPT.html) and [27. Appendix C: Zope Page Templates Reference](https://zope.readthedocs.io/en/latest/zopebook/AppendixC.html)

   **Multi Sample COA, bika.coa:Multi.Pt**

   The COA Header with the lab logo ( [link to code](https://github.com/bikalims/bika.coa/blob/green_coa/src/bika/coa/reports/Multi.pt#L126)):
		 
   		<tal:same_verifier condition="python:view.is_verifier_unique(collection)[0]">
 		 <tal:render condition="python:True">
			<div class="row section-header no-gutters">
 		 	<!-- Header Table -->
  		  	<div class="col-6 text-left">
  		    	<!-- Header Left -->
  	    	<h1>Certificate of Analysis</h1>
    	  	<h1 name='coa_num' tal:content="python: coa_num"/>
    			</div>
    			<div class="col-6 text-right">
      		  	<!-- Header Right -->
        		<img class="logo image-fluid" style="object-fit:contain"
          	   	tal:attributes="src python:view.get_toolbar_logo();style styles/logo_styles"/>
  			</div>
			</div>
	 	 </tal:render>
 
  
A portion of the results table, here we display the result for the analysis services (code):

 		<tal:results repeat="model page">
		<tal:analyses tal:define="analyses python:view.get_analyses_by(model, title=row_data[0]);">
		<tal:analysis tal:repeat="analysis analyses">
                   <td colspan="2" class="text-center"
                        tal:define="result python:model.get_formatted_result(analysis);
                        verified python:analysis.review_state in ['published', 'verified']">
                      <span class="font-weight-normal"
                            tal:condition="python:result and verified"
                            tal:content="structure result" />
                      <span class="font-weight-normal"
                            tal:condition="python:result and not verified">-</span>
                      <span class="font-weight-normal"
                            tal:condition="not:result"></span>
                      <span tal:condition="python:model.is_out_of_range(analysis)"
                            class="font-weight-normal">
			<span class="outofrange text-danger">
                              <img tal:attributes="src python:report_images['outofrange_symbol_url']"/>
               		 </span>
                     	 </span>
                	</td> …

2. ## Instrument Interface

    For a simple flat results table from the instrument. e.g. for a Lactoscope, Samples per row, Analyses per column. Sample ID in column A,  column B is ignored, results per Analysis from column C to the right.
   
    Analyses are identified in the header row:

| Sample ID | Instrument Serial Number | Predicted Fat % m/m | Predicted Protein % m/m | Predicted Lactose % m/m | Predicted Solids % m/m |
| --------- | ------------------------ | ------------------- | ----------------------- | ----------------------- | ---------------------- |
| H23-04845 | 878929 | 5.22 | 3.88 | 4.73 | 14.71 |
| H23-04844 | 878929 | 5.25 | 3.91 | 4.74 | 14.78 |
| H23-04843 | 878929 | 5.35 | 3.94 | 4.7 | 14.9 |

  **Reading the imported file**

A CSV in this case. [Code](https://github.com/bikalims/senaite.instruments/blob/main/src/senaite/instruments/instruments/perkinelmer/lactoscope/lactoscopeh23061316.py#L148)

	def parse(self):
    	order = []
    	ext = splitext(self.infile.filename.lower())[-1]
    	if ext == ".xlsx":
        	order = (self.xlsx_to_csv, self.xls_to_csv)
    	elif ext == ".xls":
        	order = (self.xls_to_csv, self.xlsx_to_csv)
    	elif ext == ".csv":
        	self.csv_data = self.infile
    	if order:
        	for importer in order:
            	try:
                	self.csv_data = importer(
                    	infile=self.infile,
                    	worksheet=self.worksheet,
                    	delimiter=self.delimiter,
                	)
                	break
            	except SheetNotFound:
                	self.err("Sheet not found in workbook: %s" % self.worksheet)
                	return -1
            	except Exception as e:  # noqa
                	pass
        	else:
            	self.warn("Can't parse input file as XLS, XLSX, or CSV.")
            	return -1
    	stub = FileStub(file=self.csv_data, name=str(self.infile.filename))
    	self.csv_data = FileUpload(stub)

    	portal_type = ""
    	lines = self.csv_data.readlines()
    	reader = csv.DictReader(lines) 

   **Parsing a row and updating field values** 

The column headers, e.g. Predicted Fat % m/m, must correspond to Analysis Keywords on the Sample that are not allowed special characters, and these are split out of the header title before comparison. [Code](https://github.com/bikalims/senaite.instruments/blob/main/src/senaite/instruments/instruments/perkinelmer/lactoscope/lactoscopeh23061316.py#L217)

	def parse_ar_row(self, sample_id, row_nr, row):
    		ar = self.get_ar(sample_id)
    		parsed = {subn(r"[^\w\d\-_]*", "", k)[0]: v for k, v in row.items()}
    		# m°C
    		parsed = {subn(r'mm', '', k)[0]: v for k, v in parsed.items() if k}
    		parsed = {subn(r'mg100g', '', k)[0]: v for k, v in parsed.items() if k}
    		parsed = {subn(r'mS', '', k)[0]: v for k, v in parsed.items() if k}
    		parsed = {subn(r'mC', '', k)[0]: v for k, v in parsed.items() if k}
    		parsed = {subn(r'-', '', k)[0]: v for k, v in parsed.items() if k}

    	warnings = False
    	for kw, v in parsed.items():
        	if kw == "InstrumentSerialNumber":
            	continue
        	if kw == "DateTimeofAnalysis":
            	continue
        	if kw == "Lab":
            	continue
        	if kw == "ProductName":
            	continue
        	if kw == "Warning":
            	continue
        	if kw == "SampleID":
            	continue
        	try:
            	analysis = self.get_analysis(ar, kw)
            	if not analysis:
                	warnings = True
                	continue
            	keyword = analysis.getKeyword
            	new_dict = {
                	"Lab": parsed["Lab"],
                	"ProductName": parsed["ProductName"],
                	"InstrumentSerialNumber": parsed["InstrumentSerialNumber"],
                	"DateTimeofAnalysis": parsed["DateTimeofAnalysis"],
                	"SampleID": parsed["SampleID"],
                	kw: parsed[kw],
            	}
            	self.parse_row(row_nr, new_dict, keyword)
        	except Exception as e:
            	self.warn(
                	msg="Error getting analysis for '${kw}': ${sample_id}",
                	mapping={"kw": kw, "sample_id": sample_id},
                	numline=row_nr,
                	# line=str(row),
            	)
            	warnings = True
    	if warnings:
        	return 0

   **Using sample_id to find the Sample**

[Code](https://github.com/bikalims/senaite.instruments/blob/main/src/senaite/instruments/instruments/perkinelmer/lactoscope/lactoscopeh23061316.py#L402). Note: Analysis Request = Sample

	def get_ar(sample_id):
    	query = dict(portal_type="AnalysisRequest", getId=sample_id)
    	brains = api.search(query, CATALOG_ANALYSIS_REQUEST_LISTING)
    	try:
        	return api.get_object(brains[0])
    	except IndexError:
        	pass

