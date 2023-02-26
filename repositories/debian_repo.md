# Debian Repositories

Debian based OS use dpkg in the background to unpack .deb files and perform
installation steps. apt is a nice wrapper on top of dpkg that handles
downloading and verifying .deb files from a package repository structure. 

## Structure

### .deb

A .deb file is an archive of a file structure. The typical file name for a
debian package follows the following template
`<Name>_<VersionNumber>-<RevisionNumber>_<Architecture>`. in our case Name will
be foo, version will be 1.0, and we will have built a package for amd64. This
will produce the following package `foo_1.0-1_amd64.deb`. The general structure
looks something like this.

foo/  
foo/DEBIAN/control  
foo/usr/bin/package_binary

#### **foo/DEBIAN/control**

```
Package: foo
Version: 1.0
Architecture: amd64
Maintainer: razrdog <razrdog@.com>
Description: some description of what foo does. 
```

#### **Pre and Post install/remove**

in addition to the above file structure you can add scripts to be run during
install to the DEBIAN directory. they are `preinst`, `postinst`, `prerm`,
`postrm`. The shell scripts will be executed and can be used to help build
directories or configure a package as part of the install process.

#### **Other Files**

the same way your package binary is copied to the correct directory during
install you can add other files to be copied to the file system at install. any
file outside of the DEBIAN directory will be copied to the host at that path.
For example placing a file at `foo/etc/foo/foo.conf` will result in
foo.conf being created in `/etc/foo/`

### Repository

In order to set up a repository I followed this guide
https://wiki.debian.org/DebianRepository/SetupWithReprepro while skipping over
creating the actual web server. Instead I push the entire directory structure
to an S3 bucket on various hosting providers.

## Security

### Signatures

#### **Debsig**

Default settings in debian are to verify a repository signature which signs the
manifest file of a repo. When keys are kept properly offline and inaccessible
and the repository mantainer is trusted this is probably an acceptable.

An additional feature is that packages can be signed individually by the
package maintainer using debsig which dpkg can verify on install. unfortunately
dpkg either enforces debsig or not across the entire distribution. Most of the
standard packages don't utilize debsig so configuring dpkg to verify will lead
to all updates and installs from the standard repo failing.

in order to enable debsig enforcement you must edit `/etc/dpkg/dpkg.conf` and
remove the `no-debsig` line. **This will break your package manager**

#### **Split-gpg**

In order to protect the repository PGP key it's possible to run your offline
repository on a QubesOS system which can be configured with Split-gpg. This
helps to protect the private key similar to how an HSM would.

## Refs

- https://www.debian.org/doc/manuals/debian-faq/pkg-basics.en.html
- https://www.debian.org/doc/manuals/maint-guide/start.en.html#needprogs
- https://wiki.debian.org/DebianRepository/SetupWithReprepro