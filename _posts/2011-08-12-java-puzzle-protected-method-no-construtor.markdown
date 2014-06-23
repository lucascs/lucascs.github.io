---
layout: post
status: publish
published: true
title: 'Java puzzle: protected method no construtor'
author:
  display_name: Lucas Cavalcanti
  login: lucascs
  email: lucas@cavalcanti.me
  url: ''
author_login: lucascs
author_email: lucas@cavalcanti.me
wordpress_id: 42
wordpress_url: http://lucas.cavalcanti.me/?p=42
date: '2011-08-12 18:45:37 -0300'
date_gmt: '2011-08-12 21:45:37 -0300'
categories:
- puzzle
tags: []
comments:
- id: 23
  author: Jonas Abreu
  author_email: jonas@vidageek.net
  author_url: http://vidageek.net
  date: '2011-08-12 18:58:47 -0300'
  date_gmt: '2011-08-12 21:58:47 -0300'
  content: null?
- id: 24
  author: Sergio Lopes
  author_email: slopes@gmail.com
  author_url: ''
  date: '2011-08-12 21:16:26 -0300'
  date_gmt: '2011-08-13 00:16:26 -0300'
  content: Chuto que chama o sobrescrito na filha... e o atributo nao tera sido setado
    ainda
- id: 25
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2011-08-13 01:48:02 -0300'
  date_gmt: '2011-08-13 04:48:02 -0300'
  content: "pois é, e por causa disso tive que fazer esse commit:\r\nhttps://github.com/caelum/vraptor/commit/45772be44c5a4660e074ee9b43cfc59e8aa6f9a0\r\n\r\nsenão
    as pessoas que sobrescrevessem a classe não iam poder usar dependências para implementar
    o sortSerializations().\r\n\r\nBizarro cair nesse caso, não?"
---
<p>Enquanto estava <a href="https://github.com/caelum/vraptor/commit/58535af3c904aa2819fcc827eb8bb4aed5cf0405">refatorando</a> um <a href="https://github.com/caelum/vraptor/pull/377">pull request do VRaptor</a> feito pelo <a href="https://github.com/acdesouza">acdesouza</a>, cai numa situação estranha do java.</p>
<p>A idéia era criar um ponto de extensão para a classe <a href="https://github.com/caelum/vraptor/blob/master/vraptor-core/src/main/java/br/com/caelum/vraptor/serialization/DefaultRepresentationResult.java">DefaultRepresentationResult</a>:</p>

{% highlight java %}
public DefaultRepresentationResult(...List<Serialization> serializations) {//construtor
    //...
    this.serializations = serializations;
    sortSerializations();
}
 /**
   * Override this method if you want another ordering strategy.
   *
   * @since 3.4.0
   */
protected void sortSerializations() {
       Collections.sort(this.serializations, new PackageComparator());
}
{% endhighlight %}

<p>Daí se a pessoa quiser mudar a ordenação é só sobrescrever o método sortSerializations(). Até aí tudo bem, mas quão perigoso é isso?</p>
<p>Daí vem o seguinte puzzle:</p>

{% highlight java %}
public class Mae {
    public Mae() {
         metodo();
    }
    protected void metodo() {
        System.out.println("Mãe");
    }
}
{% endhighlight %}
{% highlight java %}
public class Filha extends Mae {
   private final String x;
   public Filha() {
        this.x = "Testando";
   }
   @Override
   protected void metodo() {
       System.out.println(x);
   }
}
{% endhighlight %}
<p>Se eu executar:</p>
{% highlight java %}
new Filha();
{% endhighlight %}
<p>o que será impresso?</p>
