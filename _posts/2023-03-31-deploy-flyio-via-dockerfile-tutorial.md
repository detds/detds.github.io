---
title: "Como fazer deploy de uma aplicação Java Spring Boot no fly.io: um passo a passo simples"
permalink: /deply-flyio-via-dockerfile-tutorial
layout: single
header:
  overlay_color: "#00ADB5"
author_profile: true
toc: true
toc_sticky: true
excerpt: Um breve tutorial de como fazer um deploy de uma aplicação no Fly.io utilizando Dockerfile.
---

![deply-flyio-image](/assets/images/posts/2023-03-31-deploy-flyio-via-dockerfile-tutorial-image.jpg){: .align-left} Este artigo apresenta um passo a passo simples para fazer o deploy de uma aplicação **Java Spring Boot** na plataforma de computação em nuvem **fly.io** via **Dockerfile**. 

O Fly.io pode ser considerado uma ótima **alternativa ao Heroku**, principalmente para quem busca **uma solução gratuita** para o deploy de aplicações Java, uma vez que o Heroku, atualmente, exige o pagamento para o uso de seus serviços. Ele oferece recursos para hospedagem e execução de bancos de dados em sua plataforma, incluindo a opção de executar bancos de dados em contêineres Docker, além de oferecer serviços de banco de dados gerenciados, como o Fly Postgres (PostgreSQL), Redis e MySQL. Com esses recursos, é possível executar e escalar bancos de dados em um ambiente de hospedagem distribuído e altamente disponível.

