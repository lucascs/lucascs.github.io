---
layout: post
status: publish
published: true
title: 'Anatomia de uma solução: VRaptor - linkTo para jsp - parte 2'
author:
  display_name: Lucas Cavalcanti
  login: lucascs
  email: lucas@cavalcanti.me
  url: ''
author_login: lucascs
author_email: lucas@cavalcanti.me
wordpress_id: 11
wordpress_url: http://lucas.cavalcanti.me/?p=11
date: '2011-06-19 23:39:27 -0300'
date_gmt: '2011-06-20 02:39:27 -0300'
categories:
- solução
- vraptor
tags:
- vraptor
comments:
- id: 11
  author: Sérgio Lopes
  author_email: slopes@gmail.com
  author_url: http://www.caelum.com.br/
  date: '2011-06-20 12:35:31 -0300'
  date_gmt: '2011-06-20 15:35:31 -0300'
  content: "E com a EL 2.2, não resolveria o problema mais facilmente? Digo, já dá
    pra invocar métodos na EL, passar params, varargs etc\r\n\r\nAbraços"
- id: 12
  author: Arthur Carvalho
  author_email: armoucar@gmail.com
  author_url: http://twitter.com/arthurmoucar
  date: '2011-06-20 13:17:31 -0300'
  date_gmt: '2011-06-20 16:17:31 -0300'
  content: "Legal o lance do Guava.\r\nNa primeira linha que eu vi, por um momento
    pensei que era frescura. Mas se tratando em legibilidade do código, ainda valia
    apena.\r\nPra declaração do segundo Map, com certeza o efeito que causa na vizualização
    do código é muito melhor. Vou ver mais sobre a API.\r\n\r\nAgora sobre as URIs
    com id's não consegui pensar em algo tão limpo quanto está fincando.\r\nAdicionaria
    as URIs no mapa sem a parte de {campo.id} e utilizaria o varStatus do c:forEach
    mesmo.\r\n\r\nAguardo o próximo post!\r\nValeu."
