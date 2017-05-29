Provisioning a new site
=======================

## Required packages:

* nginx
* Python 3.6
* virtualenv + pip
* Git

ex. Ubuntu에서 실행하는 방법:

    sudo add-apt-repository ppa:fkrull/deadsnakes
    sudo apt-get install nginx git python36 python3.6-venv

## Nginx Virtual Host config

* nginx.template.conf 참고
* SITENAME 부분을 다음과 같이 수정 staging.my-domain.com

## Systemd service(Upstart)

* gunicorn-systemd.template.service 참고
* SITENAME 부분을 다음과 같이 수정 staging.my-domain.com

## Folder structure:
사용자 계정의 홈 폴더가 /home/username 이라고 가정

/home/username
└── sites
    └── SITENAME
         ├── database
         ├── source
         ├── static
         └── virtualenv
