# Expressões de correspondência de tipos através de padrões

O C# sempre permitiu o uso de expressões que permitiam identificar os tipos de um determinado objeto. Com a versão 7.0, e influenciada pelo que acontece no mundo das linguagens funcionais, a linguagem levou o uso deste tipo de expressões um pouco mais longe. Neste capítulo, apresentamos detalhadamente todas as novidades relacionadas com o uso deste tipo de expressões.


## Introdução

A linguagem C# sempre permitiu a aplicação de estratégias que nos permitem seguir fluxos de execução em função do tipo de um determinado objeto. Por outras palavras, a linguagem permite-nos identificar o tipo de um objeto e reagir em conformidade com o tipo inferido a partir dessa análise. Comecemos por analisar o exemplo seguinte:

```cs
public class Quadrado
{
    public double Lado{ get;}
    public Quadrado(double lado)
    {
        Lado = lado;
    }
}
public class Retangulo
{
    public double Altura { get; }
    public double Largura { get; }
    public Retangulo(double altura, double largura)
    {
        Altura = altura;
        Largura = largura;
    }
}
public static class CalculadorArea
{
    public static double CalculaArea(object figura)
    {
        if (figura is Retangulo)
        {
            var retangulo = (Retangulo)figura;
            return retangulo.Altura * retangulo.Largura;
        }
        else if (figura is Quadrado)
        {
            var quadrado = (Quadrado)figura;
            return quadrado.Lado * quadrado.Lado;
        }
        return 0;
    }
}
var q = new Quadrado(2);
Console.WriteLine(CalculadorArea.CalculaArea(q));
```

Como é possível depreender a partir do seu nome, o principal objetivo do método estático `CalculaArea` é calcular a área de uma figura que lhe é passada através do parâmetro figura. No seu interior, recorremos ao operador is para identificarmos o tipo do objeto que lhe é passado. Quando estamos perante um objeto de um dos tipos "conhecidos" (`Quadrado` ou `Retangulo`), então recorremos a uma conversão explícita (*cast*) antes de calcularmos a área. Portanto, a partir do exemplo anterior, rapidamente deduzimos que o operador is é utilizado para detetar a forma ou tipo do objeto que é passado ao método `CalculaArea`.

Antes de avançarmos, importa salientar que não recomendamos a escrita de código semelhante ao apresentado no exemplo anterior. Em C#, recomendamos (sempre que possível) a aplicação de técnicas caraterísticas das linguagens OO (orientadas a objetos). Por exemplo, o leitor com alguma experiência neste tipo de linguagens rapidamente se apercebe da possibilidade de introduzirmos uma interface que estabeleça um contrato de cálculo de área, contrato este que pode ser implementado por ambos os tipos. A introdução deste contrato simplifica o código final e permite a realização do cálculo da área sem que isso implique a deteção do tipo em * runtime*. O excerto seguinte ilustra esta aproximação:

```cs
public interface IFigura
{
    double Area{ get; }
}
public class Quadrado: IFigura
{
    public double Lado{ get;}
    public Quadrado(double lado)
    {
        Lado = lado;
    }
    public double Area => Lado * Lado;
}
public class Retangulo: IFigura
{
    public double Altura { get; }
    public double Largura { get; }
    public Retangulo(double altura, double largura)
    {
        Altura = altura;
        Largura = largura;
    }
    public double Area => Altura * Largura;
}
IFigura q = new Quadrado(2);
Console.WriteLine( q.Area );
```

A partir da análise do código anterior, rapidamente inferimos que a introdução da interface que define o contrato de cálculo da área simplifica drasticamente o código existente no interior do método estático `CalculaArea`. Com esta abordagem, cada tipo de figura passa a ser responsável por calcular a sua área. Na prática, isto significa que os consumidores dos tipos anteriores deixam de ter de detetar o tipo de objeto em causa e passam a trabalhar apenas com a abstração introduzida pela interface `IFigura`.

