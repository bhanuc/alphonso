# alphonso

Your own private offline npm registry and backup server.

Overview
--------

This project allows you to have your own npm registry that caches your packages on to the 
localdisk/S3 and saves your time and data, whenever you redownload the same package with same version. 

This server works with the necessary `npm` commands just like the npmjs.org registry. You can use it to not worry about npm going down or to store your private packages. It performs much faster than npmjs.org and can even be matched with a CDN like Cloudfront to be fast globally.

Rather than trying to copy all the data in npm, this acts more like a proxy. While npm is up, it will cache package data onto the local disk. If npm goes down, it will deliver whatever is available in the cache. This means it won't be a fully comprehensive backup of npm, but you will be able to access anything you accessed before.

This project is an independent fork of [elephant](https://github.com/dickeyxxx/elephant) which was inspired from [sinopia](https://github.com/rlidwka/sinopia). This came out of the need of running a local npm cache that was independent of the internet. This fork will remain independent due to conflicts of goals of the project, for more details see this [issue](https://github.com/dickeyxxx/elephant/issues/9#issuecomment-196822043)

This is also a [12 Factor](http://12factor.net/) app to make it easy to host on a PaaS like Heroku or in a custom Ansible/Chef/Puppet cluster.

Setup
-----

If `REDIS_URL` is set (optional) redis will be used to cache package data.

The easiest way to set this up is with the Heroku button:

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

Alternatively, you can set it up by cloning this repo:

```
$ git clone https://github.com/bhanuc/alphonso
$ cd alphonso
$ npm install
$ npm start
```

Either way, your registry is now setup and you should be able to test it by updating the packages with it:

```
$ npm update --registry http://urltomyregistry
```

See below for how to enable authorization and `npm publish`.

How it works
------------

Essentially the goal of the project is to quickly deliver current npm data even when npm is offline.  In npm there are 2 main types of requests: package metadata and tarballs.

Package metadata mostly contains what versions of a package are available. These cannot be cached for very long since the package can be updated. By default, it is cached for 60 seconds. You can modify this with `CACHE_PACKAGE_TTL`. Etags are also supported and cached to further speed up access.

The tarballs are the actual code and never change once they are uploaded (though they can be removed via unpublishing). These are downloaded one time from npmjs.org per package and version, stored in a specified folder/location. 

In the event npmjs.org is offline, alphonso will use the most recent package metadata that was requested from npmjs.org until it comes back online.

npm commands supported
----------------------

* `npm install`
* `npm update`
* `npm login`
* `npm whoami`
* `npm publish`

Authentication
--------------

Alphonso uses an htpasswd file in local disk/S3 for authentication and stores tokens in local disk. 
To set this up, first create an htpasswd file. Then upload it to `/htpasswd` in the npm directory/s3 main directory.

```
$ htpasswd -nB YOURUSERNAME >> ./htpasswd
```

Then you can login with npm. Note that the email is ignored by the server, but the CLI will force you to add one.

```
$ npm login --registry http://myregistry
Username: dickeyxxx
Password:
Email: (this IS public) jeff@heroku.com
$ npm whoami --registry http://myregistry
dickeyxxx
```

This stores the credentials in `~/.npmrc`. You can now use `npm publish` to publish packages.

**NOTE**: Because the original use-case for having private packages was a little strange, right now you need to be authenticated to upload a private package, but once they are in the registry anyone can install them (but they would have to know the name of it). Comment open a new issue on the project if you'd like to see better functionality around this.

**EXTRA:NOTE**: Test-suite is not yet added but will be added soon.
