---
title: "Minitest VS Rspec - Introdução e comparação as diferentes formas de escrever testes"
last_modified_at: 2020-12-07T10:44:00-03:00
categories:
  - Blog
tags:
  - ruby
  - tdd
  - minitest
  - rspec
---

Dando continuidade ao tema testes (<a href="/pt-BR/blog/introducao-a-testes-automatizados-com-ruby/" target="_blank">acesse o primeiro artigo</a>), irei abordar as duas principais bibliotecas utilizadas na comunidade Ruby. São elas, o <a href="https://github.com/seattlerb/minitest" target="_blank">minitest</a> e o <a href="https://github.com/rspec/rspec" target="_blank">rspec</a>.

O foco deste post está no uso prático das libs e não em fazer uma análise profunda de cada uma delas. Minha intenção com esse conteúdo é de apresentar suas diferenças e destacar as forças e fraquezas de cada uma delas na minha humilde opinião.

# Minitest

Está lib pode ser utilizada com qualquer projeto Ruby e vem instalada por padrão em aplicações <a href="https://guides.rubyonrails.org/testing.html#rails-meets-minitest" target="_blank">Ruby on Rails</a>. E para fazer a sua introdução, quero destacar um trecho existente no README do projeto:

> minitest doesn't reinvent anything that ruby already provides, like: classes, modules, inheritance, methods. This means you only have to learn ruby to use minitest...

> **Tradução livre:** minitest não reinventa nada que o ruby já fornece, como: classes, módulos, herança, métodos. Isso significa que você só precisa aprender ruby para usar o minitest...

Ou seja, uma vez que você aprende os recursos essenciais desta ferramenta você só precisará fazer uso da linguagem.

Por conta disso, uma das coisas que mais me impressiona nesta gem é a sua baixa curva de aprendizado, até o fim deste post você aprenderá o necessário para fazer um uso pleno do que ela tem a te oferecer. 😉

A seguir confira um exemplo no qual testaremos uma calculadora que sabe como realizar as operações de soma (`.add`) e subtração (`.sub`):

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'

  gem 'minitest' , '~> 5.14', '>= 5.14.2'
end

module Calc
  extend self

  def add(a, b); number(a) + number(b); end
  def sub(a, b); number(a) - number(b); end

  private

    def number(value)
      return value if value.is_a?(Numeric)
      return value.include?('.') ? value.to_f : value.to_i if numeric_string?(value)
      raise TypeError, 'the value must be Numeric or a String with numbers'
    end

    def numeric_string?(value)
      value.is_a?(String) && value =~ /((\d+)?\.\d|\d+)/
    end
end

require 'minitest/autorun'

class CalcAdditionTest < Minitest::Test
  def test_the_operation_with_numeric_values
    assert Calc.add(1, 1) == 2
    assert Calc.add(1.5, 1) == 2.5
  end

  def test_the_operation_with_string_values
    assert Calc.add(1, '1') == 2
    assert Calc.add('1.5', 1) == 2.5
    assert_raises(TypeError) { Calc.add('a', 1) }
  end

  def test_the_operation_with_invalid_args
    assert_raises(TypeError) { Calc.add([], '1') }
    assert_raises(TypeError) { Calc.add('1.5', {}) }
  end
end
```
<small>* <a href="https://gist.github.com/serradura/1bc0dd7cad6ed7dabc243ea7e40e1fde#file-exemplo_01-rb" target="_blank">Link</a> para o gist com o código acima.</small>

Executando os testes:

{% include figure image_path="/assets/images/5-minitest-vs-rspec/executando_o_minitest.gif" alt="Executando o exemplo anterior - minitest." %}

<small>* No GIF acima eu adiciono o primeiro exemplo em um arquivo com o nome minitest.rb e o executo com o comando `ruby minitest.rb`.</small>

> <span style="font-style: normal;">**Obs:** Recomendo a leitura <a href="/pt-BR/blog/dicas-para-iniciantes-de-como-instalar-e-executar-codigo-ruby/" target="_blank">desse post</a> caso não saiba como executar os exemplos de código.</span>

Este foi o resultado gerado pela execução:

```
Run options: --seed 37929

# Running:

...

Finished in 0.001297s, 2313.0301 runs/s, 5397.0702 assertions/s.

