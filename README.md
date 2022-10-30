# development environment

This folder contains [infrastructure as code][IaC] (IaC) for a minimal development environment that supports swapping backend machines and associated GPU(s) to meet the demands of a given development task. It currently uses [terraform][terraform] with the [google cloud platform][gcpsdk] provider, but it could be adapted for other cloud platforms or providers (see the [terraform documentation][tfmdocs] for further reference).

## workflow
The expected workflow is to

- set up a development machine with `make up`, 
- connect to the machine via the associated jupyter lab server accessible from the [google cloud platform user interface][gcpui] for interactive use, 
- [ssh](#remote-connection) to the machine from a terminal or IDE such as VS Code for library development,
- toggle the machine off and on with `make stop` and `make start`, and
- destroy all associated resources with `make down`.

## prerequisites

- install [google cloud sdk][gcpsdk]
  - `gcloud init` to set project and [application default credentials][adc]
    - `gcloud auth login`
    - `gcloud auth application-default login`
- install [terraform][terraform]
  - `terraform init`
- edit [tfvars.sh](./tfvars.sh)
  - set variables using [pass][pass] or manually
    - `pass insert github_username`
    - and similar for `gcp_project`, `gcp_email` ,`gcp_credentials_file`, `gcp_notebooks_name`
    - `gcp_credentials_file` contains the path to appication default credentials. The most common value is `~/.config/gcloud/application_default_credentials.json`
    - check these are defined with `$ pass`

      ```shell
      $ pass
      Password Store
      ├── gcp_credentials_file
      ├── gcp_credentials_path
      ├── gcp_email
      ├── gcp_notebooks_name
      ├── gcp_project
      └── github_username
      ```

  - install and authenticate with [github cli][ghcli] to use gists for the post startup script
    - check `gh auth status` when complete
  - executing this script will upload [post-startup-script.sh](./post-startup-script.sh) to github gist by default
- review/edit [terraform.tfvars](./terraform.tfvars)
  - you can optionally set the post startup script url here if you are not able to set up the [github cli][ghcli] 
- `make test` will function when above are satisfied
  - upload [post-startup-script.sh](./post-startup-script.sh) to github gist
  - print `TF_VARS_*` environment variables

## usage 

The primary interface is via the [Makefile](./Makefile), which is being used here as a modular collection of short shell scripts rather than as a build system. You can fill environment variables and print each command prior to running with `make -n <target>` such as `make -n up`. Please see [GNU make][make] for further reference. The primary targets are

    make up - create -OR- update the instance
    make stop - stop the instance
    make start - start the instance
    make down - delete the instance
    
All other targets are auxiliary. The [Makefile](./Makefile) is primarily to document commands that are commonly used to work with the terraform resource(s). You can simply copy the command from the Makefile and run it manually in the terminal if you do not want to use [make][make].

## remote connection

The [Makefile](./Makefile) will run

```shell
gcloud compute config-ssh
```

to update your ssh configuration file and print the configured hostname at the end of `make up`. In order to connect from IDEs or otherwise, it may be helpful to update your `~/.ssh/config` file with something similar to (updated to reference the key files you use with google cloud platform)

```shell
Host gcp
    HostName <IP_ADDRESS>
    IdentityFile ~/.ssh/google_compute_engine
    UserKnownHostsFile=~/.ssh/google_compute_known_hosts
    IdentitiesOnly=yes
    CheckHostIP=no
    StrictHostKeyChecking=no
    RequestTTY Yes
    RemoteCommand cd /home/jupyter && sudo su jupyter
```

The `IP_ADDRESS` of the remote host is printed at the end of `make up`. You can run `gcloud compute instances list` to display the `IP_ADDRESS` of the virtual machine if you need to reference it.
If you use the container rather than disk image to setup the virtual machine, you may find an alternative `RemoteCommand` useful

```shell
Host gcp
    ...
    RemoteCommand sudo docker exec -it payload-container /bin/bash
```


[IaC]: https://en.wikipedia.org/wiki/Infrastructure_as_code
[adc]: https://cloud.google.com/docs/authentication/provide-credentials-adc
[make]: https://www.gnu.org/software/make/
[gcpsdk]: https://cloud.google.com/sdk/docs/install
[tfmdocs]: https://developer.hashicorp.com/terraform/docs
[gcpui]: https://console.cloud.google.com/vertex-ai/workbench/list/instances
[terraform]: https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/install-cli
[pass]: https://www.passwordstore.org/
[ghcli]: https://cli.github.com
