---
layout: post
status: publish
published: true
title: Deixando seu código mais funcional com Scala for comprehensions
author:
  display_name: Lucas Cavalcanti
  login: lucascs
  email: lucas@cavalcanti.me
  url: ''
author_login: lucascs
author_email: lucas@cavalcanti.me
wordpress_id: 47
wordpress_url: http://lucas.cavalcanti.me/?p=47
permalink: 2011/10/04/codigo-mais-funcional-com-scala-for-comprehensions/
date: '2011-10-04 07:00:52 -0300'
date_gmt: '2011-10-04 10:00:52 -0300'
categories:
- scala
tags:
- scala
- for comprehension
- refatoração
- programação funcional
comments:
- id: 40
  author: Rafael de F. Ferreira
  author_email: public@rafaelferreira.net
  author_url: http://www.rafaelferreira.net/
  date: '2011-10-15 21:49:53 -0300'
  date_gmt: '2011-10-16 01:49:53 -0200'
  content: Oi Lucas. Estou com preguiça de tentar, mas acho que se você especificar
    o tipo de retorno do getParameterTypes, não precisa fazer o Map(entries:_*). A
    mágica do CanBuildFrom gera o Map para você.
---
<p>Há alguns meses estava pareando com o <a href="http://twitter.com/alberto_souza">Alberto Souza</a> para melhorar o <a href="https://github.com/caelum/vraptor-scala">plugin do VRaptor para scala</a> fazendo o VRaptor navegar pelos getters (properties) do Scala.</p>
<p>O componente a ser sobrescrito no VRaptor é o <a href="https://github.com/caelum/vraptor/blob/master/vraptor-core/src/main/java/br/com/caelum/vraptor/http/route/TypeFinder.java">TypeFinder</a>, que é responsável por descobrir os tipos dos caminhos que colocamos nas URI templates do VRaptor. Ex:</p>

{% highlight java %}
@Get("/livros/{livro.id}") //==> o parâmetro livro.id é um Long, por exemplo
public void visualiza(Livro livro) {...}
{% endhighlight %}

<p>A implementação padrão é a seguinte (não precisa tentar entender o que acontece ainda, é só pra mostrar o estilo do código):</p>

{% highlight java %}
public Map<String, Class<?>> getParameterTypes(Method method, String[] parameterPaths) {
  Map<String,Class<?>> result = new HashMap<String, Class<?>>();
  String[] parameterNamesFor = provider.parameterNamesFor(method);
  for (String path : parameterPaths) {
    for (int i = 0; i < parameterNamesFor.length; i++) {
      String name = parameterNamesFor[i];
      if (path.startsWith(name + ".") || path.equals(name)) {
        String[] items = path.split("\.");
        Class<?> type = method.getParameterTypes()[i];
        for (int j = 1; j < items.length; j++) {
          String item = items[j];
          try {
            type = new Mirror().on(type).reflect().method("get" + upperFirst(item)).withoutArgs().getReturnType();
          } catch (Exception e) {
             throw new IllegalArgumentException("Parameters paths are invalid: " + Arrays.toString(parameterPaths) + " for method " + method, e);
          }
        }
        result.put(path, type);
      }
    }
  }
  return result;
}
{% endhighlight %}

<p>A idéia do código é, basicamente, pegar os caminhos ({livro.id}), partir dos nomes dos parâmetros ([livro]) e procurar o tipo (livro.getId() retorna Long). Mas o principal é perceber que isso é um típico código imperativo em java: pegar todos os dados necessários (parameterPaths, parameterNames, etc) percorrê-los e popular um mapa (result) que é o resultado esperado do método.</p>
<p>Para transformar esse código em funcional precisamos mudar o nosso pensamento. Para deixar o código mais funcional o mais natural é, a partir dos dados, aplicar funções e transformá-los em outros dados. Algo como criar uma função que tem como entrada a lista de paths e a lista de nomes e, como saída, o mapa de caminhos para tipos. Até aí, nada muito diferente da programação imperativa. A grande sacada é que não vamos chegar ao nosso objetivo executando comandos um atrás do outro, mas sim fazer composição de funções (lembra do f ∘ g do colégio?) transformando, aos poucos, a entrada na saída esperada.</p>
<p>A tradução direta do código acima para Scala, feita automaticamente com o menu 'Convert Java file to Scala' do <a href="http://www.jetbrains.com/idea/">IntelliJ</a> e levemente modificada retirando elementos desnecessários, fica assim:</p>

