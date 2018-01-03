# Tipos por valor (*value types*) com comportamento de tipos por referência (*reference types*)

A plataforma .NET sempre dividiu os tipos em duas categorias, normalmente designados por tipos por valor (também conhecidos por *value types*) e por tipos por referência (*reference types*). Cada um possui vantagens e desvantagens. Uma das vantagens inerentes ao uso dos tipos por valor reside no facto de podermos criar elementos deste tipo sem que isso resulte em alocações efetuadas na *heap* (a gestão automática de memória efetuada pelo *GC* pode introduzir algumas penalizações a nível de *performance*). Por outro lado, os elementos deste tipo são amazenados na *stack* e são copiados por valor. Por outras palavras, qualquer atribuição de um elemento deste tipo a uma variável resulta sempre na criação de uma cópia desse elemento. Na prática, isto significa que o seu uso não é recomendado quando, por exemplo, estamos perante algoritmos que efetuam cópias sucessivas de valores, já que este comportamento predefinido pode levar a um esgotamento da *stack* em alguns casos limites). Felizmente para nós, e como veremos em seguida, algumas destas questões são resolvidas com as novidades introduzidas pelo lançamento do C# 7.2.


## O qualificador `in`

Uma das novidades introduzidas pelo C# 7.2 é a passagem de referências de leitura para tipos por valor. Esta nova funcionalidade permite-nos passar valores deste tipo através de referências, que não podem ser modificadas no interior do método que as recebem. Os parâmetros anotados com este qualificador referenciam diretamente o espaço de memória da variável utilizada na sua inicialização (aquando da invocação do método). Nesta altura, o leitor poderá estar a se interrogar acerca da necessidade de introdução deste novo qualificador. Afinal de contas, a passagem por referência já é suportada desde a primeira versão da linguagem, através do uso do qualificador `ref`.

Na verdade, existe uma diferença de comportamento importante associada ao uso deste qualificador (`in`) quando comparado com o qualificador `ref`: como mencionámos, o seu uso impede a modificação do valor referenciado pelo parâmetro no interior do método. Portanto, estamos perante um novo qualificador que complementa os termos reservados `ref` e `out` e que colmata uma lacuna relacionada com a passagem por referência de elementos dos chamados tipos por valor.

Nesta altura, não devem restar dúvidas quanto ao papel do novo qualificador: é usado apenas para passar um parâmetro por referência, não permitindo a sua modificação no interior do método. Para ilustrar o uso deste novo qualificador, vamos começar por introduzir o tipo `Ponto`, caraterizado pelas propriedades `X` e `Y`:

```cs
public struct Ponto
{
    public double X;
    public double Y;
}
```

Vamos ainda supor que temos uma classe auxiliar, com um método designado por `Calcula` que é responsável por calcular a distância entre dois pontos:

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

O método recebe (através de parâmetros) dois elementos do tipo por valor (note-se como `Ponto` é representado através de uma `struct`), sendo que cada um deles necessita de (pelo menos) 16 bytes de espaço (recorde-se que cada `double` ocupa 8 bytes). Portanto, sem o qualificador `in`, cada invocação do método `Calcula` resulta numa alocação de 32 bytes. Neste caso, a realização dessas cópias é, claramente, desnecessária e pode ser omitida através do uso do qualificador `in`. A aplicação deste atributo a estes parâmetros permite.nos consumir menos memória: neste caso, apenas de 8 ou 16 bytes, dependendo este valor do tipo de arquitetura onde o código é executado. Em sistemas de 32 bits, cada apontador é representado por 4 bytes, pelo que o espaço necessário à passagem dos dois parâmetros resume-se apenas a 8 bytes.

No exemplo anterior, os ganhos a nível de espaço são reduzidos. Contudo, a situação mudaria rapidamente de figura se, por exemplo, estivermos perante invocações sucessivas de um método no interior de um ciclo. 

À semelhança do que acontecia com os qualificadores `out`e `ref`, também não podemos definir *overloads* de métodos que diferem apenas na aplicação deste novo qualificador. Como seria de esperar, não podemos aplicar este qualificador a um parâmetro que tenha sido anotado com o qualificador `out` ou `ref`. No que diz respeito à variância que carateriza o uso de genéricos, os parâmetros anotados com o qaulificador `in` são considerados não variantes (*nonvariant*).

