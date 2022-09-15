# ofed

Installs OFED via Mellanox Repositories

# Requirements

None

# Role Variables

Please see `defaults/main.yml`.

* `ofed_repository_url` Gives a URL to the repository holding the OFED driver packages. Defaults to the [Mellanox public package repository](https://linux.mellanox.com/public/repo/mlnx_ofed)
* `ofed_version` Specifies the OFED driver version to be installed, default is `latest`
* `ofed_target_release` Specifies the Linux distribution for the install of the OFED drivers, default is `rhel7.8`

# Dependencies

None

# Example Playbook

## Default settings

```
    - hosts: servers
      tasks:
        - include_role: stackhpc.ofed
```

# Custom settings

## Defining the package repository

There are two methods of defining the repositories used to install the OFED drivers, setting the `ofed_custom_repos` disables the URL method.

**NOTE:** If how the repository definition method changes it is recommended to purge the old repo files _before_ running the StackHPC OFED role as this can lead to unpredictable behaviour when installing packages and their dependencies.

### Defining the repository with a URL

These settings will install the OFED drivers from a local repository, and target version and the RHEL8.3 release. This expects there to be a `.repo` file at the location to configuer the repository

```
    - hosts: servers
      vars:
        ofed_repository_url: http://repo.local
        ofed_version: '5.2-2.2.0.0'
        ofed_target_release: rhel8.3
      tasks:
        - include_role: stackhpc.ofed
```

### Using a custom repository

Defining the `ofed_custom_repos` variable can be used to be more explicit with the repository configuration. This disables the url method specified with `ofed_repository_url`, `ofed_version`, and `ofed_target_release`. This can be used to define multiple repositories if required, note that the standed Mellanox OFED repository configuration is four repositories in a single repository configuration file.

```
    - hosts: servers
      vars:
        ofed_custom_repos:
          - name: Repository One
            file: two_repos_one_file.repo
            baseurl: https://repo.one.url
          - name: Repository Two
            file: two_repos_one_file.repo
            baseurl: https://repo.two.url
          - name: Repository Three
            file: detailed_repo_three.repo
            description: "Detailed repository definition"
            baseurl: https://repo.three.url
            mirrorlist: https://repo.three.mirrorlist.url
            enabled: false
            gpgkey: https://repo.three.gpgkey.url
            gpgcheck: true
            verifyssl: false
      tasks:
        - include_role: stackhpc.ofed
```

## Disable kernel update

This should skip the kernel updates:

```
    - hosts: servers
      vars:
        ofed_update_kernel: false
      tasks:
        - include_role: stackhpc.ofed
```

## Packaged dependency conflicts

If the OFED drivers fail to install with conflicts over `libibverbs` with an error similar to the following:

```
"Depsolve Error occured: \n Problem: cannot install both libibverbs-55mlnx37-1.55103.x86_64 and libibverbs-35.0-1.el8.x86_64\n  - package perftest-4.5-1.el8.x86_64 requires libefa.so.1()(64bit), but none of the providers can be installed\n  - package perftest-4.5-1.el8.x86_64 requires libefa.so.1(EFA_1.1)(64bit), but none of the providers can be installed\n  - cannot install the best candidate for the job"
```

This can be resolved from the command line by passing `--nobest` to `yum` or `dnf`, or with this playbook with the `ofed_nobest` variable.

For further issues (such as installing older OFED drivers) the `--allowerasing` and `--skipbroken` parameters can also be passed with `ofed_allowerasing` and `ofed_skipbroken` respectivly.

If the problem can be pinned to specific packages these can be excluded with the `ofed_exclude` variable which passes the string or list as the `--exclude` argument. The example below excludes two specific packages with versions included.

```
    - hosts: servers
      vars:
        ofed_nobest: true
        ofed_skipbroken: true
        ofed_allowerasing: true
        ofed_exclude:
          - libibumad-37.2-1.el8.x86_64
          - infiniband-diags-37.2-1.el8.x86_64
      tasks:
        - include_role: stackhpc.ofed
```

## Failure backing up Grub config

The kernel update updates the grub configuration to include the new kernel modules, however if a host is missing either grub configuration file `/etc/grub2.cfg` or `/etc/grub2-efi.cfg` this task can fail. The backup can be disabled with the `ofed_grub_backup` variable.

**NOTE:** It's recommended to test that the host successfully boots with the OFED Kernel Modules first before disabling the config backup, as this setting destroys the previous configuration.

```
    - hosts: servers
      vars:
        ofed_grub_backup: false
      tasks:
        - include_role: stackhpc.ofed
```
## Incorrect Kernel Modules

On installing the Mellanox OFED Drivers, the `openibd` daemon may not start with the following errors:

```
openibd: ERROR: Module mlx5_ib belong to kernel-modules which is not a part of MLNX_OFED, skipping
openibd: ERROR: Module mlx5_core belong to kernel-core which is not a part of MLNX_OFED, skipping
openibd: ERROR: Module ib_umad belong to kernel-modules which is not a part of MLNX_OFED, skipping
openibd: ERROR: Module ib_uverbs belong to kernel-modules which is not a part of MLNX_OFED, skipping
openibd: ERROR: Module ib_ipoib belong to kernel-modules which is not a part of MLNX_OFED, skipping
openibd: ERROR: Module rdma_cm belong to kernel-modules which is not a part of MLNX_OFED, skipping
openibd: ERROR: Module rdma_ucm belong to kernel-modules which is not a part of MLNX_OFED, skipping
```

This can be due to the kernel modules included in the packages from the repository not matching the kernel, while it is recommended that the [kernel modules be recompiled to match the kernel](https://docs.nvidia.com/networking/display/MLNXOFEDv531001/Installing+Mellanox+OFED#InstallingMellanoxOFED-additionalinstallationproceduresAdditionalInstallationProcedures) and provided via a local package repositry there is a workaround by enabling `FORCE_MODE` and disabling `RDMA_UCM_LOAD` in `/etc/infiniband/openib.conf`. The role can implement this with the following variables:

```
    - hosts: servers
      vars:
        ofed_manage_openib_conf: true
        ofed_force_mode: true
        ofed_rdma_ucm_load: false
      tasks:
        - include_role: stackhpc.ofed
```


License
-------

Apache

Author Information
------------------

Will Szumski
