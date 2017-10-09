# Devolução de valores por referência

Até ao momento, o C# não disponibilizava nenhuma estratégia que permitisse, de uma forma "segura", devolver um valor por referência a partir de um método. Com o lançamento do C# 7.0, esta lacuna é colmatada através da introdução do conceito de devolução de valores por referência. Portanto, a partir desta altura, e como veremos ao longo deste capítulo, já não temos de recorrer a apontadores e de efetuar as tradições operações de pin de memória para conseguirmos devolver um valor por referência a partir de um método. 


## Introdução

A linguagem C# sempre suportou o uso de parâmetros por referência (estes parâmetros são identificados através do uso do modificador `ref`). Como referimos no Capítulo 2, os parâmetros deste tipo não possuem espaço de armazenamento próprio, pelo que podem ser vistos como *aliases* para as variáveis utilizadas aquando da invocação do método. Por outras palavras, os parâmetros deste tipo podem ser vistos como identificadores, que nos permitem aceder diretamente ao espaço de armazenamento das variáveis que lhes foram passadas durante a execução do método.

O excerto seguinte ilustra esta funcionalidade:

```cs
void Main()
{
  int y = 20;
  MudaValor(ref y);
  Console.WriteLine(y); //10
}
static void MudaValor(ref int x)
{
  x = 10;
}
```

Se executarmos o código anterior, rapidamente concluímos que, após invocarmos o método `MudaValor`, a variável y possui o valor 10. Isto acontece porque o parâmetro `x` foi definido como parâmetro por referência. Portanto, no interior do método MudaValor, x não é mais do que um aliás para a variável `y`, ou seja, `x` referencia o espaço de armazenamento associado à variável `y`. Isto significa que todas as alterações efetuadas ao valor do parâmetro `x` estão, na realidade, a ser efetuadas diretamente no espaço de armazenamento associado à varável `x`.
Até ao lançamento do C# 7, este era o único uso permitido do modificador `ref` num programa escrito em C#. Note-se, contudo, que esta limitação era imposta apenas pela linguagem C# e não pelo CLR (*Common Language Runtime*). E isto porque o CLR sempre suportou a devolução de valores por referência. A devolução de um valor por referência é sinalizada através da aplicação do modificador `ref` ao tipo de retorno de um método.

Como é óbvio, existem várias restrições à aplicação deste qualificador ao tipo de retorno de um método. Assim, a utilização desta funcionalidade estava sujeita às condições seguintes (que foram impostas para simplificar o trabalho do *garbadge collector* - GC): 
* os campos de um objeto não podem ser aliases para variáveis;
* as entradas de um array não podem ser aliases para variáveis.

>**CLR (*Common Language Runtime*)**
>O CLR pode ser visto como um ambiente de execução para aplicações escritas no chamado código gerido (*managed code*). Para além de disponibilizar um ambiente de execução, o CLR fornece ainda outros serviços que contribuem para simplificar o desenvolvimento de aplicações. 

