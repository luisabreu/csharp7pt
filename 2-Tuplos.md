# Tuplos

A devolução de vários elementos a partir de um método não é suportada em C#. Para isso, ou recorríamos a parâmetros de saída ou eramos obrigados a criar novos tipos para esse fim. Como veremos em seguida, o C# 7.0 simplifica o trabalho necessário a estes cenários através do uso de tuplos. 


## Introdução

A natureza fortemente tipificada da linguagem C# introduz várias vantagens na escrita de código. Contudo, existem algumas situações onde as restrições associadas ao uso de tipificação acabam por nos obrigar a algum trabalho adicional. Por exemplo, se necessitarmos de devolver mais do que um valor a partir de um método, então somos obrigados a criar uma nova classe ou estrutura (`struct`) para armazenar esses valores. Alternativamente, podemos ainda recorrer aos chamados parâmetros de saída (parâmetros anotados com o modificador `out`) para devolver valores a partir de um método.

Na maior parte das vezes, os programadores optavam pela introdução de um novo tipo em detrimento do uso de parâmetros de saída. Esta opção justifica-se, essencialmente, por duas razões. Em primeiro lugar, porque em muitos dos métodos onde temos esta necessidade, os valores devolvidos acabam por formar uma entidade coerente (nestes casos, a introdução de um novo tipo para agrupar os vários valores devolvidos acaba por ser adequada).

> **O padrão *TryParse***
>A plataforma .NET introduz, desde a versão 2.0, um conjunto de métodos que permitem converter um valor de um tipo noutro. Estes métodos são normalmente designados por `TryParse` e caraterizam-se pelo facto de devolverem dois valores. O primeiro é representado por um `bool` e é devolvido através do tipo de retorno do método. Este valor é utilizado para indicar o resultado final da conversão (`true` indica uma conversão efetuada com sucesso). Por sua vez, o segundo é devolvido através de um parâmetro de saída e permite-nos recuperar o valor obtido a partir da conversão (quando esta é efetuada com sucesso). Neste caso, a criação de um novo tipo para representar o valor devolvido por este método faz menos sentido do que o uso de um parâmetro de saída, já que a combinação dos valores não resulta na criação de uma entidade coerente. 

Para além da questão levantada no parágrafo anterior, o uso de parâmetros de saída não é possível em alguns casos. Por exemplo, os métodos assíncronos (métodos que recorrem aos termos `async`/`await` para simplificar a escrita de código que utilizam tarefas assíncronas) não podem definir parâmetros deste tipo. No exemplo seguinte, estamos perante um método que necessita devolver dois valores. Neste caso, optámos por recorrer a um novo tipo para agrupar ambos os valores devolvidos pelo método:

```cs
struct Info
{
  public int Conta;
  public double Soma;
}
static Info EfetuaCalculos(IEnumerable<double> valores)
{
  var i = new Info();
  foreach (var v in valores)
  {
    i.Conta++;
    i.Soma += v;
  }
  return i;
}
void Main()
{
  var valores = new[] { 10D, 3D, 5D };
  var info = EfetuaCalculos(valores);
  Console.WriteLine($"Total elems: {info.Conta}; Soma:{info.Soma}");
}
```

Como é possível observar, o método `EfetuaCalculos` é responsável por contar o número de elementos e por somar os valores mantidos numa coleção (coleção esta que lhe é passada através de um parâmetro). Como mencionámos, a necessidade de recuperação de dois valores conduziu-nos à introdução de um novo tipo (designado, neste exemplo, por `Info`) que disponibiliza duas propriedades que nos permitem retornar os valores a partir do método.

## Sintaxe

Com o C# 7, este código pode ser simplificado através do uso de tuplos (designados por *tuples* na literatura inglesa). O código apresentado no excerto seguinte recorre a tuplos para simplificar o código apresentado no exemplo anterior:

```cs
(int contador, double soma) EfetuaCalculos(IEnumerable<double> valores)
{
  var conta = 0;
  var soma = 0d;
  foreach (var v in valores)
  {
    conta++;
    soma += v;
  }
  return (conta,soma);
}
var info = EfetuaCalculos(new[] { 10D, 3D, 5D });
Console.WriteLine($"Total elems: {info.contador}; Soma:{info.soma}");
``` 

