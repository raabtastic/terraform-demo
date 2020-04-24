# Getting Started with Terraform

Terraform is the most popular language for defining and provisioning infrastructure as code (IaC).

In this guide you will lay the foundation for using Terraform to manage your infrastructure by

* Creating a configuration file
* Downloading and installing a provider
* Creating and running an execution plan
* Destroying provisioned infrastructure

By completing the steps in this guide, you will demonstrate that you have Terraform correctly installed and are able to create, configure, and destroy a piece of infrastructure.

## Prerequisites

Before you can create infrastructure with Terraform, you must first install it. To install Terraform, visit [Terraform.io](https://www.terraform.io/downloads.html) and download the proper package for your operating system and architecture. Unzip the downloaded package and move it into a directory included in your system's [`PATH`](https://superuser.com/questions/284342/what-are-path-and-other-environment-variables-and-how-can-i-set-or-use-them).

Verify Terraform is correctly installed by checking the version number.

```shell
$ terraform -version
```

If you see

```shell
terraform: command not found
```

then Terraform is not yet successfully installed.

## Configure Terraform

With Terraform installed, you can now start creating infrastructure.

First create a new directory on your local machine.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

Next, create a file for your Terraform configuration code.

```shell
$ touch main.tf
```

If your system doesn't support `touch`, you can create a new file in a text editor of your choice and save it as `main.tf`. Note that configuration is not sensitive to the filename; when run, Terraform will load all files in the working directory that end in `.tf`.

Paste the following lines into the file.

```hcl
provider "docker" {
    host = "unix:///var/run/docker.sock"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

## Initialize Terraform

With the configuration file created and saved, you can now initialize Terraform with the `init` command. The `init` command will download and install all providers used in the configuration. For now, this is just the `docker` provider.

```shell
$ terraform init
```

You should see output similar to the following.

```shell

Initializing the backend...

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.docker: version = "~> 2.7"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Take note of the providers installed to ensure they match your expected configuration. In this case, you can see Terraform installed the latest version of `docker`, as expected. Terraform downloads providers into hidden subdirectories of the current working directory. 

If the output does not say Terraform has been successfully initialized, you will need to troubleshoot any error messages given. 

## Create infrastructure

Once Terraform is successfully initialized, use the `apply` command to create infrastructure.

```shell
$ terraform apply
```

The `apply` command generates an _execution plan_ based on the configuration file(s) in the current working directory. This plan details changes Terraform will make to your infrastructure to match the desired configuration. 

It may take a few minutes to generate this plan. If it runs successfully, you will see output similar to the following. 

```
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = "json-file"
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + restart          = "no"
      + rm               = false
      + shm_size         = (known after apply)
      + start            = true

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id     = (known after apply)
      + latest = (known after apply)
      + name   = "nginx:latest"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Type `yes` to execute the plan. This may take a few minutes, during which you may be prompted to allow firewall exceptions or grant other permissions. Upon successful completion, you should see output similar to the following.

```
docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Still creating... [20s elapsed]
docker_image.nginx: Still creating... [30s elapsed]
docker_image.nginx: Still creating... [40s elapsed]
docker_image.nginx: Still creating... [50s elapsed]
docker_image.nginx: Still creating... [1m0s elapsed]
docker_image.nginx: Still creating... [1m10s elapsed]
docker_image.nginx: Still creating... [1m20s elapsed]
docker_image.nginx: Still creating... [1m30s elapsed]
docker_image.nginx: Still creating... [1m40s elapsed]
docker_image.nginx: Creation complete after 1m41s [id=sha256:602e111c06b6934013578ad80554a074049c59441d9bcd963cb4a7feccede7a5nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=d2d80515cb6853814db35f033853e8ef1fa92aa04292a8dda28a04fea96b86c3]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

If you received the error ```"Cannot connect to the Docker daemon at unix:///var/run/docker.sock"```  then you will need to troubleshoot your Docker setup. On some systems you may need to change the `host` parameter in your configuration file to `localhost`.

```shell
host = "tcp://localhost:2375"
```

## Destroy infrastructure

To close the loop on your Terraform introduction, you will now destroy the infrastructure you created by running the following command.

```shell
$ terraform destroy
```

Similar to `apply`, `destroy` creates and displays an execution plan.

```
docker_image.nginx: Refreshing state... [id=sha256:602e111c06b6934013578ad80554a074049c59441d9bcd963cb4a7feccede7a5nginx:latest]
docker_container.nginx: Refreshing state... [id=d2d80515cb6853814db35f033853e8ef1fa92aa04292a8dda28a04fea96b86c3]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach            = false -> null
      - command           = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - cpu_shares        = 0 -> null
      - dns               = [] -> null
      - dns_opts          = [] -> null
      - dns_search        = [] -> null
      - entrypoint        = [] -> null
      - env               = [
          - "NGINX_VERSION=1.17.10",
          - "NJS_VERSION=0.3.9",
          - "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
          - "PKG_RELEASE=1~buster",
        ] -> null
      - gateway           = "172.17.0.1" -> null
      - group_add         = [] -> null
      - hostname          = "d2d80515cb68" -> null
      - id                = "d2d80515cb6853814db35f033853e8ef1fa92aa04292a8dda28a04fea96b86c3" -> null
      - image             = "sha256:602e111c06b6934013578ad80554a074049c59441d9bcd963cb4a7feccede7a5" -> null
      - ip_address        = "172.17.0.2" -> null
      - ip_prefix_length  = 16 -> null
      - ipc_mode          = "private" -> null
      - links             = [] -> null
      - log_driver        = "json-file" -> null
      - log_opts          = {} -> null
      - logs              = false -> null
      - max_retry_count   = 0 -> null
      - memory            = 0 -> null
      - memory_swap       = 0 -> null
      - must_run          = true -> null
      - name              = "training" -> null
      - network_data      = [
          - {
              - gateway          = "172.17.0.1"
              - ip_address       = "172.17.0.2"
              - ip_prefix_length = 16
              - network_name     = "bridge"
            },
        ] -> null
      - network_mode      = "default" -> null
      - privileged        = false -> null
      - publish_all_ports = false -> null
      - read_only         = false -> null
      - restart           = "no" -> null
      - rm                = false -> null
      - shm_size          = 64 -> null
      - start             = true -> null
      - sysctls           = {} -> null
      - tmpfs             = {} -> null

      - labels {
          - label = "maintainer" -> null
          - value = "NGINX Docker Maintainers <docker-maint@nginx.com>" -> null
        }

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id     = "sha256:602e111c06b6934013578ad80554a074049c59441d9bcd963cb4a7feccede7a5nginx:latest" -> null
      - latest = "sha256:602e111c06b6934013578ad80554a074049c59441d9bcd963cb4a7feccede7a5" -> null
      - name   = "nginx:latest" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

Notice the `-` symbols at the beginning of many lines. These indicate the configuration commands will be removed, similar to how `+` symbols indicated the inclusion of commands when the configuration plan was created with `apply`. 

To continue with the destruction of the infrastructure, type `yes` and hit ENTER. Terraform will destroy the resources it had created earlier. You should see messages similar to following which indicate successful completion.

```
docker_container.nginx: Destroying... [id=d2d80515cb6853814db35f033853e8ef1fa92aa04292a8dda28a04fea96b86c3]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:602e111c06b6934013578ad80554a074049c59441d9bcd963cb4a7feccede7a5nginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

## Next steps

You have successfully configured, created, and destroyed your first piece of infrastructure with Terraform. Building upon this foundation, you can now [build](https://learn.hashicorp.com/terraform/getting-started/build) and [configure](https://learn.hashicorp.com/terraform/getting-started/change) infrastructure from other providers, such as AWS.
