# What

This is a demo for using SOPS encrypted datafiles, in conjunction
with an encryption key managed in Google's Cloud KMS

# Why

SOPS ("Secet Operations") is a useful tool that lets you keep 90% of a repo
normally editable and readable, but makes sure that certain lines of certain
files are encrypted. 

Not only does it make editing sensitive files easier and safer, but its 
integration with cloud based KMS systems like GCP and AWS means that 
* You can restrict sensitive access to a small subset of repo privileged users
* Because of SSO mechanisms, it is easier for those users to deal with it.

# How

This repo and all required tools were set up using the steps below:

## Install SOPS
curl -sSL https://raw.githubusercontent.com/upciti/wakemeops/main/assets/install_repository | sudo bash

sudo apt install sops

## Create the config .sops.yaml

(See [the one in this repo](.sops.yaml) and adjust as needed)


## Install google-cloud-cli and log in

Follow instructions at https://docs.cloud.google.com/sdk/docs/install

## Create the KMS key

    gcloud services enable cloudkms.googleapis.com

    gcloud kms keyrings create sops-keyring --location=us-west1

    gcloud kms keys create sops-key --location=us-west1 \
      --keyring=sops-keyring --purpose=encryption

    # Note that in GCP KMS, admins do NOT have access permission to keys.
    # Even for admins, you have to explicitly add perms. Use a group!
    # Example:

    gcloud kms keys add-iam-policy-binding sops-key \
      --location=us-west1 \
      --keyring=sops-keyring \
      --member="group:trusted-users@yourdomain.com" \
      --role="roles/cloudkms.cryptoKeyEncrypterDecrypter"

    # (and then of course make sure the right people are in the group)

## Convert the values to be encrypted

After you have created a potential target file, run the following command:

sops -e -i mixed-data.enc.yaml

That file will now have sensitive lines, and ONLY the sensitive lines, encrypted