O uso de um tuplo (definido através da sintaxe literal) no excerto anterior permitiu-nos devolver mais do que valor a partir do método sem que isso implicasse a criação de um novo tipo. Neste caso, o tuplo agrupa apenas dois valores, que podem ser acedidos através dos nomes contador e soma. Note-se como, no final do código apresentado no excerto anterior (última linha), recuperamos estes valores através de uma sintaxe que se assemelha muito à sintaxe utilizada para recuperarmos os valores de um campo ou propriedade disponibilizada por um determinado tipo.

Apesar de já termos apresentado um exemplo que ilustra o uso de um tuplo, a verdade é que ainda não apresentamos uma definição oficial para este tipo de elemento. De uma forma resumida, podemos ver um tuplo como uma lista ordenada, imutável, heterógena, mas fortemente tipificada de valores, valores esses podem ser recuperados através de nomes personalizados definidos pelo programador.

> **Utilização de tupos em .NET**
>A utilização deste tipo de elementos pressupõe o uso da Framework .NET versão 4.7 ou a instalação do pacote *Nuget* `System.ValueTuple`. Para instalar o pacote *Nuget* `System.ValueTuple`, podemos executar o comando seguinte a partir de uma consola *Nuget* no *Visual Studio*:
> `install-package System.ValueTuple`

Como é possível verificar através do exemplo anterior, a simplicidade associada à criação deste tipo de elementos é um dos pontos fortes associados ao seu uso. Para além disso, a possibilidade de definição de nomes personalizados para identificar os valores agrupados num tuplo (no exemplo anterior, os valores do tuplo são identificados pelos nomes contador e soma) é outro dos aspetos positivos associados ao uso deste tipo de elementos.


## Comparação com o tipo *Tupple*

Nesta altura, o leitor com experiência em C# pode estar a interrogar-se acerca das vantagens inerentes ao uso deste novo tipo. Afinal de contas, a plataforma .NET já disponibilizava um tipo (representado pelas classes `Tupple<T, ...>`) que permitia o uso deste tipo de estruturas em C#. Infelizmente, o uso de tuplos criados a partir deste tipo sofre de alguns problemas.
Em primeiro lugar, os tipos `Tupple<T,...>` são classes (portanto, estamos perante um tipo por referência, também conhecidos por *reference types*). Na prática, isto significa que o seu uso está sujeito a uma pequena penalização associada à alocação de espaço na *heap* (e correspondente limpeza efetuada pelo *Garbadge Collector*). Em aplicações onde a performance é essencial, o uso massivo deste tipo de elementos pode acabar por levar a uma degradação na performance do sistema. Este problema é resolvido através da introdução dos novos tipos por valor (*value type*) `System.ValueTuple<...>` para representar os tuplos.

Outra das diferenças entre os novos tipos `System.ValueTuple<...>` e os tipos `System.Tupple<T,...>` reside no facto de o novo tipo utilizar campos mutáveis. Esta decisão deve-se ao cenário típico que motivou a introdução dos novos tuplos: substituição de variáveis que são passadas através de parâmetros por referência ou de saída a métodos que necessitam de devolver vários valores. Nestes casos, a possibilidade de modificação dos valores dos campos dos tuplos permite-nos evitar o uso de variáveis temporárias para efetuar a manipulação intermédia desses valores.


## Representação literal

O exemplo apresentado no início deste capítulo introduziu a sintaxe literal utilizada na criação de tuplos. Nesta secção, vamos analisar detalhadamente as regras que regem a representação literal deste tipo de elementos. O exemplo seguinte apresenta aquela que é, porventura, a representação literal mais simples de um tuplo:

```cs
var dados = ("Luis", "Funchal");
```

A variável `dados` apresentada no exemplo anterior guarda um tuplo composto por dois valores, do tipo `string`. Neste caso, e como recorremos a uma variável implicitamente tipificada (note-se o uso do termo `var`), o compilador é responsável por inferir o tipo de cada um dos valores que compõem o tuplo (como referimos, ambos os valores são do tipo `string`). 

