---
title: "Dica: Como instalar e executar código Ruby (para iniciantes)"
last_modified_at: 2022-10-07T21:48:00-03:00
categories:
  - Blog
tags:
  - iniciante
  - ruby
---

- [Links para instalar Ruby/Rails](#links-para-instalar-rubyrails)
- [Executando código Ruby](#executando-código-ruby)
- [Concluindo](#concluindo)
- [Desafios](#desafios)

Olá, nesse post irei compartilhar referências com excelentes explicações de como preparar um ambiente de desenvolvimento Ruby/Rails e demonstrar de maneira prática como executar arquivos **.rb**.

## Links para instalar Ruby/Rails

O primeiro link que recomendo é o <a href="https://gorails.com/setup" target="_blank">https://gorails.com/setup</a> que te apresenta como preparar um ambiente Ruby em diferentes sistema operacionais como: <a href="https://gorails.com/setup/ubuntu" target="_blank">Linux/Ubuntu</a>, <a href="https://gorails.com/setup/windows" target="_blank">Windows</a> e <a href="https://gorails.com/setup/osx" target="_blank">macOS (OSX)</a>.

A segunda recomendação é a documentação oficial do <a href="https://guides.rubyonrails.org/development_dependencies_install.html" target="_blank">Ruby on Rails</a> que te ensina como instalar o framework e suas dependências (git, banco de dados e etc).

## Executando código Ruby

Primeiro, verifique se o Ruby está instalado em seu ambiente. Para fazer isso, acesse o seu terminal e execute `ruby -v`.

Se tudo der certo você verá algo como:
```sh
ruby 3.1.2p20 (2022-04-12 revision 4491bb740a)
```

Uma vez que o Ruby está instalado, basta criar um arquivo com a extensão `.rb` e executá-lo com o comando `ruby`.

Veja um passo a passo de como fazer isso:
1. Crie o arquivo: `ola_mundo.rb`
2. Abra esse arquivo com o editor de sua preferência e escreva o seguinte trecho de código: `puts "Ola mundo"`
3. Execute o arquivo. Ex: `ruby ola_mundo.rb`

Veja a seguir um GIF com cada um dos passos acima.

{% include figure image_path="/assets/images/4-instalando-ruby/1_executando_codigo_ruby.gif" alt="Verificando a versão do Ruby e executando um arquivo .rb" caption="Verificando a versão do Ruby e executando um arquivo .rb" %}

## Concluindo

Meu objetivo com este conteúdo é em orientar iniciantes em como consumir o conteúdo do meu e de outros blogs/sites afim de conseguirem testar os exemplos de código disponíveis.

Gostou do conteúdo? Deixe seu comentário aqui embaixo contando o que achou. Valeu! 😉

## Desafios

<a href="/pt-BR/posts/" target="_blank">Leia os outros posts do blog</a> e crie arquivos `.rb` com os exemplos de código para então executá-los e modificá-los (melhor forma de aprender é praticando).

---

Já ouviu falar do **ada.rb - Arquitetura e Design de Aplicações em Ruby**? É um grupo focado em práticas de engenharia de software com Ruby. Acesse o <a href="https://t.me/ruby_arch_design_br" target="_blank">canal no telegram</a> e junte-se a nós em nosso <a href="https://meetup.com/pt-BR/arquitetura-e-design-de-aplicacoes-ruby/" target="_blank">meetup mensal</a> (100% on-line).
