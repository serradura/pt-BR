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

<span id="service-objects">Mas antes de falar dela, gostaria de destacar a abordagem mais praticada pela comunidade nos dias de hoje, no caso, a criação de <a href="https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/" target="_blank">service objects</a></span>. Em geral, os services objects ficam localizados na pasta `app/services` de uma aplicação Rails e tem como responsabilidade concentrar as regras de negócio da aplicação, permitindo assim que outras camadas fiquem mais coesas (Ex: controllers e models). Porém, essa abordagem tem sido alvo de muitas críticas <a href="https://avdi.codes/service-objects/" target="_blank">[1]</a><a href="https://www.codewithjason.com/rails-service-objects/" target="_blank">[2]</a>. A razão disso é que o resultado mais comum é a criação de classes enormes e com excesso de responsabilidades. Dificultando assim a manutenção e evolução do código mediante o aumento de complexidade por conta de requisitos de negócios cada vez mais sofisticados.

<a href="https://github.com/discourse/discourse/blob/ae47bcb2691612e478df78a328036124617edfa7/app/services/" target="_blank">Clique aqui</a> para visualizar a pasta de `app/services` do `Discourse`. E <a href="https://github.com/discourse/discourse/blob/ae47bcb2691612e478df78a328036124617edfa7/app/services/user_merger.rb#L237">clique aqui</a> para ver um exemplo de uma operação complexa (119 linhas) de um dos métodos do service object dessa aplicação.

---

Por muito tempo fui adepto a essa prática, mas volta e meia me via sendo penalizado por fazer uso dela. Por conta disso passei a procurar por soluções alternativas e que fossem utilizadas pela comunidade, afim de atingir o seguinte objetivo:
> *ter uma camada saúdavel (**fácil de entender**, **manter/modificar** e **testar**) de regras de negócio nas minhas aplicações Ruby/Rails*.

Durante essa jornada fiz uso de gems como: <a href="https://rubygems.org/gems/interactor" target="_blank">`interactor`</a> (+3 milhões de downloads), <a href="https://rubygems.org/gems/trailblazer-operation" target="_blank">`trailblazer-operation`</a> (+1 milhão de downloads), <a href="https://rubygems.org/gems/dry-transaction" target="_blank">`dry-transaction`</a> (quase 1 milhão de downloads), <a href="https://rubygems.org/gems/dry-monads" target="_blank">`dry-monads (do notation)`</a> (+2 milhões de downloads).

Mas ao longo desse processo tive diversas alegrias e tristezas no uso de cada uma delas (talvez faça sentido escrever um post somente sobre isso). Sendo que a melhor experiência que tive foram com as gems do <a href="https://dry-rb.org/" target="_blank">dry-rb</a>.

E, embora o ecossistema `dry-rb` fosse promissor por favorecer um desenvolvimento bem <a href="https://en.wikipedia.org/wiki/SOLID" target="_blank">SOLID</a>, era muito comum constar uma grande aversão a essas gems por conta do estilo funcional que elas possuem.

Enfim, por conta dessas experência (ao longo de anos) e a dificuldade em capacitar desenvolvedores Jr, Pl e Sr. Resolvi criar uma gem que tivesse uma aproximação com o Rails (para evitar essa aversão) e que promovesse boas práticas de desenvolvimento (leia-se SOLID). Pois bem, bora ver código para entender o que a <a href="https://github.com/serradura/u-case" target="_blank">u-case</a> tem a oferecer.

> **Nota:** A fim de facilitar a experimentação, farei uso do <a href="https://bundler.io/guides/bundler_in_a_single_file_ruby_script.html" target="_blank">bundler inline</a> em alguns exemplos. Assim será possível copiar e colar os trechos de código em um arquivo `.rb` e ao executá-los (`ruby exemplo_da_gem_u-case.rb`) o bundler resolverá todas as dependências.

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
2. A regra de negócio, definido pelo método `call!`.<br/>
3. Um resultado de sucesso ou de falha, definido pelos métodos `Success(result: {})` ou `Failure(result: {})`.

