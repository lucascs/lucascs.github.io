---
layout: post
status: publish
published: true
title: Sobre acoplamento e injeção de dependências em Ruby
author:
  display_name: Lucas Cavalcanti
  login: lucascs
  email: lucas@cavalcanti.me
  url: ''
author_login: lucascs
author_email: lucas@cavalcanti.me
wordpress_id: 60
wordpress_url: http://lucas.cavalcanti.me/?p=60
date: '2012-03-16 11:30:14 -0300'
date_gmt: '2012-03-16 14:30:14 -0300'
categories:
- ruby
- oo
tags:
- ruby
- acoplamento
- injeção de dependências
comments:
- id: 246
  author: Alberto Souza
  author_email: alots.ssa@gmail.com
  author_url: http://alots.wordpress.com
  date: '2012-03-16 12:53:54 -0300'
  date_gmt: '2012-03-16 15:53:54 -0300'
  content: "Fiquei com um pouco de medo :). A solução é bem esperta, como é de praxe
    no seu caso. Para o caso do Mock eu acho que pode ser um pouco pior. Acho que
    se você estubar aquele método, nos seus testes vai parecer que aquela instancia
    tem sim ele... Mas na hora que você rodar vai dar caca. É o problema da alta flexibilidade
    de mexer nas classes :). \r\n\r\nPara mim, usar a constante ali me deixa preocupado
    com a legibilidade. O primeiro pensamento da pessoa, pelo menos na maioria dos
    casos, vai ser o de procurar uma classe com o nome EnviadorDeEmail e ela vai ter
    que descobrir que isso é só uma variavel...."
- id: 249
  author: Leonardo
  author_email: leonardo.marcelino@gmail.com
  author_url: ''
  date: '2012-03-16 13:10:24 -0300'
  date_gmt: '2012-03-16 16:10:24 -0300'
  content: "Olá Lucas, muito bom o post.\r\n\r\nUm approach  que acho bacana sobre
    injeção de dependências em Ruby, e fornecer um o serviço no próprio método e deixar
    como padrão algum serviço\r\n\r\nPor exemplo:\r\n\r\nclass MinhaClasse\r\n  #...\r\n
    \ def processo_complicado(enviador = EnviadorDeEmail.new)\r\n     email = gera_email
    \r\n     enviador.envia email\r\n  end\r\nend\r\n\r\nNeste caso eu poderia chamar
    o metodo processo_complicado sem passar nenhum argumento e ele usaria o serviço
    EnviadorDeEmail por padrão, ou então caso eu desejasse uma outra implementação
    eu poderia chamar o metodo processo_complicado passando esse novo serviço:\r\n\r\nprocesso_complicado
    EnviadorDeEmailNaoBlocante.new\r\n\r\nAbraços,\r\n\r\nLeonardo"
- id: 251
  author: Rafael Ponte
  author_email: rponte@gmail.com
  author_url: http://www.rponte.com.br/
  date: '2012-03-16 13:51:06 -0300'
  date_gmt: '2012-03-16 16:51:06 -0300'
  content: "Provavelmente não podemos considerar como \"perigo\", mas no final temos
    um objeto com uma dependência não explicita.\r\n\r\nOutro ponto, será que neste
    seu cenário a classe  EnviadorDeEmail atuaria como uma variável global, não? Se
    for o caso, variáveis globais costumam trazer mais problemas do que gostaríamos."
- id: 252
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2012-03-16 13:54:37 -0300'
  date_gmt: '2012-03-16 16:54:37 -0300'
  content: "Você pode pensar nessa variável como se fosse uma interface do java. Acontece
    a mesma coisa, vc vai procurar a classe, daí é uma interface e vc precisa procurar
    a implementação...\r\n\r\nse você fizer algo do tipo:\r\n<pre lang=\"ruby\">\r\n#
    deve ter o metodo:\r\n# envia(email)\r\nEnviadorDeEmail = ....\r\n</pre>\r\nvc
    criou uma \"interface\" em ruby."
