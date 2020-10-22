---
title: "Introdução a gem u-case"
last_modified_at: 2020-10-23T00:19:00-03:00
categories:
  - Blog
tags:
  - ruby
  - u-case
---

Seja bem vindo(a) ao meu primeiro post. Já aviso que estou começando e <a href="https://twitter.com/serradura" target="_blank">conto com teu feedback</a> para me auxiliar a melhorar a maneira como compartilho conhecimento aqui no blog. 😉

---

A gem <a href="https://github.com/serradura/u-case" target="_blank">u-case</a> é um projeto que tenho me dedicado há mais de 1 ano. O projeto tem como objetivo facilitar o desenvolvimento e modelagem da camada de regras de negócio de aplições Ruby. Embora a gem possa ser utilizada com qualquer codebase, usarei o contexto de uma aplicação Ruby on Rails para apresentar o seu uso.

Mas antes de falar dela, gostaria de destacar a abordagem mais praticada pela comunidade nos dias hoje, no caso, a criação de <a href="https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/" target="_blank">service objects</a>. Em geral, os services objects ficam localizados na pasta `app/services` de uma aplicação Rails e tem como responsabilidade concentrar as regras de negócio da aplicação, permitindo assim que outras camadas fiquem mais coesas (Ex: controllers e models). Porém, não é de hoje que essa abordagem tem sido motivo de crítica <a href="https://avdi.codes/service-objects/" target="_blank">[1]</a><a href="https://www.codewithjason.com/rails-service-objects/" target="_blank">[2]</a>. A razão disso é que o resultado mais comum é a criação de classes enormes e com excesso de responsabilidades. Dificultando assim a manutenção e evolução do código mediante o aumento de complexidade por conta de requisitos de negócios cada vez mais sofisticados.

<a href="https://github.com/discourse/discourse/blob/ae47bcb2691612e478df78a328036124617edfa7/app/services/" target="_blank">Clique aqui</a> para visualizar a pasta de `app/services` do `Discourse`. E <a href="https://github.com/discourse/discourse/blob/ae47bcb2691612e478df78a328036124617edfa7/app/services/user_merger.rb#L237">clique aqui</a> para ver um exemplo de uma operação complexa (119 linhas) de um dos métodos do service object dessa aplicação.

---

Por muito tempo fui adepto a essa prática, mas volta e meia me via sendo penalizado por fazer uso dela. Por conta disso passei a procurar por soluções alternativas utilizadas pela comunidade visando atingir o seguinte resultado:
> *ter uma camada saúdavel (**fácil de entender**, **manter** e **testar**) de regras de negócio nas minhas aplicações Ruby/Rails*.

Durante essa jornada fiz uso de gems como: <a href="https://rubygems.org/gems/interactor" target="_blank">`interactor`</a> (+3 milhões de downloads), <a href="https://rubygems.org/gems/trailblazer-operation" target="_blank">`trailblazer-operation`</a> (+1 milhão de downloads), <a href="https://rubygems.org/gems/dry-transaction" target="_blank">`dry-transaction`</a> (quase 1 milhão de downloads), <a href="https://rubygems.org/gems/dry-monads" target="_blank">`dry-monads (do notation)`</a> (+2 milhões de downloads).

Mas ao longo desse processo tive diversas alegrias e tristezas no uso de cada uma delas (talvez faça sentido escrever um post somente sobre isso). Confesso que a melhor experiência que tive foram com as gems do <a href="https://dry-rb.org/" target="_blank">dry-rb</a>.

E, embora o ecossistema `dry-rb` fosse promissor por favorecer um desenvolvimento bem <a href="https://en.wikipedia.org/wiki/SOLID" target="_blank">SOLID</a>, era muito comum constar uma grande aversão a essas gems por conta do estilo funcional que elas possuem.

Enfim, por conta dessas experência (ao longo de anos) e a dificuldade em capacitar desenvolvedores Jr, Pl e Sr (viciados em escrever códigos nem um pouco expressivo/entendível) resolvi criar uma gem que tivesse uma aproximação com o Rails (evitando essa aversão) e que promovesse boas práticas de desenvolvimento (leia-se SOLID). Pois bem, bora ver código para entender o que a gem <a href="https://github.com/serradura/u-case" target="_blank">u-case</a> tem a oferecer.

