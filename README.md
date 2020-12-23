# IPA

IPA stands for InfoPasażer Archives. InfoPasażer is a site maintained by PKP (National Polish Railways)
that shows information about all trains (even not owned by PKP), eg. their current position and delay. The site is great, but trains
disappear as soon as they reach their destinations; hence the idea of archiving it.

This project consists of:

- library for obtaining data from InfoPasażer: `station_api.py` `train_api.py` 
- script for updating the data and storing it in the database: `src` directory
- simple REST API: `api` directory
- simple web frontend using that API: `html` directory

## Requirements

Python 3 is used as the main language with BeautifulSoup and MySQL connector libraries. Data is stored in MariaDB.
Frontend is served by Apache 2.4 using mod_wsgi and Flask.

## Using as a CLI tool

Mainly for testing purpose, there are scripts for printing the contents directly from InfoPasażer to console.

### Printing station details

Go to infopasazer.intercity.pl and find interesting station, then copy `stationid` value from URL and run:

    ./station_api.py stationid

### Printing train details

Go to infopasazer.intercity.pl and find interesting train, then copy `trainid` value from URL and run:

    ./train_api.py trainid

## Storing the data

### Creating database

Create new MySQL database and edit `ipa_config.py` with credentials. Then create database schema:

    ./ipa.py create_schema

### Updating train info

This will go to every stations defined in `ipa_config.py` and fetch data for every train available:

    ./ipa.py update_trains

Please, do not abuse InfoPasażer site by executing this too often! To keep the data up to date add
this line to crontab.

## Checking current data

After gathering some data, you can use this commands to examine them.

### Printing trains

This will list all train numbers (+ names):

    ./ipa.py print_trains

### Printing train history

This will print `2700/1 SIEMIRADZKI` train history:

    ./ipa.py print_train '2700/1 SIEMIRADZKI'

## Setting up web frontend

Edit `api/flaskapp.wsgi` file and set system paths to `api` and `src` directories, and also secret key.
Then create Apache page from this template:

    <VirtualHost *:80>
        ServerName ipa.lovethosetrains.com
        DocumentRoot /opt/ipa/html

        <Directory /opt/ipa/html>
            Require all granted
            AllowOverride All
        </Directory>

        WSGIScriptAlias /api /opt/ipa/api/flaskapp.wsgi
        <Directory /opt/ipa/api>
            Require all granted
            AllowOverride All
            AddOutputFilterByType DEFLATE application/json
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/ipa-error.log
        CustomLog ${APACHE_LOG_DIR}/ipa-access.log combined
    </VirtualHost>

