---
layout: default
---

### Junior Tada Blog 👋

## O Mínimo de Qualidade Para Seu Código

Olá, este guia tem por objetivo apresentar o mínimo de qualidade que uma aplicação com rails
deve conter.

[Rubocop](https://github.com/rubocop/rubocop)

Esta é uma ferramenta que analisa (linter) e formata o código.
Adicione a gem no seu arquivo Gemfile no grupo :development, :test

```sh  
    gem 'rubocop', '~> 1.33'
```

Para utilizar seguindo o nosso [exemplo](https://juniortada.github.io/posts/build_rails_7_application_with_docker)
de aplicação com docker, entre em um terminal no bash do docker e execute:

```sh
    docker-compose run web bash

    rubocop -a
```

[Rspec](https://rspec.info/)

Essa parte precisa de uma postagem só para isso, mas fica aqui uma introdução a instalação.

```sh  
    gem 'rspec-rails', '~> 6.0'
```

Outras gem úteis:

```sh  
    gem 'ffaker', '~> 2.21'
    gem 'vcr', '~> 6.1'
    gem 'simplecov', '~> 0.21.2'
```