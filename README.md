# Rude Distributed Asset Management

Rude is a distributed asset management system for web projects.

- rude is a command-line driven
- rude is distributed
- rude plays nicely with git
- rude stores assets in CouchDB
- rude integrates with Amazon S3

Rude keeps your git repositories _lean_ and focused on the code.

## Problem Statement

Platform as a Service (PaaS) providers are increasingly popular nowadays.
They let programmers focus on development, and leave systems adminstration to dedicated companies.
Deployment is typically triggered by pushing a Git repository to the service provider.

Either you must choose to include your binary assets in your Git deployment,
or choose to host them elsewhere.
Most people will agree that the foremer solution is not really a solution.
Git retains all data indefinitely, and a repository can quickly bloat into multi-GBs
if you keep shuffling assets in and out.
The latter solution is very manual and error prone.

Rude is a reliable solution to the above problem.

## Installation

Installation of the Rude package is done via NPM:

    npm install -g rude

_Dependencies_:

- Rude uses a local [CouchDB](http://couchdb.apache.org/) database to store assets.
- **CouchDB must be installed separately**

### CouchDB on Mac OS X

We recommend using [homebrew](http://mxcl.github.com/homebrew/) to install CouchDB.

    brew install couchdb

Homebrew will guide you how to make CouchDB auto-start if you require.

## Assets

Rude manages assets for web projects.

A web project may contain a lot of images,
videos, gzipped downloads, etc.
These are inappropriate for storage in Git.
Git sees a project in its entirety, 
and shuffling large binary assets in and out will bloat your repository.
Rude lets you separate your assets, 
without losing the guaruntees afforded by Git.

Rude tracks your assets in a single inventory file that is lean,
and easy to track using Git.

## How it Works

Rude creates and manages a local file called your **manifest**.
The manifest is a list of all your asset names, and their SHA-1 hash.
Your program can ask the Rude module for an asset by name,
rude consults the manifest to lookup the assets hash,
then returns a fully-qualified URL pointing to the matching asset.

When running in development,
Rude will store and retreive assets from a local CouchDB database.
When your application is ready for production,
Rude can upload your entire set of assets to S3, or another web server.

Rude switches where it retreives assets from by consulting the environment variable `RUDE_PREFIX`.
When running locally, this is set to `http://localhost:5984`, your local CouchDB instance.

## Command Line Usage

> **Status** Alpha

Create a new local repository:

    $ rude init
    [OKAY] New local rude database initialized at http://localhost:5984/rude

Rude expects to be initialized at the root of your Git checkout directory.
The `init` command is idempotent, calling `init` on an existing project
will not overwrite an existing setup.

### Tracking Assets

> **Status** Alpha

Track assets by adding them to the Rude database,
assets are tracked by file name only:

    $ rude add path/to/file.ext
    [OKAY] New asset added to local database as 'file.ext'
    [INFO] Asset id 012927f794d2462ae6d5b0bcf1ee01bfb16571cc

Assets are stored in a single bucket; each asset requires a unique name.
Rude will error if you attempt to over write an existing asset.
You can explicitly force updating an asset with `-f`:

    $ rude add -f path/to/new/file.ext
    [OKAY] Existing asset 'file.ext' replaced
    [INFO] Asset id 7e792aa144129cec0c25b1e2bd55bee50d30b866

Each time you add or replace an asset, 
Rude will upate a file at the root of your repository called
`assets.json`.
Rude will also upload the asset to your local CouchDB database.

Your `assets.json` file _should_ be tracked by git,
and is explicitly formatted to make merging easier.
This file contains explicit pointers to all your assets,
each represented by a unique SHA-1 hash.
Since this file is tracked by git,
Rude will always return URLs that match the currently checked out branch.

- You should not track assets in Git.

### Rude Manifest

> **Status** Alpha

The Rude manifest is a single file at the root of your Git repository
called `assets.json`.
It is a JSON encoded mapping between asset names to their hash sum.

    {
        "helloworld": "7e792aa144129cec0c25b1e2bd55bee50d30b866", 
        "goodbye": "f4485f480d8e52aca885ddadbeed186bc2682500"
    }

This file _should_ be tracked in Git, in lieu of tracking the actual assets.

#### Managing the Assets File with Git

Rude makes no attempt to get into the revision control business.
Git does that very well, and it would be counter-productive to track revisions in two places.
What Rude really does is make it _very easy_ to track project assets with git.
Rather than adding the large [BLOBs](http://en.wikipedia.org/wiki/Binary_large_object) directly,
you just add pointers each asset.

> Rude is just a _pointer resolver_ mapping names to hashes, and hashes to fully-qualified URLs

Since the assets file is tracked by Git, it is formatted by Rude in a Git-friendly way.
Git tracks changes per-line, so Rude writes one asset per line.
The file is still valid JSON, just padded with useful whitespace.
A Git diff between revisions will _only_ show the assets that have changed.

## Project Usage

At runtime, your project asks Rude for assets by name,
and Rude returns their URLs.
The URLs returned depend on runtime configurations.
When running locally in development,
Rude returns URLs from your local CouchDB database.

### Node.js

> **Status** Alpha

Rude will auto-configure itself based on environment variables:

- Set `RUDE_PREFIX` to your assets server (default `http://localhost:5984/rude`)
- Set `RUDE_ASSETS_FILE` to your asset menifest (default `assets.json`)

Require Rude somewhere in your project:

    var rude = require('rude').config()
    var url  = rude('helloworld')
    
    console.log(url)

This will echo something like:

    http://localhost:5984/rude/7e792aa144129cec0c25b1e2bd55bee50d30b866/helloworld.jpg

#### Express

Rude integrates fairly well with [Express](https://npmjs.org/package/express) 
and [Jade](https://npmjs.org/package/jade).

Require Rude and attach it to your Express app:

    var app  = require('express')()
    var rude = require('rude').config()
    
    app.locals.rude = rude
    
    app.get('/', ...)

You can then call Rude from your Jade template wherever you require a link to your asset:

    body
        img.icon( src="#{rude('coming_soon.png')}", alt="Logo" )
        ...
    
A similar technique can be used in any template language that allows function calls.

#### Advanced

You can also pass in configurations programatically with:

    var prefix = 'https://server.com/assets'
    var config = JSON.parse(fs.readFileSync('myfile.json'))
    
    var rude = require('rude').config(prefix,config)
    console.log( rude('helloworld') )

A synchronous read _should_ be okay here, since it's only called on startup.

## Sharing

> **Status** Alpha

A system is not distributed unless without sharing between peers.
Rude uses CouchDB replication to synchronize repositories.

You can manage replication using CouchDB, or have Rude trigger replication manually:

    $ rude push https://some-server/rude

This will replicate your database changes to the remote server.
You can pull down changes from the server with 

    $ rude pull https://some-server/rude

## Production

> **Status** Alpha

When your project is ready for production,
Rude can upload your assets to their appropriate servers, or distribution networks.

### Amazon S3

You must set your AWS credentials using the
environment variables `AWS_S3_KEY` and `AWS_S3_SECRET`.

    $ rude publish s3://region/bucket-name
    [INFO] Uploading to S3
      √ Image1.png
      √ Image2.png
    [DONE] Use environment variable RUDE_PREFIX=https://bucket-name.s3-us-west-2.amazonaws.com

Region _must_ be one of the following:

- `us-standard` (US Standard)
- `us-west-2` (US West Oregon)
- `us-west-1` (US West Northern California)
- `eu-west-1` (EU Ireland)
- `ap-southeast-1` (Asia Pacific Singapore)
- `ap-northeast-1` (Asia Pacific Tokyo)
- `sa-east-1` (South America Sao Paulo)

Further more, your region _must_ agree with the region where your bucket has been created.

### SSH

SSH publishing requires SSH keys, run:

    $ rude publish git://server.com:path/to/assets

## Command Line Tips

Get the URL of an asset e.g. `asset.png` 

    rude list | grep asset.png | awk -s '{print $4}'

# Todo

- set explicit content-types when adding assets
- better errors when trying to re-add an existing asset (give the other name)
- fine-grained replication between CouchDBs
- add ability to format hash-id via `RUDE_PREFIX`
    - e.g. `0000-0000`, `0000/0000`
    - splitting the hashes into multiple subdirectories will make S3 management more sane

# License

Copyright (C) 2012 Jacob Groundwater

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
