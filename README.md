# Pit Distributed Asset Management

Pit is a distributed asset management system for games projects.

- pit is a command-line driven
- pit is distributed
- pit plays nicely with git
- pit stores assets in CouchDB

Pit keeps your git repositories _lean_ and focused on the code.

## Problem Statement

Games tend to have large binary assets such as pictures, videos, textures, sounds and more.
Either you must choose to include your binary assets in your Git repository,
or choose to host them elsewhere.
Most people will agree that the foremer solution is not really a solution.
Git retains all data indefinitely, and a repository can quickly bloat into multi-GBs
if you keep shuffling assets in and out.
The latter solution is very manual and error prone.

Pit is a reliable solution to the above problem.

## Installation

Installation of the Pit package is done via NPM:

    npm install -g pit

_Dependencies_:

- Pit uses a local [CouchDB](http://couchdb.apache.org/) database to store assets.
- **CouchDB must be installed separately**

### CouchDB on Mac OS X

We recommend using [homebrew](http://mxcl.github.com/homebrew/) to install CouchDB.

    brew install couchdb

Homebrew will guide you how to make CouchDB auto-start if you require.

## Assets

Pit manages assets for game projects.

A game project may contain a lot of images,
videos, sounds, etc.
These are inappropriate for storage in Git.
Git sees a project in its entirety, 
and shuffling large binary assets in and out will bloat your repository.
Pit lets you separate your assets, 
without losing the guaruntees afforded by Git.

Pit tracks your assets in a single inventory file that is lean,
and easy to track using Git.

## How it Works

Pit creates and manages a local file called your **manifest**.
The manifest is a list of all your asset names and paths, and their SHA-1 hash.
Pit will store and retreive assets from a CouchDB database.
It integrate well with git flow such as checking-out a branch will trigger checking-out of the assets as they were in that commit.

You can decide your CouchDB configuration which can range from simple single database to distributed locally-available databases.
Out-of-the-box Pit uses your local CouchDB instance at `http://localhost:5984`

## Command Line Usage

> **Status** Alpha

Create a new local repository:

    $ pit init
    [OKAY] New local pit database initialized at http://localhost:5984/pit

Pit expects to be initialized at the root of your Git checkout directory.
The `init` command is idempotent, calling `init` on an existing project
will not overwrite an existing setup.

### Tracking Assets

> **Status** Alpha

Track assets by adding them to the Pit database,
assets are tracked by file name only:

    $ pit add path/to/file.ext
    [OKAY] New asset added to local database as 'file.ext'
    [INFO] Asset id 012927f794d2462ae6d5b0bcf1ee01bfb16571cc

Assets are stored using their relative path to the git root.
If you add an asset that is already exist (meaning having the same hash as exist asset) it will not be added again to CouchDB and will only add another entry to your **manifest** file.

Each time you add an asset, 
Pit will upate a file at the root of your repository called
`assets.json`.
Pit will also upload the asset to your local CouchDB database.

Your `assets.json` file _MUST_ be tracked by git,
and is explicitly formatted to make merging easier.
This file contains explicit pointers to all your assets,
each represented by a unique SHA-1 hash.

- You should not track assets in Git.

## Checking-out Assets

> **Status** Alpha

**NOT IMPLEMENTED YET**
When your checkout commit from git pit will be invoked using git-hook and will checkout the assets define in the **manifest** file.
It will remove (delete) assets that exist in your working directory but not in your **manifest** file and will write the assets as they were in that specific commit.

### Pit Manifest

> **Status** Alpha

The Pit manifest is a single file at the root of your Git repository
called `assets.json`.
It is a JSON encoded mapping between asset relative path to their hash sum.

    {
        "helloworld": "7e792aa144129cec0c25b1e2bd55bee50d30b866", 
        "dir1/dir2/goodbye": "f4485f480d8e52aca885ddadbeed186bc2682500"
    }

This file _MUST_ be tracked in Git, in lieu of tracking the actual assets.

#### Managing the Assets File with Git

Pit makes no attempt to get into the revision control business.
Git does that very well, and it would be counter-productive to track revisions in two places.
What Pit really does is make it _very easy_ to track project assets with git.
Rather than adding the large [BLOBs](http://en.wikipedia.org/wiki/Binary_large_object) directly,
you just add pointers each asset.

> Pit is just a _pointer resolver_ mapping names to hashes, and hashes to binary files

Since the assets file is tracked by Git, it is formatted by Pit in a Git-friendly way.
Git tracks changes per-line, so Pit writes one asset per line.
The file is still valid JSON, just padded with useful whitespace.
A Git diff between revisions will _only_ show the assets that have changed.

## Sharing

> **Status** Alpha

A system is not distributed unless without sharing between peers.
Pit uses CouchDB replication to synchronize repositories.

You can manage replication using CouchDB, or have Pit trigger replication manually:

    $ pit push https://some-server/pit

This will replicate your database changes to the remote server.
You can pull down changes from the server with 

    $ pit pull https://some-server/pit

## Command Line Tips

Get the URL of an asset e.g. `asset.png` 

    pit list | grep asset.png | awk -s '{print $4}'

# Todo

- set explicit content-types when adding assets
- better errors when trying to re-add an existing asset (give the other name)
- fine-grained replication between CouchDBs
- add ability to format hash-id via `RUDE_PREFIX`
    - e.g. `0000-0000`, `0000/0000`
    - splitting the hashes into multiple subdirectories will make S3 management more sane

# Relation to Rude

Pit was forked from rude project. Rude already did most of the heavy-lifting of assets management and pit only shift it towrds game projects that need the assets locally and be tracked using relative path as apposed to only names.

# License

Copyright (C) 2012 Jacob Groundwater
Copyright (C) 2014 Ido Ran

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