Uma vez que não definimos explicitamente os nomes utilizados na identificação desses valores, então a sua recuperação só pode ser feita através do acesso direto aos campos definidos pelo tipo que serve de suporte ao tuplo (como veremos um pouco mais à frente, os tuplos são representados programaticamente em C# por elementos dos tipos `System.TupleValue<...>`). Na prática, isto significa que a recuperação dos valores do tuplo dados é feita através do uso dos nomes `dados.Item1` e `dados.Item2`, conforme ilustrado em seguida:

```cs
Console.WriteLine(dados.Item1); //imprime Luis
Console.WriteLine(dados.Item2); //imprime Funchal
```

Como referimos, e se assim o desejarmos, podemos indicar explicitamente o nome que deve ser utilizado para aceder aos valores guardados em cada um dos campos que compõem o tuplo através de uma sintaxe semelhante à seguinte:

```cs
var dados = (nome: "Luis", morada: "Funchal");
```

Neste caso, os valores armazenados pelo tuplo foram identificadas explicitamente, mas o tipo de cada uma desses valores continua a ser inferido pelo compilador. Note-se que, quando optamos por identificar explicitamente os valores armazenados pelo tuplo através de nomes personalizados, então somos forçados a atribuir nomes a todos os valores mantidos no tuplo. Por outras palavras, o código seguinte resulta num erro de compilação:

```cs
var dados = (nome: "Luis", "Funchal");
```

Como seria de esperar, um mesmo nome não pode ser utilizado para identificar valores distintos agrupados num mesmo tuplo.

Finalmente, e se assim o desejarmos, podemos identificar explicitamente os nomes e os tipos de cada um dos valores guardados num tuplo. O excerto seguinte ilustra esta estratégia:

```cs
(string nome, string morada) dados = ("Luis", "Funchal");
```

Neste caso, a expressão `(string nome, string morada)` identifica um tuplo que agrupa dois valores, do tipo `string`. Estes valores podem ser recuperados através dos nomes `nome` e `morada`.

A linguagem permite ainda a indicação dos nomes utilizados para identificar os valores guardados pelos tuplos em ambos os lados de uma expressão de inicialização de uma variável. Qual será, então, o comportamento do compilador perante uma expressão semelhante seguinte:

```cs
(string nome, string morada) dados =  (n: "Luis", m: "Funchal");
```

Neste caso, o compilador acaba por gerar um aviso (*warning*) que alerta para o facto de os nomes `n` e `m` serem ignorados devido ao tuplo destino especificar nomes diferentes para identificar os valores armazenados nos campos (portanto, a atribuição anterior só pode ter acontecido devido a um erro do programador). Este aviso é gerado apenas quando estamos perante atribuições diretas. Portanto, se modificarmos ligeiramente o código anterior para algo semelhante ao seguinte, então deixamos de obter um aviso durante a compilação:

```cs
var tuplo1 = (n: "Luis", m: "Funchal");
(string nome, string morada) dados =  tuplo1;
```

Neste caso, e como veremos mais à frente, estamos perante uma inicialização de um tuplo a partir de um outro tuplo compatível. Ambos agrupam o mesmo número de valores e cada um dos valores do primeiro tuplo possui exatamente o mesmo tipo do valor que ocupa a mesma posição no segundo (uma "conversão" semelhante à anterior, onde os valores são do mesmo tipo, mas são identificados através de nomes personalizados diferentes é designada por conversão por identidade - *identity conversion*).


## Desconstrução de tuplos

A linguagem também permite efetuar a desconstrução de um tuplo através do uso das chamadas expressões de desconstrução. As expressões deste tipo permitem-nos inicializar um conjunto de variáveis a partir dos valores agrupados por um tuplo. Analisemos o exemplo seguinte:

```cs
var tuplo = (Nome: "Paulo", Morada: "Lisboa" );
(string n, string m) = tuplo;
Console.WriteLine(n); //Paulo
```

No exemplo anterior, a declaração `(string n, string m)` introduz duas variáveis, designadas por `n` e `m`, que são inicializadas a partir dos valores identificados pelos nomes `Nome` e `Morada` do tuplo guardado na variável `tuplo`.

Se quisermos, podemos deixar a aferição dos tipos das variáveis para o compilador. No excerto seguinte, apresentamos duas instruções que produzem o mesmo resultado e que ilustram esta estratégia:

```cs
(var n, var m) = tuplo;
var(n, m) = tuplo;
```

Conforme é possível verificar através do exemplo seguinte, também podemos recorrer à desconstrução de tuplos para atribuirmos os valores dos campos de um tuplo a variáveis que foram declaradas previamente:

```cs
string n;
string m;
var tuplo = (Nome: "Paulo", Morada: "Lisboa" );
(n, m) = tuplo;
```

Neste caso, a expressão de desconstrução foi utilizada para atribuirmos os valores agrupados por um tuplo a duas variáveis que foram declaradas previamente.


### Mais sobre a desconstrução de tuplos

A desconstrução é uma operação que não está limitada aos tuplos. Na realidade, esta operação pode ser aplicada a qualquer objeto de um tipo que define um método público de desconstrução. Este método deve ser designado por `Deconstruct` e pode ser definido como método de instância ou como método de extensão de um tipo (no caso dos tipos `System.ValueTuple<...>`, os métodos de desconstrução são definidos através de métodos de extensão da classe `System.TupleExtensions`).

Atualmente, os métodos de desconstrução devem possuir uma assinatura semelhante à apresentada no excerto seguinte:

```cs
public void Deconstruct(out T1 xi, ..., out Tn xn)
{
}
```

A lista de parâmetros de saída define os valores que podem ser recuperados através de uma expressão de desconstrução a partir de um objeto deste tipo. Para ilustrarmos uma implementação típica deste tipo de métodos, vamos supor que temos uma classe designada por Ponto, que armazena as coordenadas de um determinado ponto:

```cs
public class Ponto
{
  public int X { get; }
  public int Y { get; }
  public Ponto(int x, int y)
  {
     X = x;
     Y = y;
  }
  public void Deconstruct(out int x, out int y)
  {
     x = X;
     y = Y;
  }
}
static void Main()
{
 var p = new Ponto(10, 20);(var x, var y) = p;
 Console.WriteLine($"{x}, {y}"); //10,20
}
```

No exemplo anterior, as variáveis locais `x` e `y` são inicializadas através do uso de uma expressão de desconstrução. Quando essa expressão é executada, o método `Deconstruct` é invocado e as variáveis acabam sendo inicializadas no interior desse método (note-se o uso de parâmetros de saída). 


## Implementação dos tuplos

Como mencionámos, os tuplos são implementados através do uso dos tipos `System.ValueTuple<T,...>`. Por outras palavras, todos os exemplos apresentados anteriormente acabam por resultar na instanciação de um dos vários tipos por valor (*value type*) genéricos `System.ValueTuple<T,...>` introduzidos pela plataforma .NET (apesar de apenas termos utilizado tuplos que agrupam 2 valores e de o tipo com maior número de valores apenas permitir 8 valores, a verdade é que o compilador permite lidar com tuplos de qualquer número de elementos).

Cada um dos vários tipos `System.ValueTuple<T,...>` introduz tantos campos quanto os parâmetros de tipo utilizados. Esses campos são designados respetivamente por `Item1`, `Item2`, .... Os nomes personalizados utilizados na criação de tuplos literais são mapeados pelo compilador nos campos expostos pelos tipos `System.ValueTuple<...>` que é usado para representar esse tuplo. Por outras palavras, a atribuição de nomes aos valores armazenados por um tuplo pode ser vista como açucar sintático, que é suportado apenas pelo compilador.

Portanto, se recuperarmos o último exemplo apresentado:

```cs
(string nome, string morada) dados =  (n: "Luis", m: "Funchal");
```

É possível afirmarmos que esta instrução será transformada pelo compilador em código semelhante ao seguinte:

```cs
var dados = new System.ValueTuple<string, string>("Luis", "Funchal");
```

Isto significa que os valores do tuplo identificados pelos nomes `nome` e `morada` são, na verdade, guardados em campos designados por `Item1` e `Item2`. Ou seja, uma instrução semelhante à seguinte:

```cs
Console.WriteLine(dados.nome);
Será transformada pelo compilador em algo semelhante ao seguinte:
Console.WriteLine(dados.Item1);
```

Nesta altura, importa referir que, se assim o desejarmos, podemos utilizar diretamente os identificados `Item1`, `Item2`, etc. no código C# que escrevemos para recuperamos os valores guardados por um tuplo. Esta natureza "dupla" dos tuplos acaba por introduzir algumas restrições no que diz respeito aos nomes que podem ser utilizados na identificação dos valores armazenados por um tuplo criado através da sintaxe literal. Assim, e para além dos nomes `Item1`, `Item2`, etc., também não podemos utilizar nomes que identifiquem outros membros disponibilizados pelos tipos `System.ValueTuple<...>`.

Analisemos os exemplos apresentados no excerto seguinte:

```cs
var opcao1 = (ToString: "Teste", Morada: "Funchal"); // erro: uso de membro 
exposto pelo System.ValueTuple
var opcao2 = (Item2: "L", Item1: "LL"); //erro: posicao incorreta
var opcao3 = (Item1: "ok", Item2: 10); //ok
```

A primeira instrução apresentada resulta num erro de compilação, já que `ToString` identifica o nome de um membro exposto pelos tipos `System.ValueTuple<...>`. A segunda instrução também produz um erro de compilação porque os nomes `Item1` e `Item2` só podem ser utilizados para identificar valores guardados, respetivamente, no primeiro e segundo campos de um tuplo. Finalmente, o terceiro exemplo apresentado não produz quaisquer erros de compilação e mostra como podemos utilizar corretamente os nomes dos campos expostos pelos tipos `System.ValueTuple<...>`.


## Conversões entre tuplos

Como seria de esperar, a linguagem suporta a conversão entre tuplos. O primeiro aspeto a reter reside no facto de os nomes utilizados na criação de um tuplo literal não influenciarem este tipo de operação. Assim, um tuplo criado através de uma sintaxe literal pode ser convertido em qualquer outro tuplo, desde que os tipos dos seus campos sejam conversíveis nos tipos dos campos do tuplo destino. 

Recordemos, então, um exemplo apresentado previamente na secção onde apresentámos a sintaxe utilizada na criação de tuplos literais:

```cs
var tuplo1 = (n: "Luis", m: "Funchal");
(string nome, string morada) dados =  tuplo1;
```

Neste caso, ambos os tuplos (`tuplo1` e `dados`) são representados por instâncias do tipo `System.ValueTuple<string, string>`. Apesar de os valores guardados em cada tuplo serem identificados por nomes diferentes, a verdade é que não existe qualquer impedimento na atribuição de `tuplo1` a `dados`. Estas "conversões" (se é que podemos chamá-las assim) são designadas por *identity conversions*.

Em alguns casos, podemos ter conversões entre tuplos mesmo quando os tipos dos campos não são exatamente os mesmos. Por exemplo, o excerto seguinte apresenta um exemplo válido de uma inicialização de uma variável a partir da conversão de um tuplo literal:

```cs
(string nome, byte idade) t = (null, 5);
```

Neste caso, e uma vez que existem conversões implícitas entre os tipos dos valores dos campos do tuplo `(null, 5)` e os tipos dos valores do tuplo da variável `t`, então o compilador não produz qualquer erro durante a compilação. `null` é um valor especial, que pode ser atribuído a qualquer tipo de objeto "anulável", logo pode ser atribuído a um campo do tipo `string`. Por sua vez, o valor inteiro 5 também pode ser convertido implicitamente no tipo `byte`. Logo, a conversão entre os tuplos anteriores é possível. Este tipo de conversão costuma ser designado por conversão implícita entre tuplos (*implicit typle conversion*).

Note-se que este tipo de conversão só pode ser aplicado quando os tipos são indicados explicitamente ou quando o compilador consegue obter um tipo válido a partir das expressões atribuídas a cada campo do tuplo. Analisemos o exemplo seguinte:

```cs
var t1 = ("Luis", 5);
var t2 = (null, 5);
```
Neste caso, a primeira instrução é executada sem quaisquer problemas. Isto acontece porque as expressões utilizadas permitem inferir o tipo de cada um dos campos do tuplo. O mesmo já não acontece com a segunda instrução, uma vez que o compilador não é capaz de inferir o tipo do primeiro campo do tuplo a partir da expressão `null`.

Como os tuplos são considerados tipos por valor (*value types*), então também estão sujeitos a eventuais operações de *boxing*. Como os nomes dos valores agrupados por um tuplo não fazem parte da sua representação em runtime, então também não fazem parte do objeto obtido a partir da operação *boxing*. Nestes casos, a recuperação do tuplo pode ser feita, por exemplo, através de uma operação de "conversão" de identidade (*identity conversion*), que foi apresentada num dos parágrafos anteriores. O exemplo seguinte ilustra este tipo de operações:

```cs
var info = (nome: "Luis", morada: "Funchal");
object box = info; //boxing
(string n, string m) info3 = info; //unboxing
```

Neste caso, e apesar do tuplo inicial permitir o acesso aos seus valores através dos nomes `nome` e `morada`, a operação de *unboxing* utilizada acaba por nos devolver um tuplo cujos valores podem ser acedidos através dos nomes `n` e `m`.

Para além de estarem sujeitos a operações de boxing, os tuplos também podem ser *nullables* (*nullable tuples*). Como seria de esperar, a conversão entre um tuplo e um *nullable* é possível (*implicit nullable conversion*), conforme ilustrado através do excerto seguinte:

```cs 
(int x, int y)? t = (1, 2);
```

Apesar de os exemplos apresentados até ao momento não o mostrarem, a verdade é que as conversões entre tuplos são importantes, especialmente quando estamos perante *overloads* de métodos que esperam parâmetros destes tipos. Suponhamos, então, que temos os seguintes overloads de um método designado por `M1`:

```cs
void M1((int x, int y) arg){...};
void M1((object x, object y) arg){...};
M1((1,2)); // prim método
M1(("luis", "funchal")); //segundo
```

Como seria de esperar, a primeira invocação resulta na execução do método que espera um tuplo do tipo `(int, int)`. No segundo caso, não existe nenhum overload do método que disponibilize um tuplo que agrupa valores do mesmo tipo dos do tuplo passado ao método. Contudo, e uma vez que existe uma conversão entre o tipo string e o tipo object, então o segundo método acaba por ser o escolhido.


## Comparação entre tuplos

A comparação entre dois tuplos é outra das operações que é suportada pela linguagem/plataforma .NET. Uma vez que estamos a falar de tipos por valor (*value types*), então sabemos que a comparação pode ser efetuada através do uso do método Equals. O exemplo seguinte ilustra o uso deste método:

```cs
var o1 = (m: 10, n: 20);
var o2 = (10, 20);
Console.WriteLine(o1.Equals(o2)); //true
```

Com base no que foi dito anteriormente, rapidamente chegamos à conclusão de que a expressão anterior produz o resultado `true`. Cada tuplo agrupa dois elementos, do mesmo tipo, que possuem exatamente os mesmos valores. Note-se que, ao contrário do que acontece com o nome, o tipo do campo é utilizado na comparação dos tuplos. Por exemplo, o resultado da comparação seguinte é diferente do obtido no exemplo anterior:

```cs
var o1 = (m: 10, n: 20);
var o2 = (10, 20.0);
Console.WriteLine(o1.Equals(o2)); //false
```

Neste caso, o segundo campo do segundo tuplo possui um tipo diferente do do primeiro, pelo que o resultado da comparação obtida à custa da execução do método `Equals` produz o resultado false.

A comparação de tuplos não pode ser feita através do uso do operador `==` (à semelhança, aliás, do que acontece por predefinição com os elementos do tipo *value types*). Apesar do tipo `System.ValueTuple<T,...>` efetuar o *override* do método `Equals`, ele acabou por não introduzir uma implementação personalizada para o operador `==`. Este "esquecimento" deve-se, em primeiro lugar, à natureza genérica dos tipos `System.ValueTuple<...>`. Para além disso, e como veremos, uma eventual implementação baseada na delegação ao método `Equals` pode conduzir, em alguns cenários, à obtenção de valores contraditórios.

Esta não implementação dos operadores `==` e `!=` por parte dos tipos `System.ValueTuple<...>` pode parecer estranha à primeira vista. Contudo, existem boas justificações para esta "omissão". A recomendação para implementação personalizada do operador `==` num tipo passa pela delegação no operador `==` de cada um dos campos que o constituem. Portanto, a implementação recomendada para um tuplo constituído por dois campos passaria pela utilização de uma expressão do tipo `x1 == x1 && y1 == y1`. Infelizmente, esta operação não pode ser implementada pelos tipos que servem de base aos tuplos (`System.ValueTuple<...>`), já que estamos perante tipos genéricos, que não aplicam quaisquer restrições aos tipos dos parâmetros genéricos utilizados. Esta falta de restrições impede-nos de implementar o operador `==` à custa da delegação, já que podemos mesmo estar perante um tipo que não disponibilize esse operador.

Nesta altura, poderíamos pensar que uma delegação alternativa no método `Equals` poderia constituir uma alternativa viável para a implementação do operador `==`. Uma vez que este método é definido pela classe `System.Object` (`object`), então pode ser invocado sobre qualquer elemento válido. Infelizmente, uma implementação baseada no uso deste método produziria resultados inesperados em algumas situações. Analisemos, por exemplo, as comparações entre valores do tipo `double`. Suponhamos que queremos comparar dois valores `double.NaN`. Como é possível verificar através do excerto seguinte, a comparação baseada no uso deste método produz um resultado diferente do produzido pelo operador `==`:

```cs
var iguais = double.NaN.Equals(double.NaN); // true
var diferentes = double.NaN == double.NaN; // false
``` 

Tendo em atenção este comportamento, rapidamente concluímos que a reutilização do método Equals não é uma boa solução para o problema em mãos. Se isso fosse feito, e supondo que estaríamos perante um tuplo onde pelo menos um dos valores agrupados é do tipo `double` e possui o valor `double.NaN`, então os resultados obtidos seriam difíceis de explicar. O excerto seguinte tenta ilustrar este cenário:

```cs
//comparacao direta double.NaN
var dif = double.NaN == double.NaN; // false
// agora com tuplos
var t1 = (double.NaN, double.NaN);
var t2 = (double.NaN, double.NaN);
var ig = t1.Equals(t2); //true
var dif2 = t1 == t2; // true (supondo delegação na implementação Equals)
```

Como é possível verificar, a comparação direta dos tuplos produziria um resultado diferente da comparação direta dos valores agrupados em cada um deles (isto, claro, se a implementação do operador `==` fosse efetuada à custa do método `Equals` definido pelo tipo de cada campo do tuplo). Nesta altura, parece-nos que a não implementação explícita do operador `==` (e do operador `!=`) acabou por se revelar uma decisão acertada.


## Onde serão utilizados os tuplos?

A introdução dos novos tuplos em C# (acompanhados do açucar sintático usado na sua definição através de literais) abre-nos as portas para muitos cenários. A recuperação de vários valores a partir de um método é, com toda a certeza, o cenário onde este tipo de elementos será utilizado mais frequentemente. Mas há mais. Por que não recorrer a este tipo de elementos quando necessitarmos de recuperar dados provenientes de uma consulta a uma base de dados que não constituem propriamente uma entidade? Nestes casos, o uso de tuplos permite-nos representar vários campos mantidos numa tabela, sem que isso implique a criação de um novo tipo.


## Conlusão

Este capítulo apresentou os tuplos e mostrou como podemos recorrer a estes elementos para devolvermos vários elementos a partir de um método. Depois de apresentar a sintaxe literal, o capítulo analisou ainda outras caraterísticas importantes associadas ao uso deste tipo de elementos, como, por exemplo, a implementação interna deste tipo de elementos ou a conversão entre tuplos. 

No próximo capítulo, continuamos a nossa análise das novidades introduzidas pelo C# 7.0 e vemos como o uso dos chamados *ref locals* nos permitem devolver valores por referência a partir de uma função. 


### Bibliografia

["C# Design Notes for Apr 6, 2016"](https://github.com/dotnet/csharplang/blob/master/meetings/2016/LDM-2016-04-06.md) 
["Tupples"](https://docs.microsoft.com/en-us/dotnet/articles/csharp/csharp-7#tuples) 
["Why do C# 7 Value Tuples implement the Equals method but not the double equals operator"](http://stackoverflow.com/questions/42170040/why-do-c-sharp-7-valuetuples-implement-the-equals-method-but-not-the-double-equa) 
["Support for == and != on ValueTuple"](https://github.com/dotnet/roslyn/issues/13155) 
["Quickstart guide for tuples"(]https://github.com/dotnet/roslyn/blob/master/docs/features/tuples.md) 
["Tuples in C# 7"](https://www.thomaslevesque.com/2016/07/25/tuples-in-c-7/) 
["Tackling Tuples: Understanding the new C# 7 Value Type"](http://our.componentone.com/2017/01/30/tackling-tuples-understanding-the-new-c-7-value-type/) 