> **Nota:** A fim de facilitar a experimentação, farei uso do <a href="https://bundler.io/guides/bundler_in_a_single_file_ruby_script.html" target="_blank">bundler inline</a> em alguns exemplos de código. Assim será possível copiar e colar os trechos de código em um arquivo `.rb` e ao executá-los (`ruby exemplo_da_gem_u-case.rb`) o bundler resolverá as dependências.

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'u-case', '~> 4.2.1'
end

class Sum < Micro::Case
  attributes :a, :b

  def call!
    if a.is_a?(Numeric) && b.is_a?(Numeric)
      Success result: { number: a + b }
    else
      Failure result: { message: '`a` e `b` devem ser numéricos' }
    end
  end
end

# == Resultado de sucesso ==

result = Sum.call(a: 2, b: 2)

p result             # #<Success (Micro::Case::Result) type=:ok data={:number=>4} transitions=1>
puts result.success? # true
puts result[:number] # 4
puts result.data     # { :number => 4 }

# == Resultado de falha ==

result = Sum.call(a: '2', b: 2)

p result              # #<Failure (Micro::Case::Result) type=:error data={:message=>"`a` e `b` devem ser numéricos"} transitions=1>
puts result.failure?  # true
puts result[:message] # `a` e `b` devem ser numéricos
puts result.data     # { :message => "`a` e `b` devem ser numéricos" }
```

Todo caso de uso possuí a seguinte estrutura:
1. Um conjunto de atributos (método `attributes`), ou seja, serão os dados de *input*.
2. A regra de negócio, definido pelo método `call!`.
3. Um resultado de sucesso ou de falha, definido pelos métodos `Success(result: {})` ou `Failure(result: {})`.

> Se analisarmos bem, um caso de uso nada mais é do que um **_input_** (os atributos), uma ação (processamento), que retornará um **_output_** (o resultado).

Sendo que esse resultado é um `Micro::Case::Result`, que como podemos ver no exemplo acima possuí os seguintes métodos:
- `#success?` retornará `true` caso o retorno do `call!` tenha sido com o método `Success()`.
- `#failure?` retornará `true` caso o retorno do `call!` tenha sido com o método `Failure()`.
- `#data` retorna o hash usado na propriedade `:result` dos métodos `Success()` ou `Failure()`.
- `#[]` permite acessar os valores contidos no `result.data`.

> **Ponto importante:** o valor retornado na propriedade `:result` dos métodos `Success()` e `Failure()` tem sempre de ser um `Hash`.

O que você achou até aqui, tranquilo?

Pois bem, me acompanhe para entender o real poder do `u-case`. Que nada mais é do que favorecer o uso de composição.

Para exemplificar isso, criaremos um novo caso de uso que permitirá adicionar três há um número existente.

```ruby
class Add3 < Micro::Case
  attribute :number

  def call!
    return Success result: { number: number + 3 } if number.is_a?(Numeric)

    Failure result: { message: '`number` deve ser numérico' }
  end
end

result = Add3.call(number: 1)

puts result.data # { :number => 4 }
```

Agora, que tal criarmos um caso de uso para adicionar nove?

Como disse anteriormente, o real poder do `u-case` está na capacidade que o mesmo tem em promover o uso de composição.

Para isso, faremos uso de um novo recurso, no caso, usaremos o `Micro::Cases.flow()`.

```ruby
class Add3 < Micro::Case
  attribute :number

  def call!
    return Success result: { number: number + 3 } if number.is_a?(Numeric)

    Failure result: { message: '`number` deve ser numérico' }
  end
end

Add9 = Micro::Cases.flow([Add3, Add3, Add3])

result = Add9.call(number: 1)

puts result.data # { :number => 10 }
```

E aí, o que acho? Facinho né?

Agora, bora fazermos um caso de uso um pouquinho mais rebuscado que somará dois números e então adicionará três ao resultado da soma.

