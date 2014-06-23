---
layout: post
status: publish
published: true
title: 'Anatomia de uma solução: VRaptor - linkTo para jsp - parte 1'
author:
  display_name: Lucas Cavalcanti
  login: lucascs
  email: lucas@cavalcanti.me
  url: ''
author_login: lucascs
author_email: lucas@cavalcanti.me
wordpress_id: 9
wordpress_url: http://blog.cavalcanti.me/?p=9
date: '2011-06-17 11:05:56 -0300'
date_gmt: '2011-06-17 14:05:56 -0300'
categories:
- solução
- vraptor
tags:
- vraptor
comments:
- id: 2
  author: Luiz Fernando Signorelli
  author_email: luiz.sg@gmail.com
  author_url: http://luizsignorelli.com
  date: '2011-06-17 11:43:21 -0300'
  date_gmt: '2011-06-17 14:43:21 -0300'
  content: "Estive pensando nisso outro dia.\r\nAguardo ansioso pelo próximo post.\r\n\r\nabraço"
- id: 3
  author: Kenneth Reis
  author_email: javaki@gmail.com
  author_url: ''
  date: '2011-06-17 11:52:36 -0300'
  date_gmt: '2011-06-17 14:52:36 -0300'
  content: "Cara, idéia interessante!\r\nMuito bom"
- id: 4
  author: Washington Botelho
  author_email: wbotelhos@gmail.com
  author_url: http://wbotelhos.com.br
  date: '2011-06-17 13:14:31 -0300'
  date_gmt: '2011-06-17 16:14:31 -0300'
  content: "Se a solução já pegar o contexto ficará ainda melhor. (:\r\n\r\nParabéns
    pelo blog. o/"
- id: 6
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2011-06-17 15:28:25 -0300'
  date_gmt: '2011-06-17 18:28:25 -0300'
  content: Sim, a idéia é já retornar a URI pronta pra ser usada =)
- id: 7
  author: Guevara
  author_email: eguevara2012@gmail.com
  author_url: http://guevara2012.wordpress.com
  date: '2011-06-17 15:56:44 -0300'
  date_gmt: '2011-06-17 18:56:44 -0300'
  content: "Beleza Lucas?\r\nEstive estudando um pouco de Python e Django esses tempos.
    Descobri no Django recursos muito interessantes que resolvem esses problemas no
    desenvolvimento e que poderiam ser usados tb no VRaptor, veja.\r\nUsando o mesmo
    exemplo de visualização, na html ficaria:\r\n\r\n<a href=\"{% url detail_funcionario
    funcionario.id %}\" rel=\"nofollow\">Visualizar</a>\r\n\r\nE na urls.py:\r\n\r\nurl(r'^funcionario/detail/(?P\\d+)/$',
    views.detail, name=\"detail_proprietario\"),  \r\n\r\nEle usa urls (path) nomeadas,
    repare o \"name\", ou seja, basta chamar o nome da url ao invés de chamar pelo
    caminho exato na html. Com isto, o método detail pode ser alterado que não irá
    afetar os outros links que estiverem usando o mesmo método.\r\nSe quiser dar uma
    olhada:\r\nhttps://docs.djangoproject.com/en/dev/topics/http/urls/#naming-url-patterns\r\n\r\nGrande
    abraço!"
- id: 8
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2011-06-17 16:03:55 -0300'
  date_gmt: '2011-06-17 19:03:55 -0300'
  content: |-
    Olá,

    É uma funcionalidade bem interessante, daria para fazer algo nesse estilo também.

    Abre uma issue pra isso por favor?
    http://github.com/caelum/vraptor/issues

    Abraço
- id: 9
  author: Guevara
  author_email: eguevara2012@gmail.com
  author_url: http://guevara2012.wordpress.com
  date: '2011-06-17 16:34:18 -0300'
  date_gmt: '2011-06-17 19:34:18 -0300'
  content: "Lucas, abri uma issue lá, veja se ficou legal:\r\nhttps://github.com/caelum/vraptor/issues/368\r\nAbraço!!"