- id: 253
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2012-03-16 14:08:18 -0300'
  date_gmt: '2012-03-16 17:08:18 -0300'
  content: |-
    Todas as classes são variáveis globais em ruby ;)

    Na verdade todas as dependências estão explicitas na implementação (vc quem declarou com os news quais são elas),
    e escondidas no uso da classe,  o que é o melhor dos casos, já que quem usa o EnviadorDeEmail não precisa saber quais
    são as dependências dele
- id: 254
  author: Pedro Matiello
  author_email: pmatiello@gmail.com
  author_url: ''
  date: '2012-03-16 14:22:06 -0300'
  date_gmt: '2012-03-16 17:22:06 -0300'
  content: "&gt; Todas as classes são variáveis globais em ruby ;)\r\nMas quão normal
    é fazer algo assim em Ruby: \"AlgoQuePareceNomeDeUmaClasse = NomeDeUmaClasseDeVerdade\"?
    Se isso não é muito comum, a classe é global mas não corre um grande risco de
    ser reatribuída.\r\n\r\nAinda: e se eu quiser usar duas implementações diferentes
    (i.e. tenho EnviadorDeEmailTextoPuro e EnviadorDeEmailHtml) e preciso escolher
    qual vou usar no momento do uso, e não na inicialização da aplicação?"
- id: 255
  author: Rafael Ponte
  author_email: rponte@gmail.com
  author_url: http://www.rponte.com.br/
  date: '2012-03-16 15:18:27 -0300'
  date_gmt: '2012-03-16 18:18:27 -0300'
  content: Bem, não podendo haver DI através de construtor ou setter eu normalmente
    não considero uma dependência explicita. Isso dificulta mockar (pra ruby nem tanto),
    mas ainda assim eu estava me referindo as DI na MinhaClasse.
- id: 256
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2012-03-16 15:28:59 -0300'
  date_gmt: '2012-03-16 18:28:59 -0300'
  content: |-
    Não sei o quão normal é... Acho que quase não é usado.

    se tem duas implementações a serem usadas simultaneamente não dá pra usar isso... mas em java com container de DI tb não funciona ;)
- id: 257
  author: Lucas Cavalcanti
  author_email: lucas@cavalcanti.me
  author_url: ''
  date: '2012-03-16 15:31:39 -0300'
  date_gmt: '2012-03-16 18:31:39 -0300'
  content: |-
    A idéia era que, se eu precisar do enviador de emails na Minha classe eu simplesmente faço um EnviadorDeEmail.new... é explicita ;)

    a configuração vai definir qual é a implementação. "Injeção de Dependência" via sobrescrita de "constante" de tipo ;)
- id: 260
  author: Pedro Matiello
  author_email: pmatiello@gmail.com
  author_url: http://pmatiello.blogspot.com/
  date: '2012-03-16 16:57:29 -0300'
  date_gmt: '2012-03-16 19:57:29 -0300'
  content: "Mas ainda existe a possibilidade de construir o objeto manualmente se
    a injeção é por construtor.\r\n\r\nAinda, no exemplo do post, eu diria que a dependência
    não é realmente injetada, mas buscada pela classe cliente. Comparado com o código
    similar em Java, há a vantagem de ser fácil configurar o resultado da busca para
    ambientes diferentes com as atribuições no ponto de entrada da aplicação (em Java
    a gente usaria uma factory e um monte de código entediante ou algo do tipo). E
    também é muito mais fácil testar em Ruby porque o RSpec tem grandes poderes."
- id: 265
  author: Rafael de F. Ferreira
  author_email: public@rafaelferreira.net
  author_url: http://www.rafaelferreira.net
  date: '2012-03-16 19:05:42 -0300'
  date_gmt: '2012-03-16 22:05:42 -0300'
  content: "Esse é o grande problema dos containers de DI. Reduzem a flexibilidade
    do modelo OO à de módulos estáticos tipo C ou ML.\r\n\r\nSe a escolha da dependência
    é global, não há grande vantagem em relação à dependência direta da implementação."
