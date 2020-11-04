---
title: "[Em construção] Introdução a testes automatizados (TDD) com Ruby"
hidden: true
last_modified_at: 2020-11-01T23:58:00-03:00
categories:
  - Blog
tags:
  - ruby
  - tdd
---

Olá, se tem uma coisa que pessoas e equipes de auto desempenho possuem é **confiança**. Sendo que em desenvolvimento de software é muito importante ter certeza que o teu código funciona na medida que é modificado. A técnica mais comum e intuitiva para aumentar confiança na medida que se promove mudanças é a escrita de roteiros de testes, que serão realizados manualmente. Ou seja, um ser humano, executará uma série de ações em sequência afim de avaliar se os resultados esperados foram obtidos.

Testes realizados por seres humanos são válidos, mas é um recurso lento e caro se comparado a um processo automatizado. Por conta disso, nossa industria desenvolveu técnicas e ferramentas que permitem que boa parte desse processo seja o mais automático possível.

Existem várias técnicas para se testar software. Neste artigo, utilizarei como exemplo o TDD (**T**est **D**riven **D**evelopment), que pode ser traduzido como: **desenvolvimento orientado a testes**. O TDD é a prática de
se escrever os testes primeiro, isto é, você implementa testes (que naturalmente vão falhar) e só então
adiciona o código necessário para que eles passem.

Esse post será dividido em duas partes, a primeira abordará o que é e por que utilizar TDD, e a última te ensinará como usar na prática! Como? Você aprenderá na medida que escreve a sua própria lib de testes 💪.

## O que é TDD?

Tdd é um ciclo de desenvolvimento no qual você escreve primeiro o teste e depois a implementação. Sendo essas as suas etapas:

### Primeiro passo: <span style="font-weight: normal;">escrever um teste que irá falhar</span>.

{% include figure image_path="/assets/images/3-tdd/1_Red.png" alt="Tdd: Red = Teste falhando" caption="Tdd: Red = Teste falhando." %}

Primeiro, você tem o <span style="color: red;">*red mode*</span>, no qual é escrito um teste que falha. Nele você descreverá o comportamento que o código deverá ter. Seja uma série de ações ou apenas uma parte/unidade do comportamento esperado.

### Próximo passo: <span style="font-weight: normal;">fazer o teste passar</span>.

{% include figure image_path="/assets/images/3-tdd/2_Red-Green.png" alt="Tdd: Red + Green = Fazer o teste passar com o mínimo de código necessário" caption="Tdd: Red + Green = Fazer o teste passar com o mínimo de código necessário." %}

Nessa etapa, você escreverá apenas a quantidade necessária para fazer o teste ficar verde, ou seja, entrar no <span style="color: green;">*green mode*</span>.

### Último passo: <span style="font-weight: normal;">refatorar o código produzido</span>.

{% include figure image_path="/assets/images/3-tdd/3_Red-Green-Refactor.png" alt="Tdd: Red + Green + Refactor = Refinar a estrutura do código sem quebrar o comportamento." caption="Tdd: Red + Green + Refactor = Refinar a estrutura do código sem quebrar o comportamento." %}

Nessa etapa, se torna possível <span style="color: blue;">refinar o código</span> ao mesmo tempo que se tem garantias de que o mesmo funciona. Ou seja, o teste tem que continuar verde.

> <span style="font-style: normal;">**Dica:** Só refine o código se for realmente necessário! 🙂</span>

**Significado de refatoração**: É o processo de melhorar a estrutura de um código/sistema enquanto se preserva o comportamento existente.

### Por fim: <span style="font-weight: normal;">repita o ciclo</span>.

{% include figure image_path="/assets/images/3-tdd/4_Red-Green-Refactor-Cycle.png" alt="Tdd: Red + Green + Refactor (Cycle) = Repita o ciclo para a próxima implementação." caption="Tdd: Red + Green + Refactor (Cycle) = Repita o ciclo para a próxima implementação." %}

Repita o ciclo para a próxima funcionalidade/comportamento a ser entregue.
1. Escreva um <a style="color: red;" href="#primeiro-passo-escrever-um-teste-que-irá-falhar">teste que falha</a>.
2. Faça o <a style="color: green;" href="#próximo-passo-fazer-o-teste-passar">teste passar</a>.
3. <a style="color: blue;" href="#próximo-passo-refatorar-o-código-produzido">Refatore o código</a> sempre que necessário.

A ideia é repetir esse ciclo para cada funcionalidade que o sistema receber, até que ele seja considerado finalizado.

## Por que fazer TDD?

Ao fazer isso você e sua equipe obterão os seguintes benefícios:
1. **Erradicar o medo de modificar o sistema/código fonte.**
  Pelo fato dos testes estarem verdes, será possível alterar a lógica de negócio e comportamentos da aplicação uma vez que as especificações estejam garantidas.
2. **Refatoração se torna algo fácil e seguro de ser feita.**
  Caso algum teste falhe durante um *refactoring* será possível identificar o problema e então corrigí-lo.

A curto prazo tudo isso pode parecer improdutivo, já que você irá escrever mais código do que o necessário para chegar no resultado desejado. Mas no longo prazo, será um diferencial ter essa confiança para modificar o sistema conforme for preciso.

## Como fazer TDD?

