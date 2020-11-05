---
title: "Introdução a testes automatizados (TDD) com Ruby"
last_modified_at: 2020-11-05T02:12:00-03:00
categories:
  - Blog
tags:
  - ruby
  - tdd
---

Olá, se tem uma coisa que pessoas e equipes de auto desempenho possuem é **confiança**. Sendo que em desenvolvimento de software é muito importante ter certeza que o teu código funciona na medida que é modificado. A técnica mais comum e intuitiva para aumentar confiança de que as mudanças não estão gerando *bugs* é a realização de testes manuais. Ou seja, um ser humano, executará uma série de ações em sequência afim de avaliar se o sistema funciona conforme o esperado.

Testes realizados por seres humanos são válidos, mas é um recurso lento e caro se comparado a um processo automatizado. Por conta disso, nossa industria desenvolveu técnicas e ferramentas que permitem uma automação de boa parte desses processos.

Existem várias técnicas para se testar software. Mas neste artigo abordarei o uso do TDD (**T**est **D**riven **D**evelopment), que pode ser traduzido como: **desenvolvimento orientado a testes**. O TDD é a prática de
escrever os testes antes da implementação, isto é, você implementa testes (que naturalmente vão falhar) e só então adiciona o código necessário para que eles passem.

Esse post será dividido em duas partes, a primeira abordará o que é e por que utilizar TDD, e a última te ensinará como usar na prática! Como? Você aprenderá na medida que escreve a sua própria lib de testes 💪.

## O que é TDD?

Como dito, TDD é um ciclo de desenvolvimento no qual você escreve primeiro o teste e depois a implementação. Veja a seguir cada uma de suas etapas:

### Primeiro passo: <span style="font-weight: normal;">escrever um teste que irá falhar</span>.

{% include figure image_path="/assets/images/3-tdd/1_Red.png" alt="Tdd: Red = Teste falhando" caption="Tdd: Red = Teste falhando." %}

Primeiro, você tem o <span style="color: red;">*red mode*</span>, no qual é escrito um teste que falha. Nele você descreverá o comportamento que o código deverá ter. Seja uma série de ações ou apenas uma parte/unidade do comportamento esperado.

### Próximo passo: <span style="font-weight: normal;">fazer o teste passar</span>.

{% include figure image_path="/assets/images/3-tdd/2_Red-Green.png" alt="Tdd: Red + Green = Fazer o teste passar com o mínimo de código necessário" caption="Tdd: Red + Green = Fazer o teste passar com o mínimo de código necessário." %}

Nessa etapa, você escreverá o código necessário para fazer o teste passar, ou seja, entrar no <span style="color: green;">*green mode*</span>.

### Último passo: <span style="font-weight: normal;">refatorar o código produzido</span>.

{% include figure image_path="/assets/images/3-tdd/3_Red-Green-Refactor.png" alt="Tdd: Red + Green + Refactor = Refinar a estrutura do código sem quebrar o comportamento." caption="Tdd: Red + Green + Refactor = Refinar a estrutura do código sem quebrar o comportamento." %}

Nesse momento se torna possível <span style="color: blue;">refinar o código</span>, já que temos garantias que o mesmo funciona. Ou seja, podemos melhorar a implementação desde que os testes continuem passando.

**Significado de refatoração**: É o processo de melhorar a estrutura de um código/sistema enquanto se preserva o comportamento existente.

> <span style="font-style: normal;">**Dica:** Só refine o código se for realmente necessário! 🙂</span>

### Por fim: <span style="font-weight: normal;">repita o ciclo</span>.

{% include figure image_path="/assets/images/3-tdd/4_Red-Green-Refactor-Cycle.png" alt="Tdd: Red + Green + Refactor (Cycle) = Repita o ciclo para a próxima implementação." caption="Tdd: Red + Green + Refactor (Cycle) = Repita o ciclo para a próxima implementação." %}

Repita o ciclo para a próxima funcionalidade/comportamento a ser entregue.
1. Escreva um <a style="color: red;" href="#primeiro-passo-escrever-um-teste-que-irá-falhar">teste que falha</a>.
2. Faça o <a style="color: green;" href="#próximo-passo-fazer-o-teste-passar">teste passar</a>.
3. <a style="color: blue;" href="#próximo-passo-refatorar-o-código-produzido">Refatore o código</a> (melhore o que for possível).

A ideia é repetir esse ciclo para cada funcionalidade do sistema até que o mesmo seja considerado finalizado (pronto para produção).

## Por que fazer TDD?

Ao fazer isso você e sua equipe obterão os seguintes benefícios:
1. **Erradicar o medo de modificar o sistema/código fonte.**
  <br/>Pelo fato dos testes estarem verdes, será possível alterar a lógica de negócio e comportamentos da aplicação uma vez que as especificações estão garantidas.
