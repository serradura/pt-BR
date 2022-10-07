---
title: "Entendendo o `self` e `mixins` no Ruby"
last_modified_at: 2022-10-06T14:30:00-03:00
categories:
  - Blog
tags:
  - iniciante
  - ruby
  - mixin
  - module
  - ruby object model
---

- [O `self`](#o-self)
  - [Contexto? Escopo?](#contexto-escopo)
  - [Dúvida: dá onde surgiu esse `puts`?](#dúvida-dá-onde-surgiu-esse-puts)
  - [Experimento com métodos de instância](#experimento-com-métodos-de-instância)
- [Os mixins](#os-mixins)
    - [Fazendo mixins com `include`](#fazendo-mixins-com-include)
    - [Fazendo mixins com `extend`](#fazendo-mixins-com-extend)
- [Recap](#recap)
- [Conclusão](#conclusão)
- [Exercício extra](#exercício-extra)

Um dos recursos mais confusos para quem está iniciando com Ruby é o funcionamento do `self`.

Então o objetivo deste post é explicar seu funcionamento ao responder as seguintes perguntas: Qual problema o self resolve? Como ele funciona?

## O `self`

O `self` é uma palavra reservada do Ruby que dá acesso ao objeto atual.

Sendo que o objeto atual tem sempre haver com o contexto, escopo em que o mesmo é usado.

### Contexto? Escopo?

Sim, contexto ou escopo tem haver com o momento em que o código é executado.

Falar é fácil... Mostre me código!

Veja nos comentários do código (`#`) abaixo, o resultado que <a href="/pt-BR/blog/dicas-para-iniciantes-de-como-instalar-e-executar-codigo-ruby" target="_blank">a execução irá gerar</a>.

```ruby
def quem_sou_eu
  puts self.inspect
end

def mostre_minha_classe
  puts self.class.inspect
end

quem_sou_eu
# main

mostre_minha_classe
# Object
```

**Detalhes sobre o código acima:**
1. O método `puts` é responsável por imprimir o resultado no terminal
2. O método `.inspect` faz uma representação do objeto como *string*
3. O método `.class` retorna a classe que foi utilizada para gerar o objeto.

Perceba que no exemplo acima, o método `quem_sou_eu` retornou `main`, e o `mostre_minha_classe` indica que esse `main` é uma instância da classe `Object`.

> **Observação:** Esse `main` é o que chamamos de **top level scope** (escopo do nível superior), todo código Ruby executa a partir dele. O mesmo é gerado pelo próprio Ruby e tudo que for declarado nele estará disponível em qualquer outro ponto do código.

### Dúvida: dá onde surgiu esse `puts`?

Quando não for definido o objeto alvo do método, o Ruby usará o `self` por padrão.

Como o `puts` é acessível de qualquer objeto, perceba que o uso do `self` para invocá-lo se torna opcional.

```ruby
def quem_sou_eu_SEM_self_no_puts
  puts self.inspect
end

def quem_sou_eu_USANDO_self_no_puts
  self.puts self.inspect
end

quem_sou_eu_SEM_self_no_puts
# main

quem_sou_eu_USANDO_self_no_puts
# main
```

Legal né?

### Experimento com métodos de instância

```ruby
def quem_sou_eu
  puts self.inspect
end

class Person
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def who_am_i
    # Como disse em uma explicação mais acima,
    # todo método declarado no top level scope (main)
    # ficará acessível de qualquer ponto do código.
    quem_sou_eu
  end
end

quem_sou_eu
# main

person = Person.new('Serradura')

person.who_am_i
# #<Person:0x000000010297d9e0 @name="Serradura">

puts person.name
# Serradura
```

Antes de explicar o código acima, vamos relembrar o que é o `self`:

> É uma palavra reservada do Ruby que dá acesso ao objeto atual.

Veja que o `quem_sou_eu` imprimiu `main` e o `person.who_am_i` que faz uso do `quem_sou_eu` imprimiu `#<Person:0x000000010297d9e0 @name="Serradura">`.

> O código `0x000000010297d9e0` irá variar a cada execução e significa o endereço de memória do objeto.

Pode ser que você não soubesse que é possível acessar qualquer método declarado no `main`, não é?

Mas assim... Declarar métodos no `main` / `top level scope` para reusar em qualquer ponto do código **é uma péssima prática!**

Beleza, mas como reaproveitar métodos sem precisar usar essa estratégia?
**Resposta:** Através de `mixins`.

## Os mixins

A tradução `to mix-in` é misturar. Logo, mixin é uma forma de misturar código! 😅

Beleza, mas como definir um mixin?
**Resposta:** Declarando um módulo (`module`) e fazendo uso deles.

#### Fazendo mixins com `include`

```ruby
module InspecionadorDeObjetos
  def quem_sou_eu
    puts self.inspect
  end

  def mostre_minha_classe
    puts self.class.inspect
  end
end
```

Agora vejamos como misturar esses métodos em outro escopo/contexto:

```ruby
module InspecionadorDeObjetos
  def quem_sou_eu
    puts self.inspect
  end

  def mostre_minha_classe
    puts self.class.inspect
  end
end

class Person
  include InspecionadorDeObjetos

  attr_reader :name

  def initialize(name)
    @name = name
  end
end

person = Person.new('Serradura')

person.quem_sou_eu
# #<Person:0x000000010297d9e0 @name="Serradura">

person.mostre_minha_classe
# Person

quem_sou_eu
# (NameError) undefined local variable or method `quem_sou_eu' for main:Object
```

Veja acima que o `include InspecionadorDeObjetos` disponibiliza os métodos do módulo na instância da classe `person.quem_sou_eu`.

Perceba também, que o `quem_sou_eu` não está mais disponível no `main`/`top level scope`.

> **Nota:** A partir de agora irei usar apenas `top level scope` para me referir ao `main`, lembrando que um é sinônimo do outro.

Mas assim Serra, tem como disponibilizar os métodos main no `top level scope`?

Tem sim! 😈

#### Fazendo mixins com `extend`

```ruby
module InspecionadorDeObjetos
  def quem_sou_eu
    puts self.inspect
  end

  def mostre_minha_classe
    puts self.class.inspect
  end
end

class Person
  include InspecionadorDeObjetos

  attr_reader :name

  def initialize(name)
    @name = name
  end
end

person = Person.new('Serradura')

person.quem_sou_eu
# #<Person:0x000000010297d9e0 @name="Serradura">

person.mostre_minha_classe
# Person

extend InspecionadorDeObjetos

quem_sou_eu
# main

mostre_minha_classe
# Object
```

Perceba que o método `extend` tem a capacidade de modificar o contexto no qual o mesmo foi utilizado.  Por conta disso, é possível invocar os métodos `.quem_sou_eu` e `.mostre_minha_classe` no top level scope após usarmos o `extend InspecionadorDeObjetos`.

Beleza Serra, então tu tá querendo me dizer que posso usar o extend em qualquer objeto? Qualquer objeto mesmo?

Resposta: Sim!

```ruby
module InspecionadorDeObjetos
  def quem_sou_eu
    puts self.inspect
  end

  def mostre_minha_classe
    puts self.class.inspect
  end
end

class Person
  extend InspecionadorDeObjetos
end

Person.quem_sou_eu
# Person

Person.mostre_minha_classe
# Class

uma_string_qualquer = 'uma_string_qualquer'

uma_string_qualquer.extend(InspecionadorDeObjetos)

uma_string_qualquer.quem_sou_eu
# "uma_string_qualquer"

uma_string_qualquer.mostre_minha_classe
# String

'uma_outra_string_qualquer'.quem_sou_eu
# (NoMethodError) undefined method `quem_sou_eu' for "uma_outra_string_qualquer":String
```

Veja nos exemplos acima, que os métodos `.quem_sou_eu` e `.mostre_minha_classe` estão acessíveis para a classe `Person` e para a `"uma_string_qualquer"`. Porém, perceba que o `extend` modificou o escopo do objeto em si e não de outros que tenham o mesmo tipo, como é o caso da string `"uma_outra_string_qualquer"` que não tem o método `.quem_sou_eu`.

## Recap

O `self` é:
1. Uma palavra reservada do Ruby que dá acesso ao objeto atual.
2. Invocado sempre que um método não tiver um objeto alvo. Ex:
   - `puts 1` equivale a `self.puts 1`
   - `include MyModule` equivale a `self.include MyModule`
   - `extend MyModule` equivale a `self.extend MyModule`

O `mixin` é:
1. Uma forma de compartilhar métodos/comportamentos entre classes e objetos.
2. Use `include SomeModule` para disponibilizar os métodos nas instâncias do objeto.
3. Use `extend SomeModule` para disponibilizar os métodos no escopo do objeto.

## Conclusão

Espero que esse artigo te ajude melhor a entender o funcionamento do `self` e como o `self`/escopo dos objetos é respeitado mesmo com o uso de `mixins`.

Gostou do conteúdo? Deixe seu comentário aqui embaixo contando o que achou. Valeu! 😉

## Exercício extra

Execute o código abaixo e procure entender qual será o funcionamento do `self` no uso com classes e herança.

```ruby
class InspecionadorDeObjetos
  def self.quem_sou_eu
    puts self.inspect
  end

  def self.mostre_minha_classe
    puts self.class.inspect
  end

  def quem_sou_eu
    puts self.inspect
  end

  def mostre_minha_classe
    puts self.class.inspect
  end
end

class Person < InspecionadorDeObjetos
end

InspecionadorDeObjetos.quem_sou_eu
InspecionadorDeObjetos.mostre_minha_classe

InspecionadorDeObjetos.new.quem_sou_eu
InspecionadorDeObjetos.new.mostre_minha_classe

Person.new.quem_sou_eu
Person.new.mostre_minha_classe
```

---

Já ouviu falar do **ada.rb - Arquitetura e Design de Aplicações em Ruby**? É um grupo focado em práticas de engenharia de software com Ruby. Acesse o <a href="https://t.me/ruby_arch_design_br" target="_blank">canal no telegram</a> e junte-se a nós em nosso <a href="https://meetup.com/pt-BR/arquitetura-e-design-de-aplicacoes-ruby/" target="_blank">meetup mensal</a> (100% on-line).
