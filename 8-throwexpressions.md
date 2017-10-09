# Expressões *throw*

A proliferação dos membros que podem ser definidos através de expressões obrigou a equipa a repensar o papel do termo *throw*. Até ao momento, todas as referências a esse termo eram consideradas instruções. Com o lançamento do C# 7.0, a equipa foi obrigada a repensar o papel deste termo. Assim, e como veremos em seguida, este termo passa também a poder ser utilizado em expressões.


## Termo *throw*

Em C#, *throw* sempre foi considerada uma instrução (normalmente designada por *statement* na literatura inglesa). Devido a isso, existem algumas restrições que impedem o seu uso em determinados locais. Por exemplo, o seu uso não é permitido em:
1. Expressões condicionais;
2. Expressões definidas à custa do operador *null coalescing*;
3. Expressões *lambda*.

Portanto, na prática, isto significava que não era possível gerar exceções a partir das expressões semelhantes às seguintes:

```cs
void Main()
{
    Func<int, int> t = (i) => i > 10 ? throw new Exception() : 20; //ERRO
    t(20);
}
```

Nestes casos, uma possível solução passava pela introdução de um método, cujo único papel era gerar a exceção. O excerto seguinte ilustra esta aproximação:

```cs
void Main()
{
    Func<int, int> t = (i) => i > 10 ? GeraExcecao() : 20; //ERRO em C# 5
    t(20);
}

int GeraExcecao()
{
    throw new Exception();
}
```

Como vimos, o C# 7.0 aumentou a lista de membros de um tipo cujos corpos podem ser definidos à custa de expressões *lambda*. Portanto, esta possível proliferação de locais onde teríamos de recorrer a *hacks* semelhantes ao apresentado no exemplo anterior fez com que a equipa reconsiderasse o uso do termo *throw*. Assim, a partir da versão 7, throw passa a poder ser utilizado como expressão.

Na prática, nada muda no que diz respeito à sua sintaxe. A diferença prende-se mesmo com o facto de podermos passar a utilizar este termo em locais que nos estavam vedados até ao momento.

As expressões throw devem respeitar as regras seguintes:

```cs
throw_expression
    : 'throw' null_coalescing_expression
    ;
null_coalescing_expression
    : throw_expression
    ;
```

Uma expressão *throw* está sujeita a algumas regras. Em primeiro lugar, estas expressões não produzem um tipo. Para além disso, estas expressões possuem uma conversão implícita para qualquer tipo, pelo que, em termos de sintaxe, o seu valor pode ser atribuído a campos ou variáveis de qualquer tipo. No que diz respeito às regras de análise de fluxo, uma variável `v` só é considerada definitivamente inicializada antes da expressão `null_coalescing_expression` se lhe tiver sido atribuído um valor antes da expressão `throw_expression`. Qualquer variável v é considerada como definitivamente inicializada após a `throw_expression`.

Uma expressão throw é permitida nos contextos seguintes:
1. Como segundo ou terceiro operando do operador ternário `?:`;
2. Como segundo operando do operador *null-coalescing*  `??`;
3. Como corpo de uma expressão *lambda*.

Portanto, ainda existem alguns locais onde o uso de expressões *throw* não é permitida. Por exemplo, este tipo de expressões não pode ser utilizado em expressões booleanas (`&&` ou `||`) ou delimitada por parêntesis ([*Parenthesized expressions*](https://msdn.microsoft.com/library/aa691352)). 


## Alguns exemplos práticos

No excerto seguinte, utilizamos recorremos a uma expressão *throw* para gerar uma exceção quando o valor passado a um setter de uma propriedade não foi inicializado corretamente:

```cs
public class Pessoa
{
    private string _nome = "";
    private string _ultimoNome = "";
    public string Nome
    { 
        get => _nome; 
        set => _nome = value ?? "";
    }
    public string UltimoNome
    { 
        get=> _ultimoNome;
        set => _ultimoNome = value ?? throw new ArgumentNullException(nameof(value));
    }
    public string NomeCompleto => $"{Nome} {UltimoNome}";
}
```

Outro cenário possibilitado pela introdução de expressões *throw* é a possibilidade de geração de uma exceção a partir de uma expressão de inicialização. Até ao lançamento da versão 7.0, este tipo de código tinha de ser definido no interior de um construtor. O excerto seguinte ilustra este cenário:

```cs
public class Opcoes
{
  private Configuracao _config = 
         CarregaDefinicoes() ?? throw new InvalidOperation();
}
```

Neste caso, e como seria de esperar, o compilador acaba por colocar a inicialização no interior do construtor. Importa ainda ter em atenção que o código no excerto anterior foi apresentado apenas para ilustrar a nova funcionalidade e o seu uso em produção deve ser feito com algum cuidado, uma vez que a geração de exceções a partir de um construtor não é recomendada.


## Conclusão

Neste capítulo, analisamos as repercussões associadas ao facto de o termo *throw* passar a poder gerar expressões. Como vimos, estamos perante uma alteração pequena, mas que, à semelhança do que acontece com muitas das novidades, será muito útil em determinados cenários (nomeadamente, quando estamos perante o uso de operadores ternários, operadores *null coalescing* ou expressões *lambda*).

No próximo capítulo, encerramos o livro com a apresentação das novas funcionalidades relacionadas com a definição de literais numéricos.


### Bibliografia

["Throw expressions"](https://docs.microsoft.com/en-us/dotnet/articles/csharp/csharp-7#throw-expressions) 
["Throw expressions"](https://github.com/dotnet/csharplang/blob/master/proposals/csharp-7.0/throw-expression.md)