2. **Refatorar se torna algo fácil e seguro de ser feito.**
  <br/>Caso algum teste falhe durante um *refactoring* será possível identificar o problema e então corrigí-lo.

A curto prazo tudo isso pode parecer improdutivo, já que você irá escrever mais código (implementação + testes) para chegar ao resultado desejado. Mas no médio/longo prazo, será um grande diferencial ter confiança para modificar o sistema conforme for preciso.

## Como fazer TDD?

Para responder a essa pergunta te convido a escrever uma lib de testes para colocarmos toda a teoria em prática. Recomendo a leitura do <a href="/blog/as-diferentes-formas-de-declarar-comportamento-em-ruby/" title="Programação multiparadigma - As diferentes formas de declarar comportamento em Ruby">post anterior</a> já que o conteúdo do mesmo servirá de base para nos auxiliar na implementação.

Um dos conceitos chaves dessa prática é a que testes devem ser encarados como uma **fonte de verdade**. Ou seja, **os teste não podem falhar**.

E dado o conceito acima, sugiro fazermos uso de <a href="https://www.honeybadger.io/blog/a-beginner-s-guide-to-exceptions-in-ruby/" target="_blank">*exceptions*</a> para interromper a execução de nossa suíte de testes caso ocorra alguma falha.

```ruby
a = 1
b = 2

raise 'os valores devem ser iguais.' if a != b
```

O resultado do código acima será:

```ruby
RuntimeError: os valores devem ser iguais.
```

> **Obs:** Escreva nos comentários caso você tenha mais interesse em aprender sobre exceptions. <span style="font-style: normal;">😉</span>

Será que você teve a mesma ideia que eu? Que tal usarmos uma sequência de condicionais + exceptions para definir os nossos testes?

> Afim de ter uma relação com o <a href="/blog/as-diferentes-formas-de-declarar-comportamento-em-ruby/" title="Programação multiparadigma - As diferentes formas de declarar comportamento em Ruby">post anterior</a>, usaremos o exemplo de uma calculadora para colocar em prática o uso de TDD.

**Primeiro passo:** escrever um teste que falha.

```ruby
raise 'asserção falhou' if sum(1, 1) != 2
```

O que acontecerá se o código acima for executado? Dará erro! Por que? Porque o método não existe! 😅

```ruby
NoMethodError: undefined method `sum' for main:Object
```

Então, bora definir o método.

```ruby
def sum(a, b)
end

raise 'asserção falhou' if sum(1, 1) != 2
```

E ao executar o código acima, iremos ver esse resultado:

```ruby
RuntimeError: asserção falhou
```

Opa! O teste falhou. 🙌
Ou seja, entramos no <span style="color: red;">*red mode*</span>.

---
Antes de continuarmos, precisamos abrir um rápido parênteses.

**Pergunta:** Por que fazer uso da palavra asserção?<br />
**Resposta:** Por conta de seu <a href="https://www.dicio.com.br/assercao/" target="_blank">significado</a> (afirmação que se faz com muita certeza).

Logo, se um teste falhar, significa que falhou nossa afirmação / expectativa.

---

Dado que nosso teste está falhando, podemos implementar o **mínimo necessário** para ele passar.

```ruby
def sum(a, b)
  2
end

raise 'asserção falhou' if sum(1, 1) != 2
```

Ao executar o código anterior verá que o mesmo não teve erros. Ou seja, nosso teste está passando (<span style="color: green;">*green mode*</span>)!

Imagino que você já percebeu que o nosso método de soma, não soma! Mas acredite, essa é a proposta do TDD, você só escreve o código necessário para o teste passar.

E dado que não temos código o suficiente para refatorar, sugiro reiniciarmos o ciclo e escrevermos um novo teste que falha.

```ruby
def sum(a, b)
  2
end

raise 'asserção falhou' if sum(1, 1) != 2 # nil
raise 'asserção falhou' if sum(1, 3) != 4 # RuntimeError: asserção falhou
```

Dado a nova falha, podemos implementar o código necessário para ele passar.

```ruby
def sum(a, b)
  a + b
end

raise 'asserção falhou' if sum(1, 1) != 2
raise 'asserção falhou' if sum(1, 3) != 4
```

Uhuuuu! Os testes voltaram a passar.

Curtiu? Mas assim, dado que os testes estão passando que tal começarmos a refatorar a nossa "lib de testes"?

Para isso, sugiro criarmos um método `assert` que lançará uma exception caso o valor do argumento seja `false`.

```ruby
def assert(truthy)
  raise 'asserção falhou' unless truthy