> Se analisarmos bem, um caso de uso nada mais é do que um **_input_** (os atributos), uma ou mais ações (processamento), que retornará um **_output_** (o resultado).

Sendo que esse resultado é um `Micro::Case::Result`, que como podemos ver no exemplo acima possuí os seguintes métodos:
- `#success?` retornará `true` caso o retorno do `call!` tenha sido com o método `Success()`.
- `#failure?` retornará `true` caso o retorno do `call!` tenha sido com o método `Failure()`.
- `#data` retorna o hash usado na propriedade `:result` dos métodos `Success()` ou `Failure()`.
- `#[]` permite acessar os valores contidos no `result.data`.

> **Ponto importante:** o valor retornado na propriedade `:result` dos métodos `Success()` e `Failure()` tem sempre de ser um `Hash`.

O que você achou até aqui, tranquilo?

Pois bem, agora peço que me acompanhe para entender o real poder do `u-case`. Que nada mais é do que favorecer o uso de composição.

E para exemplificar isso, criaremos um novo caso de uso que permitirá somar três há um determinado número.

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

Agora, que tal criarmos um caso de uso para somar nove?

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

E aí, o que achou? Facinho né?

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

Hummmm... O que aconteceria se fizessemos uma composição com outra composição?

Para testarmos isso, sugiro criarmos algo que somará dois números e então adicionará nove, sendo que essa última operação será uma composição.

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

E aí, achou fácil? Perceba que é possível fazer composição tanto com casos de usos isolados, ou com outros `flows` (esse é o termo usado para indicar a composição de dois ou mais casos de uso).