3 runs, 7 assertions, 0 failures, 0 errors, 0 skips
```

Agora que habemus código. Bora analisar o que foi utilizado?

## Entendendo a implementação do módulo `Calc`

Primeiramente, vamos entender a implementação de `Calc` que é um módulo contendo dois métodos estáticos `.add` e `.sub`, isso se torna possível pelo fato de usarmos `extend self`.

```ruby
module Calc
  extend self

  def add(a, b); number(a) + number(b); end
  def sub(a, b); number(a) - number(b); end

  # ...
end
```

Para auxiliar os métodos públicos, temos o método privado `.number` que tem a responsabilidade de obter um valor numérico, o mesmo faz uso do `.numeric_string?` (que também é privado) para identificar se o valor é uma string contendo um `integer` ou `float`. Caso os valores não sejam válidos, será lançada uma exception `TypeError` com a mensagem `the value must be Numeric or a String with numbers`.

```ruby
module Calc
  # ...

    private

    def number(value)
      return value if value.is_a?(Numeric)
      return value.include?('.') ? value.to_f : value.to_i if numeric_string?(value)
      raise TypeError, 'the value must be Numeric or a String with numbers'
    end

    def numeric_string?(value)
      value.is_a?(String) && value =~ /((\d+)?\.\d|\d+)/
    end
end
```

## Entendendo os testes com Minitest

Antes de analisar como testar, precisamos entender a estrutura que envolve os testes.

```ruby
require 'minitest/autorun'

class CalcAdditionTest < Minitest::Test
  def test_the_operation_with_numeric_values
  end

  def test_the_operation_with_string_values
  end

  def test_the_operation_with_invalid_args
  end
end
```

1. `require 'minitest/autorun'` executará o minitest após o ruby interpretar o arquivo *.rb* (numa aplicação <a href="https://guides.rubyonrails.org/testing.html#rails-meets-minitest" target="_blank">Ruby on Rails</a> isso já vem configurado).
2. Declaramos uma classe Ruby `CalcAdditionTest` que herda de `Minitest::Test` para conter todos os cenários de teste.
3. Declaramos métodos iniciando com `test_` para indicar ao **minitest** quais são os testes.

Perceba que essa estrutura é Ruby puro, já que declaramos toda a estrutura fazendo uso de uma classe e métodos.

Pois bem, dado que entendemos a estrutura que envolvem os testes, vamos analisar como eles são feitos:

```ruby
def test_the_operation_with_numeric_values
  assert Calc.add(1, 1) == 2
  assert Calc.add(1.5, 1) == 2.5
end

def test_the_operation_with_string_values
  assert Calc.add(1, '1') == 2
  assert Calc.add('1.5', 1) == 2.5
  assert_raises(TypeError) { Calc.add('a', 1) }
end

def test_the_operation_with_invalid_args
  assert_raises(TypeError) { Calc.add([], '1') }
  assert_raises(TypeError) { Calc.add('1.5', {}) }
end
```

Usamos o método `assert` que funciona da mesma forma como o que implementamos no post <a href="/pt-BR/blog/introducao-a-testes-automatizados-com-ruby/" target="_blank">Introdução a testes automatizados (TDD) com Ruby</a>. Ou seja, o teste irá falhar caso o resultado da expressão seja `false` (Ex: `Calc.add(1, 2) == 2`) .

O outro método utilizado no teste é o `assert_raises`, que verifica se uma determinada *exception* foi lançada durante o teste.

> **Dica:** Confira a <a href="https://www.rubydoc.info/gems/minitest/5.14.2/Minitest/Assertions" target="_blank">doc do `Minitest::Assertions`</a> para entender os outros métodos disponíveis para se fazer uma asserção.

Seguindo a dica acima, poderíamos fazer uso do <a href="https://www.rubydoc.info/gems/minitest/5.14.2/Minitest/Assertions#assert_equal-instance_method" target="_blank">`assert_equal`</a> para verificar a igualdade entre dois valores inteiros e <a href="https://www.rubydoc.info/gems/minitest/5.14.2/Minitest/Assertions#assert_in_delta-instance_method" target="_blank">`assert_in_delta`</a> para comparar floats. Ex:

```ruby
def test_the_operation_with_numeric_values
  assert_equal(2, Calc.add(1, 1))

  assert_in_delta(2.1232, Calc.add(1, 1.1232), 0.01)