Para responder a essa pergunta te convido a escrever uma lib de testes para colocarmos o TDD em prática. Usarei o <a href="/blog/as-diferentes-formas-de-declarar-comportamento-em-ruby/" title="Programação multiparadigma - As diferentes formas de declarar comportamento em Ruby">post anterior</a> como base para nos auxiliar na implementação e exploração dos conceitos.

Um dos conceitos chaves dessa prática é a que testes devem ser encarados como uma **fonte de verdade**. Ou seja, **os teste não podem falhar**, se apenas um falhar essa fonte ficou comprometida.

Dado o conceito acima, que tal usarmos <a href="https://www.honeybadger.io/blog/a-beginner-s-guide-to-exceptions-in-ruby/" target="_blank">*exceptions*</a> para interromper a execução de nossa suíte de testes em caso de falha?

```ruby
a = 1
b = 2

raise 'os valores devem ser iguais.' if a != b
```

O resultado pelo código acima será:

```ruby
RuntimeError: os valores devem ser iguais.
```

> Escreva nos comentários caso você tenha mais interesse em aprender sobre exceptions. <span style="font-style: normal;">😉</span>

Será que você teve a mesma ideia que eu? E se usarmos uma sequência de condicionais que podem lançar exceptions para iniciar a definição dos nossos testes?

Afim de ter uma relação com o <a href="/blog/as-diferentes-formas-de-declarar-comportamento-em-ruby/" title="Programação multiparadigma - As diferentes formas de declarar comportamento em Ruby">post anterior</a>, usaremos o exemplo de uma calculadora para colocar em prática o uso de TDD.

Como já foi dito, primeiro precisamos escrever um teste que falha:

```ruby
raise 'asserção falhou' if sum(1, 1) != 2
```

O que acontecerá se o código acima for executado? Dará erro! Por que? Porque o método não existe! 😅

```ruby
NoMethodError: undefined method `sum' for main:Object
```

Então, bora definir o método para evitar que esse erro?

```ruby
def sum(a, b)
end

raise 'asserção falhou' if sum(1, 1) != 2
```

Resultado do código acima será:

```ruby
RuntimeError: asserção falhou
```

Opa! O teste falhou. 🙌
Ou seja, entramos no <span style="color: red;">*red mode*</span>.

---
Antes de continuarmos, precisamos abrir um rápido parênteses: Por que estamos usando a palavra **asserção**?

Essa palavra pode ser <a href="https://www.dicio.com.br/assercao/" target="_blank">definida</a> como: Afirmação que se faz com muita certeza.

Usando nosso último exemplo, lançaremos um erro caso a soma de 1 + 1 for diferente de 2.

```ruby
raise 'asserção falhou' if sum(1, 1) != 2
```
---

Dado que estamos nosso teste está falhando, bora implementar o **mínimo** necessário para ele passar.

```ruby
def sum(a, b)
  2
end

raise 'asserção falhou' if sum(1, 1) != 2
```

Ao executar o código anterior, verá que o mesmo não teve erros (nenhuma exception foi lançada). Com isso, entramos no <span style="color: green;">*green mode*</span>.

Imagino que você já percebeu que o nosso método de soma, não soma! Mas acredite, essa é a proposta do TDD, você só escreve o código necessário para o teste passar. Quanto menos código para manter, melhor.

E como ainda não temos código o suficiente para refatorar, sugiro reiniciarmos o ciclo e escrevermos um novo teste que falha.

```ruby
def sum(a, b)
  2
end

raise 'asserção falhou' if sum(1, 1) != 2 # nil
raise 'asserção falhou' if sum(1, 3) != 4 # RuntimeError: asserção falhou
```

Dado a falha anterior, podemos implementar o código necessário para ele passar.

```ruby
def sum(a, b)
  a + b
end

raise 'asserção falhou' if sum(1, 1) != 2 # nil
raise 'asserção falhou' if sum(1, 3) != 4 # nil
```

Uhuuuu! Os testes voltaram a passar. Legal né?

Mas assim, que tal começarmos a refatorar a nossa "lib de testes"? Afinal podemos refatorá-la enquanto nossos testes passarem! 🤯

Para isso, sugiro criarmo um método `assert` que lançará uma exception caso o valor do argumento seja `false`.

```ruby
def assert(truthy)
  raise 'asserção falhou' unless truthy
end
```

Obs: Podemos usar `unless` para negar uma condição, ou seja, é o mesmo que fazer `if !false`.

A seguir, veja o exemplo completo da nosso método de asserção sendo usado para testar nossa implementação:

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
```

To-do...

## Agradecimentos

Quero agradecer a colaboração do <a href="github.com/tomascco" target="_blank">@tomascco</a>, <a href="github.com/mploureno" target="_blank">@mploureno</a> e <a href="github.com/joaomarcos96" target="_blank">@joaomarcos96</a> na revisão deste post. Muito obrigado! 👏👏👏

---

Valeu! 🙂

---

Já ouviu falar do **ada.rb - Arquitetura e Design de Aplicações em Ruby**? É um grupo focado em práticas de engenharia de software com Ruby. Acesse o <a href="https://t.me/ruby_arch_design_br" target="_blank">canal no telegram</a> e junte-se a nós em nosso <a href="https://www.meetup.com/pt-BR/design-e-arquitetura-de-aplicacoes-ruby/events/" target="_blank">meetup mensal</a> (100% on-line).
