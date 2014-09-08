# Nodyn.io Website

To build the site:

    $ cd _harp
    $ harp compile ./ ../

This will generate the website into the current directory.

To run the staging server for local development.

    $ cd _harp
    $ harp server

Then browse to http://localhost:9000

## Staging and production servers

The staging server is http://staging.nodyn.io. When changes are pushed to the
`master` branch of this repository, the staging site is regenerated.

When changes are pushed to the `production` branch of this repository, the 
production website is regenerated. To easily push staging content to production
run `$ git push origin master:production`.
