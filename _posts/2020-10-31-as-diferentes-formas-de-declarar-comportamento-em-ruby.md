---
title: "Programação multiparadigma - As diferentes formas de declarar comportamento em Ruby"
last_modified_at: 2020-11-01T10:15:00-03:00
categories:
  - Blog
tags:
  - ruby
  - oop
  - fp
  - sp
---

Olá, o objetivo deste post é de apresentar as diferentes formas que a linguagem Ruby permite você declarar comportamentos.

Mas o que seria um comportamento? Um comportamento é uma unidade de código (encapsulada dentro de métodos ou funções) que executam uma ação.

Ao longo deste post irei demonstrar como utilizar **métodos** em diferentes contextos (`global`, `classe`, `instâncias` e `módulos`) e como utilizar **funções**.

## Métodos globais

Métodos globais são métodos acessíveis em qualquer ponto da aplicação, inclusive, essa é a maneira mais simples (e perigosa) de se definir comportamentos em Ruby.

```ruby
def sum(a, b)
  a + b
end

p sum(1, 2)

p self
p self.class
p self.object_id
```

Ao executar o código acima, você verá o resultado abaixo (exceto o object_id que será definido para cada execução):
```ruby
3              # p sum(1, 2)

main           # p self
Object         # p self.class
70335232078200 # p self.object_id
```

Vamos entender o código acima:
1. O retorno (output) do método `sum(1, 2)` foi `3` como esperado.
2. Ao inspecionar `p self` (`p` é um atalho para `puts some_object.inspect`) obtivemos `main` <a href="https://nandovieira.com.br/ruby-object-model-self" target="_blank">[1]</a><a href="https://engineering.appfolio.com/appfolio-engineering/2018/8/9/rubys-main-object-does-what" target="_target">[2]</a> que para fins práticos podemos assumir como o escopo global.
3. O resultado de `p self.class` é `Object`, ou seja, `main` é uma instância de `Object` (o Ruby faz um `Object.new` assim que o interpretador começar a ler os arquivos *.rb* do seu programa).
4. Essa instância como todo objeto em Ruby terá um `object_id` (endereço na memória), que será único, já que só existirá uma única instância chamada `main`.

