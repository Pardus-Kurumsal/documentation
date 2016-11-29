# Creating Base Docker Image

We need modified debootstrap package and a custom debian repo in order to create
a base docker image of our distro. Currently, `pardus-devel` and
`pardus-rolling` are symlinked to `pardus` script in debootstrap package.


```
# Install our debootstrap and keyring packages.
sudo apt-get install ./pardus-archive-keyring_2016.1_all.deb
sudo apt-get install ./debootstrap_1.0.84pardus1_all.deb
```

Run debootstrap.

```
sudo debootstrap pardus-devel pardus-devel
```

Create the docker image `pardus/pardus-devel`. This is the image that is
going to run our live build configuration scripts. 

```
sudo tar -C pardus-devel -c . | docker import - pardus/pardus-devel
```

You can also create `pardus/pardus-package` and `pardus/pardus-test` using the
command above.
