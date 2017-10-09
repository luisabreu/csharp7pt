# Declaração de variáveis na lista de parâmetros de saída

O C# sempre suportou o uso dos chamados parâmetros de saída (*out* parameters) desde o seu lançamento. Até ao lançamento da versão 7.0 da linguagem, o uso destes tipos de parâmetros costumava obrigar o programador a algum trabalho extra. Felizmente para nós, e como veremos ao longo deste capítulo, a utilização destes parâmetros em C# 7.0 é simplificada através da introdução da declaração de variáveis na lista de parâmetros de saída de uma função.


## Introdução aos parâmetros

Os parâmetros permitem-nos passar valores para o interior de um método. Analisemos o excerto seguinte:

```cs
static class Informacao{
 static void ImprimeIdade(int idade)
 {
  Console.WriteLine($" Voce tem: {idade}");
 }
 static void Main()
 {
  var idade = 10;
  ImprimeIdade(idade); //10
 }
}
```

O método estático `ImprimeIdade` espera um parâmetro do tipo `int`, designado por `idade`. Este parâmetro permite-nos passar um valor (neste caso, um inteiro) para o interior do método aquando da sua invocação. Parâmetros como os usados no exemplo anterior são designados por parâmetros regulares ou parâmetros por valor. Este nome deve-se ao facto de os parâmetros deste tipo possuírem espaço de armazenamento próprio, que é inicializado com o valor que é passado ao método aquando da sua execução. Eventuais alterações efetuadas num parâmetro deste tipo no interior do método não afetam a variável utilizada na inicialização desse parâmetro aquando da invocação do método.

Portanto, se, no método `ImprimeIdade` apresentado no excerto anterior, alterarmos o valor do parâmetro `idade`, essa alteração não afetaria o valor armazenado pela variável `idade`, conforme ilustrado através do excerto anterior:

```cs
static class Informacao
{
 static void ImprimeIdade(int	 idade)
 {
  idade = idade + 1;
  Console.WriteLine($" Voce tem: {idade}");
 }
 static void Main()
 {
  var idade = 10;
  ImprimeIdade(idade); //11
  Console.WriteLine(idade); // 10
 }
}
```

Se, a partir do interior de um método, pretendermos modificar a variável que lhe é passada através de um parâmetro, então temos de recorrer aos chamados parâmetros por referência. Estes parâmetros são definidos através do modificador `ref` e, ao contrário do que acontece com os parâmetros por valor, não possuem espaço de armazenamento próprio. Na prática, isto significa que estes parâmetros podem ser vistos como *aliases* para as variáveis que são passadas a um método aquando da sua invocação:

```cs
static class Informacao
{
 static void ImprimeIdade(ref int idade)
 {
  idade = idade + 1;
  Console.WriteLine($" Voce tem: {idade}");
 }
 static void Main()
 {
  var idade = 10;
  ImprimeIdade(ref idade); //11
  Console.WriteLine(idade); // 11
 }
}
```

Note-se ainda como a invocação de métodos que possuem parâmetros deste tipo também obriga ao uso do modificador `ref`. Para além disso, importa ainda reter que uma variável só pode ser passada a um parâmetro depois de ser inicializada (portanto, se não tivéssemos atribuído o valor 10 à variável idade, esta não poderia ter sido passada ao método `ImprimeIdade`).

> **Parâmetros por referência vs tipos por referência**
>Na literatura relacionada com a linguagem C# (e com a plataforma .NET), a designação “por referência" é utilizada para descrever conceitos diferentes, o que pode contribuir para confundir o leitor iniciado. Uma das confusões mais frequentes prende-se com a diferença entre tipos de parâmetros e tipos de objetos que podem ser criados em .NET. 
Assim, os tipos de objetos criados em .NET podem ser caraterizados como tipos por referência ou como tipos por valor. Classes, interfaces e *delegates* são considerados tipos por referência. Por sua vez, as estruturas (`struct`) e enumerações (`enum`) são representadas por tipos por valor. 
>As variáveis de tipos por referência guardam, como o próprio nome indica, uma referência para o local onde está o valor. Por outras palavras, os espaços de armazenamento das variáveis deste tipo contêm os endereços onde podemos encontrar os respetivos valores. Se quisermos, podemos pensar nas variáveis deste tipo como sendo apontadores para outros locais de memória que contêm o valor referenciado. Por sua vez, as variáveis do tipo por valor guardam diretamente um valor desse tipo. 
>O tipo de um objeto não está diretamente relacionado com a passagem de valores através de parâmetros, sendo que o tipo de um parâmetro pode ser representado por um tipo por referência ou por um tipo por valor (e isto independentemente de os parâmetros serem passados por valor, por referência ou de saída).

