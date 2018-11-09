Frequently Asked Questions (FAQ)
================================

Why is there a collision error when switching generation?
---------------------------------------------------------

Home Manager currently installs packages into the user environment,
precisely as if the packages were installed through
`nix-env --install`. This means that you will get a collision error if
your Home Manager configuration attempts to install a package that you
already have installed manually, that is, packages that shows up when
you run `nix-env --query`.

For example, imagine you have the `hello` package installed in your
environment

```console
$ nix-env --query
hello-2.10
```

and your Home Manager configuration contains

    home.packages = [ pkgs.hello ];

Then attempting to switch to this configuration will result in an
error similar to

```console
$ home-manager switch
these derivations will be built:
  /nix/store/xg69wsnd1rp8xgs9qfsjal017nf0ldhm-home-manager-path.drv
[…]
Activating installPackages
replacing old ‘home-manager-path’
installing ‘home-manager-path’
building path(s) ‘/nix/store/b5c0asjz9f06l52l9812w6k39ifr49jj-user-environment’
Wide character in die at /nix/store/64jc9gd2rkbgdb4yjx3nrgc91bpjj5ky-buildenv.pl line 79.
collision between ‘/nix/store/fmwa4axzghz11cnln5absh31nbhs9lq1-home-manager-path/bin/hello’ and ‘/nix/store/c2wyl8b9p4afivpcz8jplc9kis8rj36d-hello-2.10/bin/hello’; use ‘nix-env --set-flag priority NUMBER PKGNAME’ to change the priority of one of the conflicting packages
builder for ‘/nix/store/b37x3s7pzxbasfqhaca5dqbf3pjjw0ip-user-environment.drv’ failed with exit code 2
error: build of ‘/nix/store/b37x3s7pzxbasfqhaca5dqbf3pjjw0ip-user-environment.drv’ failed
```

The solution is typically to uninstall the package from the
environment using `nix-env --uninstall` and reattempt the Home Manager
generation switch.

Why are the session variables not set?
--------------------------------------

Home Manager is only able to set session variables automatically if it
manages your Bash or Z shell configuration. If you don't want to let
Home Manager manage your shell then you will have to manually source
the

    ~/.nix-profile/etc/profile.d/hm-session-vars.sh

file in an appropriate way. In Bash and Z shell this can be done by
adding

```sh
. "$HOME/.nix-profile/etc/profile.d/hm-session-vars.sh"
```

to your `.profile` and `.zshrc` files, respectively. The
`hm-session-vars.sh` file should work in most Bourne-like shells.

How do I switch configs based on architecture/machine?
------------------------------------------------------

Typically this would be done by having separate "top-level" files for your
different systems. I usually name these based on the hostname (and the username
if I have multiple users on the system) like `atlantis-rycee.nix`, for
example. Then for each user you can set up a symbolic link from
`~/.config/nixpkgs/home.nix` to the appropriate top level file.

So, if you have three machines, you might create three files

- `nixos.nix`
- `debian.nix`
- `darwin.nix`

each of these would follow the format

```nix
{ ... }:

{
  imports = [ ./common.nix ];

  # Various options that are specific for this machine/user.
}
```

where the `common.nix` file contains configuration shared across the three
systems. Of course, instead of just a single `common.nix` file you can have
multiple ones, even one per program or service.

You can get some inspiration from the [Post your home-manager home.nix
file!][1] Reddit thread.

[1]: https://www.reddit.com/r/NixOS/comments/9bb9h9/post_your_homemanager_homenix_file/
