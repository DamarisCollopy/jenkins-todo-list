# Projeto Jenkins - Alura

# Etapa 1
1) Configure o seu ambiente inicial.

    Configurando o ambiente e conectando máquina virtual:
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
                    Nome de usuário: <nome>
                    Senha: <senha>
                    Nome completo: <Jenkins ....>
                    Email: nome@email.com.br

                # Reload nas permissoes do docker
                    sudo usermod -aG docker $USER
                    sudo usermod -aG docker jenkins
                    exit
            vagrant reload
# Etapa 2
Versione o seu código:

    Passos para a configuracao do git e versionamento do codigo
    
# Criar uma conta em github.com
    
    ssh-keygen -t rsa -b 4096 -C "<seu-usuario>@gmail.com"
    
    cat ~/.ssh/id_rsa.pub
    
# Configurar a chave no github
    git config --global user.name "<seu-usuario>"
    git config --global user.email <seu-usuario>@<seu-providor>
    ssh -T git@github.com
# Fazer download do código nos anexos e copiar para o diretório compartilhado app
    cd /vagrant/jenkins-todo-list
    git init
    git add .
    git commit -m "Meu primeiro commit"
    git log
# Criar um repositório no github: jenkins-todo-list
    git remote add origin git@github.com:<seu-usuario>/jenkins-todo-list.git
    git push -u origin master

# Etapa 3
Crie o seu primeiro job, que vai monitorar o seu repositório:

    Configurando a chave privada criada no ambiente da VM no Jenkins
    
    cat ~/.ssh/id_rsa
    Credentials -> Jenkins -> Global Credentials -> Add Crendentials -> SSH Username with private key [ github-ssh ]
    
    Criando o primeiro  job que vai monitorar o repositorio
    
        Novo job -> jenkins-todo-list-principal -> Freestyle project:
        Esse job vai fazer o build do projeto e registrar a imagem no repositório.
        
    # Gerenciamento de código fonte:
        Git: git@github.com:DamarisCollopy/jenkins-todo-list.git [SSH]
        Credentials: git (github-ssh)
        Branch: master
    # Trigger de builds
        Pool SCM: * * * * *
    # Ambiente de build
        Delete workspace before build starts
    # Salvar
    # Validar o log em: Git Log de consulta periódica

# Etapa 4 Automatizar usando Jenkins

- Intalando o plugins do doker no jenkins,  fazer o jenkins subir a imagem seguindo os parametros estabelecidos no Dokerfile

- Configure o daemon do Docke, isso deve ser feito na sua maquina virtual, e necessario para o jenkins poder controlar seu docker
- 
Expor o deamon do docker

    sudo mkdir -p /etc/systemd/system/docker.service.d/
    sudo vi /etc/systemd/system/docker.service.d/override.conf
        [Service]
        ExecStart=
        ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
    sudo systemctl daemon-reload
    sudo systemctl restart docker.service
    
Sugested Plugins

    Instalando os plugins
    
    Gerenciar Jenkins -> Gerenciar Plugins -> Disponíveis
        # docker
    Install without restart -> Depois reiniciar o jenkins
    
Gerenciar Jenkins -> Configurar o sistema -> Nuvem

    # Name: docker
    
    # URI: tcp://127.0.0.1:2376
    
    # Enabled
    
    # This project is parameterized: 
    DOCKER_HOST
    tcp://127.0.0.1:2376
    
    # Voltar no job criado anteriormente
    
    # Manter a mesma configuracao do GIT para desenvolvimento
    
    # Build step 1: Executar Shell
    
    # Validando a sintaxe do Dockerfile
    
        docker run --rm -i hadolint/hadolint < jenkins-todo-list/Dockerfile
    
    # Build step 2: Build/Publish Docker Image
    
        Directory for Dockerfile:  ./jenkins-todo-list/
        
        Cloud: docker
        
        Image: django_todolist_image_build
        
#  Etapa 5 - Confguracao de controle de chave 

- Utilizamos dois ambientes que é o ambiente de desenvolvimento e o ambiente de produção.O que diferencia um ambiente do outro? É somente o arquivo .env., esse arquivo .env contém o endereço do banco, usuário, senha, algumas chaves de configuração de criptografia. Então a gente tem que separar esses arquivos porque ele não está versionado no nosso GitHub. 

- Isso é muito importante porque é o que garante a segurança da nossa aplicação.