{% highlight scala %}
def getParameterTypes(method: Method, parameterPaths: Array[String]) = {
  val result = new HashMap[String, Class[_]]
  val parameterNamesFor = provider.parameterNamesFor(method)
  for (path <- parameterPaths) {
    for (i <- 0 to (parameterNamesFor.length - 1)) {
      val name = parameterNamesFor(i)
      if (path.startsWith(name + ".") || (path == name)) {
        val items = path.split("\.")
        var clazz = method.getParameterTypes(i)
        for(j <- 1 to (items.length - 1)) {
          val item = items(j)
          try {
            clazz = new Mirror().on(clazz).reflect.method("get" + upperFirst(item)).withoutArgs.getReturnType
          } catch {
            case e => {
              throw new IllegalArgumentException("Parameters paths are invalid: " + Arrays.toString(parameterPaths) + " for method " + method, e)
            }
          }
        }
        result.put(path, clazz)
      }
    }
  }
  result
}
{% endhighlight %}

<p>Esse código não tem <del>quase</del> nada de funcional ainda, a idéia é deixá-lo mais funcional com alguns recursos do Scala.</p>
<p>O segundo <code>for</code> está passeando pelos índices, já que precisamos usar o mesmo índice para pegar o nome e o tipo dos parâmetros, que estão em listas (arrays) diferentes. Em Scala, podemos trocar:</p>

