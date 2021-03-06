---
layout: post
status: publish
published: true
title: 'Anatomia de uma solução: VRaptor - linkTo para jsp - Final'
author:
  display_name: Lucas Cavalcanti
  login: lucascs
  email: lucas@cavalcanti.me
  url: ''
author_login: lucascs
author_email: lucas@cavalcanti.me
wordpress_id: 26
wordpress_url: http://lucas.cavalcanti.me/?p=26
permalink: 2011/07/06/anatomia-de-uma-solucao-vraptor-linkto-para-jsp-final/
date: '2011-07-06 10:00:42 -0300'
date_gmt: '2011-07-06 13:00:42 -0300'
categories:
- solução
- vraptor
tags: []
comments:
- id: 19
  author: Danilo Barboza
  author_email: danilo.barboza@gmail.com
  author_url: http://twitter.com/danilo_barboza
  date: '2011-07-11 20:41:47 -0300'
  date_gmt: '2011-07-11 23:41:47 -0300'
  content: "Grande, Lucas!\r\n\r\nCurti a solução!\r\n\r\nSó faltou concatenar o <b><i>contextPath</b></i>
    antes de retornar o <b><i>router.urlFor(...)</b></i> que fica melhor ainda! (Assim
    nao precisaríamos nem do c:url! ;D)\r\n\r\nEu andei pensando e chamar ${linkTo[ProdutoController].vendasDe(6,6,2011)}
    seria EXCELENTE! Mas sem criação de classes extras em tempo de compilação ou runtime
    realmente fica impossível né? Mesmo com a EL 2.2 nos permitindo invocar métodos...\r\n\r\nValeu
    e já estou utilizando em um projeto! (Com o contextPath embutido! ;)\r\n\r\n[
    ]'s"
- id: 20
  author: Davi
  author_email: davisnog@gmail.com
  author_url: ''
  date: '2011-07-14 01:41:50 -0300'
  date_gmt: '2011-07-14 04:41:50 -0300'
  content: "Muito bom, Lucas.\r\nEspero ter essa funcionalidade como default no VRaptor.
    ;-)"
- id: 21
  author: Rafael
  author_email: rafael@memes.com.br
  author_url: ''
  date: '2011-07-15 10:17:02 -0300'
  date_gmt: '2011-07-15 13:17:02 -0300'
  content: "Oi Lucas, parabéns pelo Post.\r\nQuando estará integrado ao vraptor?!\r\nColoquei
    uma dúvida no GUJ, se puder dar uma olhada, agradeço!"
---
<p>Nas partes <a href="http://lucas.cavalcanti.me/2011/06/17/anatomia-de-uma-solucao-vraptor-linkto-para-jsp-parte-1/">1</a> e <a href="http://lucas.cavalcanti.me/2011/06/19/anatomia-de-uma-solucao-vraptor-linkto-para-jsp/">2</a> conseguimos fazer com que isso funcionasse manualmente:</p>

{% highlight html %}
${linkTo[ProdutoController].adiciona} => /produtos
{% endhighlight %}

<p>Mas se a gente precisasse colocar todas as URIs do sistema no mapa <code>linkTo</code>, não ia adiantar muita coisa. O legal é conseguir gerar esse mapa com todas as lógicas que existem no sistema de uma vez só. Para isso precisamos do componente do VRaptor que é responsável pelas rotas (URIs) das lógicas: o <b>Router</b>.<br />
Com ele você consegue a URI de uma lógica a partir da classe e do método:</p>
{% highlight java %}
String uri = router.uriFor(Controller.class, metodo, argumentos);
{% endhighlight %}
<p>com esse método conseguiríamos gerar o nosso mapa <code>linkTo</code>:</p>
{% highlight java %}
Map<String, Map<String,String>> linkTo = Maps.newHashMap();
List<Class<?>> controllers = //todos os controllers do sistema
for (Class<?> controller : controllers) {
    //*
    List<Method> metodos = new Mirror().on(controller).reflectAll().methods();

    Map<String, String> mapaDoMetodo = Maps.newHashMap();
    for(Method metodo : metodos) {
         String nome = metodo.getName();

         //aridade é o número de parâmetros de um método. Precisamos passar um array 
         //com o tamanho da aridade do método para o uriFor funcionar
         String uri = router.uriFor(controller, metodo, new Object[aridade(metodo)]);

         mapaDoMetodo.put(nome, uri);
    }

    linkTo.put(controller.getSimpleName(), mapaDoMetodo);
} 
request.setAttribute("linkTo", linkTo);

