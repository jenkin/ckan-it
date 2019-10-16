# CKAN-IT - The Italian distribution

[![Join the #pdnd-ckan channel](https://img.shields.io/badge/Slack%20channel-%23pdnd--ckan-blue.svg?logo=slack)](https://developersitalia.slack.com/messages/CMX9ZDPK3)
[![Get invited](https://slack.developers.italia.it/badge.svg)](https://slack.developers.italia.it/)
[![PDND/DAF on forum.italia.it](https://img.shields.io/badge/Forum-PDND-blue.svg)](https://forum.italia.it/c/daf)

[CKAN](https://ckan.org/) is a powerful data management system that makes data accessible – by providing tools to streamline publishing, sharing, finding and using data.

This project provides everything you need to run CKAN plus a set of extensions for supporting Italian open data in a set of Docker images.

## Tools references

The tools used in this repository are

* [CKAN](https://ckan.org/)
* [Docker](https://www.docker.com/)

## Main components

* **CKAN** version 2.6.8 with the extensions listed at the end of this document.

* **Solr** version 6.2 packaged for CKAN and with some customizations (see [italia/ckan-it-solr](https://github.com/italia/ckan-it-solr)).

* **PostgreSQL** version 10.1, modified for CKAN (see https://hub.docker.com/r/ckan/postgresql/tags, tag *latest*).

* **Redis** version 5.0.5, pulled in as a dependency from its [official Docker repository](https://hub.docker.com/_/redis).

## How to run CKAN-IT

In this repository, CKAN and its related tools are redistributed as a set of Docker containers interacting with one each other.

The `Dockerfile` and the `docker-compose.yml` files are in the root of this repository.

> NOTE: the `docker-compose.yml` file sets different environment variables that could be used to adapt and customized many platform functionalities, read more in "Environment variables" section below.

If you want a CKAN-IT instance up and running in a couple of minutes, follow these steps.

1. Create and enter an empty folder: `mkdir ckan-it && cd ckan-it/` (or use the name you prefer)
2. Download the `docker-compose.yml` from [here](https://raw.githubusercontent.com/italia/ckan-it/master/docker-compose.yml)
3. Pull and run all containers: `docker-compose up -d`

After a while you can open the CKAN-IT home [http://localhost:5000](http://localhost:5000) and login with the provided credentials.
You can follow the log stream running `docker-compose logs -f` (then ctrl+c to exit).

The following default credentials can be used to access the portal (you should change them after the first login).

```
Username: ckanadmin
Password: ckanpassword
```

If you only want to run a CKAN-IT instance and use it to manage and publish your own data, you can stop here.
In a production environment you can install and setup a proxy server in front of CKAN-IT with https support.

> WARNING: all data are stored in [Docker named volumes](https://success.docker.com/article/different-types-of-volumes)! In a production environment you should mount these volumes on local folders updating the [docker-compose configuration](https://docs.docker.com/compose/compose-file/compose-file-v2/#volumes) accordingly.

To bring down and remove the containers use `docker-compose down`.

## How to build and test CKAN-IT

If you want to build local images instead of pull them from Dockerhub, ie. for testing pourpose, you need some extra steps.

1. Clone this repo: `git clone https://github.com/italia/ckan-it.git` (if you want to clone the repo in a folder other than `ckan-it/` add the name you want after the previous command, ie. `git clone https://github.com/italia/ckan-it.git my_custom_folder`)
2. Enter the created folder: `cd ckan-it/` (or the name you have chosen in previous step, ie. `cd my_custom_folder/`)
3. Change working branch if needed: `git checkout branch-name`
4. Initialize submodules: `git submodule update --init --recursive`
5. Build images: `docker-compose -f docker-compose.yml -f docker-compose.build.yml build`
6. Run all containers using built images: `docker-compose up -d` (if you want to check logs run `docker-compose logs -f`)

## CKAN-IT harvesting (optional)

CKAN-IT can acts also as an aggregator of data sources, harvesting dataset metadata from external sources.
If you want to import data from external sources, follow these additional steps.

> WARNING: note that if `CKAN_HARVEST` variable in docker-compose is not set to `"true"` no organizations and sources are initially loaded (see below), so you must use the GUI to manually add new organizations and sources of your choice before next steps.

1. Browse to [http://localhost:5000/harvest](http://localhost:5000/harvest) to check all available sources or add new sources
2. Identify the name of the CKAN container with `docker container ls` (ie. `italia-ckan-it`) and run the following command: `docker exec -it italia-ckan-it /ckan-harvest.sh`

You can see logs during harvesting import with following command: `docker-compose logs -f`.
You can find more logs in `/var/log/ckan` folder inside the container.

### Run CKAN-IT periodic harvesting

Schedule a CRON job on the host machine to run the `/ckan-harvest.sh` script at the root of the file system of the CKAN container.

How to do this really depends on how you run the containers. When running containers with docker-compose for instance we did this by getting the container id and using `docker-exec` to run a command inside the container, as follows:
`docker exec -it italia-ckan-it /ckan-harvest.sh 2>&1 /var/log/periodic-harvest.out`

So you can schedule a periodic run of the above script, ie. every hour, with CRON on the host machine and save logs.

### Pre-load all organizations and sources

The [italia/ckan-it-harvesters](https://github.com/italia/ckan-it-harvesters) repository contains all sources harvested by the national catalog of the [Piattaforma Digitale Nazionale Dati (PDND) - previously DAF](https://pdnd.italia.it/).

If you want to clone it in your environment you must follow some additional steps:

1. Check if `data/init/harvesters` folder exists, if not add it running `git submodule add https://github.com/italia/ckan-it-harvesters data/init/harvesters`
2. Add `CKAN_HARVEST="true"` environment variable to the ckan service in `docker-compose.yml` (ie. see `docker-compose.harvest.yml`)
3. Run containers: `docker-compose up -d`
3. Wait for organizations and harvest sources loading, then run `docker exec -it italia-ckan-it /ckan-harvest.sh`
4. Follow previous section to setup a periodic harvesting

## Environment variables

The following environment variables are mandatory and should be set in order to deploy CKAN-IT. The `docker-compose.yml` file in this repository applies some exemplar values, to be used for demos and local tests.

### General variables

* CKAN_DEBUG *(format: {"true"|"false"})* - Whether to activate or not the debug log messages. It should always be false for production environments.

* CKAN_HARVEST *(format: {"true"|"false"})* - Whether to activate or not the built-in harvesters. It should be false if you want to only build your own catalog and not harvest external sources.

* CKAN_SITE_URL - The base URL of your CKAN-IT deployment.

* CKAN_ADMIN_EMAIL - The email address of the local admin user.

* CKAN_ADMIN_USERNAME - The user name of the local admin user.

* CKAN_ADMIN_PASSWORD - The password of the local admin user.

### PostgreSQL variables

* CKAN_DB_HOST - The host name of the CKAN PostgreSQL database.

* CKAN_DB_PORT - The port of the CKAN PostgreSQL database.

* CKAN_DB_USER - The user name of the CKAN PostgreSQL database.

* PGPASSWORD - The password of the CKAN PostgreSQL database.

* CKAN_SQLALCHEMY_URL *(format: {postgresql://{CKAN_DB_USER}:{PGPASSWORD}@{CKAN_DB_HOST}:{CKAN_DB_PORT}/})* - The connection string to your PostgreSQL database.

### Redis variables

* CKAN_REDIS_HOST - The host name of your Redis service.

* CKAN_REDIS_PORT - The port of your Redis service.

* CKAN_REDIS_URL *(format: redis://{CKAN_REDIS_HOST}:/{CKAN_REDIS_PORT})* - The full address of the Redis service.

### Solr variables

* CKAN_SOLR_HOST - The host name of the Solr service.

* CKAN_SOLR_PORT - The port of the Solr service.

* CKAN_SOLR_URL *(format: http://{CKAN_SOLR_HOST}:{CKAN_SOLR_PORT}/solr/ckan)* - The full URL of the Solr service.

## CKAN 2.6.8 extensions reference

  - stats
  - view
    - text_view
    - image_view
    - recline_view
  - datastore
  - [spatial](https://github.com/italia/ckanext-spatial/) (tag 2.6.8-2)
    - spatial_metadata
    - spatial_query
  - [harvest](https://github.com/ckan/ckanext-harvest/) (tag v1.1.1)
    - ckan_harvester
  - [multilang](https://github.com/italia/ckanext-multilang/) (tag 2.6.8-2)
    - multilang_harvester
  - [dcat](https://github.com/ckan/ckanext-dcat/) (tag v0.0.9)
    - dcat_rdf_harvester
    - dcat_json_harvester
    - dcat_json_interface
  - [dcatapit](https://github.com/italia/ckanext-dcatapit/) (tag 2.6.8-2)
    - dcatapit_pkg
    - dcatapit_org
    - dcatapit_config
    - dcatapit_harvester
    - dcatapit_csw_harvester
    - dcatapit_harvest_list
    - dcatapit_subcatalog_facets

## How to contribute

Contributions are welcome. Feel free to open issues and submit a pull request at any time!
