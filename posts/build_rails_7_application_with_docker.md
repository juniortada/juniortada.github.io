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
FROM ruby:3.2.3

# Copy the main application.
RUN mkdir -p /app
COPY . /app
WORKDIR /app

# Dependencies
RUN apt-get update -qq && apt-get install -y postgresql-client
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
&& apt-get install -y nodejs
RUN npm install --location=global yarn
COPY Gemfile /app/Gemfile
RUN bundle install

# Install node modules
RUN yarn install --check-files
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

ruby '3.2.3'

gem 'rails', '~> 7.1'
```

Agora vamos construir nosso container pela primeira vez
```sh
docker compose build
```

Após instalar todas as dependências do projeto inicie o container
```sh
docker compose up
```

Abra uma nova aba do terminal para acessar a máquina docker
e construir nosso aplicativo rails
```sh
docker compose run web bash

rails new . --force --database=postgresql

# caso queira criar com bootstrap
rails new . -j esbuild --css bootstrap --force -d=postgresql
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

PS: o rails 7.1 gera arquivos automáticos para Docker, caso queira volte para a versão
inicial desse post e inicie com docker compose up --build
Inicie tudo novamente com docker compose up
Caso alguma instalação de gem falhar, 
rode novamente o docker-compose build e depois docker-compose up

Acesse http://0.0.0.0:3000/ e voilà!!

PS: no primeiro acesso da página inicial click no botão create database

![Captura de tela de 2024-02-29 23-28-51](https://github.com/juniortada/juniortada.github.io/assets/6554847/7c456258-5378-47d5-a14f-11d131604be2)

[voltar](https://juniortada.github.io/)