end
```

> **Obs:** Abra o seu irb e faça a soma `1.1232 + 1` e você obterá: `2.1231999999999998`. O `assert_in_delta` permite considerar uma diferença na comparação de `floats`.

E isso, é tudo o que você precisa aprender para fazer uso do minitest. Resumindo:

1. Declare uma classe que termine com o sufixo `Test` e que herde de `Minitest::Test`. Ex: `class CalcAdditionTest < Minitest::Test; end`.
2. Declare os testes ao definir métodos que comecem com `test_`. Ex: `def test_the_operation_with_numeric_values; end`
3. Faça uso dos <a href="https://www.rubydoc.info/gems/minitest/5.14.2/Minitest/Assertions" target="_blank">métodos de asserção</a>.

Para exercitarmos, que tal implementarmos os testes para a operação de subtração?

```ruby
class CalcSubtractionTest < Minitest::Test
  def test_the_operation_with_numeric_values
    assert_equal(0, Calc.sub(1, 1))

    assert_in_delta(0.1232, Calc.sub(1.1232, 1), 0.01)
  end

  def test_the_operation_with_string_values
    assert Calc.sub(1, '1') == 0
    assert Calc.sub('1.5', 1) == 0.5
    assert_raises(TypeError) { Calc.sub('a', 1) }
  end

  def test_the_operation_with_invalid_args
    assert_raises(TypeError) { Calc.sub([], '1') }
    assert_raises(TypeError) { Calc.sub('1.5', {}) }
  end
end
```

<small>* <a href="https://gist.github.com/serradura/1bc0dd7cad6ed7dabc243ea7e40e1fde#file-exemplo_02-rb" target="_blank">Link</a> para o gist com o código completo do exemplo acima.</small>

Bem simples né?

---

O que demonstrei acima foi uma preferência pessoal minha, no caso, ter um arquivo com cada contexto dos meus testes. Mas as vezes você pode achar interessante ter os diferentes contextos em um único arquivo.

Mas a pergunta que fica é, como fazer isso com `minitest`?

R: Fazendo uso de namespaces hora! Afinal, só precisamos usar o que o Ruby já tem. 😉

```ruby
module Calc
  class AdditionTest < Minitest::Test
    def test_the_operation_with_numeric_values
      assert Calc.add(1, 1) == 2
      assert Calc.add(1.5, 1) == 2.5
    end
    # ...
  end

  class SubtractionTest < Minitest::Test
    def test_the_operation_with_numeric_values
      assert_equal(0, Calc.sub(1, 1))

      assert_in_delta(0.1232, Calc.sub(1.1232, 1), 0.01)
    end
    # ...
  end
end
```

<small>* <a href="https://gist.github.com/serradura/1bc0dd7cad6ed7dabc243ea7e40e1fde#file-exemplo_03-rb" target="_blank">Link</a> para o gist com o código completo do exemplo acima.</small>

## Concluindo a introdução ao Minitest

Espero que tenha ficado claro o quão simples o minitest é e o quanto ele promove o uso de Ruby. Minitest não reinventa a roda! 🙂

Além dos recursos apresentados até aqui, existem os conceitos de `setup` e `teardown` que nada mais é do que declarar uma operação que deve ocorrer antes (`setup`) e depois (`teardown`) de cada teste. Ex:

```ruby
class CalcAdditionTest < Minitest::Test
  def setup
    @numbers = {a: 1, b: 2}
  end

  def teardown
    @numbers.delete(:a)
    @numbers.delete(:b)
  end

  def test_the_operation_with_numeric_values
    assert_equal(3, Calc.add(@numbers[:a], @numbers[:b]))
  end
