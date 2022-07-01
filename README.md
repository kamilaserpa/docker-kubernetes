# Docker | Kubernetes

Estudo de ferramentas do Docker e Kubernetes em acompanhamento do curso [Docker para Desenvolvedores (com Docker Swarm e Kubernetes)](https://www.udemy.com/course/docker-para-desenvolvedores-com-docker-swarm-e-kubernetes/).
]
Arquivos utilizados no decorrer do curso: https://github.com/matheusbattisti/curso_docker.

## Introdução

### O que é Docker?
O Docker é um software que reduz a complexidade de setup de aplicações.
É onde configuramos containers, que funcionam com servidores para rodar nossas aplicações.
Permite criar ambientes independentes e que funcionam em diversos SO's com facilidade.

### Por que Docker?
O Docker proporciona mais velocidade na configuração do ambiente. Proporciona maior performance para executar uma aplicação do que uma Virtual Machine.

Nos livra da Matrix from Hell.
Numa empresa podem existir diversos projetos utilizando tecnologias distintas, estes projetos devem funcionar em diversos ambientes, por exemplo, na máquina dos desenvolvedores, ambiente de homologação, produção, etc. A mesma aplicação acaba sendo instalada de formas diferentes em váriso ambientes. Dessa forma a alteração de versão de uma tecnologia, ou adição de uma nova stack torna necessária alterações em todos os ambientes. O ingresso de um novo desenvolvedor demanda um dia de trabalho ou mais apenas para instalação da stack do ambiente de desenvolvimento. Com o Docker é possível configurar as imagens de ambiente de cada aplicação e executar os containers necessários.

![Matrix from Hell](_images/matrix_from_hell.png)

No Windows indica-se o `Cmder` como alternativa ao terminal, pois simula um terminal de Linux, aceitando todos os comandos compatíveis com o Docker também.

## Trabalhando com Containers

Container é um pacote de código que pode executar uma ação. Containers utilizam imagens para serem executados.
<br>Imagem</br> é o "projeto" que será executado pelo container, todas as instruções ficam declaradas nela.

Container é o Docker executando uma imagem.

Repositório Docker para encontrar imagens: https://hub.docker.com/.

### Container vs Virtual Machine (VM)

Container é uma aplicação que serve para um derminado fim. Seu tamanho é de MBs. Utilizam menos recursos e tem usos específicos, isolando funcionalidades. <br>
Já uma VM possui sistema operacional próprio, tamanho de GBs e pode executar diversas funções ao mesmo tempo, utilizando mais recursos.

### Alguns comandos


 - Verificar containers executados
   - `docker ps` ou `docker container ls` - exibe containers em execução
   - `docker ps -a` ou `docker container ls -a` - exibe todos os containers já executados.
  
 - Modo interativo
   - `docker run -it <imagem>`, a flag `-it` pode ser adicionada ao comando `run` possibilita executar comandos disponíveis no container através de terminal interativo, ex.: `docker run -it node`.

 - Execução em background
   - `docker run nginx -d`, ao adicionar a flag `-d` (detached) o terminal fica livre para receber novos comandos.

 - Expondo porta
   - containers docker não tem conexão com nada externo a eles. A flag `-p <porta_local_de_acesso>:<porta_do_container>` expõe portas para que o container fique acessível, ex.: `docker run -d -p 3000:80 nginx`, sendo possível acessar o serviço em "http://localhost:3000".

 - Parando containers
   - `docker stop <id ou nome>`, para a execução do container e libera recursos por ele utilizados

 - Reiniciando containers
   - `docker start <id ou nome>` volta a execução de um container. O comando `run` sempre cria um container novo.

 - Definindo nome do container
   - Adicionando a flag `--name` ao comando "run" definimos um nome, se não colocarmos o container recebe um nome aleatório, ex.: `docker run -d -p 80:80 --name nginx_app nginx`.

 - Verificando logs de um container
   - Podemos verificar o que aconteceu em um container através do comando `docker logs <id ou nome>`.
   - `docker logs -f <id ou nome>`, com a flag "-f" (follow) mostra os logs em tempo real. Ctrl+C para sair.

 - Removendo containers
   - `docker rm <id ou nome>`. Se o container estiver em execução podemos adicionar a flag `-f`(force) que para o container e em seguida o remove da memória local, ex.: `docker rm <id ou nome> -f`.