//...
private int aridade(Method method) {
    return method.getParameterTypes().length;
}
{% endhighlight %}

<p><sub>* ajuda do <a href="http://projetos.vidageek.net/mirror/mirror/">Mirror</a>, uma biblioteca bem legal feita pelo Jonas Abreu para fazer reflection. Já vem como dependência do VRaptor.</sub></p>
<p>Temos dois problemas com esse código. O primeiro deles é: Como conseguir uma lista de todos os controllers do sistema? O VRaptor não tem um método específico pra isso na sua API mas, como sempre, dá pra improvisar se a gente conhecer um pouco da sua API interna.</p>
<p>Um dos jeitos é pegar todas as rotas do sistema (<code>router.allRoutes()</code>) e, a partir delas construir a lista de controllers. O problema é que as rotas (<a href="https://github.com/caelum/vraptor/blob/master/vraptor-core/src/main/java/br/com/caelum/vraptor/http/route/Route.java">interface Route</a>) não têm um método direto pra isso (o mais perto seria:<code> rota.resourceMethod(requestMockada, "qqer uri").getResource().getType()</code>, não mto legal, ou reflection acessando os fields da rota).</p>
<p>Outro jeito seria interceptar o registro dos controllers e guardá-los em uma lista. Com o VRaptor é possível interceptar qualquer uma das classes anotadas com uma das anotações do VRaptor (ou até com alguma anotação sua <a href="https://github.com/caelum/vraptor/blob/master/vraptor-core/src/main/java/br/com/caelum/vraptor/ioc/Stereotype.java">anotada com @Stereotype</a>). Esse interceptador é uma classe que implementa <a href="https://github.com/caelum/vraptor/blob/master/vraptor-core/src/main/java/br/com/caelum/vraptor/ioc/StereotypeHandler.java">StereotypeHandler</a>. Então podemos fazer:</p>
{% highlight java %}
@Component
@ApplicationScoped //nesse escopo pois isso vai rodar na inicialização do VRaptor
public class LinkToHandler implements StereotypeHandler { 

     public Class<? extends Annotation> stereotype() {
          return Resource.class; // intercepta todos os controllers
     }

     private List<Class<?>> controllers = Lists.newArrayList(); //guava ajudando aqui também =)
     public void handle(Class<?> controller) {
          controllers.add(controller);
     }
     public List<Class<?>> getControllers() {
         return this.controllers;
     }
}
{% endhighlight %}

<p>Cada vez que o VRaptor acha um controller, ele vai chamar esse método handle, e adicioná-lo na lista. Assim conseguimos rodar o código que popula o mapa linkTo que eu postei acima. O código funciona muito bem, se nenhuma das rotas tem parâmetro (ou seja, se tivermos <code>@Path("/produtos/{produto.id}"</code>) como vamos passar o id do produto?). O que nos leva ao segundo problema do código: como tratar URIs que têm parâmetros?</p>
<p>Se uma URI tem parâmetro, não podemos usar a nossa estratégia de gerar o mapa inteiro já com as URIs, pois ela vai ser diferente dependendo de qual parâmetro passamos para ela. Mas como passar parâmetros usando EL padrão?</p>
{% highlight html %}
${linkTo[ProdutosController].visualiza(produto)} => seria o ideal, mas não funciona =/

relaxando um pouco:

