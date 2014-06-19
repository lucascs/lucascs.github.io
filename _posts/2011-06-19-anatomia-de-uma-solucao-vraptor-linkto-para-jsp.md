---
layout: post
title: "Anatomia de uma solução: VRaptor – linkTo para jsp – parte 2"
permalink: /2011/06/19/anatomia-de-uma-solucao-vraptor-linkto-para-jsp/
tags: solucao, vraptor
---
{% include header.html %}

Antes de começar a solução, vamos usar uma rota mais fácil:

{% highlight java %}
@Resource
public class ProdutoController {
    @Post("/produtos")
    public void adiciona(Produto produto) {...}
}
{% endhighlight %}

A idéia é conseguir fazer com que:

```
${linkTo[ProdutoController].adiciona}
```

retorne a URI `/produtos`.

Vamos, primeiro, olhar para a primeira parte e relaxar um pouquinho a sintaxe:

```
${linkTo['ProdutoController']}
```

Qual é a única forma disso funcionar? Na EL padrão você geralmente só pode navegar pelos getters, com três exceções: Lists, Arrays e Maps. Nas Lists e Arrays você pode acessar os índices (`lista[0]`, `array[1]`, etc), e nos mapas você pode acessar as chaves (`mapa['chave']`). Então pro código acima funcionar, podemos fazer o linkTo ser um mapa e adicioná-lo no request:

{% highlight java %}
Map<String, ?> linkTo = Maps.newHashMap(); // do guava, para evitar declarar os generics de novo
linkTo.put("ProdutoController", objetoMagico);

request.setAttribute("linkTo", linkTo);
{% endhighlight %}

Agora só precisamos criar um objetoMagico em que eu possa chamar o .adiciona. Isso significa que precisamos de um objeto que tenha o método getAdiciona(). Até é possível gerar uma classe assim em tempo de compilação (com o APT) ou em runtime (com ASM ou javassist), uma classe para cada controller do sistema com getters para o nome de cada método. Mas isso é um pouco complicado demais (bem mais difícil que implementar a Tag ou o ELResolver que eu estava evitando), então vamos relaxar um pouco mais a sintaxe. Como fazer isso funcionar?

```
${linkTo['ProdutoController']['adiciona']}
```

Outro mapa!

{% highlight java %}
Map<String, Map<String,String>> linkTo = Maps.newHashMap(); // viu como o guava é útil aqui? ;)

linkTo.put("ProdutoController", ImmutableMap.of("adiciona", "/produtos"));

request.setAttribute("linkTo", linkTo);
{% endhighlight %}

Usei aqui a classe ImmutableMap do guava, pro código ficar mais conciso. Isso cria um mapa imutável com uma única entrada (adiciona -> /produtos).
Agora o código `${linkTo['ProdutoController']['adiciona']}` retorna o que a gente queria: /produtos.
O legal é que os colchetes são apenas um dos jeitos de acessar chaves de um mapa, não o único. No caso em que a chave é uma String, podemos simplificar a sintaxe:
```
${linkTo['ProdutoController'].adiciona} => /produtos
```

Já chegamos bem próximos da sintaxe proposta no começo do post, só falta retirar as aspas. Mas será que eu posso simplificar a sintaxe para ficar assim?
```
${linkTo[ProdutoController].adiciona} => /produtos (?)
```

Infelizmente não. Quando fazemos isso, o JSP procura por uma variável chamada ProdutoController. Bom, não seja por isso, basta fazer uma pequena gambiarra adaptação:

{% highlight java %}
request.setAttribute("ProdutoController", "ProdutoController");
{% endhighlight %}

E pronto, o código que queríamos funciona!

```
${linkTo[ProdutoController].adiciona} => /produtos !
```

Agora é só gerar esses mapas automaticamente, um para cada controller do VRaptor. Mas antes de fazer isso, existe um problema: a solução até aqui não suporta todas as rotas possíveis no VRaptor.
E se tivéssemos:

{% highlight java %}
@Resource
public class ProdutoController {
    @Get("/produtos/{id}")
    public Produto visualiza(Long id) {...}

    @Delete("/produtos/{produto.id}")
    public void remove(Produto produto) {...}
}
{% endhighlight %}

Como fazer para gerar as URIs /produtos/4, /produtos/15, etc ainda sem Tags ou ELResolvers?
Como fazer esse tipo de código funcionar?

{% highlight xml %}
<c:forEach items="${produtos}" var="produto">
   <a href="${linkTo[ProdutoController].visualiza ??? produto.id ???}">Visualiza</a>
   <a href="${linkTo[ProdutoController].remove ??? produto ???}">Remove</a>
</c:forEach>
{% endhighlight %}

E agora, como vocês fariam isso? No próximo post eu explico o resto da solução, com um pouco mais de magia negra e detalhes internos do VRaptor que podem te ajudar a customizar várias coisas de forma bem fácil.
