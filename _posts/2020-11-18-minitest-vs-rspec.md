---
title: "Minitest VS Rspec"
hidden: true
last_modified_at: 2020-11-18T23:14:00-03:00
categories:
  - Blog
tags:
  - ruby
  - tdd
  - minitest
  - rspec
---

Dando continuidade ao tema <a href="/pt-BR/blog/introducao-a-testes-automatizados-com-ruby/" target="_blank">testes (acesse o primeiro artigo)</a>, irei abordar as duas principais bibliotecas utilizadas na comunidade Ruby. São elas, o <a href="https://github.com/seattlerb/minitest" target="_blank">minitest</a> e o <a href="https://github.com/rspec/rspec" target="_blank">rspec</a>.

# minitest

Para introduzir está lib, quero destacar um trecho existente no README do projeto:

> minitest doesn't reinvent anything that ruby already provides, like: classes, modules, inheritance, methods. This means you only have to learn ruby to use minitest...

> **Tradução livre:** minitest não reinventa nada que o ruby já fornece, como: classes, módulos, herança, métodos. Isso significa que você só precisa aprender ruby para usar o minitest...

Ou seja, uma vez que você aprende os recursos essenciais desta ferramenta você só precisará de ruby puro.

Por conta disso, uma das coisas que mais me impressiona nesta gem é a sua curva de aprendizado, até o fim deste post você aprenderá o necessário para fazer um uso pleno do que ela tem a te oferecer. 😉

A seguir confira um exemplo no qual testaremos uma calculadora que sabe como realizar as operações com argumentos numéricos, ou com strings contendo números:

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

    def numeric_str?(value)
      value.is_a?(String) && value =~ /((\d+)?\.\d|\d+)/
    end

    def number(value)
      return value if value.is_a?(Numeric)
      return value.include?('.') ? value.to_f : value.to_i if numeric_str?(value)
      raise TypeError, 'the value must be Numeric or a String with numbers'
    end
end

require 'minitest/autorun'

class Calc::AdditionTest < Minitest::Test
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

Executando o exemplo anterior:

{% include figure image_path="/assets/images/5-minitest-vs-rspec/executando_o_minitest.gif" alt="Executando o exemplo anterior - minitest." %}

> <span style="font-style: normal;">**Obs:** Recomendo a leitura <a href="/pt-BR/blog/dicas-para-iniciantes-de-como-instalar-e-executar-codigo-ruby/" target="_blank">desse post</a> caso não saiba como executar os exemplos de código.</span>

Agora, que habemus código bora analisar o que foi usado:

To-do...