- id: 13
  author: Kenneth Reis
  author_email: javaki@gmail.com
  author_url: ''
  date: '2011-06-20 15:23:12 -0300'
  date_gmt: '2011-06-20 18:23:12 -0300'
  content: "Oi Lucas, eu solucionaria isso usando reflexão e uma function declarada
    numa TLD:\r\n\r\n<b>taglib.tld</b>\r\n<code>\r\n  \r\n\r\n\t\r\n    1.1\r\n    cf\r\n
    \   http://abc.com.br/abc\r\n\r\n    \r\n        link\r\n        org.sistema.Utils\r\n
    \       java.lang.String link(java.lang.String)\r\n    \r\n\r\n</code>\r\n\r\n\r\n<b>org.sistema.Utils</b><code>\r\npackage
    org.sistema;\r\n\r\npublic class Utils{\r\n  public static String link(String
    actionPath) {\r\n        Class klazz = null;\r\n        String classNamePath =
    actionPath.substring(0, actionPath.lastIndexOf(\".\"));\r\n        String methodName
    \   = actionPath.substring(actionPath.lastIndexOf(\".\") + 1);\r\n        try
    {\r\n            klazz = Class.forName(classNamePath);\r\n        } catch (ClassNotFoundException
    ex) {\r\n            Logger.getLogger(Util.class.getName()).log(Level.SEVERE,
    null, ex);\r\n        }\r\n\r\n        Method met = null;\r\n        for (int
    i = 0; i &lt; klazz.getMethods().length; ++i)\r\n            if (klazz.getMethods()[i].getName().equals(methodName))\r\n
    \               met = klazz.getMethods()[i];\r\n\r\n        return met.getAnnotation(Post.class).value()[0];\r\n
    \   }\r\n}</code>\r\n\r\ne para usar numa jsp:\r\n<code>\r\n\r\n\r\n\r\n${cf:link('com.meusistema.controllers.ProdutoController.adiciona')}\r\n\r\n</code>\r\n\r\nPoderia
    ser usado em qualquer projeto, melhor ainda se encapsulado num .jar da vida! xD"
- id: 14
  author: Kenneth Reis
  author_email: javaki@gmail.com
  author_url: ''
  date: '2011-06-20 15:34:22 -0300'
  date_gmt: '2011-06-20 18:34:22 -0300'
  content: "Eita, a formatação saiu errada, colei no pastebin\r\n<b>taglib.tld</b>\r\nhttp://pastebin.com/dEgsu0Kk\r\n\r\n<b>org.sistema.Utils.java</b>\r\nhttp://pastebin.com/9mGyyxeR\r\n\r\nQueria
    achar um jeito de usar só o nome da Classe na function:\r\n<code>${cf:link('ProdutoController.adiciona')}"
- id: 15
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2011-06-20 15:42:07 -0300'
  date_gmt: '2011-06-20 18:42:07 -0300'
  content: "a sua solução ainda não suporta os parâmetros ;)\r\nmas também é interessante.
    Você pode convencionar que todos os controllers estão no pacote com.meusistema.controllers,
    e usar a partir de lá.\r\nNo próximo post eu mostro como saber quais são todos
    os controllers do sistema, então tudo fica mais fácil, a gente não precisa chutar."
- id: 16
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2011-06-20 18:06:33 -0300'
  date_gmt: '2011-06-20 21:06:33 -0300'
  content: |-
    Dá pra executar métodos, mas não dá pra forçar que eles retornem string. No caso em que os métodos do controller retornam void não tem muito o que fazer.

    []'s
---
<p>Antes de começar a solução, vamos usar uma rota mais fácil:</p>
<pre lang="java">
@Resource
public class ProdutoController {
    @Post("/produtos")
    public void adiciona(Produto produto) {...}
}
</pre>
<p>A idéia é conseguir fazer com que:</p>
<pre lang="html">
${linkTo[ProdutoController].adiciona}
</pre>
<p>retorne a URI <b>/produtos</b>.</p>
<p>Vamos, primeiro, olhar para a primeira parte e relaxar um pouquinho a sintaxe:</p>
<pre lang="html">
${linkTo['ProdutoController']}
</pre>
<p>Qual é a única forma disso funcionar?  Na EL padrão você geralmente só pode navegar pelos getters, com três exceções: Lists, Arrays e Maps. Nas Lists e Arrays você pode acessar os índices (lista[0], array[1], etc), e nos mapas você pode acessar as chaves (mapa['chave']). Então pro código acima funcionar, podemos fazer o linkTo ser um mapa e adicioná-lo no request:</p>
<pre lang="java">
Map<String, ?> linkTo = Maps.newHashMap(); // do guava, para evitar declarar os generics de novo
linkTo.put("ProdutoController", objetoMagico);

request.setAttribute("linkTo", linkTo);
</pre>
<p>Agora só precisamos criar um <b>objetoMagico</b> em que eu possa chamar o <b>.adiciona</b>. Isso significa que precisamos de um objeto que tenha o método getAdiciona(). Até é possível gerar uma classe assim em tempo de compilação (com o APT) ou em runtime (com ASM ou javassist), uma classe para cada controller do sistema com getters para o nome de cada método. Mas isso é um pouco complicado demais (bem mais difícil que implementar a Tag ou o ELResolver que eu estava evitando), então vamos relaxar um pouco mais a sintaxe. Como fazer isso funcionar?</p>
<pre lang="html">
${linkTo['ProdutoController']['adiciona']}
</pre>
<p>Outro mapa!</p>
<pre lang="java">
Map<String, Map<String,String>> linkTo = Maps.newHashMap(); // viu como o guava é útil aqui? ;)

linkTo.put("ProdutoController", ImmutableMap.of("adiciona", "/produtos"));

request.setAttribute("linkTo", linkTo);
</pre>
<p>Usei aqui a classe <a href="http://guava-libraries.googlecode.com/svn/tags/release09/javadoc/com/google/common/collect/ImmutableMap.html">ImmutableMap</a> do <a href="http://code.google.com/p/guava-libraries/">guava</a>, pro código ficar mais conciso. Isso cria um mapa imutável com uma única entrada (adiciona -> /produtos).</p>
<p>Agora o código <b>${linkTo['ProdutoController']['adiciona']}</b> retorna o que a gente queria: <b>/produtos</b>.<br />
O legal é que os colchetes são apenas <b>um</b> dos jeitos de acessar chaves de um mapa, não o único. No caso em que a chave é uma String, podemos simplificar a sintaxe:</p>
<pre lang="html">
${linkTo['ProdutoController'].adiciona} => /produtos
</pre>
<p>Já chegamos bem próximos da sintaxe proposta no começo do post, só falta retirar as aspas. Mas será que eu posso simplificar a sintaxe para ficar assim?</p>
<pre lang="html">
${linkTo[ProdutoController].adiciona} => /produtos (?)
</pre>
<p>Infelizmente não. Quando fazemos isso, o JSP procura por uma variável chamada ProdutoController. Bom, não seja por isso, basta fazer uma pequena <del datetime="2011-06-20T03:10:45+00:00">gambiarra</del> adaptação:</p>
<pre lang="java">
request.setAttribute("ProdutoController", "ProdutoController");
</pre>
<p>E pronto, o código que queríamos funciona!</p>
<pre lang="html">
${linkTo[ProdutoController].adiciona} => /produtos !
</pre>
<p>Agora é só gerar esses mapas automaticamente, um para cada controller do VRaptor. Mas antes de fazer isso, existe um problema: a solução até aqui não suporta todas as rotas possíveis no VRaptor.</p>
<p>E se tivéssemos:</p>
<pre lang="java">
@Resource
public class ProdutoController {
    @Get("/produtos/{id}")
    public Produto visualiza(Long id) {...}

    @Delete("/produtos/{produto.id}")
    public void remove(Produto produto) {...}
}
</pre>
<p>Como fazer para gerar as URIs /produtos/4, /produtos/15, etc ainda sem Tags ou ELResolvers?<br />
Como fazer esse tipo de código funcionar?</p>

{% highlight xml %}
<c:forEach items="${produtos}" var="produto">
   <a href="${linkTo[ProdutoController].visualiza ??? produto.id ???}">Visualiza</a>
   <a href="${linkTo[ProdutoController].remove ??? produto ???}">Remove</a>
</c:forEach>
{% endhighlight %}

<p>E agora, como vocês fariam isso? No próximo post eu explico o resto da solução, com um pouco mais de magia negra e detalhes internos do VRaptor que podem te ajudar a customizar várias coisas de forma bem fácil.</p>
