# Rude Distributed Asset Management

Rude is a distributed asset management system for web projects.

- rude is a command-line driven
- rude is distributed
- rude plays nicely with git

Rude keeps your git repositories _lean_ and focused on the code.

## Installation

Installation of the Rude package is done via NPM:

    npm install -g rude

_Dependencies_:

- Rude uses a local [CouchDB](http://couchdb.apache.org/) database to back its collection of assets.
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

## Usage

> **Status** Alpha

Create a new local repository:

    $ rude init
    [OKAY] New local rude database initialized at http://localhost:5984/rude

Rude expects to be initialized at the root of your Git checkout directory.

### Tracking Assets

> **Status** Alpha

Track assets by adding them to the Rude database:

    $ rude add path/to/file.ext
    [OKAY] New asset added to local database as 'file.ext'
    [INFO] Asset id 012927f794d2462ae6d5b0bcf1ee01bfb16571cc

Assets are stored in a single bucket; each asset requires a unique name.
Rude will error if you attempt to over write an existing asset.

    $ rude add -f path/to/new/file.ext
    [OKAY] Existing asset 'file.ext' replaced
    [INFO] Asset id 7e792aa144129cec0c25b1e2bd55bee50d30b866

Rude will automatically add the asset to your `.gitignore` file,
and update the Rude manifest.

### Rude Manifest

> **Status** Alpha

The Rude manifest is a single file at the root of your Git repository
called `assets.json`.
It is a simple JSON encoded mapping matching asset names to their hash sums.

    {
        "helloworld": "7e792aa144129cec0c25b1e2bd55bee50d30b866", 
        "goodbye": "f4485f480d8e52aca885ddadbeed186bc2682500"
    }

This file _should_ be tracked in Git, in lieu of tracking the actual assets.

## Usage

Rude works by returning mapping asset names to URLs in your projects and templates.
The URLs returned depend on runtime configurations.
When running locally in development,
Rude returns URLs from your local CouchDB database.

### Node.js

> **Status** Alpha

Require Rude somewhere in your project:

    var rude = require('rude')
    var rude('helloworld')
    
    console.log(rude)

This will echo something like:

    http://localhost:5984/rude/7e792aa144129cec0c25b1e2bd55bee50d30b866/helloworld.jpg

## Sharing

> **Status** Planned

A system is not distributed unless without sharing between peers.
Rude uses CouchDB replication to synchronize repositories.

You can manage replication using CouchDB, or have Rude trigger replication manually:

    $ rude push https://some-server/rude

This will replicate your database changes to the remote server.
You can pull down changes from the server with 

    $ rude pull https://some-server/rude

## Production

> **Status** Planned

When your project is ready for production,
Rude can upload your assets to their appropriate servers, or distribution networks.

### Amazon S3

    $ rude publish s3
    
### WebDAV

    $ rude publish http://server/dav

### SSH

    $ rude publish user@host:/path/to/web










