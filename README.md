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
