---
title: "Introducing B/CDD"
last_modified_at: 2022-09-08T14:52:00-03:00
categories:
  - Blog
tags:
  - ruby
  - rails
  - bcdd
  - english
---

- [The process](#the-process)
- [The different stages/deliveries](#the-different-stagesdeliveries)
- [The feedback loop](#the-feedback-loop)
- [What would a codebase look like?](#what-would-a-codebase-look-like)
- [Content to share](#content-to-share)

This article is under construction. The main idea, for now, is to expose some content about the **B**usiness/**C**ontext **D**riven **D**evelopment process (B/CDD).

### The process

{% include figure image_path="/assets/images/6-bcdd/bcdd_the_process_v1.png" alt="B/CDD: The process." %}

1. **Prioritize:**
  * What problem do we need to solve? What is necessary to solve it?

2. **Check Requirements:**
  * Is acceptable the complexity and uncertainty level of what needs to be done?<br/><br/>If it is not, refine the scope/strategy before starting working on it!

3. **Build:**
  * Work within a quality level (ETC), and focus on good enough deliveries.<br/><br/>ETC = Easy To Change.

4. **Test:**
  * Ensure that your work works and that you'll have feedback if something stops working.

5. **Release:**
  * Communicate the changes and make them available.

6. **Learn:**
   * After understanding the impacts of what is done (ROI), set the following steps and try to promote improvements to the workflow.<br><br>ROI = Return On Investment

### The different stages/deliveries

{% include figure image_path="/assets/images/6-bcdd/bcdd_different_stages-deliveries_v1.png" alt="B/CDD: The different stages/deliveries." %}

1. Make it **Work**
2. Make it **Right**
3. Make it **Even Better**

They happen in different stages/deliveries and can be one or more pull requests for each of them.

### The feedback loop

{% include figure image_path="/assets/images/6-bcdd/bcdd_feedback_loop_v1.png" alt="B/CDD: The feedback loop." %}

The deliveries must always meet the minimum quality expectations.

Make it Work example:
>  A well-tested use case (the implementation can contain duplication or lack of abstractions) declared with a good name and inside a suitable namespace - the final result must be a cohesive module.

You can repeat the same stage until you have sure about prioritizing the next one.

The idea here is to move forward when there is clarity about what and how much can be done to improve what is already working in production.

### What would a codebase look like?

The following link contains a PDF with examples of different ways to organize the folders and files of a Ruby on Rails application. The B/CDD version would be like the last versions.

[Get the PDF (Ruby on Rails file structure by Rodrigo Serradura)]({{ site.url }}/pt-BR/assets/pdfs/ror_file_structure-rodrigo_serradura_v2.pdf)

Access this GitHub repository to see a Ruby on Rails application created following this process and applying the B/CDD code guidelines.

[https://github.com/serradura/todo-bcdd/](https://github.com/serradura/todo-bcdd/)

### Content to share

{% include figure image_path="/assets/images/6-bcdd/bcdd_the_process_description_v1.png" alt="B/CDD: The process description" %}

{% include figure image_path="/assets/images/6-bcdd/bcdd_feedback_loop_with_the_different_stages_v1.png" alt="B/CDD: The process with emphasis on the different stages/deliveries" %}

{% include figure image_path="/assets/images/6-bcdd/bcdd_different_stages-deliveries_description_v1.png" alt="B/CDD: The different stages/deliveries description." %}

---

Já ouviu falar do **ada.rb - Arquitetura e Design de Aplicações em Ruby**? É um grupo focado em práticas de engenharia de software com Ruby. Acesse o <a href="https://t.me/ruby_arch_design_br" target="_blank">canal no telegram</a> e junte-se a nós em nosso <a href="https://meetup.com/pt-BR/arquitetura-e-design-de-aplicacoes-ruby/" target="_blank">meetup mensal</a> (100% on-line).
