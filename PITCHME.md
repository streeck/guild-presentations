---?image=images/celery.png&size=auto 80%
@title[Intro]

---
## Outline

1. Current setup of celery in DE
2. New proposed setup by platform team
3. Future improvements
4. Discussion / Questions

---
@title[Current setup]

## Current setup

+++?image=images/jenkins_1.png&size=auto 80%

+++?image=images/jenkins_2.png&size=auto 80%

+++

```shell
virtualenv-2.7 venv
source venv/bin/activate
pip install -r requirements.txt
export INVENTORY_FILE="ansible/inventory/test"

sed -i '/\.mgt\.gen\.local/b; s/^\(el[h|a][^ ]*\)/\1.mgt.gen.local/g' $INVENTORY_FILE

ansible-playbook -v --inventory-file=$INVENTORY_FILE ansible/api_worker.yml --limit=api_worker,api_beat --diff
```

@[1-3](Done by `make requirements` in devops)
@[4](Already taken care by `deploy-pipeline` in Jenkins)
@[6](Copy pasted from devops Jenkinsfile)
@[8](Also taken care by `deploy-pipeline`)


+++

```text
ansible
├── roles
│   ├── celery
│   ├── celery_beat
│   └── deploy
├── vars
│   ├── api_beat.yml
│   ├── api_worker.yml
│   └── api.yml
├── api_worker.yml
└── api.yml
Jenkinsfile
Makefile
```

+++

### api_worker playbook

```yaml
- hosts: api_worker
  become: yes
  vars_files:
    - vars/api_worker.yml
  roles:
    - {role: celery, tags: ['api_worker_daemon']}
    - {role: monit, tags: ['api_monit']}
  tags: [api_worker]

- hosts: api_beat
  become: yes
  vars_files:
    - vars/api_beat.yml
  roles:
    - {role: celery_beat, tags: ['api_beat_daemon']}
  tags: [api_beat]
```

+++

### Jenkinsfile

```shell
./bin/inventories_to_mgt.sh
make requirements
```

+++

### Makefile

```makefile
venv:
	@virtualenv -p python2.7 --no-download venv
	venv/bin/pip install tox

requirements: venv
	venv/bin/pip install -r requirements.txt
```

+++

### Challenges

- Deployment inconsistency
- Scattered configuration and management burden
- DRY and KISS principles
- Celery role assumes application has been deployed

---
@title[Proposed setup]
## Proposed setup

+++?image=images/new_jenkins_1.png&size=auto 80%

+++?image=images/new_jenkins_2.png&size=auto 80%

+++?image=images/new_jenkins_3.png&size=auto 80%

+++

```text
ansible
├── roles
│   └── deploy
├── vars
│   └── api
│   │   ├── main.yml
│   │   ├── scheduler.yml
│   │   └── worker.yml
└── api.yml
Jenkinsfile
Makefile
```

+++

### Benefits

- Better deployment visibility and consistency
- No more assumptions
- Less configuration and management burden
- More generic structure to try new libraries (RQ, Dramatiq, etc)

---
@title[More to come]

## Future improvements

+++

### Application tarballs in Artifactory

Build once and forget...

+++

### Pipeline and playbook in application repo

```text
pipeline
├── artifactory
├── deployment.jenkinsfile
├── playbook.yml
├── vars.yml
└── requirements.yml
src
Makefile
```

+++

### Shift devops maintainability burden

- Deploy role in a separate repo
    * Versioned
    * More flexibility to apply changes

---

### All of this is up for discussion, feedback and improvements!

---

# Discussion / Questions?