- id: 10
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2011-06-17 17:08:38 -0300'
  date_gmt: '2011-06-17 20:08:38 -0300'
  content: ficou sim, valeu =)
- id: 17
  author: 'Anatomia de uma solução: VRaptor &#8211; linkTo para jsp &#8211; Final
    | Lucas Cavalcanti&#039;s Blog'
  author_email: ''
  author_url: http://lucas.cavalcanti.me/2011/07/06/anatomia-de-uma-solucao-vraptor-linkto-para-jsp-final/
  date: '2011-07-06 10:19:50 -0300'
  date_gmt: '2011-07-06 13:19:50 -0300'
  content: '[...] partes 1 e 2 conseguimos fazer com que isso funcionasse manualmente:
    ${linkTo[ProdutoController].adiciona} [...]'
- id: 22
  author: Otávio
  author_email: otavio@otavio.com.br
  author_url: ''
  date: '2011-07-15 10:56:09 -0300'
  date_gmt: '2011-07-15 13:56:09 -0300'
  content: "Oi Lucas. \r\n\r\nPoxa, eu ví esse código nascendo na tela do teu notebook
    lá no Starbucks. Realmente tu és um ninja dos códigos.\r\n\r\nJá adicionei ao
    meu reader.\r\n\r\nAbraço."
---
<p>Uma das desvantagens de usar VRaptor é que sempre temos que digitar as URIs duas vezes - no controller e na view:</p>
<pre lang="java">
@Resource
public class ProdutoController {
    @Post("/produtos/{id}")
    public void visualiza(Long id) {...}
}
</pre>
<pre lang="HTML">
<a href="<c:url value="/produtos/${produto.id}"/>">${produto.nome}</a>
</pre>
<p>Assim, durante o desenvolvimento você precisa lembrar qual é a URI correta do método, sem garantia nenhuma de que ela é a certa - se por algum motivo mudarmos o path do método visualiza todos os links para ele quebram silenciosamente.</p>
<p>Para quem programa em Rails, existe uma solução para isso, um helper que gera os links dado um controller e um método:</p>
<pre lang="ruby">
<% link_to "Um produto", :controller => "produtos", :action => "visualiza", :id => 4 %>
</pre>
<p>Num projeto que eu estou desenvolvendo em VRaptor + Scala, com o <a href="http://scalate.fusesource.org/">Scalate</a> como template engine, conseguimos chegar nisso:</p>
<pre lang="html">
${linkTo[ProdutoController](_.visualiza(3))}  => /produtos/3
</pre>
<p>E ainda ganhando checagem estática: o template não vai compilar se não existir o método visualiza em ProdutoController, que recebe um número como parâmetro. Isso é possível porque no ssp conseguimos executar qualquer método scala, com a sintaxe padrão do scala.</p>
<p>Mas será que algo parecido com isso é possível com JSP?</p>
<p>O primeiro problema é que usando a EL padrão do JSP você não pode executar qualquer método de um objeto, somente getters. Executar métodos só criando uma Tag, ou sobrescrevendo o ELResolver - ambas soluções bastante trabalhosas.</p>
<p>Semana passada eu estava pareando com o Otávio Garcia, que é um grande contribuidor do VRaptor e estava visitando a Caelum, e a gente resolveu tentar encontrar uma solução para isso.</p>
<p>Chegamos nessa solução, sem criar tags nem sobrescrever o <em>resolver</em>:</p>
<pre lang="html">
${linkTo[ProdutoController].visualiza[3]}
</pre>
<p>Será que isso funciona? Será que é possível fazer isso retornar a URI "/produtos/3"?</p>
<p>Tem idéia de como fazer isso funcionar? Como seria? No próximo post eu explico qual foi a nossa solução, com pitadas de magia negra e técnicas interessantes. ;)</p>