Infelizmente, nem sempre temos a sorte de estar perante livrarias que foram criadas com base nos princípios que norteiam as linguagens orientadas a objetos. Nesses casos, temos de recorrer a técnicas como as apresentadas no início deste capítulo. Felizmente para nós, o C# 7 introduz as novas expressões de correspondência de tipos através de padrões (*pattern matching*), que procuram aumentar a produtividade do programador através da redução do código necessário nestes cenários. Comecemos, então, por analisar o primeiro tipo de expressões deste género, designadas na literatura atual por *if type patterns*.


## *If type patterns*

A forma mais rápida de introduzirmos este conceito passa pela apresentação de um exemplo. Assim, vamos começar por alterar a função `CalculaArea` apresentada no exemplo anterior. O excerto seguinte apresenta as alterações efetuadas para passarmos a utilizar as novas expressões de correspondência de tipos através de padrões:

```cs
public static class CalculadorArea
{
    public static double CalculaArea(object figura)
    {
        if (figura is Retangulo retangulo)
        {
            return retangulo.Altura * retangulo.Largura;
        }
        else if (figura is Quadrado quadrado)
        {
            return quadrado.Lado * quadrado.Lado;
        }
        return 0;
    }
}
```

Expressões como as apresentadas no exemplo anterior são designadas por *is expressions* e são utilizadas para verificar se uma expressão possui um determinado tipo. Estas expressões são constituídas por três partes. No lado esquerdo, encontramos uma expressão que será avaliada contra o tipo indicado pela expressão da direita. Como é possível verificar, a expressão da esquerda é separada da da direita através do operador `is`. Para além do tipo, a expressão utilizada na definição do padrão (expressão da direita) introduz ainda uma variável que apenas é considerada inicializada quando a expressão total possui o valor `true`.

Portanto, ao olharmos para o exemplo anterior, rapidamente concluímos que cada uma das expressões de correspondência de tipo (através de padrões) injeta automaticamente uma variável (`retangulo` no caso dos objetos do tipo `Retangulo` e `quadrado` no caso dos objetos do tipo `Quadrado`), variável essa que apenas pode ser utilizada quando a expressão de correspondência de tipo associada possuir o valor `true`.

As regras introduzidas pela linguagem são bem claras no que diz respeito ao âmbito de cada uma das variáveis `retangulo` e `quadrado` utilizadas no exemplo anterior. O primeiro aspeto a reter reside no facto de as variáveis introduzidas no exemplo anterior possuírem tempos de vida diferentes. Assim, o âmbito da variável retangulo é o método `CalculaArea` (e não apenas o bloco associado à instrução `if`). Por sua vez, o âmbito da variável `quadrado` está limitado ao ramo da instrução `else`.

Este comportamento é introduzido pelas regras que regem os âmbitos das instruções `if-else` e deve-se ao facto de o ramo associado à instrução `else` introduzir um novo âmbito (algo que não acontece com o ramo da instrução `if`, fazendo, assim, com que esta esteja no mesmo âmbito da sua declaração - o que, neste exemplo significa que quer a instrução `if`, quer a variável `retangulo` estão visíveis no âmbito do método `CalculaArea`).


### Âmbito das variáveis introduzidas pelas expressões *is*

Nesta altura, é possível que o leitor se questione acerca do âmbito da variável introduzida pela expressão `is`. Por exemplo, é provável que o leitor deseje saber se pode, ou não, utilizar a variável `retangulo` no interior do corpo do método `CalculaArea`, antes do bloco associado à instrução `if`. A resposta a esta questão é não. 

Uma vez que o C# não permite o acesso a uma variável antes de esta ser declarada, então qualquer tentativa de acedermos à variável antes da instrução `if` resulta na geração de um erro com o código *CS0841* (*Cannot use variable before it’s declared*). Por outro lado, se tentarmos recuperar o valor da variável `retangulo` a seguir ao bloco associado à instrução `if`, então obteremos outro erro (código *CS0165*), que nos informa que não podemos utilizar uma variável não inicializada (e isto porque as variáveis introduzidas pelos padrões das expressões de correspondência de tipos só são consideradas definitivamente inicializadas quando essas expressões produzem o valor `true`):