- Como é que a gente faz isso dentro do Jenkins? Nós vamos precisar de um plugin chamado Config File Provider.

 Instalar o plugin Config File Provider

 Configurar o Managed Files para Dev - banco de desenvolvmento
 
    # Name : .env-dev
        [config]
        # Secret configuration
        SECRET_KEY = ' '
        # conf
        DEBUG=True
        # Database
        DB_NAME = "todo_dev"
        DB_USER = "devops_dev"
        DB_PASSWORD = " "
        DB_HOST = "localhost"
        DB_PORT = "3306"

  Configurar o Managed Files para Prod, banco de producao
  
    # Name: .env.-prod
        [config]
        # Secret configuration
        SECRET_KEY = ' '
        # conf
        DEBUG=False
        # Database
        DB_NAME = "todo"
        DB_USER = "devops"
        DB_PASSWORD = " "
        DB_HOST = "localhost"
        DB_PORT = "3306"

    No job: jenkins-todo-list-principal importar o env de dev para teste:  Nesse caso, pra testar, nós vamos utilizar o ambiente de Dev, o arquivo .env de dev. Nunca se deve         apontar testes pra produção.

    Adicionar passo no build: Provide configuration Files
    
    File: .env-dev
    
    Target: ./to_do/.env

    Adicionar passo no build: Executar Shell

  Criando o Script para Subir o container com o arquivo de env e testar a app:
  
    #!/bin/sh

    # Subindo o container de teste // Repare no --name=django_todolist_image_build, esse e o nome do meu container
    
    docker run -d -p 82:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/jenkins-todo-list-                         principal/to_do/.env:/usr/src/app/to_do/.env --name=todo-list-teste django_todolist_image_build

    # Testando a imagem
    
    docker exec -i todo-list-teste python manage.py test --keep
    exit_code=$?

    # Derrubando o container velho
    
    docker rm -f todo-list-teste

    if [ $exit_code -ne 0 ]; then
        exit 1

    fi

- A abordagem escolhida permite que a mesma imagem construída no processo seja utilizada para testes, validações e também para rodar em produção, visto que o arquivo .env de       cada ambiente
  é passado na execução do container

# Etapa 5 Parametros para subir no dockerhub

 Instalar o plugin: Parameterized Trigger 

 Modificar o Job para startar com 2 parametros:
    # Geral:
    Este build é parametrizado com 2 parametros de string
    
        Nome: image
        Valor padrão: <seu-usuario-no-dockerhub>/django_todolist_image_build

        Nome: DOCKER_HOST
        Valor padrão: tcp://127.0.0.1:2376

 No build step: Build / Publish Docker Image
 
    # Mudar o nome da imagem para: <seu-usuario-no-dockerhub>/django_todolist_image_build
    
    # Marcar: Push Image e configurar **suas credenciais** no dockerhub

 Mudar no job de teste a imagem para: ${image}
 
    docker run -d -p 82:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/jenkins-todo-list       principal/to_do/.env:/usr/src/app/to_do/.env --name=todo-list-teste ${image}
    

Etapa 6 - Integracao com o slack

   - Agora a gente vai começar a montar o nosso job que vai fazer o deploy pra desenvolvimento, vai publicar nossa aplicação no ambiente de desenvolvimento.
     Pra isso a gente vai começar a configurar algumas partes do job que vão notificar o desenvolvedor que tá pronta a aplicação pra ele rodar.
   - Pra isso a gente vai usar o Slack, o Slack é uma das ferramentas mais comuns de comunicação que a gente tem no mercado, e ela é simples de trabalhar e tem uma integração
     boa com o Jenkins.
   - Link Slack https://app.slack.com

    # Criar app no slack: cursoalurajenkinssede.slack.com
    
    URL básico: <Url do Jenkins app no seu canal do Slack>
    Token de integração: <Token do Jenkins app no seu canal do Slack>

    # Instalar o plugin do slack: Gerenciar Jenkins > Gerenciar Plugins > Disponíveis: Slack Notification
    
        # Configurar no jenkins: Gerenciar Jenkins > Configuraçao o sistema > Global Slack Notifier Settings
        
            # Slack compatible app URL (optional): <Url do Jenkins app no seu canal do Slack>
            
            # Integration Token Credential ID : ADD > Jenkins > Secret Text
            
                # Secret: <Token do Jenkins app no seu canal do Slack>
                # ID: slack-token
                
            # Channel or Slack ID: pipeline-todolist

    # As notificações vão funcionar da seguinte maneira:
    
    Job: todo-list-desenvolvimento será feito pelo Jenkinsfile (em andamento)
    Job: todo-list-producao: Ações de pós-build > Slack Notifications: Notify Success e Notify Every Failure

