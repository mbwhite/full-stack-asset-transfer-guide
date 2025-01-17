# full-stack-asset-transfer-guide

> NOTE to Early Users Please ensure that you clone and build the docker image for the ansible-collection
## Scenario

- Asset transfer, based on the secured assest transfer scenario.
- Token based settlement of the final amount
- Organizations: 
    - Escrow (as endorsing org)
    - Regulator (as ordering org)
    - Bank/Finance Company (two of)




## Parts of the system

- Chaincode for the private asset transfer
- Client Application using Gateway to send in the transactions
    - tempted to suggest that the regulator has a basic CLI based 
    - the two banks can have a very simple UI if possible to demonstrate the REST API

- Tokens..... yes

## Orchestration of System to provision

## 0. Important Setup Steps 

Ahead of the [PR](https://github.com/IBM-Blockchain/ansible-collection/pull/610/files) being available

```
git clone git@github.com:IBM-Blockchain/ansible-collection.git
cd ansible-collection
git fetch origin pull/610/head:ofs-ansible
git checkout ofs-ansible
docker build -t ofs-ansible .
```

### 1. Create a KIND local Cluster

This will start a local KIND cluster running in a docker container; there are some minor variations beween K8S clusters, for example default storage class, which this will configure for you.

```shell
just kind
```


### 2. Deploy the Fabric Operator and Console

This will deploy the Fabric Operator and the Fabric Operations console via two Ansible Playbooks, and some configuration variables. 


- Creation of the operator: `ansible-playbook ./infrastructure/01-operator-install.yml`
- Creation of the console:  `ansible-playbook ./infrastructure/02-console-install.yml`

The configuration file is `vars.yml`

```yaml
# The type of K8S cluster this is using
target: kind
arch: amd64

# k8s namespace for the operator and console
namespace: fabricinfra

# Console name/domain
console_name: hlf-console
console_domain: localho.st

#  default configuration for the console
# password reset will be required on first login
console_email: admin
console_default_password: password

# different k8s clusters will be shipped with differently named default storage providers
# or none at all.  KIND for example has one called 'standard'
console_storage_class: standard
```

To run these you either need Ansible installed locally and a number of Ansible plugins; it is easier therefore to run a docker based version, that includes all the requirements. issue `just console` use this container to run both playbooks

This is running the following two commands - it's using a docker container as it's easy to that then get the python environmment setup.

```
docker run --rm -v ${HOME}/.kube/:/home/ibp-user/.kube/ -v $(pwd)/infrastructure/operator_console_playbooks:/playbooks --network=host fabric-ansible:latest ansible-playbook /playbooks/01-operator-intstall.yml    
docker run --rm -v ${HOME}/.kube/:/home/ibp-user/.kube/ -v $(pwd)/infrastructure/operator_console_playbooks:/playbooks --network=host fabric-ansible:latest ansible-playbook /playbooks/02-console-install.yml
```

or

```shell
just console  
```

The console will be running on `https://fabricinfra-hlf-console-console.localho.st/` - give it a try in your favourite browser. Be aware though it will complain as the certificate for HTTPS is not setup.
The default password and admin account is defined in the `vars.yml` above - you will need to change this on first login

It is best to create an API key to be able to work with teh Console/Operator programmatically, the `just console` command does this for you. For reference the command is (change the password here if you've altered in the console)

```
curl -X POST https://fabricinfra-hlf-console-console.localho.st:443/ak/api/v2/permissions/keys -u admin:password -k -H 'Content-Type: application/json' -d '{"roles": ["writer", "manager"],"description": "newkey"}'
```


This producces json, that the script will parse into a auth-vars.yml for ansible to use in step 3.


### 3. Create Two org network within the console  *AS-IS*

This is the standard Ansible Collection network.

```
just fabric-network
```

When this has completed, there will be a 2 org network ready to go. 

### 3. Confirm the Fabric Sail Descriptor   **TO-BE**

A declarative description is in the `infrastructure/fabric-sail.yaml`

From this we can create several resouces, however the Ansible playbooks to create this set of Fabric nodes is what we want

```
fabgen --file ./infrastructure/fabric-sail.yaml --out ./infrsatructure/created_playbooks/ --template sail
```

If you look in the `./infrsatructure/created_playbooks/` You'll see a number of playbooks created
Each of these needs to be run to create the resources required

```bash
ansible-playbook ....
```