Você pode acessar um dos meus projetos no github [clicando aqui](https://github.com/detds/project-spring-java-17) onde fiz o deploy de uma aplicação Java com web serviços, utilizando Spring Boot e banco de dados PostgreSQL,


## 1. Instalação do Flyctl

Acesse a [documentação do site oficial do Fly.io](https://fly.io/docs/getting-started/installing-flyctl/) e siga as instruções de instalação de acordo com o sistema operacional utilizado.


## 2. Configuração do Flyctl

Após a instalação do Flyctl, abra o terminal e execute o comando:

```shell
$ flyctl auth login
```

## 3. Criar o aplicativo Postgres

Antes de criarmos o banco de dados, é necessário criar um aplicativo Postgres na plataforma.

No terminal, execute o comando a seguir para criar o aplicativo postgres no fly.io:

```shell
$ flyctl postgres create
```

Será solicitado que você escolha um nome para o aplicativo postgres e pressione Enter.

Em seguida, escolha uma região para implantação do aplicativo. Escolha a região que desejar digitando o número correspondente e pressionando Enter.

> **Observação:** Anote as credenciais de acesso ao banco de dados exibidas na tela, pois você não terá outra chance de vê-las.

Para verificar se o volume foi criado com sucesso, execute o seguinte comando:


```shell
$ volumes list -a NOME_DO_APLICATIVO
```

Esse comando exibirá a lista de volumes associados ao seu aplicativo. Certifique-se de que o volume que você acabou de criar está presente na lista. Se estiver, significa que o volume foi criado com sucesso e está pronto para ser utilizado no deploy da aplicação.

## 4. Criar o banco de dados

Agora que o aplicativo Postgres está criado, podemos criar o banco de dados dentro dele. Vamos utilizar o **pgAdmin**, uma ferramenta gráfica de administração do Postgres. É necessário configurar o proxy do aplicativo Postgres no Fly.io com a máquina local. Para configurar o proxy, execute o seguinte comando:

```shell
$ flyctl proxy PORTA_LOCAL:PORTA_FLY -a NOME_DO_APLICATIVO
```

Substitua "PORTA_LOCAL" por uma porta disponível em seu computador que será usada para se comunicar com o proxy e "PORTA_FLY" pela porta do banco de dados Postgres no Fly.io, que geralmente é 5432. Substitua também "NOME_DO_APLICATIVO" pelo nome do aplicativo que você criou no [Passo 3](#3-criar-o-aplicativo-postgres). Esse comando abrirá uma conexão entre a porta local escolhida e o aplicativo Postgres no Fly.io. 

> Mantenha o comando em execução enquanto estiver usando o pgAdmin.

Abra o pgAdmin, faça login usando as credenciais que você criou no [Passo 3](#3-criar-o-aplicativo-postgres). Selecione o servidor Postgres que foi criado no Fly.io e clique em "Create" para criar um novo banco de dados.

Digite o nome do banco de dados que deseja criar e clique em "Save". Agora, você pode verificar se o banco de dados foi criado com sucesso, selecionando o servidor Postgres e verificando se o novo banco de dados está na lista de bancos de dados disponíveis.

Com o ambiente preparado, agora podemos seguir para o processo de deploy da aplicação Java Spring Boot.

## 5. Configurar aplicação Spring Boot para se conectar ao banco de dados

Agora que o banco de dados está configurado e em execução, precisamos atualizar nossa aplicação Spring Boot para se conectar a ele. 

> É importante lembrar que as informações de conexão do banco de dados, como nome do banco, nome de usuário e senha, devem ser armazenadas de forma segura e não expostas em código ou arquivos.

Vamos atualizar o arquivo `application.properties` da nossa aplicação Spring Boot para utilizar variáveis de ambiente. Abra o arquivo e adicione as seguintes linhas:

```java
spring.datasource.url=${DATASOURCE_URL}
spring.datasource.username=${DATASOURCE_USERNAME}
spring.datasource.password=${DATASOURCE_PASSWORD}
```

Dessa forma, a aplicação usará as informações de conexão do banco de dados definidas nas variáveis de ambiente.

Agora nossa aplicação está configurada para se conectar ao banco de dados Postgres no fly.io.


## 6. Criar o arquivo JAR

Para fazer o deploy da nossa aplicação Java Spring Boot no fly.io, precisamos criar o arquivo JAR que contém o código compilado e todos os recursos necessários para rodar a aplicação. 

Antes de gerarmos o arquivo JAR da nossa aplicação, é necessário que as variáveis de ambiente estejam configuradas corretamente no sistema. Caso contrário, podem ocorrer erros durante a compiação. Para isso, podemos usar os comandos abaixo para setar as variáveis no ambiente, lembrando que essas variáveis devem estar compatíveis com as que foram utilizadas no arquivo `application.properties`:

> Observação: para evitar que os comandos `export` fiquem salvos no histórico do terminal, é possível utilizar um espaço antes do comando.

```shell
$  export DATASOURCE_URL=jdbc:postgresql://<HOST>:<PORT>/<DBNAME>
$  export DATASOURCE_USERNAME=<USER>
$  export DATASOURCE_PASSWORD=<PASSWORD>
```

Substitua `<USER>`, `<PASSWORD>`, `<HOST>`, `<PORT>` e `<DBNAME>` pelos valores correspondentes do seu banco de dados Postgres.

Uma vez que as variáveis de ambiente estejam configuradas, podemos gerar o arquivo JAR com o Maven. Certifique-se de que você está no diretório raiz do seu projeto e execute o seguinte comando:

```shell
$ mvn clean package
```

Esse comando irá executar a fase de empacotamento do Maven e gerar o arquivo JAR no diretório `target`. Verifique se o arquivo JAR foi gerado com sucesso antes de prosseguir para o próximo passo.

## 7. Criar o Dockerfile

Para fazer o deploy da aplicação no Fly.io, precisamos criar um arquivo **Dockerfile** que irá conter as configurações necessárias para construir a imagem da nossa aplicação. O Dockerfile é importante porque o Fly.io utiliza o Docker como tecnologia base para o deploy das aplicações, então é necessário seguir a sua estrutura e especificações.

Uma das coisas que precisamos definir no Dockerfile é a imagem base a ser utilizada. Essa imagem precisa ser compatível com a versão do Java definida na aplicação. Geralmente, as imagens mais comuns são baseadas em distribuições Linux, como o **Alpine Linux** ou o **Ubuntu**. Além disso, o tamanho da imagem base pode afetar o desempenho e a utilização de memória da aplicação. Imagens menores geralmente são mais rápidas e consomem menos recursos, enquanto imagens maiores podem ser mais lentas e exigir mais recursos. No entanto, é importante lembrar que algumas imagens serão depreciadas com o tempo, então é importante verificar a documentação e manter a imagem atualizada. A escolha de imagens base oficiais pode ser feita no Docker Hub.

Crie um arquivo chamado `Dockerfile` na raiz do projeto. Adicione as seguintes linhas de código no Dockerfile:

```dockerfile
# Define a imagem base
FROM amazoncorretto:17-alpine-jdk

# Define as variáveis de ambiente
ENV DATASOURCE_URL=${DATASOURCE_URL}
ENV DATASOURCE_USERNAME=${DATASOURCE_USERNAME}
ENV DATASOURCE_PASSWORD=${DATASOURCE_PASSWORD}

# Define o diretório de trabalho dentro do container
WORKDIR /app

# Copia o arquivo jar do projeto para dentro do container
COPY target/nome-do-arquivo.jar /app/app.jar

# Define a porta em que a aplicação estará disponível
EXPOSE 8080

# Define o comando de inicialização da aplicação
CMD ["java", "-jar", "/app/app.jar"]
```

Certifique-se de substituir nome-do-arquivo.jar pelo nome do arquivo gerado no [passo 6](#6-criar-o-arquivo-jar).

## 8. Criar a aplicação Fly.io 

Agora vamos criar a aplicação no Fly e configurar o arquivo `fly.toml`. Para isso, utilize o comando `fly launch` na raiz do projeto para criar e configurar o arquivo `fly.toml`.

No arquivo `fly.toml`, vamos definir a seção `[env]` com as informações das variáveis de ambiente compatíveis com o Dockerfile e com as variáveis setadas no fly. Isso é importante para que a aplicação possa se conectar corretamente ao banco de dados.

Para configurar as variáveis de ambiente no arquivo `fly.toml`, adicione as seguintes linhas:

```toml
[env]
DATASOURCE_URL = ${SPRING_DATASOURCE_URL}
DATASOURCE_USERNAME = ${SPRING_DATASOURCE_USERNAME}
DATASOURCE_PASSWORD = ${SPRING_DATASOURCE_PASSWORD}
```

Agora, vamos configurar as variáveis de ambiente usando o comando flyctl secrets set. Execute os seguintes comandos no terminal:

```shell
$ flyctl secrets set SPRING_DATASOURCE_URL=jdbc:postgresql://<HOST>:<PORT>/<DBNAME>
$ flyctl secrets set SPRING_DATASOURCE_USERNAME=<USER>
$ flyctl secrets set SPRING_DATASOURCE_PASSWORD=<PASSWORD>
```

> Observação: como dito antes, você pode utilizar espaço antes dos comandos para que não fiquem salvos no histórico do terminal.

Lembre-se de substituir `<USER>`, `<PASSWORD>`, `<HOST>`, `<PORT>` e `<DBNAME>` pelas informações correspondentes ao seu banco de dados postgres.

Por fim, salve e feche o arquivo `fly.toml`. Pronto, agora a sua aplicação está pronta para ser implantada no Fly.

## 9. Deploy da aplicação

Agora que temos tudo configurado, podemos fazer o deploy da nossa aplicação com o seguinte comando:

```shell
$ flyctl deploy
```

Aguarde até que a aplicação seja implantada. Isso poderá demorar alguns minutos.

Quando o deploy estiver finalizado e se tudo estiver configurado corretamente, você poderá acessar sua aplicação. Para abrir a aplicação em um browser, utilize o comando:

```shell
$ flyctl open
```

## 10. Conclusão

Podemos dizer que o Fly.io oferece uma maneira simples e eficiente de fazer o deploy de aplicativos Java utilizando Dockerfile. É possível criar e configurar um aplicativo e um banco de dados PostgreSQL em poucos minutos. A utilização de variáveis de ambiente tanto no arquivo de configuração do Spring Boot quanto no Dockerfile e no aplicativo Fly, possibilita a segurança das informações sensíveis do banco de dados, evitando que as credenciais sejam expostas publicamente.

Além disso, o Fly.io possui um dashboard que permite monitorar e gerenciar o aplicativo em tempo real, oferecendo informações sobre o uso de recursos, logs e muito mais. Com a plataforma, podemos escalar facilmente a aplicação, adicionar recursos de balanceamento de carga, entre outras funcionalidades. Tudo isso sem precisar se preocupar com a infraestrutura ou com as complexidades do gerenciamento de servidores. Além disso, dependendo do plano escolhido, é possível fazer deploy gratuitamente, tornando-o uma ótima alternativa ao Heroku e outros serviços de hospedagem.

Em resumo, a utilização do Fly.io é uma opção interessante para quem busca uma plataforma eficiente, segura e fácil de usar para o deploy de aplicações Java e outros tipos de aplicativos.

## 11. Links úteis

Você pode encontrar mais informações sobre o Fly.io em seu site oficial. Eles também possuem uma documentação bem detalhada e uma comunidade ativa no fórum. Além disso, eles têm uma conta no Twitter onde compartilham atualizações e notícias.

Seguem os links para acessar mais informações sobre o Fly.io:

- Site oficial: [https://fly.io/](https://fly.io/)
- Documentação: [https://fly.io/docs/](https://fly.io/docs/)
- Comunidade: [https://community.fly.io/](https://community.fly.io/)
- Twitter: [https://twitter.com/flydotio](https://twitter.com/flydotio)

Espero que ajude!