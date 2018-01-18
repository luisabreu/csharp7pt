# Outras novidades do C# 7.2

Este capítulo condensa as restantes novidades do C# 7.2 que ainda não foram apresentadas nos restantes capítulos do livro. Assim, começamos por analisar o novo tipo de acesso definido à custa dos termos `private protected`, para, em seguida, analisarmos as novidades introduzidas pela versão 7.2 da linguagem no que diz respeito à passagem de valoresa métodos através de parâmetros.


## Acesso `private protected`

O novo tipo de acesso `private protected` pode ser aplicado a um membro de um tipo e indica que esse membro pode ser acedido pela própria classe ou por outras classes derivadas que tenham sido definidas na mesma *assembly* da classe que disponibiliza esse membro. Convém não confundir este tipo de acesso com o acesso `protected internal`. Um membro anotado com este último pode ser acedido por outras classes que tenham sido declaradas na mesma *assembly* que contém o tipo que disponibiliza esse membro.


## Parâmetros nomeados não definidos no final

Com esta alteração, a linguagem passa a permitir o uso de parâmetros nomeados em qualquer lugar, desde que sejam utilizados na sua posição correta. A principal motivação associada a esta nova funcionalidade é reduzir o número de caracteres necessário ao uso de parâmetros nomeados. Com a introdução dos parâmetros nomeados, passou a ser normal recorrer a este tipo de parâmetros para identificar eventuais valores literais passados a um método. Por exemplo, suponhamos o exemplo seguinte:

```cs
void ProcessaVencimento(bool chefe, string nome, string idade){
    //...
}
```

Antes da introdução desta nova funcionalidade, teríamos de escrever o código seguinte se quisermos processar o vencimento de um trabalhador não chefe:

```cs
void FazQqCoisa()
{
    var nome = "Luis";
    var idade = 30;
    ProcessaVencimento( chefe: false, nome: nome, idade: idade);
}
```

A partir desta altura, podemos escrever simplesmente o seguinte:

```cs
void FazQqCoisa()
{
    var nome = "Luis";
    var idade = 30;
    ProcessaVencimento( chefe: false, nome, idade);
}
```

Neste exemplo, recorremos a um parâmetro nomeado para identificar claramente o objetivo do valor do tipo `bool` que é passado ao método `ProcessaVencimento`. A partir de C# 7.2 e uma vez que as variáveis utilizadas na inicialização dos restantes parâmetros possuem nomes "claros", então podemos passá-las através de parâmetros posicionais (conforme ilustrado no exemplo anterior).

Com esta nova funcionalidade, a regra que proíbia a utilização de parâmetros posicionais após parâmetros nomeados foi abolida e, às regras definidas no ponto 7.5.1.1 da especificação, foi adicionada uma nova que diz o seguinte:

> An unnamed argument corresponds to no parameter when it is after an out-of-position named argument or a named params argument.

O objetivo desta nova regra é impedir a invocação de métodos como os apresentados no exemplo seguinte:

```cs
void DoIt(bool a = true, bool b = true, bool c = true) 
{

}
var valorB = false;
DoIt(c: false, valorB); //oops
```

O primeiro argumento identifica o valor passado ao terceiro parâmetro (portanto, está colocado "fora da sua posição normal"), pelo que o seguinte não pode ser um parâmetro posicional.

Esta nova alteração pode introduzir algumas questões na invocação de métodos que possuem *overloads*. Por exemplo, analisemos as declarações seguintes:

```cs
void M(int x, int y) 
{ 

}
void M<T>(T y, int x) 
{ 

}
void M2()
{
    M(3, 4);
    M(y: 3, x: 4); //M(int, int)
    M(y: 3, 4); //  M<T>(T, int)
}
```

No exemplo anterior, podem restar algumas dúvidas quanto às últimas duas invocações. Contudo, a adição da regra mencionada anterior torna claro qual o método que deve ser invocado.


## Conclusão

Ao longo deste capítulo, apresentámos as principais caraterísticas associadas ao uso do novo qualificador de acesso `private protected` e à utilização de parâmetros nomeados em funções. Como vimos, o novo qualificador veio colmatar uma lacuna existente na linguagem C#. Assim, a partir da versão 7.2, já é possível criar membros compativeis com o conceito de *ProtectedAndInternal* suportado pelo CLR. Com a remoção da restrição de colocação de parâmetros nomeados no final da lista de argumentos, a linguagem simplifica o código necessário em casos onde apenas estamos interessados em identificar um ou outro parâmetro nomeado com o objetivo de melhorar a legibilidade do nosso código.


### Bibliografia

[private protected](https://github.com/dotnet/csharplang/blob/master/proposals/csharp-7.2/private-protected.md) <br>
[Champion "Non-trailing named arguments"](https://github.com/dotnet/csharplang/issues/570) <br>

[Anterior](10-referencesemanticswithvaluetypes.md) [Índice](index.md)