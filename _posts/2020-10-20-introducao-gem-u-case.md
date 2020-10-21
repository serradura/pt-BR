---
title: "Introdução a gem u-case (EM CONSTRUÇÃO)"
last_modified_at: 2020-10-21T00:39:00-03:00
categories:
  - Blog
tags:
  - ruby
  - use cases
  - u-case
---

Seja bem vindo(a) ao primeiro post do meu blog. Já aviso que estou começando e <a href="https://twitter.com/serradura" target="_blank">conto com teu feedback</a> para me auxiliar a melhorar a maneira como compartilho conhecimento aqui no blog. 😉

---

A gem <a href="https://github.com/serradura/u-case" target="_blank">u-case</a> é um projeto que tenho me dedicado há mais de 1 ano. O projeto tem como objetivo facilitar o desenvolvimento e modelagem da camada de regras de negócio de aplições Ruby. Embora a gem possa ser utilizada com qualquer aplicação Ruby, usarei o contexto de uma aplicação Ruby on Rails para apresentar o seu uso.

Mas antes de falar dela, gostaria de destacar a abordagem mais praticada pela comunidade hoje em dia, no caso, a criação de <a href="https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/" target="_blank">service objects</a>. Em geral, os services objects ficam localizados na pasta `app/services` e tem como responsabilidade absorver regras de negócio e dessa forma tornar outras camadas mais coesas (controllers e models). Mas não é de hoje que essa abordagem tem sido motivo de crítica na comunidade <a href="https://avdi.codes/service-objects/" target="_blank">[1]</a><a href="https://www.codewithjason.com/rails-service-objects/" target="_blank">[2]</a>. A razão disso é que o resultado mais comum é a criação de classes enormes com excesso de responsabilidades, que se tornam cada vez mais difíceis de serem mantidas e testadas na medida que a aplicação cresce.

<a href="https://github.com/discourse/discourse/blob/ae47bcb2691612e478df78a328036124617edfa7/app/services/" target="_blank">Clique aqui</a> para visualizar a pasta de `services` do `discourse`. <a href="https://github.com/discourse/discourse/blob/ae47bcb2691612e478df78a328036124617edfa7/app/services/user_merger.rb#L237">Veja</a> um exemplo de uma operação de alta complexidade (119 linhas em um único método) de um dos services desse projeto.

---

Por muito tempo fui adepto a essa prática, mas volta e meia me via sendo penalizado por fazer uso dela. Por conta disso passei a procurar por soluções utilizadas pela comunidade para me auxiliar na seguinte missão:
> *ter uma camada saúdavel (**fácil de entender**, **manter** e **testar**) de regras de negócio das minhas aplicações Ruby/Rails*.

Durante essa jornada fiz uso de gems como: <a href="https://rubygems.org/gems/interactor" target="_blank">`interactor`</a> (+3 milhões de downloads), <a href="https://rubygems.org/gems/trailblazer-operation" target="_blank">`trailblazer-operation`</a> (+1 milhão de downloads), <a href="https://rubygems.org/gems/dry-transaction" target="_blank">`dry-transaction`</a> (quase 1 milhão de downloads), <a href="https://rubygems.org/gems/dry-monads" target="_blank">`dry-monads (do notation)`</a> (+2 milhões de downloads).

Mas tive diversas alegrias e tristezas no uso de cada uma delas (talvez faça sentido escrever um post somente sobre isso). Confesso que a melhor experiência que tive foram com as gems do <a href="https://dry-rb.org/" target="_blank">dry-rb</a>. Dica: conheça esse incrível ecossistema.

Embora o ecossistema `dry-rb` fosse promissor por favorer um desenvolvimento bem <a href="https://en.wikipedia.org/wiki/SOLID" target="_blank">SOLID</a>, era muito comum constar uma grande aversão a essas gems por conta do estilo funcional que elas possuem.

Enfim, por conta dessas experência (ao longo de anos) e a dificuldade em capacitar desenvolvedores Jr, Pl e Sr (viciados em escrever códigos nem um pouco expressivo/entendível) resolvi criar uma gem que tivesse uma aproximação com o Rails e promovesse boas práticas de desenvolvimento (leia-se SOLID). Pois bem, bora ver código e entender o que a gem <a href="https://github.com/serradura/u-case" target="_blank">u-case</a> tem a oferecer.

A fim de facilitar a experimentação, farei uso do <a href="https://bundler.io/guides/bundler_in_a_single_file_ruby_script.html" target="_blank">bundler inline</a> em cada exemplo de código. Ou seja, tu poderá copiar e colar os trechos de código em um arquivo `.rb` e ao executá-los (`ruby exemplo_da_gem_u-case.rb`) o bundler resolverá as dependências.

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'u-case', '~> 4.2.0'
end

class Multiply < Micro::Case
  # 1. Defina o input como atributos
  attributes :a, :b

  # 2. Defina o método `call!` com a regra de negócio
  def call!

    # 3. Envolva o resultado do caso de uso com os métodos
    #    `Success(result: *)` ou `Failure(result: *)`
    if a.is_a?(Numeric) && b.is_a?(Numeric)
      Success result: { number: a * b }
    else
      Failure result: { message: '`a` e `b` devem ser numéricos' }
    end
  end
end

# == Resultado de sucesso ==

result = Multiply.call(a: 2, b: 2)

p result             # <Success (Micro::Case::Result) type=:ok data={:number=>4} transitions=1>
puts result.success? # true
puts result[:number] # 4

# == Resultado de falha ==

result = Multiply.call(a: '2', b: 2)

p result              # <Failure (Micro::Case::Result) type=:error data={:message=>"`a` e `b` devem ser numéricos"} transitions=1>
puts result.failure?  # true
puts result[:message] # `a` e `b` devem ser numéricos
```