A falta de suporte da linguagem à devolução de valores por referência obrigava-nos, quando necessário, a criarmos métodos que devolviam apontadores (considerado código não seguro - *unsafe code*) e a fixar memória através do uso dos chamados pinned objects. Esta fixação era necessária devido à gestão dinâmica de memória efetuada pelo *garbadge collector* (GC), já que, sem ela, não era possível garantir que o apontador devolvido a partir de um método referenciava uma posição de memória válida. Ao leitor interessado em aprender mais sobre o funcionamento interno CLR, recomendamos o livro [CLR via C#](https://www.amazon.co.uk/CLR-via-C-Jeffrey-Richter/dp/B00PVXRSMO). 

Apesar de a implementação da devolução de valores por referência em C# ser possível (esta questão já tinha sido abordada algumas vezes no passado, tendo mesmo chegado a ser criado um [protótipo interno de teste](https://blogs.msdn.microsoft.com/ericlippert/2011/06/23/ref-returns-and-ref-locals/)), a existência de outras funcionalidades consideradas mais importantes e que necessitavam de ser trabalhadas fez com que a sua implementação tivesse sido adiada até agora. Se pensarmos um pouco, rapidamente concluímos que o número de cenários onde esta funcionalidade fazia sentido era algo limitado, pelo que a decisão da equipa era perfeitamente aceitável. Com o C# 7, esta situação está ultrapassada e passamos a ter uma forma de devolver uma referência para uma posição de memória ocupada por um elemento.


## Um exemplo simples: acesso direto aos campos de um objeto

Para ilustramos o uso desta funcionalidade, vamos começar por analisar o exemplo apresentado em seguida (a Figura 3.1 apresenta o resultado final obtido):

```cs
public class Carro
{
    private int velocidade;
    private int distancia;
    public ref int ReferenciaVelocidade() => ref velocidade;
    public ref int ReferenciaDistancia() => ref distancia;
    override string ToString() => $"V: {velocidade}; Kms: {distancia}";
}
void Main()
{
    var carro = new Carro();
    carro.ReferenciaDistancia() = 10;
    carro.ReferenciaVelocidade() = 20;
    Console.WriteLine(carro);
}
```

A classe `Carro` possui dois campos privados, designados por `velocidade` e `distancia`. Para além dos campos, a classe disponibiliza ainda dois métodos que nos permitem aceder diretamente ao espaço de armazenamento desses campos (note-se como o modificador `ref` é utilizado no tipo de retorno do método e na instrução `return` que é responsável por devolver um valor do método). Como mencionámos, podemos pensar nestes métodos como métodos que nos permitem aceder diretamente ao local de armazenamento dos campos de uma instância do tipo.

O excerto anterior mostra-nos ainda outra caraterística interessante associada ao uso deste tipo de métodos: eles passam a poder ser utilizados no lado esquerdo de uma instrução de atribuição. Repare-se como, no exemplo anterior, ambas as invocações dos métodos são usadas para modificar os valores dos campos internos do objeto `Carro` que foi criado no início do método `Main`.

A devolução de um valor por referência está sujeita a uma regra importante: o compilador tem de ser capaz de garantir que o valor por referência é "seguro-para-retornar", ou seja, tem de conseguir garantir que a referência retornada sobrevive ao tempo de vida do método que a devolve. Por exemplo, se analisarmos o excerto anterior, facilmente concluímos que os campos `km` e `velocidade` continuam a existir após o final da execução dos métodos `ReferenciaDistancia` e `ReferenciaVelocidade`. Logo, podem ser devolvidos por referência a partir destes métodos.

Se assim o desejarmos, também podemos fazer com uma função deste tipo devolva por referência um elemento mantido num *array*. O excerto seguinte ilustra esta estratégia:

```cs
ref int ObtemSegundoValor(int[] valores)
{
    // codigo real deveria verificar o array antes de aceder ao segundo elemento
    return ref valores[1];
}
int[] totais = { 10, 20, 30};
ObtemSegundoValor(totais) = 40;
Console.WriteLine(totais[1]);//40
```

Neste caso, `ObtemSegundoValor` devolve o segundo item de um *array* de inteiros que lhe é passado através de um parâmetro (isto é possível pois o *array* que contém esse elemento sobrevive ao método). Depois de recuperarmos um aliás para essa posição de memória, limitamo-nos a modificar o seu valor (atribuição do valor 40). Finalmente, recorremos ao método `Console.WriteLine` para confirmarmos que a alteração foi efetuada com sucesso.
A devolução por referência de elementos de arrays aplica-se apenas aos *arrays* e não a qualquer indexador. Não pode, por exemplo, ser aplicada a *strings* (e isto apesar de as *strings* poderem ser encaradas como um *array* de caracteres). Por outras palavras, o código apresentado no excerto seguinte não compila:

```cs
public static ref char ObtemPrimeiroCaracter(string nome) => ref nome[0];
```

A devolução por referência de propriedades de objetos dos chamados tipos por valor (*value types*) é possível, mas está sujeita a algumas limitações. Analisemos o exemplo seguinte:


```cs
public struct Pessoa
{
    public string Nome;
    public string Morada;
}
static ref string ObtemNome(Pessoa pessoa)
{
    return ref pessoa.Nome; //erro!
}
void Main()
{
    Pessoa p = new Pessoa { Nome = "Luis", Morada = "Funchal" };
    ObtemNome(p) = "Outro";
} 
```

Neste caso, `Pessoa` é uma estrutura (`struct`) composta por dois campos. O método `ObtemNome` tenta devolver uma referência para o campo `Nome` da estrutura `Pessoa` que lhe é passada. Se tentarmos compilar este código, obtemos um erro, que nos informa que não é possível devolvermos esse campo por referência. Isto acontece porque os valores deste tipo (`struct`) são passados por valor aos métodos. Portanto, a invocação do método anterior resulta na criação de uma nova cópia da estrutura referenciada pela variável `p`, cópia essa que é utilizada apenas no interior do método. Logo, o código escrito acaba por devolver uma referência para um valor local, que não sobrevive o método que o tenta devolver por referência.
A solução para este problema passa por transformarmos o parâmetro pessoa num parâmetro por referência, conforme ilustrado em seguida:

```cs
static ref string ObtemNome(ref Pessoa pessoa)
{
    return ref pessoa.Nome;
}
```

A partir desta altura, o parâmetro pessoa passa a ser um aliás da variável `p`, pelo que já pode ser utilizado como valor devolvido por referência a partir desse método.


## Tipos de elementos que podem ser devolvidos por referência

Como referimos na secção anterior, o compilador tem de ser capaz de verificar que o elemento devolvido por referência sobrevive ao método que o retorna. A partir desta regra, é possível concluirmos que os valores devolvidos por referência a partir de um método podem ser oriundos de:
1.	parâmetros que lhe são passados por referência (parâmetros anotados com o modificador `ref`) ou parâmetros de saída (qualificados pelo termo `out`);
2.	referências obtidas a partir da invocação de outros métodos que também devolvem valores por referência (isto é, métodos onde o tipo do valor devolvido é anotado com o termo `ref`);
3.	referências para campos de objetos que os sobrevivem;
4.	variáveis por referência definidas no seu interior (apenas em alguns casos).

O primeiro caso é, provavelmente, o mais simples de perceber. No exemplo seguinte, o método `DevolveString` recebe uma *string* através de um parâmetro por referência. Logo, esse parâmetro pode ser utilizado como elemento devolvido pela função:

```cs
void Main() 
{
    var nome = "Paulo";
    DevolveString(ref nome) = "Luis";
    Console.WriteLine(nome); // Luis
}
public static ref string DevolveString(ref string nome)
{
    //efetua algumas modificacoes ao nome
    return ref nome; //ok
}
```

Se o parâmetro nome não for anotado com o valor ref, então deixa de ser considerado como parâmetro de referência, pelo que deixa de poder ser devolvido pelo método `DevolveString`:

```cs
void Main() 
{
    var nome = "Paulo";
    DevolveString(ref nome) = "Luis";
    Console.WriteLine(nome); // Luis
}
public static ref string DevolveString(string nome)
{
    return ref nome; //erro: nome nao e passado por referencia
}
```

>**Parâmetros de saída (*out*)**
>Apesar de o exemplo anterior se concentrar no uso de um parâmetro por referência, a regra também é aplicada quando estamos perante parâmetros de saída (isto é, parâmetros anotados com o termo `out`).

O segundo caso é "seguro para retornar" apenas se todas as referências obtidas a partir da invocação de outros métodos por referência forem consideradas como "seguras para retornar". Quando isto acontece, o compilador sabe que o valor retornado por referência não pode ser local ao método que o devolve. O exemplo seguinte tenta ilustrar este ponto:

```cs
public static ref int TestaInvocacao(ref int a, ref int b) 
{ 
    int a1 = 10;
    int b1 = 20;
    (1) return ref EscolhePrimeiroZero(ref a, ref b); //OK
    (2) return ref EscolhePrimeiroZero(ref a1, ref b); //oops, poderia devolver local
}
public static ref int EscolhePrimeiroZero(ref int x1, ref int x2) 
{
    if( x1 == 0 ) 
        return ref x1; 
    return ref x2;
}
```

No excerto anterior, a função `TestaInvocacao` apresenta duas opções para devolver um valor do tipo `int` por referência. A opção (1) pode ser utilizada sem qualquer problema, já que o valor devolvido sobrevive ao método que o devolve (neste caso, estamos a falar de um dos parâmetros por referência que o próprio método `TestaInvocacao` recebeu, logo estamos a falar de um valor que continua a ser válido após a conclusão da sua invocação). O mesmo já não acontece com a opção (2). Neste cenário, existe a possibilidade de `EscolhePrimeiroZero` devolver a variável `b`, variável essa que é local ao método `TestaInvocacacao`. Portanto, o compilador não permite o uso desta instrução como tipo de retorno do método `TestaInvocacao`.
A devolução de um valor para o campo de um objeto a partir de um método de instância já foi ilustrada pelo exemplo utilizado quando introduzimos o conceito de devolução por referência. Como seria de esperar, a devolução de um elemento deste tipo não está limitada a métodos de instância. Por exemplo, no excerto seguinte apresentamos um método estático que devolve uma referência para um objeto que lhe é passado através de um parâmetro:

```cs
void Main() {
    var p = new Pessoa { Nome = "Paulo"};
    ObtemNome(p) = "Luis";
    Console.WriteLine(p.Nome);//Luis
}
public class Pessoa 
{
    public string Nome;
    public static ref string ObtemNome(Pessoa p)
    {
        return ref p.Nome;
   }
}
public static ref string ObtemNome(Pessoa p)
{
    return ref p.Nome;
}
```

Neste caso, o compilador sabe que o método `ObtemNome` devolve um campo de um objeto que sobrevive à sua execução. Note-se ainda como a devolução de um campo de um objeto passado através de parâmetro não obriga à aplicação do modificador `ref` ou `out` ao parâmetro (ao contrário do que aconteceria se quiséssemos devolver por referência um elemento do mesmo tipo do parâmetro).

O último caso requer algum cuidado. Como veremos na próxima secção, a linguagem também suporta o conceito de variável por referência. Uma variável por referência não é mais do que um aliás para uma posição de memória associada a outra variável. As variáveis deste tipo podem ser devolvidas por referência desde que os elementos por elas referenciados sobrevivam ao método (que as devolve por referência). No excerto seguinte, alteramos o método `ObtemSegundoValor` introduzido na secção anterior de forma a mostrarmos como podemos devolver corretamente uma variável por referência a partir desse método:

```cs
ref int ObtemSegundoValor(int[] valores) 
{
    ref var i = ref valores[1];
    return ref i;
}
```

Neste caso, `i` é uma variável por referência que, passe a redundância, referencia a posição de memória do segundo item de um *array* que foi passado ao método através de um parâmetro. Uma vez que este *array* sobrevive ao método, então a posição de memória da variável por referência `i` pode ser devolvida por referência a partir deste método.


## Variáveis por referência

Apesar de a possibilidade de utilização de um método que devolve uma referência para uma posição de memória ser uma funcionalidade interessante, a verdade é que existem casos onde é preferível guardarmos a referência devolvida numa variável. Note-se que a especificação do tipo de dados existente nessa posição de memória não é suficiente nestes casos. 
Por exemplo, suponhamos que alteramos o código anterior para o seguinte:

```cs
void Main()
{
    var carro = new Carro();
    int a = carro.ReferenciaDistância();
    var b = carro.ReferenciaVelocidade();
    a = 20;
    b = 30;
    Console.WriteLine($"a: {a}; b: {b}"); //20, 30
    Console.WriteLine(carro); //0, 0
}
```

Apesar de o código compilar corretamente, os resultados não são os esperados. E isto porque apesar de o compilador conseguir inicializar e inferir corretamente o tipo das variáveis (note-se o uso do termo `var` na inicialização da variável `b`), a verdade é que estas são vistas como variáveis regulares, sendo inicializadas apenas com os valores existentes nas posições de memória devolvidas pelos métodos. 

Se quisermos que estas variáveis passem a ser interpretadas como identificadores alternativos das posições de memória devolvidas pelo método, então, e à semelhança do que acontece com os parâmetros por referência, elas também têm de ser anotadas com o modificador `ref`, conforme ilustrado em seguida:

```cs
ref int a = ref carro.ReferenciaDistância();
ref var b = ref carro.ReferenciaVelocidade();
```

A partir desta altura, não existem quaisquer dúvidas quanto ao papel das variáveis: devem ser utilizadas como referências alternativas (*aliases*) para as posições de memória obtidas a partir da expressão utilizada na sua inicialização. Nesta altura, importa ainda reter que uma variável por referência necessita sempre de ser inicializada aquando da sua declaração. Para além disso, estas variáveis só podem ser inicializadas com posições de memória (e não com valores). 

Por outras palavras, o excerto seguinte resulta num erro de compilação porque a invocação do método sem o termo `ref` acaba por devolver um inteiro (e não a posição de memória ocupada por esse inteiro):

```cs
ref int a = carro.ReferenciaDistância(); //erro: ReferenciaDistância invocado sem ref
```

A existência desta regra permite-nos definir claramente o papel de cada variável e impede o uso incorreto acidental de uma variável. Nesta altura, resta-nos ainda referir que uma variável por referência pode ainda ser inicializada a partir de uma outra variável, conforme ilustrado através do excerto seguinte:

```cs
var i = 10;
ref var j = ref i; // j e um alias para i
```

A introdução da devolução de valores por referência e das variáveis por referência serão especialmente úteis quando necessitamos de desenvolver algoritmos eficientes e queremos evitar as cópias associadas ao uso dos chamados tipos por valor (*value types*). É provável que esta seja uma funcionalidade com pouco uso para a grande maioria dos programadores, mas resta-nos o conforto de saber que se precisarmos de eficiência, já não temos de recorrer diretamente a apontadores como acontecia até ao lançamento da versão 7 do C#.


## Conclusão

Este capítulo dedicou-se à análise de todas as particularidades associadas à devolução de valores por referência a partir de um método. Depois de explicar a motivação que levou à introdução desta nova funcionalidade, o capítulo introduziu ainda o conceito de variável por referência. Como vimos, estas variáveis podem ser utilizadas para referenciarem outras posições de memória obtidas à custa da invocação de um método que devolve um valor por referência.

No próximo capítulo, continuamos a apresentar as novas funcionalidades introduzidas pela linguagem C# 7.0, com especial enfase no uso das expressões de correspondência de tipos através de padrões (*pattern matching*).


### Bibliografia

["C# 7: Better Performance with Ref locals, and ref and Async returns"](http://blog.somewhatabstract.com/2017/02/06/c7-better-performance-with-ref-locals-and-ref-and-async-returns/?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+SomewhatAbstract+%28Somewhat+Abstract%29)
["Ref returns and ref locals"](https://blogs.msdn.microsoft.com/ericlippert/2011/06/23/ref-returns-and-ref-locals/) 
["Tip 26 - C# 7 Ref Returns and Ref Locals"](https://www.devu.com/advanced-csharp-tips/tip-26-c-7-0-ref-returns-and-ref-locals/) 
["C# 7: Ref Returns, Ref Locals, and how to use them"](https://www.danielcrabtree.com/blog/128/c-sharp-7-ref-returns-ref-locals-and-how-to-use-them) 
["C# 7 Additions – ref Variables"](http://structuredsight.com/2016/08/31/c-7-additions-ref-variables/) 
["C# Design Meeting Sep 1 2015"](https://github.com/dotnet/csharplang/blob/0709623617764880198f972d66845e88a89fe149/meetings/2015/LDM-2015-09-01.md)