Para além de métodos, o qualificador pode ser utilizado em qualquer tipo de membro que espera parâmetros (portanto, este qualificador também pode ser utilizado para anotar parâmetros de *delegates*, expressões *Lambda*, funções locais, *indexers* e operadores).

Como seria de esperar, o compilador é capaz de garantir que os parâmetros anotados com o qualificador `in` não são modificados no interior do método. Isto acontece porque o compilador trata-os de forma semelhante aos campos anotados com o termo `readonly` (que é utilizado apenas na definição de campos de leitura). Para além disso, o compilador impede ainda que um parâmetro anotado com o qualificador `in` seja passado a outro método que espera parâmetros anotados pelos termos `ref` ou `out`. O excerto seguinte ilustra este ponto:

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

Por sua vez, a passagem de um parâmetro anotado com o qualificador `in` a outro método através de um parâmetro que não possua nenhum qualificador aplicado é possível e, como seria de esperar, resulta na criação de uma cópia do valor desse parâmetro:

```cs
public void Testa2(double valor2)
{
    //valor2 foi criado a partir de uma copia de valor
    //...
}
public void Testa(in double valor)
{
    Testa2(valor); //copia do parâmetro criada
}
```

A utilização do qualificador `in` para caraterizar parâmetros que recebem elementos dos chamados tipos por referência é possível, mas não traz grandes benefícios (excluíndo, claro, o facto de aplicação deste qualificador impedir a modificação do valor do parâmetro no interior do método).




## *ref readonly* de tipos por valor

A versão 7.2 da linguagem introduz ainda o conceito de devolução por referência de elementos dos chamados tipo por valor: para isso, temos de anotar o tipo de retorno do membro com os termos `ref readonly`. Qualquer tentativa de modificar um valor anotado com estes elementos é automaticamente detetada pelo compilador e transformada num erro de compilação. Uma vez que o compilador não consegue saber se os membros desse tipo por valor modificam a estrutura, então acaba por criar uma cópia do valor retornado (o que acaba por garantir que o valor devolvido nunca é modificado). 

Regressando ao nosso exemplo baseado no tipo `Ponto`, é bem provável que existam várias operações que necessitem de utilizar o chamado ponto de origem, caraterizado pelas coordenadas (0,0). Nestes casos, podíamos anotar o tipo de retorno de um método ou propriedade utilizada para este objetivo com os termos `ref readonly`:

```cs
public struct Ponto
{
    public double X;
    public double Y;

    private static Ponto _origem = new Ponto();
    public static ref readonly Ponto Origem => ref _origin;
}
```

A partir desta altura, o comportamento associado à recuperação do valor da propriedade dependerá sempre da forma como a variável for declarada:

```cs
var origem = Ponto.Origem;
``` 

No exemplo anterior, a variável `origem` contém uma cópia do valor devolvido pela propriedade estática `Origem`. No excerto seguinte, a utilização dos termos `ref readonly` faz com que `origem2` referencie diretamente o espaço de memória referenciado pelo campo estático `_origem` (devolvido a partir da propriedade `Origem`):

```cs
ref readonly var origem2 = Ponto.Origem;
```

A partir desta altura, qualquer tentativa de modificar o valor referenciado pela variável `origem2` resulta num erro de compilação. 

Nesta altura, o leitor poderá estar a se interrogar acerca das regras que regem os valores que podem ser devolvidos a partir de um membro cujo tipo de retorno foi qualificado pelos termos `ref readonly`. Nesta altura, é possível retornar:
1. referências para variáveis alocadas na *heap*;
2. parâmetros anotados com o qualificador  `in`;
3. parâmetros de saída (`out`);
4. campos de estruturas (`struct`) desde que o recetor também seja seguro para ser devolvido desta forma;
5. um `ref readonly` obtido a partir da invocação de outro método se todos os valores passados aos parâmetros desse método forem seguros para retornar.

Como seria de esperar, o valor `this` não é seguro para devolver a partir de um membro de uma`struct`. Para além disso, os chamados *rvalues* também não podem ser devolvidos a partir deste tipo de membros.