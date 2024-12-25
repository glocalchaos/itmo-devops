# Лабораторная №2
## Создание роли docker-role
`tasks/main.yml`

```yml
---
# tasks file for docker
- name: update
  apt:
    update_cache: yes
- name: Установить зависимости
  apt:
    name: 
      - apt-transport-https
      - ca-certificates
      - curl
      - software-properties-common
    state: present

- name: Добавить Docker GPG ключ
  apt_key:
    url: "{{ docker_gpg_apt_key_url }}"
    state: present

- name: Добавить Docker репозиторий
  apt_repository:
    repo: deb [arch="{{ arch }}"] "{{ docker_repo }}"

- name: Установить Docker
  apt:
    name: docker-ce
    state: "{{ docker_version }}"

- name: Убедиться, что Docker запущен
  service:
    name: docker
    state: started
    enabled: true
```

`defaults/main.yml` переменные по умолчанию
```yml
---
# defaults file for docker
docker_version: "latest"
arch: "amd64"
docker_gpg_apt_key_url: "https://download.docker.com/linux/ubuntu/gpg"
docker_repo: "https://download.docker.com/linux/ubuntu focal stable"
```

## Использование роли в основном проекте
`requirements.yml`

```yml
---
- name: docker-role
  src: https://gitlab.com/ansible-roles9394643/docker-role/
  scm: git
  version: main
```

`deploy_app.yml`
```yml
---
- hosts: app
  become: yes
  roles:
    - docker-role

- name: Убедиться, что git установлен
      become: yes
      apt:
        name: git
        state: present
        
    - name: start
      community.docker.docker_container:
        name: "{{ container_name }}"
        image: timurbabs/django
        state: started
        ports:
          - "{{ container_ports }}"
```