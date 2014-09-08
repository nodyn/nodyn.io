# http://nodyn.io

This is the repository for the Nodyn website http://nodyn.io

## Generating the site

    $ cd _harp
    $ harp compile ./ ../

## Running the harp server for development

    $ cd _harp
    $ harp server

## Pushing changes to the staging server

    $ git push origin master

This will cause the staging CI server to build the site and publish it at http://staging.nodyn.io

## Pushing changes to production

    $ git push origin master:production

This will cause the production CI server to build the site from the `production` branch and publish it at http://nodyn.io
