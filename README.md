# What is this repo for?

This is a demo for using SOPS encrypted datafiles, in conjunction
with an encryption key managed in Google's Cloud KMS

The instructions for using it with other backends such as AWS KMS,
Azure, or Hashicorp Vault and others, are very similar. This guide just
happens to focus on GCP.

# Why use SOPS

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

Make sure you DO NOT COMMIT THE FILE TO git BEFORE CALLING SOPS ON IT!!

---

# Keep calm and carry on

You are now basically all set! Use git as normal. You can even edit and
update the non-sensitive parts of the .enc file as you usually do.
But if you need to edit anything in **ENC[xxxxxx]**  then call
"sops thatfile.enc.yaml"

It will automatically decrypt the file and invoke an $EDITOR, presuming
your cloud credentials are current.
If not, then in the case of google, you may need to do

    gcloud auth application-default login

If a program needs to access the file to use the values normally, then
make sure it has valid cloud credentials, and then either use a SOPS library,
or call the executable with

    sops -d thatfile.enc.yaml

It will dump the values to stdout, so make sure you handle the data in
an appropriately secure manner.