```ruby
class Sum < Micro::Case
  attributes :a, :b

  def call!
    if a.is_a?(Numeric) && b.is_a?(Numeric)
      Success result: { number: a + b }
    else
      Failure result: { message: '`a` e `b` devem ser numéricos' }
    end
  end
end

class Add3 < Micro::Case
  attribute :number

  def call!
    return Success result: { number: number + 3 } if number.is_a?(Numeric)

    Failure result: { message: '`number` deve ser numérico' }
  end
end

SumAndAdd3 = Micro::Cases.flow([Sum, Add3])

result = SumAndAdd3.call(a: 4, b: 5)

puts result.data # { :number => 12 }
```

Também foi fácil né? Bacana!

**Como isso foi possível?**

Veja que o resultado de `Sum` é `Success result: { number: a + b }`, e o atributo de `Add3` é `:number`. Ou seja, o **_output_** de `Sum` tornou-se o **_input_** do `Add3`.

Hummmm... O que aconteceria se fizessemos uma composição com uma outra existente?

Para testarmos isso, sugiro criarmos algo que somará dois números e então adicionará nove, sendo que essa última operação será uma outra composição.

```ruby
class Sum < Micro::Case
  attributes :a, :b

  def call!
    if a.is_a?(Numeric) && b.is_a?(Numeric)
      Success result: { number: a + b }
    else
      Failure result: { message: '`a` e `b` devem ser numéricos' }
    end
  end
end

class Add3 < Micro::Case
  attribute :number

  def call!
    return Success result: { number: number + 3 } if number.is_a?(Numeric)

    Failure result: { message: '`number` deve ser numérico' }
  end
end

Add9 = Micro::Cases.flow([Add3, Add3, Add3])

SumAndAdd9 = Micro::Cases.flow([Sum, Add9])

result = SumAndAdd9.call(a: 4, b: 5)

puts result.data # { :number => 18 }
```

E aí, achou fácil? Perceba que é possível fazer composição tanto com casos de usos isolados, ou com outros `flows` (esse é o termo usado para indicar a composição de 2 ou mais casos de uso).

Ainda há muito a ser dito, mal tocamos a ponta do iceberg. Mas com esses recursos já seria possível compor casos de usos bem sofisticados em qualquer aplicação Rails. Ex:

```ruby
# app/use_cases/users/creation/process.rb

module Users::Creation
  Process = Micro::Cases.flow([
    NormalizeParams,
    ValidateParams,
    Persist,
    SyncWithCRM
  ])
end
```

Você pode conferir um exemplo completo <a href="https://github.com/serradura/u-case/blob/2317f7c5e402c683cae8e7f55e636a42d15f4a27/test/micro/case/MWRF/steps/04_using_static_composition_test.rb" target="_blank">nesse arquivo de teste do projeto</a>.

## Concluindo

A intencão desse post é te apresentar de maneira rápida e objetiva o que é a gem `u-case` e como começar a fazer uso dela.

Nos próximos posts irei abordar diversas outras funcionalidades e conceitos relacionados a mesma.

Se tu curtiu esse conteúdo, sugiro acessar a <a href="https://github.com/serradura/u-case/blob/main/README.pt-BR.md" target="_blank">documentação em pt-BR</a> da gem e a assistir uma palestra na qual apresento a gem `u-case` além de todo um histórico de como aplicações Ruby on Rails tem sido organizadas nos últimos 15 anos.

<small>* PS: Tem um desafio após o vídeo.</small>

<iframe width="560" height="315" src="https://www.youtube.com/embed/ySNzVfmYy5g" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Desafio

Descubra o que o método `result.transitions` é capaz de fazer. Segue um exemplo prontinho para você executar e analisar.

**Dica:** <a href="https://github.com/serradura/u-case/blob/main/README.pt-BR.md#como-entender-o-que-aconteceu-durante-a-execu%C3%A7%C3%A3o-de-um-flow" target="_blank">confira a documentação dessa funcionalidade</a>.

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'u-case', '~> 4.2.1'
  gem 'amazing_print', '~> 1.2.2'
end

class Add3 < Micro::Case
  attribute :number

  def call!
    return Success result: { number: number + 3 } if number.is_a?(Numeric)

    Failure result: { message: '`number` deve ser numérico' }
  end
end

Add9 = Micro::Cases.flow([Add3, Add3, Add3])

result = Add9.call(number: 1)

puts result.data # { :number => 10 }

ap result.transitions
```

Valeu!
