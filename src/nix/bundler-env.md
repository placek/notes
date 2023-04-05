# bundlerEnv

## Private gems repositories are not reachable

### Example

```
trying https://rubygems.pkg.github.com/futurelearn/gems/geoip_fixtures-0.2.0.gem
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (22) The requested URL returned error: 401
error: cannot download geoip_fixtures-0.2.0.gem from any mirror
error: builder for '/nix/store/0gx7xbq8bxz6hxa0jr182bq8sx2d95g5-geoip_fixtures-0.2.0.gem.drv' failed with exit code 1
error: 1 dependencies of derivation '/nix/store/sxkf9a9m5zs364kx1f1kbrn078m5b4lk-ruby2.7.7-geoip_fixtures-0.2.0.drv' failed to build
error: 1 dependencies of derivation '/nix/store/ixbc5szzbrxbb7jgk9cqfv5sx2wlbw2c-futurelearn-gems.drv' failed to build
error: build of '/nix/store/g4528bpk7x4i0s3h5zxa06q8543ih4h2-wrapped-ruby-futurelearn-gems.drv', '/nix/store/ixbc5szzbrxbb7jgk9cqfv5sx2wlbw2c-futurelearn-gems.drv' failed
```

### Solution

Add a function that rewrites the source URLs:

```
add-credentials-to-source = host: credentials: { source, ... }@all:
  all // {
    source = source // {
      remotes = builtins.map (url: builtins.replaceStrings [ "https://${host}" ] [ "https://${credentials}@${host}" ] url) source.remotes;
    };
  };
```

Tell bundlerEnv which gems should be treated with additional care:

```
# rubygems_credentials = "my_user:ghp_CMSDfghSDFGncvjui92352evs1234twevs92";
rubygems_credentials = builtins.getEnv "BUNDLE_RUBYGEMS__PKG__GITHUB__COM";
…
gems = pkgs.bundlerEnv {
  …
  gemConfig = pkgs.defaultGemConfig // {
    geoip_fixtures = add-credentials-to-source "rubygems.pkg.github.com" rubygems_credentials;
    …
  };
};
```

## Collision between gems

### Example

```
building '/nix/store/7ar23qkbvqxj1q1ykffmxzh5vmhi0cqk-futurelearn-gems.drv'...
error: collision between `/nix/store/hrvb38yn7wi480ylcvqxvlaxwn4f0z6d-ruby2.7.7-opensearch-persistence-1.0.0/lib/ruby/gems/2.7.0/cache/bundler/git/opensearch-rails-6b8987d107b0da869cc66cfd21a6a6e1364a074d/index' and `/nix/store/pxqns9mql0g2rfz6ihc71c5ylm6kn7hf-ruby2.7.7-opensearch-model-1.0.0/lib/ruby/gems/2.7.0/cache/bundler/git/opensearch-rails-6b8987d107b0da869cc66cfd21a6a6e1364a074d/index'
error: builder for '/nix/store/7ar23qkbvqxj1q1ykffmxzh5vmhi0cqk-futurelearn-gems.drv' failed with exit code 255
error: 1 dependencies of derivation '/nix/store/4mb3iar5j68d2rd74ibfz861bmfqws2p-docker-layer-futurelearn-image.drv' failed to build
error: 1 dependencies of derivation '/nix/store/35hsn9ir4s3zkkkhpc16j5mf0pllmlld-docker-image-futurelearn-image.tar.gz.drv' failed to build
flag needs an argument: 'i' in -i
See 'docker load --help'.
```

### Solution

Add `ignoreCollisions` option to the bundlerEnv section:

```
gems = pkgs.bundlerEnv {
  …
  ignoreCollisions = true;
};
```
