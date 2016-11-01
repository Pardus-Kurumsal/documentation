# Debian Repo Management

We use [aptly][1] to manage our debian repositories.

## Getting a local mirror

**TODO**

## Creating snapshots from the local mirrors

Assuming we have the following local mirrors

```
$ aptly mirror list
List of mirrors:
 * [debian-testing-contrib]: http://ftp.debian.org/debian/ testing [udeb]
 * [debian-testing-main]: http://ftp.debian.org/debian/ testing [udeb]
 * [debian-testing-non-free]: http://ftp.debian.org/debian/ testing [udeb]
```

here is how you can create the snapshots from the local mirrors.

```
$ aptly snapshot create debian-testing-main1 from mirror debian-testing-main
$ aptly snapshot create debian-testing-contrib1 from mirror debian-testing-contrib
$ aptly snapshot create debian-testing-non-free1 from mirror debian-testing-non-free
```

## Creating local repos

The local repos are going to contain our own packages and the modified debian
packages. Following commands create the local repos `pardus-devel-main`,
`pardus-devel-contrib`, `pardus-devel-contrib`.

```
$ aptly repo create -component="main" -distribution="pardus-devel" -comment="Pardus devel main component" pardus-devel-main
$ aptly repo create -component="contrib" -distribution="pardus-devel" -comment="Pardus devel contrib component" pardus-devel-contrib
$ aptly repo create -component="non-free" -distribution="pardus-devel" -comment="Pardus devel non-free component" pardus-devel-non-free
```

## Adding custom packages to the local repos

In the following command, aptly recursively goes through all the folders under
the directory `path/to/packages` and adds the binary and source packages it
finds to the local repo `pardus-devel-main`.

```
$ aptly repo add pardus-devel-main path/to/packages/
```

You can also add a single package to any local repo by explicitly giving the
*.deb file path.

## Merging snapshots

First create snapshots of the local repos. Note that if a local repo is empty,
you can't take its snapshot.

```
$ aptly snapshot create pardus-devel-main1 from repo pardus-devel-main
```

And then merge the snapshots as follows.

```
$ aptly snapshot merge pardus-devel-main-merged1 debian-testing-main1 pardus-devel-main1 
```

## Publish the snapshots

Publish the snapshots as follows.

```
$ aptly publish snapshot -architectures="i386,source,amd64" -gpg-key="B8B5F2D5" -component=main,contrib,non-free -distribution=pardus-devel pardus-devel-main-merged1 debian-testing-contrib1 debian-testing-non-free1 pardus
```

Because `pardus-devel-contrib` and `pardus-devel-non-free` local repos are
empty, we can not take their snapshots. When we add packages to these local
repos, We merge their snapshots with their corresponding debian testing
snapshots and publish them. For the time being, we publish
`debian-testing-contrib1` and `debian-testing-non-free1` snapshots, instead of
`pardus-devel-contrib-merged1` and `pardus-devel-non-free-merged1`.


## Debian installer

When we build `debian-installer` package, we also get a
`debian-installer-images_20160630pardus1_amd64.tar.gz` file. In order to build
an ISO file, we should extract this into
`/srv/mirrors/public/pardus/dists/pardus-devel/main`.

**TODO:** I don't know if this is the proper way of doing this!


[1]:https://www.aptly.info/
