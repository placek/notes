# Package pinning

## nixpkgs channel

```
let
  revision = "8ad5e8132c5dcf977e308e7bf5517cc6cc0bf7d8";
  pkgs = import (builtins.fetchTarball { url = "https://github.com/NixOS/nixpkgs/archive/${revision}.tar.gz"; }) {};
in
...
```

## Single derivation

The version pinning of single derivation is done by providing a source for its definition. It can be done by providing a tarball or github commit of nixpkgs (or other source) and import its derivation, like:

```
let
  pkgs = import (builtins.fetchTarball {
    url = "https://github.com/NixOS/nixpkgs/archive/8ad5e8132c5dcf977e308e7bf5517cc6cc0bf7d8.tar.gz";
  }) {};

  ruby_2_7_7 = pkgs.ruby;
in
...
```
or
```
let
  pkgs = import (builtins.fetchGit {
    # Descriptive name to make the store path easier to identify
    name = "my-ruby-version-nixpkgs";
    url = "https://github.com/NixOS/nixpkgs/";
    ref = "refs/heads/nixpkgs-unstable";
    rev = "8ad5e8132c5dcf977e308e7bf5517cc6cc0bf7d8";
  }) {};

  ruby_2_7_7 = pkgs.ruby;
in
...
```

Easy way to find the exact version of the derivation is to use [Package search site by Marcelo Lazaroni](https://lazamar.co.uk/nix-versions). This solution is providing some cached build as well so the derivations has no need to be compiled when installed.

Most advanced version of such search is to use [nixpkgs repository](https://github.com/NixOS/nixpkgs) and search for commits with packagename and version in commit message.