O C# também possibilita o uso dos chamados parâmetros de saída (normalmente designados na literatura por *out parameters*). Estes parâmetros são utilizados quando queremos delegar a inicialização de uma variável num método. Ao contrário do que acontece com os parâmetros por referência, os parâmetros deste tipo não necessitam de ser inicializados antes de serem passados a um método. Por outro lado, e à semelhança do que acontece com os parâmetros por referência, os parâmetros de saída também não possuem um espaço de armazenamento associado.

Para que um parâmetro seja considerado como sendo de saída, a sua declaração tem de ser precedida do modificador `out` (modificador este que também tem de ser utilizado aquando da invocação do método). O exemplo seguinte apresenta o código utilizado na declaração e consumo deste tipo de parâmetros:

```cs
static class Informacao
{
 static void InicializaEImprimeIdade(out int idade)
 {
  idade = 10;
  Console.WriteLine(idade);
 }
 static void Main()
 {
  int i;
  InicializaEImprimeIdade (out i); //10
  Console.WriteLine(i); //10
 }
}
```

Como é possível verificar, o modificador `out` é usado simultaneamente na declaração e na invocação do método. A variável passada ao método não necessita de ser inicializada antes de ser passada ao método, sendo considerada inicializada apenas após o método ter sido concluído normalmente. Os parâmetros deste tipo têm sempre de ser inicializados no interior do método quando este é concluído com sucesso (isto é, quando estamos perante métodos que terminaram sem gerarem qualquer erro ou exceção).

> **Tipos de parâmetros**
>Para além dos 3 tipos de parâmetros anteriores, a linguagem suporta ainda o uso de parâmetros do tipo *params*. Os parâmetros deste tipo permitem-nos passar um número indeterminado de parâmetros a um método aquando da sua invocação. Programaticamente, estas variáveis são representadas por *arrays*, que são anotados com o modificador `params`.

Tipicamente, os parâmetros de saída são utilizados em métodos que necessitam de devolver mais do que um valor (como, per exemplo, o padrão seguido pelas bibliotecas da plataforma .NET nos métodos `Try...`, usados para efetuar conversões ou obter membros de dicionários). Nestes casos, o resultado devolvido pelo método indica se a operação foi, ou não, efetuada com sucesso, devendo o valor obtido ser retornado através de um parâmetro de saída. O método `DateTime.TryParse` é um destes métodos e o seu uso é ilustrado no excerto seguinte:

```cs
DateTime data;
if (DateTime.TryParse("10/02/2017", out data))
{
  Console.WriteLine($"Data convertida: {data}");
}
else
{
  Console.WriteLine("Sem conversão");
}
```

## Sintaxe

A declaração explícita de uma variável local utilizada para recuperar o valor convertido devolvido através do parâmetro de saída era obrigatória até ao momento. Com o C# 7, o código escrito nestes cenários é reduzido, já que a linguagem passa a permitir a declaração da variável na própria lista de parâmetros que é passada ao método durante uma invocação. O excerto seguinte adapta o código anterior, fazendo com que este passe a utilizar esta nova funcionalidade:

```cs
if (DateTime.TryParse("10/02/2017", out DateTime data))
{
  Console.WriteLine($"Data convertida: {data}");
}
else
{
   Console.WriteLine("Sem conversão");
}
```

Se quisermos, podemos reduzir ainda mais o código digitado através do uso das chamadas variáveis locais implicitamente tipificadas (*implicitly typed local variables*). Nestes casos, podemos substituir o tipo da variável pelo uso do identificador `var`, conforme ilustrado em seguida:

```cs
if (DateTime.TryParse("10/02/2017", out var data))
{
  Console.WriteLine($"Data convertida: {data}");
}
else
{
  Console.WriteLine("Sem conversão");
}
```

A declaração de variáveis na lista de parâmetros introduz alguns benefícios. A mais visível de todas é a redução do código utilizado. Note-se que esta redução não está limitada apenas ao texto associado à declaração da variável. Por exemplo, suponhamos o seguinte exemplo:

```cs
class Pessoa
{
  // outros membros da classe...
  public int? Idade
  {
    get
    {
       int i = 0;
       return int.TryParse(this.idadeString, out i) ? i : (int?)null;
    }
  }
}
```