{% highlight scala %}
for (i <- 0 to (parameterNamesFor.length - 1)) {
{% endhighlight %}

<p>por:</p>

{% highlight scala %}
for (i <- parameterNamesFor.indices) { // indices é o plural de index ;)
{% endhighlight %}

<p>No terceiro <code>for</code> precisamos ignorar o primeiro elemento do caminho: já sabemos o seu tipo e vamos navegar a partir dele. O código com o jeitão java abaixo tem uma tradução bem mais legal em Scala (e em várias outras linguagens).</p>

{% highlight scala %}
val items = path.split("\.")
for(j <- 1 to (items.length - 1)) {
   val item = items(j)
{% endhighlight %}

<p>O que queremos é percorrer a lista de items jogando fora o primeiro elemento:</p>

{% highlight scala %}
val items = path.split("\.")
for (item <- items.drop(1))
{% endhighlight %}

<p>ou ainda:</p>

{% highlight scala %}
for (item <- path.split("\.").drop(1))
{% endhighlight %}

<p>Só com os dois detalhes acima (além de renomear variáveis e remover o try..catch, que nunca é obrigatório em Scala) já temos um código bem mais limpo, mas ainda não bom (ou funcional) o suficiente:</p>

{% highlight scala %}
def getParameterTypes(method: Method, paths: Array[String]) = {
  val result = new HashMap[String, Class[_]]
  val names = provider.parameterNamesFor(method)
  for (path <- paths) {
    for (i <- names.indices) {
      val name = names(i)
      if (path.startsWith(name + ".") || (path == name)) {
        var clazz = method.getParameterTypes(i)
        for(item <- path.split("\.").drop(1)) {
          clazz = new Mirror().on(clazz).reflect.method("get" + upperFirst(item)).withoutArgs.getReturnType
        }
        result.put(path, clazz)
      }
    }
  }
  result
}
{% endhighlight %}

<p>O próximo passo é eliminar os dois primeiros <code>for</code>'s aninhados usando um recurso do Scala chamado <a href="http://www.scala-lang.org/node/111"><code>for comprehensions</code></a>. Os dois <code>for</code>'s são equivalentes a:</p>

{% highlight scala %}
for (path <- paths; i <- names.indices) {
{% endhighlight %}

<p>Isso é bem mais do que só economizar código, mas a explicação completa sobre o que é isso precisaria de um post inteiro. Para entender melhor o poder de <code>for comprehensions</code>, vamos fazer o inline da variável <code>name</code>:</p>

{% highlight scala %}
for (path <- paths; i <- names.indices) {
      if (path.startsWith(names(i) + ".") || (path == names(i))) {
          //resto do código
      }
}
{% endhighlight %}

<p>Tudo que temos dentro do <code>for</code> agora é o <code>if</code>. Nesse caso, podemos transferir a responsabilidade de filtrar os elementos para o próprio <code>for</code>:</p>

{% highlight scala %}
for (path <- paths; i <- names.indices; if path.startsWith(names(i) + ".") || (path == names(i))) {
   //resto do código
}
{% endhighlight %}

<p>De novo, isso é bem mais do que só economizar código. Transformamos um código procedural (<code>for -> for -> if</code>) em um código funcional. É como se tivéssemos descrito o domínio da nossa função (voltando ao colégio):<br />

```
D = { path ∈ paths, i ∈ indices tais que path começa com names(i)}<br />
```

E o que está dentro do <code>for</code> será executado para cada elemento do domínio <code>D</code> - é uma função do domínio <code>D</code> numa imagem <code>Im</code>. Louco, não?</p>
<p>Código até agora:</p>

{% highlight scala %}
def getParameterTypes(method: Method, paths: Array[String]) = {
  val result = new HashMap[String, Class[_]]
  val names = provider.parameterNamesFor(method)
  for (path <- paths; i <- names.indices; if path.startsWith(names(i) + ".") || (path == names(i))) {
    var clazz = method.getParameterTypes(i)
    for(item <- path.split("\.").drop(1)) {
      clazz = new Mirror().on(clazz).reflect.method("get" + upperFirst(item)).withoutArgs.getReturnType
    }
    result.put(path, clazz)
  }
  result
}
{% endhighlight %}

<p>================<br />
<b>BÔNUS:</b></p>
<p>Ainda dá pra deixar o código acima mais funcional (não necessariamente mais legível, mas isso é outra história).</p>
<p>Veja esse código:</p>

{% highlight scala %}
var clazz = method.getParameterTypes(i)
for(item <- path.split("\.").drop(1)) {
  clazz = new Mirror().on(clazz).reflect.method("get" + upperFirst(item)).withoutArgs.getReturnType
}
{% endhighlight %}

<p>O que estamos fazendo, no fim das contas, é transformar a lista de itens (<code>path.split("\\.").drop(1)</code>) em uma <code>clazz</code> a partir de um valor inicial (<code>method.getParameterTypes(i)</code>). Isso é uma operação bem comum em programação funcional chamada <a href="http://en.wikipedia.org/wiki/Fold_(higher-order_function)">fold</a>. Então podemos usar o <code>foldLeft</code> e evitar o uso da variável mutável (var) clazz:</p>

{% highlight scala %}
val items = path.split("\.").drop(1)
val clazz = items.foldLeft(method.getParameterTypes(i)) { (clazz, item) =>
   new Mirror().on(clazz).reflect.method("get" + upperFirst(item)).withoutArgs.getReturnType
}
{% endhighlight %}

<p>================<br />
<b>BÔNUS 2:</b><br />
Outra coisa não muito funcional que ainda estamos fazendo é a geração do mapa de resultado:</p>

{% highlight scala %}
val result = new HashMap[String, Class[_]]
for (path <- paths ....) {
   val clazz = .....
   result.put(path, clazz)
}
result
{% endhighlight %}

<p>Em programação funcional não deveríamos usar variáveis mutáveis (o valor do objeto <code>result</code> muda durante a execução do código). Em algumas linguagens como Erlang e Haskell isso é até uma regra da linguagem. A idéia é gerar esse mapa através apenas de aplicação e composição de funções.</p>
<p>Para fazer isso, precisaremos de outro recurso das <code>for comprehensions</code>: o <b>yield</b>. O que ele faz é pegar o retorno de cada iteração do <code>for</code> e gerar uma nova lista. Por exemplo:</p>

{% highlight scala %}
val novaLista = for (i <- List(1,2,3)) yield i + 2
println(novaLista) // ==> List(3, 4, 5)
{% endhighlight %}

<p>O que vamos fazer aqui é transformar cada item do nosso domínio <code>D</code> (o que está dentro do <code>for</code>) em uma tupla <code>(path, clazz)</code> que representa uma entrada (chave,valor) do nosso mapa:</p>

{% highlight scala %}
val entries =
   for (path <- paths; i <- names.indices; 
         if path.startsWith(names(i) + ".") || (path == names(i))) yield {
     val items = path.split("\.").drop(1)
     val clazz = items.foldLeft(...) { (clazz, item) =>
        new Mirror().on(clazz)....getReturnType
     }
     path -> clazz // jeito de criar a tupla (path, clazz) em scala
   }
{% endhighlight %}

<p>Agora a variável <code>entries</code> é a lista de tuplas produzida pela aplicação da função que está dentro do <code>for</code> nos elementos do domínio <code>D</code>, definido pela <code>for comprehension</code>.</p>
<p>E como eu transformo isso num <code>Map</code> agora? O legal é que o <code>Map</code> do Scala já tem um construtor que recebe uma lista de tuplas!</p>

{% highlight scala %}
val result = Map(entries:_*) // na verdade recebe um varargs, por isso o :_*, que explode a lista
{% endhighlight %}

<p>Código final:</p>

{% highlight scala %}
def getParameterTypes(method: Method, paths: Array[String]) = {
  val names = provider.parameterNamesFor(method)
  val entries =
     for (path <- paths; i <- names.indices; 
           if path.startsWith(names(i) + ".") || (path == names(i))) yield {
        val items = path.split("\.").drop(1)
        val clazz = items.foldLeft(method.getParameterTypes(i)) { (clazz, item) =>
          new Mirror().on(clazz).reflect.method("get" + upperFirst(item)).withoutArgs.getReturnType
        }
        path -> clazz
     }
  Map(entries:_*)
}
{% endhighlight %}

<hr/>
E aí, melhor ou pior que o código java correspondente?</p>
