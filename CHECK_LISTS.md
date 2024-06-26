Here are some check lists I use to develop and maintain E-Maj.
=============================================================

**Produce a change** into the **emaj extension** or related components
--------------------------------------------------------------

- specify the change
- code the change (step by step)
	- data structures
	- functions
	- other components
- test the change
	- adjust test scenari, if needed
	- tools/regress, with options t + m + p
	- tools/copytoexpected.sh when test result files can be considered as reference
- emaj upgrade procedure
	- data structures
	- functions : tools/sync_fct_in_upgrade_script.pl
	- test with tools/regress, with options T + VWX
- check error messages coverity : tools/check_error_messages.pl
- update the CHANGES.md file
- update the documentation
	- odt files, in French and English
	- rst files in docs/fr and docs/en + make html + check local output results
- commit the change
	- git status + git diff
	- git commit -am "message"
	- git push
- emaj_web (see Emaj_web checklist)
- update the slides, if needed


**Produce a change** into **Emaj_web**
----------------------------------

- specify the change
- code the change into the test environment (/var/www/html/emaj_web)
- test the change with the web interface
- check the language files consistency: ./tools/checkLang.pl
- synchronize the emaj_web repo : ~/proj/divers/tools/syncEmajWeb.sh
- commit the change
	- cd emajweb repo root directory (~/proj/emaj_web)
	- git status + git diff
	- git commit -am "message"
	- git push

Setup a new **Postgres major or minor version**
-------------------------------------------

- download and copy the postgres sources .tar.gz file into ~/pg
- decompress
	- cd pg
	- tar -xzf postgresql-x.y.z.tar.gz or tar -xjf postgresql-x.y.z.tar.bz2
	- cd postgresql-x.y.z
- compile
	- ./configure --with-libxml --prefix=/usr/local/pgxyz
	- make
	- make check
	- cd contrib
	- make
	- cd ..
	- sudo make install
	- cd contrib
	- sudo make install
- create/update the symbolic links towards the version
	- ln -s /usr/local/pgxy pgx
	- ln -s ~/pg/postgresql-x.y postgresql-x
- (re)create the new cluster
	- if a new major version
		- modify tools/emaj_tools.profile to add the new version to the supported versions list
		- modify tools/emaj_tools.env to add the new version into the supported versions list and setup the constants for this version
	- tools/create_cluster.sh <major version number>
- adjust tools
	- adjust the tools/regress.sh script
	- adjust the tools/copy2Expected.sh script
- execute non regression tests
	- cd test
	- mkdir <major version number>
	- cp -r <previous version number>/expected/<major version number>/expected
	- ln -s ../sql/ major version number>/sql
	- rebuild the regression tests result files for the new version
		- tools/regress.sh
		- adjust the code if needed (see previous checks-lists)
		- documentation: adjust the compatibility version matrix


Create a new **E-Maj version**
--------------------------

- Update the documentations MàJ doc pour upgrade 
	- odt and rst files in French and in English
		- odt: version number (page 1 and footer)
		- rst: version number in conf.py
		- files list in §8.2
		- add comment about the upgrade procedure, if needed
		- refresh the Emaj_web screenshots
- Update the slides, if needed
	- .odp files in French and English
- git commit -am 'Tag the documentation to prepare the new coming version.'
- test all distributed scripts
	- createdb -p 5414 -h localhost -U postgres testemaj
	- psql ... postgres
		- create extension emaj cascade;
		- \i sql/emaj_prepare_emaj_web_test.sql
		- \i sql/emaj_demo.sql
			- SELECT emaj.emaj_demo_cleanup();
		- \i sql/emaj_prepare_parallel_rollback_test.sql
			\! ./client/emajParallelRollback.php -g 'emaj parallel rollback test group' -m BEFORE_PROC_2 -s 3 -v ...
		- \i sql/emaj_upgrade_after_postgres_upgrade.sql
		- \i sql/emaj_uninstall.sql
	- dropdb -p 5414 -h localhost -U postgres testemaj
- check and compress the git environment
	- git status
	- git checkout master
	- delete useless branch if any: git branch -D ...
	- git gc
- save the emaj environment
- create the new version
	- cd ~/proj
	- sh emaj/tools/create_version_prepare.sh 'x.y.z' '<version date in yyyy-mmm-dd format>'
	- cd emaj
		- check the environment
		- git commit -a -m 'Setup the new x.y.z version.'
		- git tag -a v<x.y.z> -m 'Version x.y.z.'
	- cd ..
	- sh emaj/tools/create_version_complete.sh 'x.y.z'
	- cd emaj-x.y.z
		- tools/regress.sh : for all functions, then tools/copy2expected.sh
		- git commit -a -m 'In emaj-x.y.z version, adjust the content and prepare the environment for potential development.'
	- cd ..
- prepare the new devel environment
	- remove the n-2 version of the extension install script: git rm sql/emaj--x.y-2.z.sql
	- rerun all tests
		- tools/regress.sh
		- tools/copyToExpected.sh
	- commit
		git commit -am "Prepare a new emaj--x.y.z--devel.sql empty upgrade script and adjust the regression test environment for the next release."
- mark the new version in the other repos
	- cd ../emaj_doc
		- git commit -a -m 'Setup the new x.y.z version'
	- cd ../emaj_web
		- modify the version number at the beginning of the libraries/versions.inc.php file in proj/www_emajweb
		- ~/proj/divers/tools/syncEmajWeb.sh
		- git sur ~/proj/emaj_web
			- git commit -a -m 'Setup the new x.y.z version'
			- git gc
			- git tag -a vx.y.z -m 'Version x.y.z.'
- synchronize the 3 emaj, emaj_doc and emaj_web github repo
	- git push
	- git push --tags
	- check that the tag is visible on the github repos
- create the new version on the readthedocs site for both languages
	- https://readthedocs.org/projects/emaj -> sheet admin -> menu versions -> activate the new version
- review issues on github
- pgxn : check and upload the tar file to create the new version
	- https://manager.pgxn.org
- save the entire environment of the new version
- publish the announcement on pgsql-announce
