# PKS CIFS and JQ addon

## What does this do?

This will ensure that PKS clusters can mount CIFS/SMB stores using the FlexVolume.
https://github.com/Azure/kubernetes-volume-drivers/tree/master/flexvolume/smb

It installs JQ and CIFS-utils.   Note that PKS on Azure clusters already have CIFS Utils installed, but JQ isn't in the system path.

## How do I install it?

1. Open a shell prompt on a BOSH CLI with access to your PKS bosh director, such as Ops Manager.
2. Export your BOSH credentials to the enviornment.  These can be accessed via the Ops Manager GUI -> BOSH Director Tile -> Credentials Tab -> Bosh Commandline Credentials.

e.g.
```
export BOSH_CLIENT=ops_manager BOSH_CLIENT_SECRET=fakesecret BOSH_CA_CERT=/var/tempest/workspaces/default/root_ca_certificate  BOSH_ENVIRONMENT=10.0.0.10
```
3. Copy or clone this repository onto this BOSH CLI workstation and create+upload the BOSH release to the director

```
git clone https://github.com/svrc-pivotal/pks-cifs-utils && cd pks-cifs-utils
cd jq-release
bosh create-release --force
bosh upload-release ./dev_releases/jq-release/jq-0+dev.1.yml

```
4. Configure the addon from this repo.  Need to discover a current BOSH Kubo release on your director.
```
cd ..
sed "s/KUBO_VERSION/$(bosh releases | grep "^kubo " | head -1 | cut -f2 | sed 's/\*//')/" ./addon.yml
bosh -n update-config --name=pks-cifs-jq --type=runtime ./addon.yml
```
5. Update your PKS clusters via the PKS CLI and/or Ops Manager "Apply Pending Changes" button with the PKS upgrade errand enabled.  This addon will automatically be installed on all master and worker nodes with the default manifest `addon.yml`

