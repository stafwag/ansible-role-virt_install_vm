# cloud-init templates

These are ```cloud-init``` templates. Configure cloud image with ```cloud-init```

[simple_debian](simple_debian) is a symlink to [debian/11/simple_debian/](debian/11/simple_debian/) remain compatibable  with playbooks that uses this directly.

# Operation systems

## Debian GNU/Linux

* **Debian 11 (bullseye)**

  [debian/11/simple_debian](debian/11/simple_debian) A cloud-init v1 template to set up a Debian 11 GNU/Linux system. The network config is executed as part of the cloud-init ```user-data```. An interface configuration is created in ```/etc/network/interfaces.d``` and ```ifup``` is executed to bring up the interface.


* **Debian 12 (bookworm)**

  [debian/12/simple_debian](debian/11/simple_debian) A cloud-init v2 template to set up a Debian 12 GNU/Linux system. The same templates should work with most GNU/Linux systems that support cloud-init v2.
