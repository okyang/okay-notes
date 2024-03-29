## Setup Environment

```bash
# refresh system's package index
sudo apt update

# install ansible
sudo apt install ansible

# on the control machine in linux
ssh-keygen # do not enter a password

# copy keys to remote machine
ssh-copy-id { username }@{ remote machine ip }

# test logging in without a password now
ssh { username }@{ remote machine ip }

# logout
exit
```

## Setup Inventory
In the inventory file replace the following items with `{}` curly braces to the appropriate value
```bash
{ inventory item name } ansible_host={ remote machine ip } ansible_user={ remote machine user}
```

For example:
```bash
owen-raspi ansible_host=10.0.0.193 ansible_user=owen
```

## Run Playbook

```bash
ansible-playbook -i inventory playbook.yml
```