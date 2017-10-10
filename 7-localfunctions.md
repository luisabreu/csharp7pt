# Funções locais

A partir da versão 7.0, o C# passa suportar a definição das chamas funções locais. Na prática, e como veremos em seguida, as funções locais não são mais do que funções definidas no interior de outras funções. Ao introduzir este tipo de funções a linguagem passa a disponibilizar uma solução elegante para um conjunto de problemas com que se deparam os programadores que necessitam de criar métodos assíncronos ou que devolvem um *iterator*. 


## Introdução

O C# 7 introduz o conceito de função local. Isto significa que, a partir desta altura, passamos a poder definir métodos no interior de outros métodos. Esta última afirmação pode levar o leitor com experiência em C# a alegar que a definição de métodos no interior de outros já é suportada pela linguagem desde a versão 2.0, altura em que foram introduzidos os métodos anónimos.

A introdução deste tipo de métodos contribuiu para simplificar o código necessário quando necessitamos de instanciar um *delegate* (ex.: quando queremos tratar um evento através de um *event handler*), já que passámos a ter uma forma de declarar um método e um *delegate* para esse método (antes da introdução desta funcionalidade, tínhamos de definir um método nomeado e proceder à inicialização do delegate a partir desse novo método nomeado).

Com a introdução do LINQ ([Language Integrated Query](https://msdn.microsoft.com/library/a73c4aec-5d15-4e98-b962-1274021ea93d)) na versão 3.0, a linguagem simplificou ainda mais a criação de métodos anónimos através da introdução das expressões *lambda*. Apesar de isso ser verdade, e como veremos no final do capítulo, a realidade é que esta nova funcionalidade resolve alguns dos problemas inerentes ao uso deste tipo de métodos. Para já, vamos concentrar-nos apenas em justificar a adição desta nova funcionalidade e na apresentação das regras de sintaxe que regem o seu uso.


## Por quê definir métodos no interior de outros métodos

Atualmente, existem muitas classes que acabam por introduzir métodos locais privados que apenas são consumidos a partir de um outro método disponibilizado por essa classe. Por um lado, esta separação do código em vários métodos contribui para melhorar a organização da classe. Por outro lado, esta separação pode acabar por dificultar a legibilidade do código, especialmente quando o programador não conhece a classe em causa e está a ler o código fora de um IDE moderno (*Integrated Development Environment*).

Nestes casos, a definição de um método no interior de outro método é ideal, já que permite organizar o código sem que isso implique uma diminuição da sua legibilidade (ao encontrarmos um método no interior de outro, rapidamente nos apercebemos de que esse método apenas pode ser invocado a partir do corpo do método onde foi definido).

Nesta altura, o leitor pode ser levado a pensar que a possibilidade de definição de métodos locais privados não traz nada de novo. Afinal, se um método é apenas invocado a partir de um outro método dessa classe, por que não omitimos a sua definição por completo e colocamos as suas instruções no local onde ele é invocado?

Existem pelo menos dois cenários onde essa solução não é apropriada: métodos que devolvem um *iterator* e métodos assíncronos (`async`/`await`). Em ambos os casos, o compilador acaba por injetar código que pode produzir resultados que, à primeira vista, parecem inesperados. A introdução das funções locais surgiu para dar resposta a estes dois casos específicos.

A forma mais fácil de nos apercebermos dos problemas que podem ocorrer nestas situações passa pela apresentação de um exemplo simples. Vamos supor que necessitamos de implementar um método que permite efetuar a iteração dos caracteres pertencentes a um determinado intervalo, intervalo esse que é delimitado por dois parâmetros que lhe são passados.

No excerto seguinte, apresentamos uma primeira solução que dispensa por completo o uso de funções locais privadas:

```cs
public class Auxiliar
{
    public IEnumerable<char> PercorreCaracteres(char start, char end)
    {
        if ((start < 'a') || (start > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                     nameof(start), "Carácter início incorreto.");
        }
        if ((end < 'a') || (end > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                     nameof(end), "Carácter final incorreto.");
        }
        if (end <= start)
        {
            throw new ArgumentException(
               $"{nameof(end)} não pode ser inferior a {nameof(start)}");
        }
        for (var c = start; c < end; c++)
        {
            yield return c;
        }
    }
}
```

À primeira vista, tudo parece bem com o método anterior. Este método começa por efetuar algumas validações antes de recorrer ao termo `yield` para enumerar os caracteres existentes no intervalo. O excerto seguinte ilustra um dos problemas clássicos que costuma apanhar os mais programadores mais distraídos:

```cs
var res = new Auxiliar().PercorreCaracteres('f', 'a');
Console.WriteLine("Criado");
foreach (var letra in res)
{
    Console.Write($"{letra}, ");
}
```

Uma análise rápida do código apresentado permite-nos concluir que invocação da função PercorreCaracteres conduz à geração de uma exceção. A pergunta importante é onde é que a exceção será gerada? Será que ela ocorre na linha onde é feita a invocação do método (primeira linha do excerto anterior)? Ou será na linha onde efetuamos a enumeração dos resultados devolvidos pelo método?

A resposta certa é a segunda, ou seja, a exceção será gerada na linha onde efetuamos a enumeração dos resultados devolvidos pelo método. A explicação para este comportamento deve-se ao facto de os *iterators* serem representados por máquinas de estado que efetuam o *tracking* dos itens enumerados (o leitor interessado pode obter mais detalhes acerca desta transformação em http://csharpindepth.com/Articles/Chapter6/IteratorBlockImplementation.aspx).

Na prática, isto significa que o código definido no interior do método `PercorrerCaracteres` só é executado quando avançamos o *iterator*, o que, no exemplo anterior, só acontece quando tentamos percorrer os itens através de um ciclo `foreach`. Uma possível solução para este problema consiste em passarmos o código de criação para um método local privado, conforme ilustrado no excerto seguinte:

```cs
public class Auxiliar
{
    public IEnumerable<char> PercorreCaracteres(char start, char end)
    {
        if ((start < 'a') || (start > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                  nameof(start), "Carácter início incorreto.");
        }
        if ((end < 'a') || (end > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                  nameof(end), "Carácter final incorreto.");
        }
        if (end <= start)
        {
            throw new ArgumentException(
              $"{nameof(end)} não pode ser inferior a {nameof(start)}");
        }
        return Cria(start, end);
    }
    private IEnumerable<char> Cria(char start, char end)
    { 
        for (var c = start; c < end; c++)
        {
            yield return c;
        }
    }
}
```

Esta pequena alteração é suficiente para forçar a execução das validações no local esperado. Por outras palavras, a alteração anterior foi suficiente para fazer com que a exceção passe a ser gerada aquando da invocação do método `PercorreCaracteres`. Neste caso, a nossa classe é extremamente simples, pelo que a introdução de um método privado acaba por não causar grandes transtornos.

Apesar disso, é fácil concluirmos que em classes mais complexas, a introdução de um ou mais métodos deste género não simplifica a vida de quem tem de perceber o seu funcionamento. Foi a pensar nestes cenários que a linguagem passou a suportar o uso de funções locais.
O excerto seguinte mostra como podemos obter os mesmos resultados através do uso deste tipo de funções:

```cs
public class Auxiliar
{
    public IEnumerable<char> PercorreCaracteres(char start, char end)
    {
        if ((start < 'a') || (start > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                nameof(start), "Carácter início incorreto.");
        }
        if ((end < 'a') || (end > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                nameof(end), "Carácter final incorreto.");
        }
        if (end <= start)
        {
            throw new ArgumentException(
                $"{nameof(end)} não pode ser inferior a {nameof(start)}");
        }
        
        return Cria(start, end);
        
        IEnumerable<char> Cria(char s, char e)
        { 
            for (var c = s; c < e; c++)
                yield return c;
        }
    }
}
```

Na nossa opinião, esta última versão não deixa quaisquer dúvidas quanto às funções desempenhadas pelo método `Cria`. As regras inerentes ao uso de funções locais impedem ainda que esta função seja invocada por engano fora do método `PercorreCaracteres`.

O local onde a função é definida fica ao critério do programador. Geralmente, este tipo de funções costuma ser definido no final do método pai, permitindo-lhes, assim, acesso a todos as variáveis existentes no interior do âmbito desse método pai. Se optarmos por definir a função noutro ponto, então apenas teremos acesso às variáveis que tiverem sido definidas até ao local onde a função foi definida.


## Pormenores de implementação

As funções locais são transformadas pelo compilador em métodos estáticos do tipo que contém o membro onde elas foram definidas (as funções locais podem ser definidas no interior de métodos, propriedades, construtores ou finalizadores de um tipo).

Neste caso, e uma vez que estamos perante um *iterator*, então o compilador acaba por criar uma classe que é utilizada para controlar a enumeração dos valores devolvidos pelo *iterator*. No excerto seguinte, apresentamos código C# semelhante ao código final que é gerado pelo compilador para o exemplo anterior:

```cs
public class Auxiliar
{
    // Methods
    public IEnumerable<char> PercorreCaracteres(char start, char end)
    {
        if ((start < 'a') || (start > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                            "start", "Carácter início incorreto.");
        }
        if ((end < 'a') || (end > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                           "end", "Carácter final incorreto.");
        }

        if (end <= start)
        {
            throw new ArgumentException(
               string.Format("{0} não pode ser inferior a {1}", "end", "start"));
        }
        return <PercorreCaracteres>g__Cria0_0(start, end);
    }
    // Nested Types
    [CompilerGenerated]
    private sealed class <<PercorreCaracteres>g__Cria0_0>d : IEnumerable<char>, 
                IEnumerable, 
                IEnumerator<char>, 
                IDisposable, 
                IEnumerator
    {
       //codigo removido
    }
    [IteratorStateMachine(typeof(<<PercorreCaracteres>g__Cria0_0>d)), CompilerGenerated]
    internal static IEnumerable<char> <PercorreCaracteres>g__Cria0_0(
                                                           char s, char e)
    {
        this.<c>5__1 = s;
        while (this.<c>5__1 < e)
        {
            yield return this.<c>5__1;
            char ch = this.<c>5__1;
            this.<c>5__1 = (char) (ch + '\x0001');
        }
    }
}
```

Como referimos, no interior de uma função local, podemos utilizar qualquer variável que tenha sido declarada até ao local da sua definição. Portanto, se nos concentrarmos no exemplo anterior, rapidamente concluímos que podemos utilizar os parâmetros `start` e `end` a partir do interior da função local `Cria`. 

Na prática, isto significa que podemos omitir os parâmetros definidos inicialmente pelo método `Cria` e passar a controlar a enumeração dos caracteres através do uso direto dos parâmetros esperados pelo método `PercorreCaracteres`. O excerto seguinte apresenta estas alterações:

```cs
public class Auxiliar
{
    public IEnumerable<char> PercorreCaracteres(char start, char end)
    {
        if ((start < 'a') || (start > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                 nameof(start), "Carácter início incorreto.");
        }
        if ((end < 'a') || (end > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                 nameof(end), "Carácter final incorreto.");
        }
        if (end <= start)
        {
            throw new ArgumentException(
               $"{nameof(end)} não pode ser inferior a {nameof(start)}");
        }        
        return Cria();        
        IEnumerable<char> Cria()
        { 
            for (var c = start; c < end; c++)
                yield return c;
        }
    }
}
```

Nestes cenários, o código gerado pelo compilador é ligeiramente diferente do apresentado anteriormente, já que é necessário capturar os valores dos parâmetros (sem esta operação de captura, os parâmetros não seriam propagados até ao método "final" `Cria`). Para isso, o compilador introduz uma nova classe, com tantos campos quantos os necessários para guardar os valores das variáveis "externas" que são utilizadas no interior da função local.

O excerto seguinte ilustra o código final que poderia ser gerado pelo compilador ao encontrar uma função local que consome variáveis definidas fora dessa função:

```cs
public class Auxiliar
{
    public IEnumerable<char> PercorreCaracteres(char start, char end)
    {
        <>c__DisplayClass0_0 class_ = new <>c__DisplayClass0_0();
        class_.start = start;
        class_.end = end;
        if ((class_.start < 'a') || (class_.start > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                  "start", "Carácter início incorreto.");
        }
        if ((class_.end < 'a') || (class_.end > 'z'))
        {
            throw new ArgumentOutOfRangeException(
                  "end", "Carácter final incorreto.");
        }
        if (class_.end <= class_.start)
        {
            throw new ArgumentException(
              string.Format("{0} não pode ser inferior a {1}", "end", "start"));
        }
        return class_.<PercorreCaracteres>g__Cria0();
    }
}
```

Como é possível inferir a partir do excerto anterior, a função local não é injetada no próprio tipo como método estático. Nestas situações, a função local passa a ser definido como método de instância da classe que é injetada pelo compilador para permitir a propagação dos valores dos parâmetros.


## Outras curiosidades

As funções locais permitem a aplicação dos chamados [*Caller Info Attributes*](https://msdn.microsoft.com/en-us/library/mt653988.aspx) aos parâmetros que definem. Para além disso, as funções locais acabam por esconder outros métodos que possuam o mesmo nome.
O excerto seguinte ilustra este ponto:

```cs
void Main()
{
    new Test().Verifica();
}
public class Test
{
    public void Verifica()
    {
        Print();
        void Print() => Console.WriteLine("Local");
    }    
    public void Print() => Console.WriteLine("Instance");
}
```


### Funções locais vs métodos anónimos

Até ao C# 2.0, a única forma de definirmos delegates passava pelo uso de métodos nomeados adicionados a classes. Com o lançamento da versão 2.0, a linguagem passou a permitir a definição de *delegates* através do uso de método anónimos. Com o lançamento da versão 3.0, a linguagem introduziu ainda o conceito de expressão *lambda*, que passou a ser a forma mais usual utilizada na definição deste tipo de métodos. Na maior parte das situações, a escolha da estratégia utilizada na definição do *delegate* (método anónimo vs expressão *lambda*) resume-se a uma questão de gosto pessoal.

>**Métodos anónimos vs expressões *lambda***<br>
>O leitor interessado pode obter mais informações sobre o uso e caraterísticas de cada uma destas funcionalidades no [guia da linguagem C#](https://docs.microsoft.com/dotnet/articles/csharp/programming-guide/statements-expressions-operators/anonymous-methods). <br>

Antes do lançamento do C# 7.0, este tipo de expressões (métodos anónimos e expressões *lambda*) permitiam-nos criar métodos que, à primeira vista, podem parecer muito semelhantes às funções locais apresentadas neste capítulo. Contudo, e como referimos no início deste capítulo, existem algumas diferenças entre as funções locais e os métodos anónimos usados para criar delegates. A principal resume-se ao facto de a invocação de um método anónimo ser sempre efetuada através de um *delegate*. A partir daqui, é fácil concluirmos que a performance associada ao uso de funções locais tende a ser melhor (especialmente quando estamos perante cenários de recursão).

Mas há mais. As funções locais podem ser invocadas em qualquer local do método onde foram definidas. Por sua vez, os métodos anónimos só podem ser invocados depois de terem sido definidos. Finalmente, as instruções definidas no corpo de um método anónimo estão sujeitas a algumas limitações (que não se aplicam às instruções definidas no interior de funções locais). Por exemplo, o uso do termo `yield` não é permitido no interior deste tipo de expressões.
Conforme se pode comprovar pelo exemplo usado neste capítulo, é possível implementar um *iterator* através de uma função local, mas tal não é possível se utilizarmos um método anónimo.


## Conclusão

O C# 7.0 introduz o conceito de função local. Como vimos, as funções locais não são mais do que funções que são definidas no interior de outras funções, cujo principal objetivo é simplificar a criação de métodos assíncronos ou de métodos que devolvem *iterators*.

No próximo capítulo, continuamos a analisar as novas funcionalidades introduzidas pelo #C 7.0 e vamos apresentar as principais novidades relacionadas com a geração de exceções.


### Bibliografia

["Local functions"](https://docs.microsoft.com/en-us/dotnet/articles/csharp/whats-new/csharp-7#local-functions) <br>
["Thoughts on C# 7 Local Functions"](https://asizikov.github.io/2016/04/15/thoughts-on-local-functions/) 
