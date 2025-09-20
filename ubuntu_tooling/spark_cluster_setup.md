## Setup Spark Cluster with Ansible and Docker

**Project Outline**
```bash
~/ansible/spark-cluster/
├── hosts.ini                   # Inventory
├── spark-docker.yml            # Main playbook
├── group_vars/                 # Stored varriables
│   └── all.yml
├── templates/
│   ├── master-compose.yml.j2   # Docker-compose for master node
│   └── worker-compose.yml.j2   # Docker-compose for worker nodes
```
---

**hosts.ini**
- See Ansible setup file

---

**group_vars/all.yml**
```yaml
spark_version: 3.5.1
spark_master_host: 192.168.40.xxx
docker_network: spark-net
spark_image: bitnami/spark:{{ spark_version }}
```

---

**spark-docker.yml** # Template
```yaml
- name: Prepare Spark Docker cluster
  hosts: all
  become: true
  vars_files:
    - group_vars/all.yml

  tasks:
    - name: Install Docker and dependencies
      apt:
        name:
          - docker.io
          - docker-compose
        state: present
        update_cache: true

    - name: Ensure Docker service is running
      service:
        name: docker
        state: started
        enabled: true

    - name: Create Spark user-defined Docker network
      shell: "docker network create {{ docker_network }}"
      args:
        creates: "/var/lib/docker/network/files/{{ docker_network }}"
      ignore_errors: yes

    - name: Deploy Docker Compose file for master
      template:
        src: templates/master-compose.yml.j2
        dest: /home/michael/docker-compose.yml
      when: "'spark_master' in group_names"

    - name: Deploy Docker Compose file for worker
      template:
        src: templates/worker-compose.yml.j2
        dest: /home/michael/docker-compose.yml
      when: "'spark_workers' in group_names"

    - name: Launch Spark services
      shell: docker-compose -f /home/michael/docker-compose.yml up -d
      args:
        chdir: /home/michael
```
---

**templates/master-compose.yml.js** # Jinja templated docker-compose.yml for Spark container
```yaml
version: "3"

services:
  spark-master:
    image: {{ spark_image }}
    container_name: spark-master
    environment:
      - SPARK_MODE=master
    ports:
      - "7077:7077"
      - "8080:8080"
    networks:
      - {{ docker_network }}

networks:
  {{ docker_network }}:
    external: true
```


**templates/worker-compose.yml.j2** 
```yaml
version: "3"

services:
  spark-worker:
    image: {{ spark_image }}
    container_name: spark-worker
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://{{ spark_master_host }}:7077
    ports:
      - "8081:8081"
    networks:
      - {{ docker_network }}

networks:
  {{ docker_network }}:
    external: true
```

---

## Run the Playbook

```bash
ansible-playbook -i hosts.ini spark-docker.yml
```


## After Deployment
- Visitt the Spark Master UI: `http://192.168.40.xxx:8080`
- Should see both workers auto-regustered to the cluster
- Test a job

```bash
docker exec -it spark-master spark-submit --master spark://192.168.40.xxx:7077 examples/src/main/python/pi.py 10
```