> <span style="font-style: normal;">**Obs:**</span> Escreva no comentário caso queira entender melhor como funciona o **self**/**Ruby Object Model**.

### Reflexão: Como agrupar comportamentos quando só se usa métodos globais?

Resposta: Aplicando prefixos no nome dos métodos!

Para explicar a resposta acima, imagine o seguinte requisito: adicionar um novo cálculo de multiplicação (*multiply*) e agrupá-lo a operação de soma (*sum*). Como foi dito na resposta, podemos prefixar os métodos com um termo em comum (*calc_*). Ex:

```ruby
def calc_sum(a, b)
  a + b
end

def calc_multiply(a, b)
  a * b
end

p calc_sum(3, 3)      # 6
p calc_multiply(3, 3) # 9
```

> <span style="font-style: normal;">**Nota:**</span> Essa forma de se programar é chamada de **programação estruturada**, ou, **programação procedural**.

## Métodos de classe

Métodos de classe, também conhecidos como métodos estáticos. São métodos que fazem parte da definição de uma classe, ou seja, esses métodos não farão parte dos objetos ([instâncias](#métodos-de-instância)) criados por essas classes.

**Como declarar métodos de classe?**

Resposta: Usando `def self.nome_do_metodo`.

Veja um exemplo a seguir.

```ruby
class Calc
  def self.sum(a, b)
    a + b
  end

  def self.multiply(a, b)
    a * b
  end
end

p Calc.sum(3, 3)      # 6
p Calc.multiply(3, 3) # 9
```

Perceba que os métodos *sum* e *multiply* foram definidos na classe **_Calc_**.

Também perceba como essa forma de desenvolver é [idêntica a definição de métodos procedurais/globais](#reflexão-como-agrupar-comportamentos-quando-só-se-usa-métodos-globais). Ex:

```ruby
def calc_sum(a, b)
  a + b
end

class Calc
  def self.sum(a, b)
    a + b
  end
end
```

Métodos estáticos tem as mesmas propriedades de métodos procedurais. Sendo bem comum que métodos como esses fiquem enormes e se tornem difíceis de manter e testar (será o <a href="/pt-BR/blog/introducao-a-testes-automatizados-com-ruby/" target="_blank">próximo post</a> do blog).

## Métodos de instância

Métodos de instância são métodos que só serão acessíveis ao instanciar uma classe. No Ruby, fazemos isso através do método `.new`. Ex:

```ruby
class Calc
  def self.sum(a, b)
    a + b
  end

  def self.multiply(a, b)
    a * b
  end
end

calc = Calc.new

p calc.sum(2, 3)  # Você verá o erro abaixo:
# undefined method `sum' for #<Calc:0x0000...> (NoMethodError)
```

**Pergunta:** Por que não foi possível acessar os métodos `sum` e `multiply` do objeto `calc`?<br/>
**Resposta:** Porque os métodos são de classe! 😅

**Pergunta:** E como transformar esses métodos de classe em métodos de instância?<br/>
**Resposta:** Removendo `self` da declaração deles! 🙂

Exemplo:

```ruby
class Calc
  def sum(a, b)
    a + b
  end

  def multiply(a, b)
    a * b
  end
end

calc = Calc.new

p calc.sum(2, 3)      # 5
p calc.multiply(2, 3) # 6
```

Viu? Funcionou! 😊

## Métodos estáticos em módulos

Existe uma outra forma de se ter métodos estáticos em Ruby, essa outra forma é fazendo uso de módulos. Para declarar um módulo, use a palavra `module` ao invés de `class`. Ex:

```ruby
module Calc
  def self.sum(a, b)
    a + b
  end

  def self.multiply(a, b)
    a * b
  end
end

p Calc.sum(3, 3)      # 6
p Calc.multiply(3, 3) # 9

# ---

Calc.new # undefined method `new' for Calc:Module (NoMethodError)
```

Módulos tem características especiais que não serão retratadas nesse post, mas perceba que:
1. Foi possível declarar métodos estáticos e fazer uso deles.
2. Não é possível instanciar um `module`, sendo essa uma das características que os diferenciam das classes.

### Namespaces: Outra característica especial dos módulos

Namespaces é um container capaz de incluir vários tipos de itens como: classes, módulos e constantes. Para exemplificar criarei o module `Calc` que terá os módulos `Sum`, `Multiply` e que responderão a um único método (`.call`). Ex:

```ruby
module Calc
  module Sum
    def self.call(a, b)
      a + b
    end
  end

  module Multiply
    def self.call(a, b)
      a * b
    end
  end
end

p Calc::Sum.call(3, 3)      # 6
p Calc::Multiply.call(3, 3) # 9
```

**Pergunta:** Sabe o que fizemos ao criar módulos com métodos estáticos que fazem uma única coisa?<br/>
**Resposta:** Uma função!

Mas assim, quanto código para fazer algo tão simples né?

Na minha opinião isso complicou demais. Não seria melhor se Ruby tivesse um suporte real para declarar funções? 🤔

(Spoiler: Ruby tem suporte a funções sim! 🙌)

## Funções

**Aviso:** Existem diferentes formas de se declarar funções em Ruby, mas nesse post apresentarei apenas uma!

Para declarar uma função, utilize a sintaxe `-> (args) {}`. Ex:

```ruby
module Calc
  Sum = -> (a, b) { a + b }

  Multiply = -> (a, b) { a * b }
end

p Calc::Sum.call(3, 3)      # 6
p Calc::Multiply.call(3, 3) # 9
```

> <span style="font-style: normal;">**Dica:**</span> Para saber mais, <a href="https://ruby-doc.org/core-2.6/Proc.html" target="_blank">consulte a doc oficial</a> sobre esse recurso incrível.

## Concluindo

Esse post tem como objetivo introduzir os recursos que classificam Ruby como uma **liguagem multiparadigma** (conforme apresentado na <a href="https://en.wikipedia.org/wiki/Ruby_(programming_language)" target="_blank">wikipedia</a>), pelo fato dela suportar os paradigmas:
- Procedural
- Orientado a objetos
- Funcional

**Pergunta:** Tu desconhecia que Ruby tem recursos funcionais? 😱

Sem problemas, confira a palestra (30 minutos) que fiz e que apresenta o quão funcional Ruby é! 😉

<iframe width="560" height="315" src="https://www.youtube.com/embed/M2fKC92Ov0k?start=9429" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

Os exemplos de código deste post estão disponíveis nesse <a href="https://gist.github.com/serradura/dbadb71027262614e64728e5606fc8de" target="_blank">gist</a>.

Gostou do conteúdo? Deixe seu comentário aqui embaixo contando o que achou. Valeu! 😉

---

Já ouviu falar do **ada.rb - Arquitetura e Design de Aplicações em Ruby**? É um grupo focado em práticas de engenharia de software com Ruby. Acesse o <a href="https://t.me/ruby_arch_design_br" target="_blank">canal no telegram</a> e junte-se a nós em nosso <a href="https://www.meetup.com/pt-BR/design-e-arquitetura-de-aplicacoes-ruby/events/" target="_blank">meetup mensal</a> (100% on-line).
