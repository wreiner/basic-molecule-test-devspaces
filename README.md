# Testing Ansible Role With Molecule on OpenShift DevSpaces

Basic example on how to test Ansible roles with Molecule on OpenShift DevSpaces.

This is based on the [Ansible Development on OpenShift Dev Spaces](https://github.com/redhat-developer-demos/ansible-devspaces-demo) demo.

## Basic Setup

- Install ansible and molecule

```
python3 -m venv ansiblevenv
source ansiblevenv/bin/activate
pip install ansible
```

- Create a role from templates

```
ansible-galaxy role init nginx
```

- Initialize molecule

```
molecule init scenario
```

- Update information in `nginx/meta/main.yml`

```
---
galaxy_info:
  role_name: nginx
  namespace: your-namespace
...
```

## Implement role and configure Molecule

- Implement role

- Configure Molecule

The file [nginx/molecule/default/molecule.yml](nginx/molecule/default/molecule.yml) used here shows an example of how to 

```
dependency:
  name: galaxy
  options:
    requirements-file: requirements.yml
```

```
platforms:
  - name: molecule-ubi9-1
    image: registry.access.redhat.com/ubi9/ubi
    workingDir: '/tmp'
```

- Implement converge playbook, it installs the role

For the connection `community.okd.oc` is used to access the started pod.

- Implement create and destroy playbooks

In the example, both use the pod template in `nginx/molecule/default/templates/deployment.yml`.
The image which was set through `platforms > image` in `molecule.yml` is used as the pod image.

- Implement the verifying playbook

The role will be tested using Ansible. For this a playbook needs to be defined.

Creating a new role with the `ansible-galaxy role init` command, all default directories are being created.
So is the `tests` directory created, with a `test.yml` playbook file and a test `inventory` file.

To run the tests on OpenShift, again the connection must be set to `community.okd.oc`, therefore the verification playbook
is implemented in `nginx/molecule/default/verify.yml` rather than the `test.yml`.

## Setup DevSpaces tasks

DevSpaces can be configured using `devfile.yml`.

To run the tests, a tooling container is needed on where the molecule commands are being run. This is configured through the `components` directive.

The available tasks can be configured through the `commands` directive. Every command is set to run on a previously defined `component`.

Commands need to be run inside the roles directory, in the example `nginx`. When starting the DevSpace, the repository is cloned locally. The name of the clone directory is the same as the name of the repository, in this example it is `basic-molecule-test-devspaces`, because the repository address is `https://github.com/wreiner/basic-molecule-test-devspaces`.

## Start a DevSpace and run the test

Make sure DevSpaces is installed and configured correctly.

- Open the DevSpaces dashboard and click `Create Workspace`.
- Input the repository address `https://github.com/wreiner/basic-molecule-test-devspaces` into the field `Import from Git > Git repo URL` and click `Import from Git > Create & Open`.
- Make sure the extension `YAML` is installed (should be automatically be installed through `.vscode/extensions.json` and the Ansible extension)
- Load local devfile for the workspace by clicking `Workspace > Show and Run Commands (F1) > DevSpaces: Restart Workspace From local dev file` and choosing the devfile.yml
- Run a task defined in the devfile by clicking `Workspace > Run Task > devfile > 6.Molecule: run the full molecule test`
