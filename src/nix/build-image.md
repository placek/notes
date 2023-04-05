# dockerTools.buildImage

## copyToRoot requires kvm to be provided

### Example

```
building '/nix/store/i2bfn81lz8fp3df9n4sl2wxp3jbmj8wq-run-as-root.sh.drv'...
building '/nix/store/plsvzrn7c7h5n6vy6qr763am7anrm3lb-vm-run-stage2.drv'...
building '/nix/store/a5ywb47s9qq8drv33h4mil4mdfhlp4n0-vm-run.drv'...
error: a 'x86_64-linux' with features {kvm} is required to build '/nix/store/2h2sb90zii9zlj5skfh04synwlf4g55q-docker-layer-futurelearn-image.drv', but I am a 'x86_64-linux' with features {benchmark, big-parallel, nixos-test, uid-range}
flag needs an argument: 'i' in -i
See 'docker load --help'.
```

### Solution

Add this line to the end of `/etc/nix/nix.conf`:

```
system-features = nixos-test benchmark big-parallel kvm
```

and restart nix-daemon:

```
sudo systemctl restart nix-daemon
```

## Building image fails after suggesting to little space

### Example

```
❯ docker load -i (nix-build .local.nix -A image --no-out-link)
this derivation will be built:
  /nix/store/dnpnpzbb30x04afyxi0l3i4dwyhqdsa7-docker-image-futurelearn-image.tar.gz.drv
building '/nix/store/dnpnpzbb30x04afyxi0l3i4dwyhqdsa7-docker-image-futurelearn-image.tar.gz.drv'...
Adding layer...
tar: Removing leading `/' from member names
tar: temp/layer.tar: Wrote only 4096 of 10240 bytes
tar: Error is not recoverable: exiting now
error: builder for '/nix/store/dnpnpzbb30x04afyxi0l3i4dwyhqdsa7-docker-image-futurelearn-image.tar.gz.drv' failed with exit code 2
       note: build failure may have been caused by lack of free disk space
flag needs an argument: 'i' in -i
See 'docker load --help'.
```

### Solution

Make `/run/user/<USER_ID>` tmpfs bigger:

```
❯ sudo mount -o remount,size=10G /run/user/1000
❯ df -h
Filesystem      Size  Used Avail Use% Mounted on
…
tmpfs            10G  416K   10G   1% /run/user/1000
```