end
```

> <span style="font-style: normal;">**Obs:** Podemos usar `unless` para negar uma condição, ou seja, é o mesmo que fazer `if !false`.</span>

A seguir, veja o uso do novo método de asserção para fazer os testes da implementação:

```ruby
# == testing lib ==
def assert(truthy)
  raise 'asserção falhou' unless truthy
end

# == implementation ==
def sum(a, b)
  a + b
end

# == tests ==
assert sum(1, 1) == 2
assert sum(1, 3) == 4
```

**Próximo requisito:** fazer com que o método de soma seja capaz de transformar *strings* em *números*.

Veja a seguir o teste relacionado ao novo requisito:

```ruby
# == testing lib ==
def assert(truthy)
  raise 'asserção falhou' unless truthy
end

# == implementation ==
def sum(a, b)
  a + b
end

# == tests ==
assert sum(1, 1) == 2 # nil
assert sum(1, 3) == 4 # nil
assert sum('3', 3) == 6 # TypeError: no implicit conversion of Integer into String
```

Se executarmos o código acima iremos obter o erro `TypeError: no implicit conversion of Integer into String` ao tentar somar uma string com um número. Para evitar esse erro, sugiro ignorarmos qualquer argumento que não seja numérico.

```ruby
# == testing lib ==
def assert(truthy)
  raise 'asserção falhou' unless truthy
end

# == implementation ==
def sum(a, b)
  a + b if a.is_a?(Numeric) && b.is_a?(Numeric)
end

# == tests ==
assert sum(1, 1) == 2 # nil
assert sum(1, 3) == 4 # nil
assert sum('3', 3) == 6 # RuntimeError: asserção falhou
```

Pronto, agora o teste voltou a falhar!

**Próximo passo:** fazer o teste passar.

Veja abaixo como utilizei o método `is_a?` para saber se o argumento é uma `String` e o método `to_i` para transformar uma string em um inteiro.

```ruby
# == testing lib ==
def assert(truthy)
  raise 'asserção falhou' unless truthy
end

# == implementation ==
def sum(a, b)
  a = a.to_i if a.is_a?(String)
  b = b.to_i if b.is_a?(String)

  a + b
end

# == tests ==
assert sum(1, 1) == 2
assert sum(1, 3) == 4
assert sum('3', 3) == 6
```

Ao testar o código acima, verá que os testes voltaram a passar!

**Próximo passo:** Refatorar!

Nessa etapa podemos analisar se existe alguma oportunidade para melhorar a implementação sem alterar o comportamento. Analise o código abaixo afim de encontrar algo para refinar:

```ruby
def sum(a, b)
  a = a.to_i if a.is_a?(String)
  b = b.to_i if b.is_a?(String)

  a + b
end
```

Ao meu ver podemos eliminar a repetição relacionada a verificação do argumento + transformação em número. Para isso, criaremos um novo método `numeric_value` para cuidar dos argumentos.

```ruby
# == testing lib ==
def assert(truthy)
  raise 'asserção falhou' unless truthy
end

# == implementation ==
def numeric_value(arg)
  arg.is_a?(String) ? arg.to_i : arg
end

def sum(a, b)
  numeric_value(a) + numeric_value(b)
end

# == tests ==
assert sum(1, 1) == 2
assert sum(1, 3) == 4
assert sum('3', 3) == 6
```

Viu? Nossos testes continuram verdes. o/

> **Dica:** <a href="https://guru-sp.github.io/tutorial_ruby/construcoes-simples.html" target="_blank">Clique aqui</a> para conhecer mais sobre o operador ternário `?:` que foi utilizado no método `numeric_value`.

Mas assim, dado o que foi dito no <a href="/blog/as-diferentes-formas-de-declarar-comportamento-em-ruby/" target="_blank">post anterior</a>, que tal transformarmos esses <a href="/blog/as-diferentes-formas-de-declarar-comportamento-em-ruby/#métodos-globais">métodos globais</a> em <a href="/blog/as-diferentes-formas-de-declarar-comportamento-em-ruby/#métodos-de-classe">métodos de uma classe</a>?

Veja abaixo a transformação dos métodos `sum` e `numeric_value` em métodos da classe `Calc`.

```ruby
# == testing lib ==
def assert(truthy)
  raise 'asserção falhou' unless truthy
end

# == implementation ==
class Calc
  def self.numeric_value(arg)
    arg.is_a?(String) ? arg.to_i : arg
  end

  def self.sum(a, b)
    numeric_value(a) + numeric_value(b)
  end
end

