# Implementação do E-SUS com Docker
<hr>

Esse tutorial é baseado no repo do [Flávio Souza](https://github.com/FlavioSouzaSantos/eSUS-Docker). Duas imagens são criadas, uma de banco de dados e a outra do servidor. Em cada pasta desse repo está os arquivos **Dockerfile**. Se você ainda não possui o Docker na sua máquina acesse o [site oficial](https://docs.docker.com/engine/install/) ou siga algum tutorial de instalação.

A imagem do banco de dados é baseada no PostgreSQL 9.6.13-alpine. Para construir a imagem acesse a pasta **database** e faça a build rodando o comando:

```sudo docker build -t esus_database:1.0 .```

> Lembre-se que no final do comando precisa conter um ponto (.)

Para criar a imagem do webserver, como o próprio Flávio especifica, o sitema precisa do *systemd*, um gerenciador de sistemas e serviços para o sistema operacional Linux. Por isso, a imagem criada tem como base a imagem do [**centos/systemd**](https://hub.docker.com/r/centos/systemd/). O processo é semelhante ao anterior, acesse a pasta **webserver** e rode o comando:

```sudo docker build -t esus_webserver:1.0 .```

> Lembre-se que no final do comando precisa conter um ponto (.)

Os arquivos **Dockerfile** presente nas pastas são instruções para o Docker
montar as imagens. Se você tem dúvidas ou quer aprender mais acesse a [documentação de referência](https://docs.docker.com/reference/dockerfile/).

As imagens foram criadas, mas o processo ainda não terminou, é preciso que ambas se comuniquem. O Docker oferece um sistema de [*bridge*](https://docs.docker.com/engine/network/), uma network que permite os dois containers se comunicar. Vamos criar essa rede usando o comando:

```sudo docker network create esus_network```

As imagens foram criadas e se comunicam. Cada imagem é um o modelo que serve para executar um container, enquanto o container é uma aplicação ou serviço de software que é executado. Por isso, precisamos criar os containers. O comando para criar o container de banco de dados é:

```sudo docker container run -d --name esus_container_database -p 5433:5432 --net esus_network --cpus=1 -m 1gb esus_database:1.0```

Basicamente estamos dizendo para o Docker criar o container com base na imagem do banco de dados e mapear a porta de acesso do PostgresSQL para rede virtual do Docker.

Vamos criar o container do servidor, como destacado pelo Flávio, o container do [**centos/systemd**](https://hub.docker.com/r/centos/systemd/) precisa ser executado com um parâmetro adicional: **--privileged**. Esse parâmetro executa o container no modo privilegiado. Além disso é preciso definir um *cgroup*, um recurso do kernel do Linux que permite organizar processos de forma hierárquica e distribuir os recursos do sistema entre eles. 

Como ele explica, é preciso 'criar um volume bindando o cgroup da máquina host para o container'. [Volumes](https://docs.docker.com/engine/storage/volumes/) são o mecanismo para persistir dados gerados e usados ​​por contêineres Docker. O resultado é o comando:

```sudo docker container run -d --name esus_container_webserver -v /sys/fs/cgroup/:/sys/fs/cgroup:ro --privileged --env APP_DB_URL=jdbc:postgresql://esus_teste_database:5432/esus --env APP_DB_USER=postgres --env APP_DB_PASSWORD=esus -p 8080:8080 --net esus_network --cpus=1 -m 2gb esus_webserver:1.0```

Agora com os containers criados é preciso baixar e instalar o E-SUS. No site oficial (https://sisaps.saude.gov.br/esus/), a última versão é a 5.2.46. Para acessar o container segue os comandos:

Acessar o container e o bash

```sudo docker exec -it esus_teste_webserver /bin/bash ```

Acesse a pasta downloads

```cd /home/downloads/ ```

Use o **wget** para fazer download

```wget https://arquivos.esusab.ufsc.br/PEC/qQYkhiuXlligrwmg/5.2.46/eSUS-AB-PEC-5.2.46-Linux64.jar --no-check-certificate```



