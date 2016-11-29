# Non-native Debian Packaging

Here we create and maintain a nonnative debian package `slm` from scratch. The
same commands also apply to existing nonnative debian packages. As an example we
demonstrate patching an existing debian package `hello`.

## General setup

Add the following to your ~./profile

```
export DEBEMAIL="your.email.address@example.org"
export DEBFULLNAME="Firstname Lastname"
```
and source it.

```
$ source ~/.profile
```

## Nonnative debian packaging from scratch

Import the very first upstream tarball.

```
$ mkdir packages && cd packages
$ apt-get install build-essential debmake debhelper git-buildpackage pristine-tar
$ curl -sL http://<host>:<port>/<user>/slm/archive/v0.1.0.tar.gz > slm-0.1.0.tar.gz
$ mkdir slm && cd slm
$ git init
$ gbp import-orig --pristine-tar -u 0.1.0 --filter=.drone.yml ../slm-0.1.0.tar.gz
$ git branch
* master
  pristine-tar
  upstream
```

Create and modify debian/ directory

```
$ debmake -u 0.1.0 # upstream version
```

Add a `.drone.yml` which builds the debian package. Note that we filtered out the
`.drone.yml` in the command above just in case the upstream might be building with
Drone CI as well. Example `.drone.yml` for building the debian package is given
below.

```
$ cat .drone.yml 
build:
  package:
    image: pardus/pardus-package
    volumes:
      - /tmp/packages:/packages
    commands:
      - echo "TODO lintian -i /packages/$(basename $DRONE_REPO)/*.changes"
      - apt-get update
      - apt-get install build-essential equivs devscripts
      - mkdir -p build-area
      - cd build-area
      - dpkg-source -x /packages/$(basename $DRONE_REPO)/*.dsc build-$(basename $DRONE_REPO)
      - cd build-$(basename $DRONE_REPO)
      - mk-build-deps -i -r
      - dpkg-buildpackage -b -uc
      - cd ..
      - cp  -vt /packages/$(basename $DRONE_REPO) *.udeb *.deb *.changes 2>/dev/null || true
  test:
    image: pardus/pardus-test
    volumes:
      - /tmp/packages:/packages
    commands:
      - apt-get update
      - apt-get install /packages/$(basename $DRONE_REPO)/*.deb
  deploy:
    image: pardus/pardus-test
    volumes:
      - /tmp/packages:/packages
    commands:
      - apt-get update
      - echo "TODO deployment container (pardus/pardus-deploy) is yet to be prepared"
      - ls -al /packages/$(basename $DRONE_REPO)
      - echo "deployment commands go here"
branches:
  - master
```

Make debian ignore `.drone.yml`

```
$ echo 'extend-diff-ignore = ".drone.yml"' >> debian/source/local-options
```

You may want to add a `debian/watch` file. Here is an example watch file for Gogs.
```
version=3
opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/<project>-$1\.tar\.gz/ \
  http://<host>:<port>/<user>/slm/releases .*/v?(\d\S*)\.tar\.gz
```

See [this][1] complete guide for watch files.

Create a remote repo `slm` on Gogs and activate it on Drone CI.

Commit your changes and update debian changelog.

```
$ git add .
$ git commit -m "Initial package"
$ gbp dch -R -c
```
Build the source package under `/tmp/build-area`

```
$ rm -rf /tmp/build-area
$ gbp buildpackage --git-export-dir=/tmp/build-area --git-pristine-tar --git-pristine-tar-commit -S -us -uc
```

`rsync` the source package. Also creates `/tmp/packages/` directory on your build server.

```
$ d=/tmp/packages/$(basename $(git rev-parse --show-toplevel))/ && rsync -av --rsync-path="rm -rf $d && mkdir -p $d && rsync" /tmp/build-area/* build@10.10.20.13:$d
```

If build server is your local machine, run the following instead.

```
$ mkdir -p /tmp/packages && rsync -av /tmp/build-area/* /tmp/packages/$(basename $(git rev-parse --show-toplevel)) --delete
```

Also push all required branches. Pushing the master branch start the build on Drone CI, see `drone.yml` above.
Drone CI builds from package from `rsync`ed source package, not from the git repo.

```
$ git remote add origin http://<host>:<port>/packages/slm.git
$ git push -u origin --all
```

If everything is OK, tag and push the tags.

```
$ gbp buildpackage --git-tag-only
$ git push --tags
```

When upstream releases a new version (say v0.1.1), following command will do the
heavy-lifting for you using the watch file.

```
$ gbp import-orig --pristine-tar --filter=.drone.yml --uscan
```


If you forget --pristine-tar option in `import` commands, run this command.

```
$ pristine-tar commit ../PKGNAME_NEWVERSION.orig.tar.gz
```

Update changelog again.

```
$ gbp dch -R -c
```

## Repackaging an existing debian package

Import `hello` deb package from Ubuntu xenial. Make sure you have the
corrsponding deb-src in your `/etc/apt/sources.list`.

```
$ gbp import-dsc --download hello/xenial --pristine-tar --filter=.drone.yml
```

## Applying patches

`gbp pq` is a handy tool for managing patches.

```
$ gbp pq import # import patches to patch-queue/master branch
# Hack, hack, hack...
$ git add .
$ git commit -m "New patch to do blah blah"
$ gbp pq switch # switch back to master
$ gbp pq export # apply patches
$ git status # See what happened
$ git add debian
$ git commit -m "Patch(es) applied ..."
$ git branch -D patch-queue/master
```

If `patch-queue/master` branch already exists, run `gbp pq rebase` instead of
gbp `pq import`. See` debian/patches/series` for the applied patches.


**NOTE:** Once `slm` or `hello` packages are imported, they maintained
the same way. Only the import commands differ. If you are maintaining a a nonnative
package which does not exists in any official repo (e.g., the package `slm`),
use `import-orig` to import tarballs. Otherwise use `import-dsc` to import from
official repos like we did for the package `hello`.



[1]: https://wiki.debian.org/debian/watch
