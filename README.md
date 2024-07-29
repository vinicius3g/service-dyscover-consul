## Agent Consul
Esse é um projeto de estudo sobre a ferramenta consul que gerencia serivores
o objetivo desse estudo é: 

- criar um cluster de servidores

  - consulserver01
  - consulserver02
  - consulserver03

e fazer os 3 se comunicarem

### Iniciando
para iniciar o projeto suba a imagem do docker com o comando

``docker compose up -d``

para entrar no container do consul use essa linha de comando

``docker compose exec -it consulserver01 sh``

estando dentro do container rode o seguinte comando para poder iniciar um servidor 

``consul agent -dev``

tendo sucesso voce tera uma mensagem dessa dorma no terminal

``Starting Consul agent...
           Version: '1.10.12'
           Node ID: 'a6e385b8-3a95-8579-8420-0ee15113bd54'
         Node name: 'consul01'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, DNS: 8600)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false, Auto-Encrypt-TLS: false``

agora estando dentro do container para verificar o status da conexão digite a seguinte linha de comando em outro terminal 

### Criação de clusters
***ps: cada uma das configuracoes/comandos expostos aqui devem ser criados em cada um dos serviços:***

**consulserve01**
**consulserve02**
**consulserve03**

tendo subido o container entreem um dos servidores disponiveis

``docker compose exec -it consulserver01 sh``

verifique o ip com o comando 

``ifconfig``

voce tera um retorno parecedo com esse:

``eth0      Link encap:Ethernet  HWaddr 02:42:AC:1B:00:03  
          inet addr:172.27.0.3  Bcast:172.27.255.255  Mask:255.255.0.0``

estando ainda dentro do container crie duas pastas de configuraçao do consul nesses caminhos

``mkdir /etc/consul.d``

``mkdir /var/lib/consul``

tendo  feito esses passos vamos configurar o servidor
com essa linha de comando

``consul agent -server -bootstrap-expect=3 -node=consulserver01 -bind=172.27.0.4 -data-dir=/var/lib/consul -config-dir=/etc/consul.d``

explicaçao dos comandos:

**consul agent -server** => que tipo der serviço voce quer subir, no caso do tipo server se a flag não for passada ele ira como client
**-bootsstrap-expect=3** => com quantos serviços voce quer se comunicar
**-node=consulserver01** =>  define o nome do node, se não from passado ele dara um nome randomico
**-bind=172.27.0.3** => define o ip de comunicação
**-data-dir=/var/lib/consul** => diretorio onde o consul guarda os arquivos
**-config-dir=/etc/consul.d** =>  caminhos onde são gardados os arquivos de configuraçao
**-retry-join=172.27.0.3** => ja conecta com o cluster 

para facilitar e agilizar o processo de configuraçao dos servidores foi criado um arquivo de configuraçao dentro da pasta /servers/server01 ao qual facilita a criaçao não precisando passar o comando inteiro, tendo essa  psat com esse arquivo de confugiraçao criado após ter o container em pé basta entrar no mesmo e rodar o seguinte comando para pegar as configs:

``consul agent -config-dir=/etc/consul.d``

logo após

``consul members``

se tudo correu bem até o momento voce tera o seguinte resultado 

``Node            Address          Status  Type    Build    Protocol  DC   Segment
consulserver01  172.27.0.3:8301  alive   server  1.10.12  2         dc1  <all>``

para conectar um server ao outro basta rodar esse comando:

``consul join 172.27.0.4`` => ip do server

### Primeiro Client

Configurando o consulclient01

com o container rodando entre no consulclient01 e passe o seguinte comando 

``consul agent -bind=172.27.0.5 -data-dir=/var/lib/consul -config-dir=/etc/consul.d``

feito isso ele ira reconhecer o novo serviço como um client, logo após isso rode o comando join para conectar aos demais serviços

### Registrando serviço

Dentro da pasta /clients/consul01 existe um arquivo chamado services.json ao qual é responsavel pelo cadastro de todos os serviços que serão criado e utilizados.

### Trabalhando com criptografia

 para deixar o ambiente e a conexão entre os servers mais segura é aconselhavel implementar um keygen de criptografia que pode ser gerado pelo seguinte comando 

 ``consul keygen``

 fazendo isso basta colocar dentro do arquivo do server no campo encrypt

### User interface e dicas para produção

Com **client_addr** e  **ui_config** adicionado ao arquivo de server é possivel acessar o user interface do consul para fazer manipulaçoes e configuraçoes.
depois de todas configuraçoes feitas e o container rodando bastar acessar a **localhots:5800** e ira aparecer o iu para visualizaçao.

Para rodar em produção o minimo que se deve ter é:

  - TLS
  - minimo de 3 servers
  - encrypt