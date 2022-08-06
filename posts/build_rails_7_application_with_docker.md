---
layout: default
---

### Junior Tada Blog 👋

## Criando aplicação demo Ruby on Rails 7 com Docker

Olá, este guia tem por objetivo criar um hello world 
com Ruby on Rails 7 dentro de um container Docker.

Primeiro vamos criar um arquivo com o nome Dockerfile
```sh
# Dockerfile
FROM ruby:3.1.2

# Copy the main application.
RUN mkdir -p /app
COPY . /app
WORKDIR /app

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
COPY Gemfile /app/Gemfile
RUN bundle install

CMD ["rails", "server", "-b", "0.0.0.0"]
```

Agora vamos criar um arquivo com o nome docker-compose.yml
para instalar banco de dados postgresql e redis
```yml
# docker-compose.yml
version: '3.3'
services:
  db:
    image: postgres:14-alpine
    volumes:
      - "db-data:/var/lib/postgresql/data"
    environment:
      PGDATA: /tmp
      POSTGRES_HOST_AUTH_METHOD: trust
  redis:
    image: redis:7-alpine
    volumes:
      - "redis-data:/data"
  web:
    container_name: demo
    build:
      context: .
      args:
        - ENV_BUILD=development0
    volumes:
      - '.:/app'
    depends_on:
      - db
      - redis
      # - firefox
    environment:
      APP_DATABASE_URL: postgres://postgres@db
      REDIS_URL: redis://redis
      RAILS_ENV: development
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    ports:
      - "3000:3000"
volumes:
  db-data:
  redis-data:
```

Agora vamos criar um arquivo com o nome Gemfile
```sh
# frozen_string_literal: true

source 'https://rubygems.org'
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby '3.1.2'

gem 'rails', '~> 7.0'
```

Agora vamos construir nosso container pela primeira vez
```sh
docker-compose build
```

Após instalar todas as dependências do projeto inicie o container
```sh
docker-compose up
```

Abra uma nova aba do terminal para acessar a máquina docker
e construir nosso aplicativo rails
```sh
docker-compose run web bash

rails new . --force --database=postgresql
```

Desligue o container com o comando 'CTRL + C'

Altere as propriedades de root sobre os arquivos criados (atenção para o ponto no final do comando)
```sh
chown -R $USER:$USER .
```

Configure o arquivo config/database.yml
```yml
default: &default
  primary: &primary_default
    adapter: postgresql
    encoding: unicode
    pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
    schema_search_path: "public"
    username: postgres
    password:
    host: db

development:
  primary:
    <<: *primary_default
    database: app_development

test:
  primary:
    <<: *primary_default
    database: app_test

production:
  primary:
    <<: *primary_default
    database: <%= ENV["APP_DATABASE_URL"] %>
    username: app
    password: <%= ENV["APP_DATABASE_PASSWORD"] %>
```

Inicie tudo novamente com docker-compose up
Caso alguma instalação de gem falhar, 
rode novamente o docker-compose build e depois docker-compose up

Acesse http://0.0.0.0:3000/ e voilà!!

![Captura de tela de 2022-07-02 00-21-55](https://user-images.githubusercontent.com/6554847/176985440-bcab25aa-abbd-4194-beee-434a48d141f4.png)

[voltar](https://juniortada.github.io/)