end
```

<small>* <a href="https://gist.github.com/serradura/1bc0dd7cad6ed7dabc243ea7e40e1fde#file-exemplo_04-rb" target="_blank">Link</a> para o gist com o código completo do exemplo acima.</small>

A ideia do teste acima é definir no `setup` a variável de instância `@numbers` com um hash e suas propriedades `:a` e `b`. Já no `teardown`, a ideia é deletar as propriedades de `@numbers`.

Embora o exemplo acima seja um tanto simplista, esse recurso é útil para demonstrar como é fácil definir um estado comum e temporário entre diferentes testes, que poderia ser um registro no banco de dados, arquivo em disco e etc...

# Rspec

É uma alternativa ao [**minitest**](#minitest) que também permite a escrita de testes automatizados. O mesmo, estimula o uso de BDD (Behavior Driven Development) no qual o foco está em expressar o comportamento do sistema. BDD em testes é uma extensão do <a href="/pt-BR/blog/introducao-a-testes-automatizados-com-ruby/" target="_blank">TDD</a>, o motivo disso é que no começo do TDD era muito comum encontrar testes que eram especificações das implementações, ou seja, como o software realiza as operações a nível de código e não quais as consequências/comportamentos do mesmo (BDD).

Para facilitar o uso deste conceito, o **rspec** te dará uma DSL (Domain-specific language), que nada mais é do que uma forma de escrever código usando uma série de abstrações focadas em resolver um problema. No caso do rspec, te dar uma linguagem focada em especificar comportamentos através de testes. Vejamos um exemplo:

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'

  gem 'rspec', '~> 3.10'
end

module Calc
  # ... mesmo código dos exemplos com minitest
end

require 'rspec/autorun'

RSpec.describe Calc do
  describe '.add' do
    context 'with numeric values' do
      it 'returns the sum of the values' do
        expect(Calc.add(1, 1)).to eq(2)

        expect(Calc.add(1.5, 1)).to eq(2.5)
      end
    end

    context 'with string values' do
      context 'and all of them are numbers' do
        it 'returns the sum of the values' do
          expect(Calc.add(1, '1')).to eq(2)

          expect(Calc.add('1.5', 1)).to eq(2.5)
        end
      end

      context "and some of them isn't a number" do
        it 'raises an error' do
          expect { Calc.add('a', 1) }.to raise_error(TypeError)
        end
      end
    end

    context 'with some invalid argument' do
      it 'raises an error' do
        expect { Calc.add([], '1') }.to raise_error(TypeError)

        expect { Calc.add('1.5', {}) }.to raise_error(TypeError)
      end
    end
  end
end
```
<small>* <a href="https://gist.github.com/serradura/8d2ef8dfd7c9a71e467b7461f58fa70c#file-exemplo_01-rb" target="_blank">Link</a> para o gist com o código completo do exemplo acima.</small>

Veja o resultado padrão ao executar o exemplo acima:

`ruby testing_calc_using_rspec.rb`

```
....

Finished in 0.00475 seconds (files took 0.09566 seconds to load)
4 examples, 0 failures
```

Agora, veja o resultado no formato de documentação:

`SPEC_OPTS='--format documentation' ruby testing_calc_using_rspec.rb`

```
Calc
  .add
    with numeric values
      returns the sum of the values
    with string values
      and all of them are numbers
        returns the sum of the values
      and some of them isn't a number
        raises an error
    with some invalid argument
      raises an error

Finished in 0.00567 seconds (files took 0.10348 seconds to load)
4 examples, 0 failures
```

## Entendendo os testes com Rspec

Assim como fizemos com o minitest, vamos primeiro analisar a estrutura que envolve os testes.

```ruby
require 'rspec/autorun'

RSpec.describe Calc do
  describe '.add' do
    context 'with numeric values' do
      it 'returns the sum of the values' do
        # ...
      end
    end

    context 'with string values' do
      context 'and all of them are numbers' do
        it 'returns the sum of the values' do
          # ...
        end
      end

      context "and some of them isn't a number" do
        it 'raises an error' do
          # ...
        end
      end
    end

    # ...
  end
end
```

1. `require 'rspec/autorun'` executará o rspec após o ruby interpretar o arquivo *.rb*. (você pode utilizar o <a href="https://github.com/rspec/rspec-rails" target="_blank">rspec-rails</a> para realizar essa e outras configurações numa aplicação Ruby on Rails)
2. Fazemos uso do `RSpec.describe` para declarar o objeto do teste, no caso, o módulo `Calc`.
3. Utilizamos `describe '.add'` para indicar o método que será o escopo dos testes.
4. Passamos a declarar os diferentes contextos (`context`) que serão usados para testarmos os diferentes comportamentos da implementação.
5. Por fim, utilizamos `it`, para implementar os testes.