# == tests ==
assert Calc.sum(1, 1) == 2
assert Calc.sum(1, 3) == 4
assert Calc.sum('3', 3) == 6
```

Por fim, sugiro adicionarmos um método para realizar multiplicações (`Calc.multiply`).

Veja a seguir a implementação + testes do novo método:

```ruby
# == testing lib ==
def assert(truthy)
  raise 'asserção falhou' unless truthy
end

# == implementation ==
class Calc
  def self.numeric_value(arg)
    arg.is_a?(String) ? arg.to_i : arg
  end

  def self.sum(a, b)
    numeric_value(a) + numeric_value(b)
  end

  def self.multiply(a, b)
    numeric_value(a) * numeric_value(b)
  end
end

# == tests ==
assert Calc.sum(1, 1) == 2
assert Calc.sum(1, 3) == 4
assert Calc.sum('3', 3) == 6

assert Calc.multiply(2, 2) == 4
assert Calc.multiply(3, 3) == 9
assert Calc.multiply('2', 4) == 8
```

Tranquilo né?

Perceba que nesse caso nós implementamos o método e os testes de uma única vez. Ou seja, nem sempre será necessário passar por todos os passos do TDD.

## Concluindo

O poder do TDD está em desenvolver somente o código necessário para fazer o teste passar e na segurança de ter uma automação que verifica se o sistema funciona conforme o esperado.

Recomendo que você pratique e estude o máximo possível sobre esse tema (testes), porque uma vez que você tem a funcionalidade garantida, será possível explorar coisas como: refactoring/design de código, melhorias de performance e etc... Ou seja, testes prepara o terreno para você explorar mais e mais.

No próximo post abordarei sobre o uso das principais bibliotecas de testes utilizadas em projetos Ruby: o <a href="https://github.com/seattlerb/minitest" target="_blank">minitest</a> e o <a href="https://github.com/rspec/rspec" target="_blank">rspec</a>.

Valeu! 🙂

## Agradecimentos

Quero agradecer a colaboração do <a href="github.com/tomascco" target="_blank">@tomascco</a>, <a href="github.com/mploureno" target="_blank">@mploureno</a> e <a href="github.com/joaomarcos96" target="_blank">@joaomarcos96</a> na revisão deste conteúdo. Muito obrigado! 👏

## Desafios

**1)** Aprimore o método `numeric_value` do código abaixo:
```ruby
class Calc
  def self.numeric_value(arg)
    arg.is_a?(String) ? arg.to_i : arg
  end

  def self.sum(a, b)
    numeric_value(a) + numeric_value(b)
  end
end
```
Novos requisitos:

1. Transformar `String` com números e `.` em `floats`.
2. Lançar uma exceção caso algum dos argumentos não seja numérico, ou uma string com números (inteiros ou `floats`).

**2)** Analise o código abaixo e procure descobrir com o auxílio da <a href="https://ruby-doc.org/core-2.7.2/" target="_blank">documentação do Ruby</a> os métodos necessários para fazer a classe `UnitTest` funcionar.

```ruby
# == testing lib ==
class UnitTest
  def self.call
    tests = public_instance_methods.select do |method|
      method.to_s.start_with?('test_')
    end

    tests.each do |test|
      self.new.send(test)

      print '.'
    end

    puts "\n\nTodos os testes passaram!"
  end

  private

  def assert(truthy)
    raise 'asserção falhou' unless truthy
  end
end

# == implementation ==
class Calc
  def self.numeric_value(arg)
    arg.is_a?(String) ? arg.to_i : arg
  end

  def self.sum(a, b)
    numeric_value(a) + numeric_value(b)
  end

  def self.multiply(a, b)
    numeric_value(a) * numeric_value(b)
  end
end

# == tests ==
class CalcTest < UnitTest
  def test_sum
    assert Calc.sum(1, 1) == 2
    assert Calc.sum(1, 3) == 4
    assert Calc.sum('3', 3) == 6
  end

  def test_multiply
    assert Calc.multiply(2, 2) == 4
    assert Calc.multiply(3, 3) == 9
    assert Calc.multiply('2', 4) == 8
  end
end

# == running the tests ==
CalcTest.call
```

Resultado gerado pelo código acima:

```
..

Todos os testes passaram!
```

> <span style="font-style: normal;">**Obs:** `CalcTest.call` é o método utilizado para executar todos testes de `CalcTest`.</span>

---

Já ouviu falar do **ada.rb - Arquitetura e Design de Aplicações em Ruby**? É um grupo focado em práticas de engenharia de software com Ruby. Acesse o <a href="https://t.me/ruby_arch_design_br" target="_blank">canal no telegram</a> e junte-se a nós em nosso <a href="https://www.meetup.com/pt-BR/design-e-arquitetura-de-aplicacoes-ruby/events/" target="_blank">meetup mensal</a> (100% on-line).
