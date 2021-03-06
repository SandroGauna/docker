About this Repo
======

This is a modified repo (with brazilian localization modules) of the official Docker image for [Odoo](https://registry.hub.docker.com/_/odoo/). See the Hub page for the full readme on how to use the Docker image and for information regarding contributing and issues.

The full readme is generated over in [docker-library/docs](https://github.com/docker-library/docs), specifically in [docker-library/docs/odoo](https://github.com/docker-library/docs/tree/master/odoo).


How to use this image
============

This image requires a running PostgreSQL server.

Start a PostgreSQL server
--------
`$ docker run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo --name db postgres:9.4`

Start an Odoo instance
--------
`$ docker run -p 8069:8069 --name odoo --link db:db -t joaopaulosouza/odoo-docker:8.0`