Perceba como essa estrutura remete a uma especificação (`rspec` tem a ver com *specification*): **descreva** (`describe`) o **contexto** (`context`) de como o objeto do teste (`it`) deverá se comportar.

Essa proposta permite uma abordagem bem expressiva e que quando bem escrita permite que você tenha uma especificação funcional do seu código. Pois ela serve tanto para documentar quanto para testar se os comportamentos da implementação estão de acordo com o esperado.

E é graças a esse conceito/estrutura que é possível formatar a saída dos testes como uma documentação:

```
Calc
  .add
    with string values
      and all of them are numbers
        returns the sum of the values
      and some of them isn't a number
        raises an error
```

Agora que entendemos a estrutura que envolvem os testes, vamos analisar como eles são declarados:

```ruby
it 'returns the sum of the values' do
  expect(Calc.add(1, 1)).to eq(2)

  expect(Calc.add(1.5, 1)).to eq(2.5)
end

it 'raises an error' do
  expect { Calc.add('a', 1) }.to raise_error(TypeError)
end
```

Usamos `expect` para envolver o resultado que receberá as asserções:
- `expect(Calc.add(1, 1)).to eq(2)` para verificar se o resultado da soma é igual a dois.
- `expect { Calc.add('a', 1) }.to raise_error(TypeError)` para verificar se o resultado foi o lançamento de uma exception.

Damos o nome de `matchers` aos métodos utilizados para realizar as asserções com rspec, acesse esse <a href="https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers" target="_blank">link</a> para verificar todos os disponíveis por padrão.

### Implementando os testes com `one-liner syntax`

Existe uma forma alternativa de realizar os testes, chamada de <a href="https://relishapp.com/rspec/rspec-core/v/3-10/docs/subject/one-liner-syntax" target="_blank">`one-liner syntax`</a> (sintaxe de uma linha). Veja como o teste anterior ficará ao usarmos essa nova sintaxe:

```ruby
RSpec.describe Calc do
  describe '.add' do
    context 'with numeric values' do
      it { expect(Calc.add(1, 1)).to eq(2) }
      it { expect(Calc.add(1.5, 1)).to eq(2.5) }
    end

    context 'with string values' do
      it { expect(Calc.add(1, '1')).to eq(2) }
      it { expect(Calc.add('1.5', 1)).to eq(2.5) }

      context "and some of them isn't a number" do
        it { expect { Calc.add('a', 1) }.to raise_error(TypeError) }
      end
    end

    context 'with some invalid argument' do
      it { expect { Calc.add([], '1') }.to raise_error(TypeError) }
      it { expect { Calc.add('1.5', {}) }.to raise_error(TypeError) }
    end
  end
end
```
<small>* <a href="https://gist.github.com/serradura/8d2ef8dfd7c9a71e467b7461f58fa70c#file-exemplo_02-rb" target="_blank">Link</a> para o gist com o código completo do exemplo acima.</small>

Ficou bem mais simples, né?

Mas essa mudança tem um impacto nas saídas dos testes. Vejamos primeiro a saída padrão (que nada muda):

`ruby testing_calc_using_rspec_one_liner_syntax.rb`

```
.......

Finished in 0.00674 seconds (files took 0.08957 seconds to load)
7 examples, 0 failures
```

Porém, veja o impacto dessa sintaxe quando usamos o formato de documentação:

`SPEC_OPTS='--format documentation' ruby testing_calc_using_rspec_one_liner_syntax.rb`

```
Calc
  .add
    with numeric values
      is expected to eq 2
      is expected to eq 2.5
    with string values
      and all of them are numbers
        is expected to eq 2
        is expected to eq 2.5
      and some of them isn't a number
        is expected to raise TypeError
    with some invalid argument
      is expected to raise TypeError
      is expected to raise TypeError

Finished in 0.00369 seconds (files took 0.09457 seconds to load)
7 examples, 0 failures
```

Hummmm, na minha opinião, a sintaxe de uma linha facilita na escrita mas dificulta a leitura no formato de documentação. Para facilitar a comparação, vejamos a saída do primeiro VS esse último exemplo:

```
# == SEM a sintaxe de uma linha ==

Calc
  .add
    with numeric values
      returns the sum of the values
    with string values
      and all of them are numbers
        returns the sum of the values

# == COM a sintaxe de uma linha ==

Calc
  .add
    with numeric values
      is expected to eq 2
      is expected to eq 2.5
    with string values
      and all of them are numbers
        is expected to eq 2
        is expected to eq 2.5
```