> **Atenção:** Perceba que os atributos ficam acessíveis no escopo do caso de uso, ou seja, eles são a porta de entrada para o processamento que ocorrerá no método `call!`. Sem eles essa relação de **input**/**output** ficará comprometida.

## Implementando algo do nosso dia dia

Como aplicação prática, irei implementar um caso de uso que criará um usuário, o mesmo será composto dos seguintes passos:
1. Normalizará os parâmetros
2. Validará os parâmetros
3. Persistirá o usuário no banco de dados (para simplificar ele irá instanciar um objeto)

```ruby
require 'uri'
require 'ostruct'
require 'securerandom'

User = Struct.new(:id, :name, :email)

module Users
  class Create < Micro::Case
    attributes :name, :email

    def call!
      normalized_name = String(name).strip.gsub(/\s+/, ' ')
      normalized_email = String(email).downcase.strip

      validation_errors = []
      validation_errors << "Name can't be blank" if normalized_name.empty?
      validation_errors << "Email is invalid" if normalized_email !~ URI::MailTo::EMAIL_REGEXP

      return Failure result: { errors: validation_errors } if !validation_errors.empty?

      user = User.new(
        SecureRandom.uuid,
        normalized_name,
        normalized_email
      )

      Success result: { user: user }
    end
  end
end

# == Resultado de Sucesso ==

result = Users::Create.call(name: 'Rodrigo', email: 'rodrigo.serradura@gmail.com')

puts result.success? # true
p result[:user]      # #<struct User id="4cc0b77b-b824-4f16-a8a7-8ee30b7017dc", name="Rodrigo", email="rodrigo.serradura@gmail.com">

# == Resultado de Falha ==

result = Users::Create.call(name: 'Rodrigo', email: 'rodrigo.serradura')

puts result.failure? # true
p result.data        # { :errors => ["Email is invalid"] }
```

Viu como foi simples definir um processo que cria um usuário?

Agora, irei implementar cada etapa em um caso de uso separado e então compor um `flow`.

```ruby
require 'uri'
require 'ostruct'
require 'securerandom'

User = Struct.new(:id, :name, :email)

module Users
  class NormalizeParams < Micro::Case
    attributes :name, :email

    def call!
      Success result: {
        name: String(name).strip.gsub(/\s+/, ' '),
        email: String(email).downcase.strip
      }
    end
  end

  class ValidateParams < Micro::Case
    attributes :name, :email

    def call!
      validation_errors = []
      validation_errors << "Name can't be blank" if name.empty?
      validation_errors << "Email is invalid" if email !~ URI::MailTo::EMAIL_REGEXP

      return Success() if validation_errors.empty?

      Failure result: { errors: validation_errors }
    end
  end

  class CreateRecord < Micro::Case
    attributes :name, :email

    def call!
      user = User.new(SecureRandom.uuid, name, email)

      Success result: { user: user }
    end
  end

  CreationProccess = Micro::Cases.flow([
    NormalizeParams,
    ValidateParams,
    CreateRecord
  ])
end

# == Resultado de Sucesso ==

result = Users::CreationProccess.call(name: 'Rodrigo', email: 'rodrigo.serradura@gmail.com')

puts result.success? # true
p result[:user]      # #<struct User id="4cc0b77b-b824-4f16-a8a7-8ee30b7017dc", name="Rodrigo", email="rodrigo.serradura@gmail.com">

# == Resultado de Falha ==

result = Users::CreationProccess.call(name: 'Rodrigo', email: 'rodrigo.serradura')

puts result.failure? # true
p result.data        # { :errors => ["Email is invalid"] }
```

> **Atenção:** as validações não precisam ser confome o exemplo acima (elas estão bem feinhas <span style="font-style: normal;">😅</span>).
>
> É possível fazer uso das validações do ActiveModel nos atributos dos seus caso de uso. <a href="https://github.com/serradura/u-case/blob/main/README.pt-BR.md#u-casewith_activemodel_validation---como-validar-os-atributos-do-caso-de-uso" target="_blank">Clique aqui</a> para conferir na documentação.
> (**Abordarei isso em outro post**)

### 💡 Insight: Testes unitários

Escreverei sobre isso em breve, mas acredito que seja perceptível o quão prático passa ser implementar testes unitários com essa abordagem. Uma vez que é possível testar o `flow` como um todo e/ou cada etapa que o compõe. Usando os exemplos anteriores da [criação de usuário](#implementando-algo-do-nosso-dia-dia), é mais prático testar:

```ruby
# Etapas que compõe o flow Users::CreationProccess
Users::NormalizeParams
Users::ValidateParams
Users::CreateRecord

# Fluxo completo
Users::CreationProccess # Micro::Cases.flow([NormalizeParams, ValidateParams, CreateRecord])
```

Do que testar um único caso de uso com toda a regra de negócio, que é o caso do primeiro exemplo:
```ruby
Users::Create
```

## Concluindo

Ainda há muito a ser dito, pois mal tocamos a ponta do iceberg. Mas com os recursos que foram abordados já será possível criar casos de usos bem interessantes.

Minha intenção foi em te apresentar de maneira rápida e objetiva o que é a gem `u-case` e como começar a fazer uso dela.

E destacar o seu poder de composição, que facilita a criação de um código mais expressivo e prático de se manter/testar. Algo bem diferente do que ocorre com o uso tradicional de service objects (como destaquei no [início deste post](#service-objects)).

Nos próximos posts irei abordar diversas outras funcionalidades (*validação*, *normalização de atributos*, *diferentes formas de compor um flow*) e conceitos relacionados.

Se tu curtiu esse conteúdo, sugiro acessar a <a href="https://github.com/serradura/u-case/blob/main/README.pt-BR.md" target="_blank">documentação em pt-BR</a> e a assistir uma palestra na qual apresento a gem `u-case` após explicar de forma prática como aplicações Ruby on Rails vem sendo organizadas nos últimos 15 anos.

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

Valeu! 🙂
