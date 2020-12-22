# django-todolist

Simple todolist write in django for general use and pipeline automation..

  - Be kind with my baby

### Quick and free tip:

> With great power comes great responsibility


### Tech

Dillinger uses a number of open source projects to work properly:

* [Django] - Django makes it easier to build better Web apps more quickly and with less code.
* [Python-Venv] - The venv module provides support for creating lightweight “virtual environments” with their own site directories
* [MySQL] - MySQL is an Oracle-backed open source relational database management system (RDBMS) based on Structured Query Language (SQL).


### Installation

Install the dependencies and start the server.

```sh
$ cd django-todolist
$ pip install -r requirements.txt
$ python manage.py migrate # Running the migrations
$ python manage.py createsuperuser # Create a superuser
$ python manage.py runserver
```


License
----

GPL

### Configurando o ambiente e conectando máquina virtual:
    # Máquina virtual com o vagrant
    # Requisitos: http://vagrantup.com e https://www.virtualbox.org/
        # Baixar e descompactar o arquivo 1110-aula-inicial.zip
        # Entendendo o Vagranfile
        # Subindo o ambiente virtualizado
            vagrant plugin install vagrant-disksize
            vagrant up
            vagrant ssh
                ps -ef | grep -i mysql # Verificando se o MySQL esta rodando
                mysql -u devops -p # Senha mestre; show databases
                mysql -u devops_dev -p # Senha mestre; show databases
                # Instalando o Jenkins
                    cd /vagrant/scripts
                # Visualizar o conteudo do arquivo de instalacao do jenkins
                    sudo ./jenkins.sh

                # Acessar:  192.168.33.10:8080
                    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

                # Credenciais
                    Nome de usuário: alura
                    Senha: mestre123
                    Nome completo: Jenkins Alura
                    Email: aluno@alura.com.br

                # Reload nas permissoes do docker
                    sudo usermod -aG docker $USER
                    sudo usermod -aG docker jenkins
                    exit
            vagrant reload

### Passos para a configuracao do git e versionamento do codigo
### Criar uma conta em github.com
    ssh-keygen -t rsa -b 4096 -C "<seu-usuario>@gmail.com"
    cat ~/.ssh/id_rsa.pub
### Configurar a chave no github
    git config --global user.name "<seu-usuario>"
    git config --global user.email <seu-usuario>@<seu-providor>
    ssh -T git@github.com
### Fazer download do código nos anexos e copiar para o diretório compartilhado app
    cd /vagrant/jenkins-todo-list
    git init
    git add .
    git commit -m "Meu primeiro commit"
    git log
### Criar um repositório no github: jenkins-todo-list
    git remote add origin git@github.com:<seu-usuario>/jenkins-todo-list.git
    git push -u origin master

### Configurando a chave privada criada no ambiente da VM no Jenkins
    cat ~/.ssh/id_rsa
    Credentials -> Jenkins -> Global Credentials -> Add Crendentials -> SSH Username with private key [ github-ssh ]
### Criando o primeiro  job que vai monitorar o repositorio
    Novo job -> jenkins-todo-list-principal -> Freestyle project:
    Esse job vai fazer o build do projeto e registrar a imagem no repositório.
    ### Gerenciamento de código fonte:
        Git: git@github.com:rafaelvzago/treinamento-devops-alura.git [SSH]
        Credentials: git (github-ssh)
        Branch: master
    ### Trigger de builds
        Pool SCM: * * * * *
    ### Ambiente de build
        Delete workspace before build starts
    ### Salvar
    ### Validar o log em: Git Log de consulta periódica    

### Expor o deamon do docker
    sudo mkdir -p /etc/systemd/system/docker.service.d/
    sudo vi /etc/systemd/system/docker.service.d/override.conf
        [Service]
        ExecStart=
        ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376

    sudo systemctl daemon-reload
    sudo systemctl restart docker.service

### Sugested Plugins
### Instalando os plugins
    Gerenciar Jenkins -> Gerenciar Plugins -> Disponíveis
        # docker
    Install without restart -> Depois reiniciar o jenkins
Gerenciar Jenkins -> Configurar o sistema -> Nuvem
    # Name: docker
    # URI: tcp://127.0.0.1:2376
    # Enabled
### This project is parameterized: 
    DOCKER_HOST
    tcp://127.0.0.1:2376
### Voltar no job criado na aula anterior
    # Manter a mesma configuracao do GIT para desenvolvimento
    # Build step 1: Executar Shell
### Validando a sintaxe do Dockerfile
docker run --rm -i hadolint/hadolint < Dockerfile
    # Build step 2: Build/Publish Docker Image
        Directory for Dockerfile: ./
        Cloud: docker
        Image: rafaelvzago/django_todolist_image_build

### Instalar o plugin Config File Provider

### Configurar o Managed Files para Dev
    # Name : .env-dev
        [config]
        # Secret configuration
        SECRET_KEY = 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'
        # conf
        DEBUG=True
        # Database
        DB_NAME = "todo_dev"
        DB_USER = "devops_dev"
        DB_PASSWORD = "mestre"
        DB_HOST = "localhost"
        DB_PORT = "3306"

### Configurar o Managed Files para Prod
    # Name: .env.-prod
        [config]
        # Secret configuration
        SECRET_KEY = 'r*5ltfzw-61ksdm41fuul8+hxs$86yo9%k1%k=(!@=-wv4qtyv'
        # conf
        DEBUG=False
        # Database
        DB_NAME = "todo"
        DB_USER = "devops"
        DB_PASSWORD = "mestre"
        DB_HOST = "localhost"
        DB_PORT = "3306"

### No job: jenkins-todo-list-principal importar o env de dev para teste:

    Adicionar passo no build: Provide configuration Files
    File: .env-dev
    Target: ./to_do/.env

    Adicionar passo no build: Executar Shell

### Criando o Script para Subir o container com o arquivo de env e testar a app:
    #!/bin/sh

    # Subindo o container de teste
    docker run -d -p 82:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/jenkins-todo-list-principal/to_do/.env:/usr/src/app/to_do/.env --name=todo-list-teste django_todolist_image_build

    # Testando a imagem
    docker exec -i todo-list-teste python manage.py test --keep
    exit_code=$?

    # Derrubando o container velho
    docker rm -f todo-list-teste

    if [ $exit_code -ne 0 ]; then
        exit 1
    fi        