Se combinarmos a declaração da variável na lista de saída com o uso de uma expressão para definir a propriedade anterior, conseguimos transformar o código anterior em algo semelhante ao seguinte:

```cs
class Pessoa
{
  // outros membros da classe...
  public int? Idade => 
        int.TryParse(this.idadeString, out var i) ? i : (int?)null;
}
```

Apesar de não parecer, o uso de declarações de variáveis em listas de parâmetros continua a "produzir" instruções (e não expressões!). Na prática, isto significa que temos de ter algum cuidado na utilização desta nova funcionalidade. Por exemplo, a declaração de variáveis na lista dos parâmetros de saída não pode ser usada em locais onde que produzem expressões. 

No excerto seguinte, tentamos recorrer a esta funcionalidade para inicializamos o valor de um campo de uma classe aquando da sua declaração. Atualmente, a linguagem permite apenas o uso de expressões neste tipo de inicialização. Portanto, se tentarmos compilar o código apresentado em seguida, então obteremos um erro de compilação:

```cs
class Pessoa
{
  private int? valor = int.TryParse("10", out var i) ? i : (int?)null;
}
```

Para além da redução do código típico utilizado quando necessitamos de utilizar parâmetros de saída, a declaração de variáveis na lista de parâmetros de saída contribui ainda para melhorar a legibilidade do código (note-se como nos exemplos anteriores, a variável passa a ser declarada apenas no local onde é utilizada aquando da invocação do método).


### Ignorar parâmetros de saída

Se não necessitarmos do valor devolvido pelo método através do parâmetro por saída, então podemos ignorá-lo. Para isso, temos apenas de substituir a declaração da variável pelo uso do carácter `_`. Para ilustrar esta técnica, vamos recuar até ao exemplo anterior que efetua o *parsing* da data e vamos supor que apenas estamos interessados em verificar se a *string* contém, ou não, uma data válida. Nesses casos, podemos recorrer a código semelhante ao seguinte:

```cs
if (DateTime.TryParse("10/02/2017", out _))
{
  Console.WriteLine("Data correta");
}
else
{
  Console.WriteLine("Data incorreta");
}
```

Ao encontrar o carácter `_` em vez de uma declaração de uma variável, o compilador sabe que deve ignorar esse valor. O número de parâmetros de saída que podem ser ignorados não é limitado. Por outras palavras, podemos ignorar quantos parâmetros quisermos. O excerto seguinte ilustra este ponto:

```cs
void Foo(out int p1, out string p2, out bool p3, out char p4)´
{
  p1 = 42;
  p2 = "bla";
  p3 = true;
  p4 = 'x';
}
Foo(out var i, out _, out _, out _);
Console.WriteLine(i); 
```

Neste exemplo, estamos apenas interessados em recuperar o valor do primeiro parâmetro de saída, pelo que nos limitámos a passar o carácter `_` para os restantes parâmetros de saída do método.


## Âmbito e tempo de vida de uma variável

Em C#, o termo âmbito (*scope*, em inglês) é uma propriedade do nome da variável e identifica o local (ou locais) onde podemos referir-nos a uma variável no texto do programa que estamos a escrever. Por sua vez, quando falamos de tempo de vida de uma variável, estamos a referir-nos ao intervalo de tempo entre o momento da sua criação e o da sua destruição. O tempo de vida de uma variável é uma característica da própria variável e, na maior parte das vezes, corresponde ao âmbito do seu nome. Contudo, isso nem sempre acontece. Em C#, o tempo de vida de uma variável pode ser estendido quando, por exemplo, nos criamos uma expressão lambda que utiliza uma variável que foi declarada fora do seu corpo.

A declaração de variáveis na lista de parâmetros de saída de uma função pode levantar algumas dúvidas quanto ao âmbito e tempo de vida dessas variáveis, especialmente quando estamos perante alguns cenários mais "exóticos". Analisemos, então, o exemplo seguinte:

```cs
void Testa()
{
  //p pode ser utilizado aqui?
  while (int.TryParse(Console.ReadLine(), out var p))
  {
    // um p em cada passo do ciclo ou o mesmo?
  }
  // p existe aqui?
}
```

O exemplo anterior introduz duas questões que necessitam de resposta: qual o âmbito da variável `p`? E qual o tempo de vida da variável `p`? Comecemos, então, pela primeira pergunta: qual o âmbito da variável `p`?


### Âmbito de uma variável declarada na lista de parâmetros de saída

