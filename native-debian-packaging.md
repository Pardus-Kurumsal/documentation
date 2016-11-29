# Native debian packaging

In native packages we don't have watch files, and there is no concept of patching.
We do not `import-orig` since there is no notion of upstream.

## Native debian package from scratch

Use `debmake` to create a template native package as follows.

```
debmake -n -u <version> # Use an initial <version> such as 2016.1, 0.1 etc.
```

Edit the template package for your needs and follow follow the same instruction
given for [nonnative packages][1] to build the source package on your local machine
and the binary packages on the continuous integration server.

## Repackaging an existing native debian package

Just like we did for nonnative debian packages, import the `package` using
`import-dsc`. Make sure you have the corresponding deb-src in your
`/etc/apt/sources.list`.

```
$ gbp import-dsc --download package/stretch --pristine-tar --filter=.drone.yml
```

You can follow the same instruction given for [nonnative packages][1] to build
your source package on your local machine and the binary packages on the continuous
integration server.

[1]: ./nonnative-debian-packaging.md