${linkTo[ProdutosController].visualiza[produto]} => isso sim, funciona! mas como?
{% endhighlight %}
<p>Para poder fazer isso, o "método" visualiza precisa ser um mapa, que contém uma chave que é o produto que a gente passou. Poxa, será que precisamos criar um mapa com todos os produtos possíveis? Seria só loucura. Precisamos então de uma <del datetime="2011-07-04T01:24:54+00:00">gambiarra</del> solução mais criativa.</p>
<p>Para que o <code>visualiza[produto]</code> funcione, precisamos que visualiza seja um mapa, mas ninguém falou que precisa ser um <code>HashMap</code> ou um <code>TreeMap</code>. A única coisa necessária é implementar <code>Map</code>! Então vamos criar um mapa hackeado que retorne a uri dependo do objeto passado. O único método que a gente precisa implementar de verdade é o <b>get</b>, que é o método chamado quando a gente faz o <code>vizualiza[produto]</code>.</p>
<p>O chato é que a interface Map tem vários métodos, e precisamos implementar todos. Mas o nosso amigo <a href="http://code.google.com/p/guava-libraries/">guava</a> tem uma classe chamada <a href="http://guava-libraries.googlecode.com/svn/trunk/javadoc/index.html?com/google/common/collect/ForwardingMap.html">ForwardingMap</a> que é bem útil nesse caso (existem várias classes Forwarding que podem ser bem úteis também):</p>
{% highlight java %}
class Linker extends ForwardingMap<Object, String> {
    
    public Linker(Class<?> controller, Method method) {...} //guarda em fields
    
    @Override
    protected Map<Object, Linker> delegate() {
        return Maps.newHashMap(); //precisamos delegar para um mapa qualquer
    }

    @Override
    public String get(Object key) {
        return router.urlFor(controller, method, new Object[] { key });
    }

}

//código da solução anterior
for(Class<?> controller : controllers) {
    Map<String, Linker> mapaDosMetodos = Maps.newHashMap();
    for(Method metodo : metodos) {
         mapaDosMetodos.put(metodo.getName(), new Linker(controller, metodo));
    }
}
{% endhighlight %}

<p>Agora se chamarmos no jsp:</p>
{% highlight html %}
${linkTo[ProdutoController].visualiza[produtoComId3]} ==> /produtos/3, legal =)
mas
${linkTo[ProdutoController].adiciona} ==> br.com.caelum...Linker@1def231 =S
{% endhighlight %}
<p>Droga, criamos outro problema. Tudo bem, só sobrescrever o <code>toString</code> do <code>Linker</code> para também retornar a uri. Antes de mostrar o código completo, como fazer para passar mais de um parâmetro?</p>
{% highlight java %}
public class ProdutoController {
   @Get("/produtos/vendas/{dia}/{mes}/{ano}")
   public void vendasDe(int dia, int mes, int ano) {...}
}
{% endhighlight %}
{% highlight html %}
${linkTo[ProdutoController].vendasDe(6,6,2011)} ==> err.. não

${linkTo[ProdutoController].vendasDe[6,6,2011]} ==> hum.. seria bom, mas também não

${linkTo[ProdutoController].vendasDe[6][6][2011]} ==> aí sim =)
{% endhighlight %}
<p>ou seja, cada vez que recebermos um parâmetro precisamos retornar um outro mapa. Tudo bem, só retornar sempre um linker, só que a cada vez com um parametro a mais preenchido:</p>
{% highlight java %}
class Linker extends ForwardingMap<Object, Linker> {

    public Linker(Class<?> type, Method method) { // para não mudar o código anterior
        this(type, method, new Object[aridade(method)], 0);
    }
    private Linker(Class<?> type, Method method, Object[] args, int index) {//guarda em fields}

    @Override
    protected Map<Object, Linker> delegate() {
        return Maps.newHashMap();
    }

    @Override
    public Linker get(Object key) {
        Object[] newArgs = args.clone(); //para evitar memorização de argumentos
        newArgs[index] = key;
        return new Linker(type, method, newArgs, index + 1); //aumentando o índice para preencher o próximo parâmetro
    }

    @Override
    public String toString() {
        return router.urlFor(type, method, args); //só mostra a uri no último momento, 
                                                               //qdo o mapa for mostrado na jsp
    }

}
{% endhighlight %}
<p>E pronto, dessa forma conseguimos suportar a geração dos links de qualquer lógica de um controller. Lembrando que se um parâmetro não é usado na URI, não precisa ser passado.</p>
<p>O legal dessa solução é que ela passa por várias coisas úteis (um pouco roubadas às vezes ;)), que são legais saber quando um problema cabeludo aparece na sua frente. Espero que tenham gostado =).</p>
<p>Código da solução final: <a href="https://gist.github.com/1064176">https://gist.github.com/1064176</a>. Em breve estará integrado ao código do VRaptor, com os devidos testes e documentação.</p>