Como é óbvio, a variável p não pode ser utilizada no interior do método `Testa` antes da instrução `while` (e isto porque, em C#, as variáveis só podem ser utilizadas depois de terem sido declaradas). Como seria de esperar, a variável pode ser usada sem quaisquer restrições no interior. 

Mas será que podemos utilizá-la depois do corpo da instrução `while`? À primeira vista, e se pensarmos na declaração de variáveis na lista de parâmetros de saída como um atalho para o código utilizado antes do lançamento do C# 7.0, então tudo apontaria para que fosse possível referir-nos à variável `p` após o final do corpo do ciclo. Por outras palavras, se pensarmos no código anterior como sendo uma simplificação do exemplo seguinte:

```cs
void Testa()
{
  int p;
  while (int.TryParse(Console.ReadLine(), out p))
  {
    // código
  }
  //ok utilizar p aqui
}
```

Então `p` deveria poder ser utilizado após o corpo do ciclo `while`. Para que isto acontecesse, seria necessário efetuar o derrame da variável para o bloco pai (comportamento este que é designado na literatura por *scope leakage*). As primeiras versões *beta* da linguagem adotaram este comportamento.

Para além de ser pouco intuitivo, este comportamento estava ainda sujeito a algumas variações, que dependiam do tipo de instrução onde a declaração de variável na lista de parâmetros de saída era efetuada. Por exemplo, se estivermos perante instruções `for`, `foreach` e `using`, então o âmbito da variável era diferente, uma vez que estas instruções já suportavam a declaração de variáveis nas expressões utilizadas, sem que isso implicasse o derrame da variável para fora do bloco onde ela foi declarada. 

No excerto seguinte, ilustramos este ponto através de um ciclo for simples:

```cs
for( var i = 0; i < 10; i++ )
{
  Console.WriteLine(i); //ok
}
Console.WriteLine(i); //erro, fora de ambito
```

Neste caso, a variável `i` apenas pode ser utilizada na expressão associada (isto é, nas expressões de inicialização, controlo e incremento) ou no interior do corpo do ciclo `for`. Se tentarmos nos referir a essa variável após o corpo do ciclo, então obtemos um erro de compilação. 

Foi a pensar na manutenção da coerência com este comportamento existente que a equipa de desenho da linguagem C# optou por alterar o comportamento inicial de derrame (descrito no início desta secção), fazendo, assim, com que, nestes casos (`for`, `foreach` e `using`), o âmbito das variáveis declaradas na lista de parâmetros estivesse limitada apenas ao corpo associado à instrução:

```cs
void Testa()
{
  for (;int.TryParse(Console.ReadLine(), out var p);)
  {
    // um p em cada passo do ciclo ou o mesmo?
    Console.WriteLine(p);//ok
  }
  //erro, ao contrario do que acontecia no ciclo while 
  //apresentado no exemplo anterior
  Console.WriteLine(p);
}
```

Felizmente para todos, a equipa da *Microsoft* acabou por simplificar estas regras de âmbito na versão final da linguagem. Assim, a versão 7.0 da linguagem optou por limitar o âmbito das variáveis declaradas na lista de parâmetros de saída aos blocos associados às instruções onde elas foram declaradas. 

Portanto, e se regressarmos ao nosso exemplo inicial, então rapidamente concluímos que a variável p apenas pode ser utilizada no interior do bloco associado ao ciclo `while`:

```cs
void Testa()
{
  while (int.TryParse(Console.ReadLine(), out var p))
  {
   // um p em cada passo do ciclo 
  }
  // p nao existe aqui!
}
```

Este "novo" comportamento também é aplicado às variáveis declaradas na lista de parâmetros de saída utilizadas em instruções `while`, `do...while`, `for`, `using` e `foreach`. Nesta altura, o único local onde a linguagem suporta o chamado *scope leakage* é quando estamos perante variáveis declaradas na lista de parâmetros de saída de métodos utilizados em instruções `if`. Portanto, o código apresentado no exemplo seguinte compila sem qualquer erro:

```cs
void Testa()
{
  if (int.TryParse("aa", out var res))
  {
    Console.WriteLine($"if: {res}");
  }
  else
  {
    Console.WriteLine($"else: {res}");
  }
  Console.WriteLine($"fora: {res}");
}
```

A introdução desta exceção pode, contudo, conduzir a resultados "inesperados" quando estamos perante instruções `if` que contêm expressões de correspondência de tipos através de padrões (no Capítulo 4, analisamos esta nova funcionalidade detalhadamente). Como veremos, estas expressões acabam por introduzir novas variáveis, que também ficam sujeitas às regras de âmbito apresentadas nesta secção. Na prática, isso significa que o código semelhante ao seguinte não compila:

```cs
void Testa()
{
  object s = "";
  if (s is int i)
  {
        Console.WriteLine($"i={i}, s={s}");
  }
  Console.WriteLine(i); // ambito valido, mas nao inicializada
}
```

Apesar de o âmbito ser válido, a variável `i` não é considerada inicializada quando a avaliação da expressão associada ao if produz o valor `false`. Portanto, o compilador acaba por não permitir a compilação do excerto anterior porque não tem a certeza de que a variável `i` tenha sido inicializada na última linha do método `Testa`.


### Tempo de vida de uma variável declarada na lista de parâmetros de saída

Agora que já analisamos as questões relacionadas com o âmbito das variáveis declaradas na lista de parâmetros de saída, podemos concentrar-nos na discussão do tempo de vida de variáveis deste tipo quando estamos perante ciclos. Vamos, então, começar por recuperar o exemplo apresentado no início desta secção:

```cs
void Testa()
{
  while (int.TryParse(Console.ReadLine(), out var p))
  {
    // um p em cada passo do ciclo ou o mesmo?
  }
}
```

Qual será o tempo de vida da variável `p` no interior do corpo do ciclo `while`? Será que, durante a execução do ciclo, temos apenas uma variável, que é introduzida aquando da primeira invocação do método na guarda do ciclo e cujo valor é atualizado em cada avaliação dessa guarda? Ou será que cada invocação do método `int.TryParse` na guarda do ciclo acaba por resultar na introdução de uma "nova" variável? A equipa da *Microsoft* optou pela segunda opção, fazendo, assim, com que cada nova invocação do método resulte na criação de uma nova variável (comportamento este que é semelhante ao de um ciclo `foreach`).

Na prática, isto significa que cada execução do corpo do ciclo utiliza uma nova variável `p`, que é inicializada através da execução do método `int.TryParse`. Parece-nos que esta é, sem qualquer dúvida, a melhor opção, especialmente quando consideramos uma possível definição de funções através do uso de expressões lambda no interior de um ciclo. Nestes casos, a utilização de uma única variável acabaria por gerar alguns resultados inesperados devido à captura (*closure*) efetuada por esse tipo de expressões. Tendo em conta a experiência dos últimos anos, a introdução de uma nova variável em cada passo do ciclo é, sem qualquer dúvida, a melhor opção.

> ***Closures*** e tempo de vida
>Os problemas associados ao uso de variáveis em closures introduzidas por expressões *lambda* acabaram por conduzir à introdução de uma breaking change no lançamento C# 5.0. Assim, essa versão da linguagem alterou o comportamento de variáveis declaradas em expressões `foreach`, fazendo com que cada execução do passo do bloco associado resultasse na introdução de uma nova variável. O leitor interessado pode obter mais informações sobre alguns dos problemas que conduziram a esta alteração neste [post](https://blogs.msdn.microsoft.com/ericlippert/2009/11/12/closing-over-the-loop-variable-considered-harmful/) escrito pelo Eric Lippert. 


## Conclusão

Neste capítulo, vimos como podemos recorrer à declaração de variáveis na lista de parâmetros de saída que são passados a uma função. Como referimos, esta nova funcionalidade contribui para reduzir o código associado a estes cenários e para melhorar a legibilidade do código final. 
No próximo capítulo, continuamos a apresentar as novas funcionalidades do C# 7.0 e vamos ver como é que o uso de tuplos simplifica o retorno de vários valores a partir de um método.


### Bibliografia

["Out Variables"](https://docs.microsoft.com/en-us/dotnet/articles/csharp/csharp-7#out-variables) 
["C# Language Design Notes for Nov 30, 2016"](https://github.com/dotnet/csharplang/blob/master/meetings/2016/LDM-2016-11-30.md) 
["Parameter passing in C#"](http://www.yoda.arachsys.com/csharp/parameters.html) 
["C# 7, 'out var' and changing variable scope"](http://www.davidarno.org/2016/11/23/c-7-and-out-var-or-how-to-seriously-screw-up-a-language/)
["C# 7, 'out var' and changing variable scope, revisited"](http://www.davidarno.org/2017/01/30/c-7-out-var-and-changing-variable-scope-revisited/)

