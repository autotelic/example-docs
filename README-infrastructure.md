# Infrastructure Definitions

We use [terraform](https://www.terraform.io) to manage our infrastructure and [AWS](https://aws.amazon.com) to store our terraform state and secrets.

## Prerequisites

* Install [Terraform](https://www.terraform.io/intro/getting-started/install.html). Ensure that the PATH environment variable contains the Terraform binary.

* Gather your AWS account keys.

* Install the [AWS command line interface](http://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html).  Run **"aws configure"** from command line.

* Install the [Google Cloud SDK](https://cloud.google.com/sdk/downloads), then [initalize it](https://cloud.google.com/sdk/docs/initializing) using your Meta Cloud gmail account.

### aws-vault

[aws-vault](https://github.com/99designs/aws-vault) is a tool to manage AWS profiles and keys.

#### Mac

Use [homebrew cask](https://github.com/caskroom/homebrew-cask):

    $ brew cask install aws-vault

#### Windows

* Please do **NOT** download the precompiled binaries. Version 4.1.0 from the release page is detected to be a Trojan, so please do not trust the executables.

* You will need a Golang compiler (https://golang.org/dl/).

* Ensure that "aws-vault" is recognized as a command from cmd or bash; if not, find the install directory from the previous command, and set the PATH variable.

* For convenience, it is highly recommended that you do the following steps to allow aws-vault to run from bash (otherwise aws-vault only works on cmd; there are input issues from bash in Windows).

In git-bash:

    > cd

(Only do the next line if .bash_profile does not exist yet)

    > touch .bash_profile
    > vim .bash_profile

* Add the line "alias aws-vault='winpty aws-vault.exe'" and save the file.
* Restart git-bash

### chamber

[chamber](https://github.com/segmentio/chamber) is a tool to read and write secrets from [AWS SSM Parameter Store](http://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html)

#### Mac / linux

* go to https://github.com/segmentio/chamber/releases/latest to grab the latest binary
* `chmod a+rwx` the binary and symlink to /user/local/bin/chamber

#### Windows

**TODO**

## Setup

### aws-vault

If you have individual account keys, just add a profile for that account in aws-vault, e.g.:

    $ aws-vault add staging

`staging` can be any name you want - it is what you provide to aws-vault to use this role.

If you have [organization](https://aws.amazon.com/organizations/) account keys, it is not necessary to get individual credentials for each account, instead aws-vault can be used to switch roles to the account and provide temporary credentials to access the state.

First, ensure you have added your Organisation AWS account access key and secret key as an aws-vault profile:

    $ aws-vault add org-account

`org-account` can be any name you want, but it must be used as the `source_profile` below.

Then, add a section to your [aws config file](http://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html) for each environment account:

    [profile staging]
    role_arn = arn:aws:iam::<account id>:role/OrganizationAccountAccessRole
    source_profile = org-account

`staging` can be any name you want - it is what you provide to aws-vault to use this role.

Once this is in place, you can then use aws-vault to obtain temporary credentials.

    $ aws-vault exec staging -- <command>

`aws-vault exec <profile>` will insert the AWS credentials for `<profile>` into the current environment. You can view the environment variables that have been created like this (in bash):

    $ aws-vault exec <profile> -- env | grep AWS

This can be very useful to debug. You can delete the sessions for a profile with

    $ aws-vault remove -s <profile>

### chamber

Chamber is a wrapper around AWS SSM parameter store, which enables us to read and write encrypted secrets. [usage docs](https://github.com/segmentio/chamber#usage). We chain it with aws-vault to insert the decrypted secrets as temporary environment variables:

    $ aws-vault exec <profile> -- chamber exec <service...> -- <command>

It may be convenient to alias commonly used chains.

Chamber doesn't provide wrappers for every SSM parameter store action, in which case you can drop down to the aws cli.

To see what "services" are available to you through chamber:

    $ aws-vault exec <profile> -- aws ssm get-parameters-by-path --path /

The first portion of the `Name` key of each parameter object is what chamber requires as a service name. The full value of that `Name` key for a parameter can be used to delete it:

    $ aws-vault exec <profile> -- aws ssm delete-parameter --name "<name>"

Although of course this will delete the history for that secret as well.

### terraform

Before you can terraform the infrastructure for a deployed enviroment, you need to initialise terraform and configure the backend (for the remote state). Change into the directory for the environment you wish to manage, then:

Fetch any terraform modules from their sources:

    $ aws-vault exec <profile> -- chamber exec <service...> -- terraform get

Initialise the remote backend and fetch any providers:

    $ aws-vault exec <profile> -- chamber exec <service...> -- terraform init

## Usage

Now you can work on the infrastructure, plan and apply

    $ aws-vault exec <profile> -- chamber exec <service...> -- terraform plan

    ...

    $ aws-vault exec <profile> -- chamber exec <service...> -- terraform apply