```cs
public static class CalculadorArea
{
    public static double CalculaArea(object figura)
    {
        if (figura is Retangulo retangulo)
        {
            return retangulo.Altura * retangulo.Largura;
        }
        return retangulo.Altura * retangulo.Largura; // <-- ERRO
        if (figura is Quadrado quadrado)
        {
            return quadrado.Lado * quadrado.Lado;
        }
        return 0;
    }
}
```

O mesmo acontece se tentarmos atribuir um valor a uma das propriedades do objeto referenciado pela variável `retangulo`.

Este erro ocorre apenas porque a variável não é considerada como inicializada. De acordo com as regras que foram introduzidas no Capítulo 1, o âmbito da variável não está limitado ao corpo da instrução `if`, podendo ser utlizada em qualquer ponto do corpo do método `CalculaArea` (que se situe após a sua declaração). Na prática, e uma vez que as variáveis introduzidas pelas expressões `is` são mutáveis, então podemos corrigir o problema anterior se inicializarmos a variável. O excerto seguinte ilustra este ponto:

```cs
public static class CalculadorArea
{
    public static double CalculaArea(object figura)
    {
        if (figura is Retangulo retangulo)
        {
            return retangulo.Altura * retangulo.Largura;
        }
        retangulo = new Retangulo(10.0, 20.0);
        return retangulo.Altura * retangulo.Largura; // <-- OK
        if (figura is Quadrado quadrado)
        {
            return quadrado.Lado * quadrado.Lado;
        }
        return 0;
    }
}
```


### Utilização de filtros adicionais com expressões *is*

Os exemplos apresentados até ao momento concentraram-se apenas na apresentação de padrões que permitem inferir o tipo de um objeto. Se quisermos, podemos ainda combinar estas expressões com outras que influenciam a avalização final da expressão de correspondência de tipo (através de um padrão). No excerto seguinte, modificamos a função anterior para mostrar como podemos restringir o cálculo da área a figuras com dimensões superiores a 1:

```cs
public static class CalculadorArea
{
    public static double CalculaArea(object figura)
    {
        if (figura is Retangulo retângulo
          && retangulo.Altura >= 1
          && retangulo.Largura >= 1)
        {
            return retangulo.Altura * retangulo.Largura;
        }
        if (figura is Quadrado quadrado
          && quadrado.Lado >= 1)
        {
            return quadrado.Lado * quadrado.Lado;
        }
        return 0;
    }
}
```