---
<p>Quem me conhece sabe que eu gosto bastante de linguagens com tipagem estática, mas vou parar um pouco para falar sobre acoplamento em linguagens com tipagem dinâmica e como isso é diferente do que eu estou acostumado a fazer normalmente.</p>
<p>Vou começar por uma execução de um método qualquer em ruby:</p>

{% highlight ruby %}
class MinhaClasse
  def faz_alguma_coisa(outro_cara)
    outro_cara.algum_metodo
  end
end
{% endhighlight %}

<p>Qual é o nível de acoplamento da <code>MinhaClasse</code> com o <code>outro_cara</code>? O mais baixo possível! Só existe o acoplamento com a interface, no sentido mais puro da palavra, ou seja, só precisamos que o <code>outro_cara</code> tenha um método chamado <code>algum_metodo</code>, que não recebe nenhum argumento. Ou seja, o baixo acoplamento vem de graça! Não existe o acoplamento de tipos, que existe nas linguagens estáticas.</p>
<p>Se existirem duas classes que tem esse método, qualquer uma delas pode ser usada:</p>

{% highlight ruby %}
class CaraLegal
  def algum_metodo
    puts "yey!"
  end

  def outro_metodo
    puts "=)"
  end
end

class CaraChato
  def algum_metodo
    puts "Zzzzz"
  end
end
{% endhighlight %}

<p>Agora se mudar um pouco a <code>MinhaClasse</code>, por exemplo:</p>

{% highlight ruby %}
class MinhaClasse
  def faz_alguma_coisa(outro_cara)
    outro_cara.algum_metodo

    outro_cara.outro_metodo
  end
end
{% endhighlight %}

<p>Quebra todos os lugares que usam o <code>CaraChato</code> ou qualquer outra classe do sistema como argumento nesse método (inclusive os mocks que não esperavam a mensagem <code>:&zwj;outro_method</code> ¬¬). Quanto mais métodos eu chamar no <code>outro_cara</code>, mais eu estou me acoplando à interface dele, e maior é a chance de quebrar algum código que usa a <code>MinhaClasse</code>. Ou seja, ao mesmo tempo em que eu estou me acoplando pouco a um objeto, estou me acoplando a todas as classes que possuem os métodos que chamei em <code>faz_alguma_coisa</code>.</p>
<p>Agora, suponha que a <code>MinhaClasse</code> precisa enviar um email, e eu resolvi implementar da seguinte maneira:</p>

{% highlight ruby %}
class MinhaClasse
  #...
  def processo_complicado
     email = gera_email 
     enviador = EnviadorDeEmail.new
     enviador.envia email
  end
end
{% endhighlight %}

<p>Agora além de estar se acoplando a uma interface, estaria também se acoplando a uma implementação! A vida me ensinou que nesse caso, para remover esse acoplamento posso usar injeção de dependências e receber esse enviador no construtor da <code>MinhaClasse</code>, por exemplo. Mas peraí, isso é Ruby, será que esse código está acoplado à implementação do <code>EnviadorDeEmail</code> mesmo? Se eu quiser trocar a implementação padrão por uma que não fica esperando o email ser enviado, por exemplo, vou ter que varrer o sistema inteiro pra fazer isso? Não. Só preciso fazer isso em algum lugar:</p>

{% highlight ruby %}
EnviadorDeEmail = EnviadorDeEmailNaoBlocante
{% endhighlight %}

<p>Se eu tenho várias classes que oferecem serviços e são usadas como dependências no meu sistema, basta eu escolher um nome para o serviço, e dar new na hora que eu precisar usar (ou coisa parecida). E no meu ponto de entrada do sistema, simplesmente faço o 'bind' desses serviços para as implementações:</p>

{% highlight ruby %}
EnviadorDeEmail = EnviadorDeEmailNaoBlocante
Cache = MemCache
GeradorDeNotaFiscal = NotaFiscal::GeradorViaSoap
#...
{% endhighlight %}

<p>Sem contar que conseguimos implementar o new pra retornar a mesma instância, já que ele é um simples método da classe. Os motivos que tínhamos para não dar new nas classes não valem aqui (pelo menos não a maioria deles). O problema é: qual é o perigo disso?</p>