## Concluindo a introdução ao Rspec

Acredito que impressione a expressividade que os testes podem ter ao fazer o uso dessa DSL. De fato, é possível escrever tanto testes como especificações.

Mas diferente do minitest, perceba que você precisará aprender e dominar a DSL ao invés de usar o que você já sabe de Ruby.

Ou seja, essa abstração tem um custo (um tanto alto) na curva de aprendizado e também de performance (minitest é mais rápido).

Para exemplificar como a curva de aprendizado é maior, lembra-se que no tópico de [conclusão do minitest](#concluindo-a-introdução-ao-minitest) eu abordei o conceito de `setup` e `teardown`? Então, no rspec você tem diferentes formas de fazer isso, como: <a href="https://relishapp.com/rspec/rspec-core/v/3-10/docs/hooks" target="_blank">before / before(:all) / after / after(:all) / around</a>, <a href="https://relishapp.com/rspec/rspec-core/v/3-10/docs/helper-methods/let-and-let" target="_blank">let / let!</a>, <a href="https://relishapp.com/rspec/rspec-core/v/3-10/docs/subject" target="_blank">subject</a>. Além de ter de aprender a precedência de cada um deles e como os mesmos afetam os contextos que estão dentro ou fora de um aninhamento.

## Concluindo a comparação e qual a minha preferência dentre os dois

Primeiro, gostaria de recomendar que você procure aprender ambos. Porque o mercado de trabalho exigirá um ou outro.

Quanto a minha preferência, sou a favor do **minitest** por conta de sua simplicidade. É muito fácil e rápido capacitar alguém a escrever testes com ele. Afinal você foca em usar Ruby e todas as asserção podem ser encontradas em <a href="https://www.rubydoc.info/gems/minitest/5.14.2/Minitest/Assertions" target="_blank">uma única página</a> de documentação e não em <a href="https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers" target="_blank">dezenas de páginas</a> como é o caso do rspec.

Além disso, faz um tempo que tenho visto um uso cada vez maior da sintaxe de uma linha nos testes com **rspec** que compromete o entendimento do formato de documentação. Algo que é destacado como um diferencial, afinal, você pode ter uma especificação funcional do seu código. Mas na prática, muita gente passou a usar essa sintaxe em conjunto com a saída padrão (pontinhos) para simplificar o processo. Daí eu pergunto, se o foco nesse caso é simplificar que tal então dar uma chance ao minitest?

Para não parecer que o rspec é pior (o que não é), algo que não foi coberto por não ser o foco do post é sua CLI que é absurdamente mais amigável e intuitiva que a do minitest (essa dor só é amenizada no Rails).

Minha questão com rspec é que ele exigirá mais de quem escreve e mantém os testes por conta dos seus conceitos e recursos, algo que não acontece pela simplicidade e objetividade do minitest.

Testes geram confiança e permitem velocidade em qualquer equipe de desenvolvimento. Por isso sou a favor de usar a ferramenta que melhor favoreça isso, a que permita fazer mais com menos. E que nesse caso, dentro do tema testes em Ruby, o melhor custo benefício é o minitest na minha humilde opinião.

Gostou do conteúdo? Deixe seu comentário aqui embaixo contando o que achou. Valeu! 😉

## Uma curiosidade...

Você sabia que o Shopify (deve ser a maior aplicação de Ruby do mundo com suas 3 milhões de linhas de código) faz uso apenas do minitest?

Acesse esse <a href="https://t.me/ruby_arch_design_br/11431" target="_blank">link</a> (conteúdo pt-BR) para conferir quais são as razões para eles preferirem o minitest ao invés do rspec.

---

Já ouviu falar do **ada.rb - Arquitetura e Design de Aplicações em Ruby**? É um grupo focado em práticas de engenharia de software com Ruby. Acesse o <a href="https://t.me/ruby_arch_design_br" target="_blank">canal no telegram</a> e junte-se a nós em nosso <a href="https://meetup.com/pt-BR/arquitetura-e-design-de-aplicacoes-ruby/" target="_blank">meetup mensal</a> (100% on-line).