[docker run command line documentation](https://docs.docker.com/engine/reference/commandline/run/).

### Mais informações sobre Comandos
Todo comando no Docker tem acesso a uma flag `--help`. Desse modo visualizamos todas as opções disponíveis, por exemplo `docker run --help`.

## Imagens

Imagens são originadas de arquivos programados para que o Docker crie uma estrutura, essa estrutura pode execitar determinadas ações em containers.
Ao executar um container baseado numa imagem, as instruções serão executadas em camadas. Por exemplo, caso haja uma alteração no arquivo Dockerfile na linha _x_, apenas dessa linha em diante é que as instruções serão reexecutadas.

Imagens podem ser baixadas em https://hub.docker.com, porém qualquer pessoa pode fazer upload de uma imagem, portanto é importante observar imagens oficiais, quantidade de downloads e quantidade de stars para então optar pela utilização de uma imagem específica.

### Criando uma imagem e avançando em containers

Para criar uma imagem é necessário um arquivo `Dockerfile` dentro do projeto.
Esse arquivo deve conter instruções para serem executadas:
 - FROM: imagem base
 - WORKDIR: diretório da aplicação
 - EXPOSE: porta da palicação
 - COPY: quais arquivos precisam ser copiados

Utilizamos o _node_ para criar uma imagem de exemplo, instalamos o express e executamos a aplicação localmente com os seguintes comandos:

```sh
  npm init -y
  npm install express
  node app.js
```

Na pasta [1_imagens_e_containers](1_imagens_e_containers) adicionamos o arquivo `Dockerfile`:

```dockerfile
  FROM node --> utiliza imagem do node
  WORKDIR /app --> doretorio que executa a aplicação no  container
  COPY package*.json ./ --> copia arquivos package.json para o diretório raiz app
  RUN npm install --> instala as dependẽncias definida sno package.json
  COPY . . --> copia os demais arquivos da aplicação para o container
  EXPOSE 3000 --> expões porta 3000, condizente com a aplicação
  CMD ["node", "app.js"] --> comando que inicializa a aplicação
```

### Executando uma imagem

Precisamos construir a imagem através do comando `docker build <diretório da imagem`>.
Em seguida executamos com `docker run <id ou nome>`. <br>
No nosso exemplo acessamos a pasta [1_imagens_e_containers](1_imagens_e_containers) pelo terminal e executamos o comando abaixo para criar a imagem ainda sem nome e tag:
  `docker build .`

Capturando id da imagem: `docker imagem ls`.

Executando um container a partir da imagem criada:
`docker -d -p 3000:3000 <id da imagem>`.

É possível criar container nomeado: 
`docker -d -p 3000:3000 --name meu_node <id da imagem>`.

Após isso a mensagem estabelecida em [app.js](1_imagens_e_containers/app.js) deve estar disponível no navegador em _http://localhost:3000_.

Para remover uma imagem execute: `docker rmi <id da imagem>`.

### Camadas das imagens

As imagens do Docker são divididas em camadas (layers).
Cada instrução no DOckerfile representa uma _layer_. 
Quando algo é atualizado, apenas as _layers_ após a linha alterada são refeitas.
O restante permanece em cache (as instruções nas linhas anteriores à alteração), tornando o build mais rápido.

### Download de imagens
Podemos fazer o download de alguma imagem do hub e deixá-la disponível em nosso ambiente.
Utilizando o comando `docker pull <nome da imagem>`.
Por exemplo podemos baixar a imagem do python: `docker pull python`.

### Múltiplas aplicações, mesmo container

Podemos inicializar vários containers com a mesma imagem. As aplicações funcionarão em paralelo.
Para testar podemos determinar uma porta diferente para cada aplicação e executar em modo _detached_.
`docker run -d -p 3000:3000 --name meu_node1 <id da imagem>`
`docker run -d -p 4000:3000 --name meu_node2 <id da imagem>`

### Alterando o nome da imagem e tag
O comando `docker tag <id da imagem> <novo nome da imagem>` é utilizado para nomear as imagens.
Também é possível modificar a tag, que funciona como uma versão da imagem com `docker tag <id da imagem> <nome>:<tag>`.
Para visualizar as imagens utilize `docker images`.
Por exemplo: 
`docker tag 73daf1d5c391 minhaimagem`
`docker tag 73daf1d5c391 minhaimagem:tag1`

## Iniciando imagem com um nome
Podemos nomear a imagem no momento da sua criação com a flag `-t`.
Por exemplo: 
 - `docker build -t meu_node3 .`
 - `docker build -t meu_node3: tag1 .`

### Reiniciando container com interatividade
Com o comando `start`é possível utilizar um container criado anteriormente, sem criar um novo. A flag `-it`pode ser utilizada com o comando _start_ também.
O comando é `docker start -it <container>`, por exemplo: 
 - `docker stop meu_node2` para a execução do container de nome "meu_node2", pode ser utilizado o id também
 - `docker ps -a` lista os caontainers existentes
 - `docker start meu_node2` reinicializa o container meu_node2
 - `docker start -i meu_node2` reinicializa o container meu_node2 habilitando o terminal interativo. `Ctrl+c` encerra a interação.

### Removendo imagens
O comando `docker rmi <id da imagem>` remove uma imagem específica.
Ao tentar remover imagens que estão sendo utilizadas por um container, é apresentado um erro no terminal.
A flag `-f` pode ser utilizada para forçar a remoção da imagem, por exemplo: `docker rmi -f <nome>:<tag>`.
