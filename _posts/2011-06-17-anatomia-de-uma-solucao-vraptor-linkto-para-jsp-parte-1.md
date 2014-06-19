---
layout: post
title: "Anatomia de uma solução: VRaptor – linkTo para jsp – parte 1"
permalink: /2011/06/17/anatomia-de-uma-solucao-vraptor-linkto-para-jsp-parte-1
tags: solucao, vraptor
---
{% include header.html %}

Uma das desvantagens de usar VRaptor é que sempre temos que digitar as URIs duas vezes – no controller e na view:

{% highlight java %}
@Resource
public class ProdutoController {
    @Post("/produtos/{id}")
    public void visualiza(Long id) {...}
}
{% endhighlight %}

{% highlight xml %}
<a href="<c:url value="/produtos/${produto.id}"/>">${produto.nome}</a>
{% endhighlight %}

Assim, durante o desenvolvimento você precisa lembrar qual é a URI correta do método, sem garantia nenhuma de que ela é a certa – se por algum motivo mudarmos o path do método visualiza todos os links para ele quebram silenciosamente.
Para quem programa em Rails, existe uma solução para isso, um helper que gera os links dado um controller e um método:

{% highlight erb %}
<% link_to "Um produto", :controller => "produtos", :action => "visualiza", :id => 4 %>
{% endhighlight %}

Num projeto que eu estou desenvolvendo em VRaptor + Scala, com o Scalate como template engine, conseguimos chegar nisso:

```
${linkTo[ProdutoController](_.visualiza(3))}  => /produtos/3
```

E ainda ganhando checagem estática: o template não vai compilar se não existir o método visualiza em ProdutoController, que recebe um número como parâmetro. Isso é possível porque no ssp conseguimos executar qualquer método scala, com a sintaxe padrão do scala.
Mas será que algo parecido com isso é possível com JSP?
O primeiro problema é que usando a EL padrão do JSP você não pode executar qualquer método de um objeto, somente getters. Executar métodos só criando uma Tag, ou sobrescrevendo o ELResolver – ambas soluções bastante trabalhosas.
Semana passada eu estava pareando com o Otávio Garcia, que é um grande contribuidor do VRaptor e estava visitando a Caelum, e a gente resolveu tentar encontrar uma solução para isso.
Chegamos nessa solução, sem criar tags nem sobrescrever o resolver:
```
${linkTo[ProdutoController].visualiza[3]}
```

Será que isso funciona? Será que é possível fazer isso retornar a URI `"/produtos/3"`?
Tem idéia de como fazer isso funcionar? Como seria? No próximo post eu explico qual foi a nossa solução, com pitadas de magia negra e técnicas interessantes.

