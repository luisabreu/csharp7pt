# Tipos por valor (*value types*) com comportamento de tipos por referência (*reference types*)

A plataforma .NET sempre classificou os tipos em tipos por valor (também conhecidos por *value types*) e tipos por referência (*reference types*). Cada um possuía vantagens e desvantagens. Uma das vantagens inerentes ao uso dos tipos por valor reside no facto de podermos criar elementos deste tipo sem que isso resulte em alocações efetuadas na *heap*. Por outro lado, eles são copiados por valor, o que, na prática, significa que o seu uso não é recomendado quando, por exemplo, estamos perante determinados tipos de algoritmos. Felizmente para nós, e como veremos em seguida, algumas destas questões são resolvidas com o lançamento do C# 7.2.


## O qualificador `in`

Uma das novidades introduzidas pelo C# 7.2 é a passagem por referência de tipos por valor. Esta nova funcionalidade permite-nos passarmos valores do tipo por valor através de referências, que acedem diretamente ao espaço de memória utilizado no armazenamento destes valores. Nesta altura, o leitor poderá estar a se interrogar acerca da necessidade de introdução deste novo qualificador (afinal de contas, a passagem por referência já é suportada desde a primeira versão da linguagem, através do uso do qualificador `ref`). 

Na verdade, existe uma diferença de comportamento importante associada ao uso deste qualificador (`in`) quando comparado com o qualificador (`ref`): para além de ser passado por referência, o uso do novo qualificador `in` impede a modificação do valor passado no interior do método. Portanto, estamos perante um novo qualificador que complementa os termos reservados `ref` e `out` e que nos permitem ter um vocabulário completo para expressar as nossas intenções.

Assim, a definição de parâmetros de tipo por valor sem qualquer qualificador resulta sempre na realização de uma cópia do valor aquando da execução do método. A alteração deste comportamento predefinido é possível através do uso dos seguintes qualificadores:
* `out`: neste caso, o método invocado pode alterar o valor do parâmetro;
* `ref`: neste caso, o método pode alterar o valor do parâmetro;
* `in`: neste caso, o parâmetro do tipo por valor é passado por referência ao método, mas não pode ser modificado no seu interior.

Nesta altura, não devem restar dúvidas quanto ao papel do novo qualificador: é usado apenas para passar um parâmetro por referência, não permitindo a sua modificação no interior do método. Para ilustrar o uso deste novo qualificador, vamos começar por introduzir o tipo `Ponto`, caraterizado pelas propriedades `X` e `Y`:

```cs
public struct Ponto
{
    public double X;
    public double Y;
}
```
Vamos ainda supor que temos uma classe auxiliar, com um método designado por `Calcula` que é usado para calcular a distância entre dois pontos:

```cs
public static class Calculadora
{
    public static double Calcula(in Ponto pt1, in Ponto pt2)
    {
        var difX = pt2.X - pt1.X;
        var difY = pt2.Y - pt2.Y;
        return Math.Sqrt(difX * difX +difY * difY);
    }
} 

Ponto pt1 = new Ponto{ X = 1.0, Y = 2.0 };
Ponto pt2 = new Ponto{ X = 2.0, Y = 4.0 };
double dist = Calculadora.Calcula(pt1, pt2);
```

O método recebe dois valores do tipo por valor (note-se como `Ponto` é representado através de uma `struct`), sendo que cada um deles necessita de (pelo menos) 16 bytes de espaço (cada `double` ocupa 8 bytes). Portanto, sem o qualificador `in`, cada invocação do método `Calcula` resultava numa alocação de 32 bytes. Neste caso, a realização dessas cópias é, claramente, desnecessária e pode ser omitida através do uso do qualificador `in`. A partir da sua aplicação, a invocação do método necessita apenas de 8 ou 16 bytes, dependendo este valor do tipo de arquitetura onde o código é executado. Em sistemas de 32 bits, cada apontador é representado por 4 bytes, pelo que o espaço necessário à passagem dos dois parâmetros se resume a 8 bytes.

No exemplo anterior, os ganhos a nível de espaço não são muitos. Contudo, a situação rapidamente muda de figura quando, por exemplo, estamos perante invocações sucessivas no interior de um ciclo. 

À semelhança do que acontecia com os qualificadores `out`e `ref`, também não podemos definir *overloads* de métodos que diferem apenas na aplicação deste novo qualificador. Por outro lado, e ao contrário do que acontecia com os qualificadores `out` e `ref`, o novo qualificador `in` não necessita de ser utilizado aquando da invocação do método e pode mesmo ser utilizado quando os valores dos parâmetros são alimentados a partir de valores literais ou de constantes.

Para além de métodos, o qualificador pode ser utilizado em qualquer tipo de membro que espera parâmetros (portanto, este qualificador também pode ser utilizado para anotar parâmetros de *delegates*, expressões *Lambda*, funções locais, *indexers* e operadores).

Como seria de esperar, o compilador é capaz de garantir que os parâmetros anotados com o qualificador `in` não são modificados no interior do método. Para isso, trata estes parâmetros como se tivessem sido anotados com o qualificador `readonly` (que é utilizado apenas na definição de campos de leitura). Para além disso, o compilador impede ainda que um parâmetro de um método anotado com o qualificador `in` seja, a partir do seu interior, passado a outro método que espera parâmetros anotados pelos termos `ref` ou `out`. O excerto seguinte ilustra este ponto:

```cs
public void Testa2(ref double valor)
{
    //...
}
public void Testa(in double valor)
{
    Testa2( ref valor); //erro!
}
```

A passagem de um parâmetro anotado com o qualificador `in` a um outro método que não possui nenhum qualificador aplicado resulta na criação de uma cópia do valor desse parâmetro:

```cs
public void Testa2(double valor)
{
    //...
}
public void Testa(in double valor)
{
    Testa2(valor); //copia do parâmetro criada
}
```