As condições adicionais são consideradas válidas porque apenas serão executadas se a primeira condição da expressão de correspondência de tipo for válida. Por outras palavras, quando a primeira condição dos testes das expressões de correspondência de tipos devolve o valor `true`, então o compilador sabe que as variáveis `retangulo` e `quadrado` já foram inicializadas, pelo que as condições seguintes podem ser avaliadas sem qualquer problema (este comportamento está completamente alinhado com o associado ao uso do operador de conjunção lógica - representado programaticamente em C# por `&&`).

Antes de analisarmos o uso de expressões de correspondência de tipos através de padrões em instruções `switch`, queremos aproveitar esta oportunidade para referir que também é possível inicializarmos a variável da expressão de tipo com base na negação da expressão de correspondência de tipo que é passada à instrução `if`. Nestes casos, a variável apenas será inicializada no ramo associado ao `else`. O excerto seguinte tenta ilustrar este ponto:

```cs
public static double CalculaArea(object figura)
{
    if (!(figura is Retangulo retangulo) )
    {
        //retangulo n inicializado aqui!
    }        
    else
    {
        //retangulo inicializado aqui...
        return retangulo.Altura * retangulo.Largura;
    }
}
```

No exemplo anterior, negamos a expressão de verificação de tipo (note-se o uso do operador negação lógica - `!`). Neste caso, `retangulo` continua a referenciar um objeto do tipo `Retangulo`, mas essa variável apenas será inicializada no interior do bloco associado à instrução `else`. Na nossa opinião, esta aproximação não deve ser seguida, uma vez que acaba por reduzir a legibilidade final do código.


## Operador *switch*

Nos exemplos anteriores, possuíamos apenas dois tipos diferentes de figuras. Apesar de a lógica apresentada poder ser estendida para n casos (tantos quantos desejarmos), a verdade é que a partir de um determinado número de casos, o uso da instrução `if` acaba por ser pouco prático e por influenciar negativamente a legibilidade do código. Se recordarmos as versões anteriores da linguagem, rapidamente nos lembramos da instrução `switch`, que é utilizada quando necessitamos de comparar uma expressão com um valor constante:

```cs
public static string Informa(int numero)
{
    switch(numero)
    {
        case 0: return "Sem componentes";
        case 1: return "Um componente";
        case 2: return "Dois componentes";
        default: return "Vários componentes";
    }
}
```

Até ao lançamento do C# 7, a instrução `switch` apenas podia ser usada para analisar padrões constantes, sendo que os valores utilizados neste tipo de expressões estavam limitados a tipos numéricos e a *strings*. Com a introdução das novas expressões de correspondência de tipos através de padrões, a linguagem removeu algumas das restrições existentes até ao momento e passou a permitir o uso deste tipo de expressões em instruções `switch`.

No excerto seguinte, alteramos o código definido no interior do método `CalculaArea` apresentado anteriormente, fazendo com que este passe a utilizar a instrução `switch` para identificar o tipo do objeto que é passado ao método:

```cs
public static class CalculadorArea
{
    public static double CalculaArea(object figura)
    {
        switch(figura)
        {
            case Retangulo retangulo:
                return retangulo.Altura * retangulo.Largura;
            case Quadrado quadrado: 
                return quadrado.Lado * quadrado.Lado;
            default:
                return 0;         
        }
    }
}
```

À primeira vista, o código apresentado no excerto anterior é muito semelhante ao código utilizado nas versões anteriores da linguagem, onde apenas eram suportados padrões constantes. Neste caso, a principal diferença reside na introdução de uma variável (colocada à frente da expressão de verificação de correspondência de tipo). Como seria de esperar, a avaliação das expressões `case` é feita de acordo com a ordem de definição, sendo que as instruções correspondentes apenas serão executadas quando a expressão de correspondência de tipo associada for verdadeira.

A entrada `default` continua a ser opcional e as eventuais instruções associadas só serão executadas quando a avaliação dos testes relativos às entradas case anteriores produzirem o resultado `false`. A proibição de fall through mantêm-se, fazendo, assim, com que o grupo de instruções associadas a uma entrada case tenha de ser terminada com uma instrução `break` ou `return`.

>**Posicionamento da entrada *default***<br>
>Tipicamente, a entrada default costuma ser apresentada após todas as entradas case. Note-se, contudo, que isso não é obrigatório. O leitor curioso pode verificar esta afirmação através da alteração do exemplo anterior (colocando, por exemplo, a entrada `default` antes de todas as entradas). Apesar disso, o código associado à entrada `default` só será executado quando nenhum dos testes associados às entradas case produzir o valor `true`. <br>


### Âmbito das variáveis introduzidas nas entradas *case*

Ao contrário do que acontece com as variáveis introduzidas em instruções `is`, as regras que gerem o âmbito das variáveis são intuitivas. Assim, o âmbito das variáveis introduzidas nestes casos está limitado ao bloco associado. Por outras palavras, a variável `retangulo` só pode ser utilizada no bloco associado à entrada `case` onde é definida. O mesmo acontece com a variável `quadrado`. O exemplo seguinte tenta ilustrar este ponto:

```cs
public static class CalculadorArea
{
    public static double CalculaArea(object figura)
    {
        double resultado;
        switch(figura)
        {
            case Retangulo retangulo:
                resultado = retangulo.Altura * retangulo.Largura;
                break;
            case Quadrado quadrado: 
                resultado = quadrado.Lado * quadrado.Lado;
                break;
            default:
                resultado = 0;         
                break;
        }
        quadrado = new Quadrado(10); //<--- ERRO
        return resultado;
    }
}
```

Se tentarmos compilar o código anterior, obtemos um erro (*CS0103*), que nos informa que o nome `quadrado` não existe no âmbito atual.

Já o tempo de vida da variável, salvo se capturada (por exemplo, por uma expressão *lambda*) é toda a instrução `switch`.


### Mistura entre tipos e valores constantes

Se assim o desejarmos, podemos misturar expressões com padrões tipos com expressões baseadas em padrões constantes. No exemplo seguinte, introduzimos um método que, com exceção do valor 0, duplica todos os inteiros recebidos (neste caso, quando o método recebe o valor 0, ele limita-se a devolver o valor -1):

```cs
public static class Contador
{
    public static int total(object obj)
    {
        switch (obj)
        {
            case 0:
                return -1;
            case int i:
                return i * 2;            
            default:
                throw new InvalidOperationException();
        }
    }
}
```

O valor constante 0 é um caso particular do tipo `int`, pelo que deve ser declarado antes do teste da verificação de tipo. Se não o fizermos, e uma vez que as entradas `case` são analisadas sequencialmente, então o código associado a esse caso (0) nunca seria executado, já que o teste de correspondência de tipo `0 int i` devolveria sempre o valor `true`. Felizmente para nós, não temos de nos preocupar com muitos dos casos semelhantes ao anterior, já que o compilador gera um erro sempre que deteta o uso de condições que nunca serão executadas (devido ao facto de existirem testes com condições mais abrangentes definidos antes dos mais específicos). Esta regra também é aplicada pelo compilador quando estamos na presença de expressões de correspondência de tipos cujos padrões utilizam tipos relacionados hierarquicamente através de herança.

A partir do exemplo anterior, rapidamente concluímos que a possibilidade de verificação de correspondência tipos através de padrões em instruções `switch` conduz à introdução de algumas alterações importantes no funcionamento deste tipo de instruções. Portanto, quando se usa combinação de padrões, e ao contrário do que acontece quando se usam constantes, a ordem de avaliação das entradas `case` importa e o comportamento do compilador foi alterado para garantir que ordem de avaliação respeita a ordem de declaração. Felizmente para nós, e como mencionámos anteriormente, o compilador já é suficientemente inteligente para detetar vários exemplos de código não executável (*unreachable code*). Apesar disso, as boas práticas recomendam a colocação de entradas case mais específicas antes das mais generalistas.


### Utilização de expressões *when*

Com o C# 6, o termo `when` passou a poder ser utilizado em conjunto com expressões `catch`, de forma a permitir o uso dos chamados filtros de exceções (*exception filters*). Com o lançamento da versão 7, o termo passa também a poder ser utilizado como guarda de uma expressão de correspondência de tipo através de um padrão. Para ilustrar esta nova funcionalidade, vamos supor que necessitamos de criar um método que verifica se estamos perante um número "válido". 

Neste exemplo simples, estamos apenas interessados em avaliar números do tipo `int`, `float` e `double`, sendo que todos os números devem ser considerados válidos com exceção dos valores `infinity` e `NaN`. O excerto seguinte apresenta uma possível solução inicial para este problema baseada no uso de expressões de verificação de tipo introduzidas nas secções anteriores:

```cs
public static class Verificador
{
    public static bool valido(object obj)
    {
        switch (obj)
        {
            case int x:
                return true;
            case float y:
                return !float.IsNaN(y) && !float.IsInfinity(y);
            case double z:
                return !double.IsNaN(z) && !double.IsInfinity(z);
            default:
                return false;
        }
    }
}
```

O código apresentado no excerto anterior é relativamente simples. Na realidade, até podemos dizer que a solução encontrada é bem compacta, especialmente quando a comparamos com o código C# 6 necessário para obtermos o mesmo resultado. Apesar disso, a verdade é que ainda podemos melhorá-lo através do uso do termo reservado `when`. O excerto seguinte apresenta as alterações efetuadas na sequência da utilização desse termo:

```cs
public static class Verificador
{
    public static bool valido(object obj)
    {
        switch (obj)
        {
            case int x:
            case float y when !float.IsNaN(y) && !float.IsInfinity(y):
            case double z when !double.IsNaN(z) && !double.IsInfinity(z):
                return true;
            default:
                return false;
        }
    }
}
```

Como é possível verificar, a introdução do termo `when` acabou por contribuir para compactar ainda mais o código utilizado. Este termo permitiu-nos adicionar condições extra à guarda principal, fazendo, assim, com que tenha sido possível agrupar todos os casos de sucesso através de uma única expressão `return` (note-se que no código anterior não existe nenhum caso de *fall through*, já que todos os blocos de instruções associados às entradas terminam com a instrução `return`).

Como seria de esperar, o termo `when` não está limitado a expressões de correspondência de tipos aplicadas a tipos primitivos e também pode ser utilizado com outro tipo objetos. Por exemplo, o excerto seguinte altera o método `CalculaArea` por forma devolver 0 quando o lado do quadrado ou a altura/largura do retângulo possuírem o valor 0:

```cs
public static class CalculadorArea
{
  public static double CalculaArea(object figura)
  {
   switch(figura)
   {
     case Retangulo retangulo when retangulo.Altura == 0 || retangulo.Largura == 0:
     case Quadrado quadrado when quadrado.Lado == 0:
         return 0;
     case Retangulo retangulo:
         return retangulo.Altura * retangulo.Largura;            
     case Quadrado quadrado:
         return quadrado.Lado * quadrado.Lado;
     default:
         throw new InvalidOperationException();
   }
 }
}
```

Sempre que algum dos elementos que caraterizam um retângulo ou um quadrado possui o valor 0, então não efetuamos nenhum cálculo e limitamo-nos a devolver diretamente o valor 0. Uma vez que as condições utilizadas nestes casos são mais específicas do que as restantes, então temos colocá-las sempre antes das restantes entradas `case` (se não o fizermos, então obteremos um erro de compilação). Note-se ainda como reutilizamos o mesmo nome de variável em algumas das entradas `case`, sem que isso resulte em qualquer erro de compilação (uma vez mais, as regras de âmbito associadas a estas variáveis possibilitam o seu uso apenas no local "adequado").

Para encerramos este capítulo, vamos ainda efetuar uma outra alteração ao código apresentado no método anterior. O excerto seguinte apresenta o aspeto do código final desse método:

```cs
public static class CalculadorArea
{
  public static double CalculaArea(object figura)
  {
    switch(figura)
    {
      case Retangulo retangulo when retangulo.Altura == 0 || retangulo.Largura == 0:
      case Quadrado quadrado when quadrado.Lado == 0:
          return 0;
      case Retangulo retangulo:
          return retangulo.Altura * retangulo.Largura;            
      case Quadrado quadrado:
          return quadrado.Lado * quadrado.Lado;
      case null:
          throw new ArgumentNullException();
      default:
          throw new InvalidOperationException();
     }
  }
}
```

Com a entrada de um teste para o padrão constante `null`, passamos a fornecer melhor *feedback* ao utilizador. Além disso, este teste permite-nos ainda alertar para o tratamento especial dado a este valor. Apesar de o valor constante `null` não possuir um tipo, a verdade é que pode ser convertido para qualquer outro objeto anulável, permitindo, assim, a compilação sem erro do exemplo anterior.


## Conclusão

Ao longo deste capítulo, vimos como as novas expressões de correspondência de tipos através de padrões nos permitem aplicar algumas das técnicas caraterísticas do mundo funcional ao código C#. Estas expressões permitem-nos introduzir variáveis, que apenas serão consideradas como sendo definitivamente inicializadas quando a expressão de tipo for avaliada com sucesso.

No próximo capítulo, apresentamos as principais alterações introduzidas pela linguagem relacionadas com o regresso de valores a partir dos chamados métodos assíncronos.


### Bibliografia

["Pattern matching"](https://docs.microsoft.com/en-us/dotnet/articles/csharp/pattern-matching) <br>
["C# 7: Pattern Matching"](http://blog.somewhatabstract.com/2017/01/23/c7-pattern-matching/) <br>

