# Tipos por valor (*value types*) com comportamento de tipos por referência (*reference types*)

Desdes os seus primórdios, a plataforma .NET caraterizou sempre os tipos em duas categorias. De um lado, temos os chamados tipos por valor (também conhecidos por *value types*). Do outro, temos os tipos por referência (*reference types*). Como seria de esperar, cada um possui vantagens e desvantagens. Uma das principais vantagens inerentes ao uso dos tipos por valor reside no facto de podermos criar elementos deste tipo sem que isso resulte em alocações efetuadas na *heap*. Como é do conhecimento geral, a gestão automática de memória efetuada pelo *GC* pode introduzir algumas penalizações a nível de *performance*, pelo que a sua não utilização pode, em alguns cenários, melhorar a *performance* de uma aplicação.

Assim, os elementos deste tipo são amazenados na *stack*, que, tradicionalmente, possui uma capacidade bem menor do que a disponibilizada pela *heap* (normalmente, o espaço de armazenamento é de cerca de 1MB). Para além disso, estes elementos são copiados por valor. Por outras palavras, uma simples atribuição ou a passagem de um valor deste tipo a um parâmetro aquando da invocação de um método resulta sempre na duplicação da memória utilizada pelo valor original. Este comportamento predefinido pode introduzir alguns problemas quando estamos perante determinados cenários, como, por exemplo, algoritmos que recursivos que acabam por efetuar cópias sucessivas de valores. Nestes casos, o comportamento predefinido destes tipos rapidamente conduz a um esgotamento da *stack*. Felizmente para nós, e como veremos em seguida, algumas destas questões são resolvidas com as novidades introduzidas pelo lançamento do C# 7.2.


## Parâmetros `in`

Uma das novidades introduzidas pelo C# 7.2 é a passagem de tipos por valor para métodos através de referências de leitura. Como é possível depreender a partir da frase anterior, esta nova funcionalidade permite-nos, a partir do interior de um método, utilizar referências para tipos por valor que lhe são passados aquando da sua invocação. Os valores referenciados por estes parâmetros não podem ser modificados no interior dos métodos. Par que Um parâmetro possua este comportamento, ele tem de ser anotado com o qualificador `in`. Normalmente, estes parâmetros são designados na literatura por parâmetros `in`.

Como referimos, os parâmetros anotados com este qualificador referenciam diretamente o espaço de memória da variável que foi utilizada na sua inicialização aquando da invocação do método. Nesta altura, o leitor poderá estar a se interrogar acerca da necessidade de introdução deste novo qualificador. Afinal de contas, a passagem por referência já é suportada desde a primeira versão da linguagem, nomeadamente através do uso do qualificador `ref`.

Existe, contudo, uma diferença de comportamento importante associada ao uso deste qualificador (`in`) quando comparado com o qualificador `ref`: como mencionámos, o seu uso impede a modificação do valor referenciado pelo parâmetro no interior do método. Portanto, estamos perante um novo qualificador que complementa os termos reservados `ref` e `out` e que veio colmatar uma lacuna relacionada com a forma como podemos passar valores a métodos através de parâmetros. Para ilustrar o uso deste novo qualificador, vamos começar por introduzir o tipo por valor designado por `Ponto` (note-se o uso do termo reservado `struct`), que é caraterizado pelas propriedades `X` e `Y`:

```cs
public struct Ponto
{
    public double X;
    public double Y;
}
```

Vamos ainda supor que temos uma classe auxiliar (`Calculadora`), com um método designado por `Calcula`, cujo papel é calcular a distância entre dois pontos:

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

void Main(){
    var pt1 = new Ponto{ X = 1.0, Y = 2.0 };
    var pt2 = new Ponto{ X = 2.0, Y = 4.0 };
    var dist = Calculadora.Calcula(pt1, pt2);
}
```

O método recebe (através de parâmetros) dois elementos do tipo por valor (uma vez mais, note-se como `Ponto` é representado através de uma `struct`), sendo que cada um deles necessita de (pelo menos) 16 bytes de espaço (um `double` necessita de 8 bytes, pelo que a `struct` necessita de 16 bytes para armazenar os valores `X` e `Y`). Portanto, sem o qualificador `in`, cada invocação do método `Calcula` resulta numa alocação de 32 bytes (resultantes da passagem por valor dos dois parâmetros). Neste caso, a realização dessas cópias não é necessária e pode ser omitida através do uso do qualificador `in`. A aplicação deste atributo a estes parâmetros permite-nos consumir menos memória. Neste caso concreto, necessitamos apenas de 8 ou 16 bytes, dependendo este valor do tipo de arquitetura onde o código é executado. E isto porque a aplicação do qualificador transforma os parâmetros em "apontadores seguros" para os valores que lhe são passados (em sistemas de 32 bits, cada apontador é representado por 4 bytes, pelo que o espaço necessário à passagem dos dois parâmetros resume-se apenas a 8 bytes).

No exemplo anterior, os ganhos a nível de espaço são reduzidos. Contudo, a situação mudaria rapidamente de figura se, por exemplo, estivessemos perante invocações sucessivas de um método no interior de um ciclo. 


### Regras para a utilização do qualificador `in`

Os parâmetros anotados com o qualificador `in` possuem um comportamento muito semelhante aos campos de leitura de uma classe (`readonly`): depois de analisados, os parâmetros deste tipo não podem ser modificados:

```cs
public static double Calcula(in Ponto pt1, in Ponto pt2)
{
    pt = new Ponto(); //oops, erro
    pt1.X = 20; //oops, erro
    var difX = pt2.X - pt1.X;
    var difY = pt2.Y - pt2.Y;
    return Math.Sqrt(difX * difX +difY * difY);
}
```

Para além disso, o compilador impede ainda que um parâmetro anotado com o qualificador `in` seja passado a outro método que define parâmetros anotados com os termos reservados `ref` ou `out`. O excerto seguinte ilustra este ponto:

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

Para além de métodos, o qualificador `in` pode ser utilizado para anotar parâmetros de *delegates*, expressões *Lambda*, funções locais, *indexers* ou operadores.

À semelhança do que acontecia com os qualificadores `out`e `ref`, também não podemos definir *overloads* de métodos que diferem apenas na aplicação deste novo qualificador. Como seria de esperar, também não podemos aplicar este qualificador a um parâmetro que tenha sido anotado com o qualificador `out` ou `ref`. No que diz respeito à variância que carateriza o uso de genéricos, os parâmetros anotados com o qualificador `in` são considerados não variantes (*invariant*).

A utilização do qualificador `in` para caraterizar parâmetros que recebem elementos dos chamados tipos por referência é possível, mas não traz grandes benefícios (excluíndo, claro, o facto de aplicação deste qualificador impedir a modificação do valor do objeto referenciado pelo parâmetro no interior do método).


### Invocação de métodos com parâmetros anotados com o qualificador `in`

A aplicação do termo `in` ao valor passado a um parâmetro `in` aquando da invocação do método não é obrigatória. Note-se, contudo, que a sua aplicação a um valor passado a um parâmetros não `in` resulta num erro de compilação. Para ilustrar este ponto, vamos começar por adicionar um novo método à classe `Calculadora`:

```cs
public static class Calculadora
{
    public static double Calcula2(in Ponto pt1, Ponto pt2)
    {
        return 0;
    }
    public static double Calcula(in Ponto pt1, in Ponto pt2)
    {
        
        var difX = pt2.X - pt1.X;
        var difY = pt2.Y - pt2.Y;
        return Math.Sqrt(difX * difX +difY * difY);
    }
} 
```

Analisemos, agora, o excerto seguinte:

```cs
Calculadora.Calcula(pt1, pt2); //ok, passados por ref de leitura
Calculadora.Calcula2(in pt1, in pt2); //erro, pt2 nao pode ser anotado com in
```

A primeira invocação compila sem qualquer erro e, em *runtime*, resulta na passagem de uma referência (de leitura) dos valores armazenados nas variáveis `pt1` e `pt2`. Por outro lado, o compilador recusa-se a compilar a segunda invocação. Neste caso, o problema reside na aplicação do qualificador `in` ao valor passado através do segundo parâmetro do método `Calcula2`. Esta aplicação não é possível, já que não estamos perante um  parâmetro `in`. 

Outra restrição aplicada aos valores passados aos parâmetros prende-se com o facto de estes parâmetros apenas poderem ser alimentados com os chamados valores do tipo *LValue* (onde o termo *LValue* é utilizado para representar expressões que identificam localizações de memória que podem ser referidas diretamente). Na prática, e supondo que temos um método designado por `Duplica` que recebe um número inteiro através de um parâmetro `in`:

```cs
public static int Duplica(in int x)
{
    return x * 2;
}
```

Então a invocação seguinte resulta num erro de compilação:

```cs
Calculadora.Duplica(in 2); //oops, erro
``` 

Contudo, a instrução seguinte já compila sem qualquer erro:

```cs
var t = 2;
Calculadora.Duplica(in t); //ok, t e um lvalue; nao e necessario usar in
```

Para além de variáveis, os parâmetros *in* podem ainda ser inicializados com campos de leitura (`readonly`). Finalmente, importa ainda referir que os valores passados a parâmetros *in* devem possuir tipos com identidades conversíveis (*identity-convertible*) na dos tipos dos parâmetros que alimentam. Por exemplo, suponhamos que estamos perante um método genérico que espera receber um objeto através de um parâmetro in. Nestes casos, os valores passados têm de poder ser convertidos no tipo do parâmetro:

```cs
public static void Generico<T>(in T obj){}

Calculadora.Generico<object>(in Guid.Empty); //oops
```

No exemplo anterior, `Guid.Empty` produz um tipo cuja identidade não é convertível no tipo `object`. Ao detetar este problema, o compilador acaba por sinalizá-lo através da geracão de um erro de compilação. Estas regras foram introduzidas para garantir a passagem segura de uma referência direta para o espaço de memória do valor passado através de parâmetro ao método aquando da sua invocação. 

Antes de terminarmos esta secção, resta-nos referir que, se ainda o desejarmos, podemos alimentar parâmetros *in* com valores "literais" (desde que estes não sejam anotados com o qualificador `in`):

O mesmo acontece com a invocação seguinte:

```cs
Calculadora.Duplica(in 2); //oops
Calculadora.Duplica(2); //ok
```

A passagem de um valor literal ao método `Duplica` segue os mesmos princípios de uma passagem por valor tradicional. Ao encontrar código semelhante ao anterior, o compilador efetua uma transformação semelhante à apresentada no excerto seguinte:

```cs
var aux = 2;
Calculadora.Duplica(in aux); //ok
```

Portanto, o compilador começa por criar uma variável que é inicializada com uma cópia do valor 2. Em seguida, essa cópia é passada através de uma referência de leitura ao método `Duplica`. Portanto, a passagem do valor literal não invalida nenhuma das regras apresentadas até ao momento, já que, na realidade, é transformada pelo num *lvalue* que, por sua vez, é usado para inicializar o parâmetro aquando da invocação do método. Na realidade, ao permitir a passagem direta do valor literal, o compilador acaba apenas por nos poupar algum trabalho extra.

A discussão anterior permite-nos concluir que  criação automática de variáveis temporárias é necessária em alguns casos para garantir que os parâmetros anotados com o qualificador `in` recebem sempre *lvalues*. Para além do cenário anterior (onde um valor literal foi passado ao método sem ser anotado com o qualificador `in`), a aplicação de um valor predefinido a um parâmetros deste tipo também pode resultar na criação de um valor temporário:

```cs
public static int Duplica(in int x  = 2)
{
    return x * 2;
}

Calculadora.Duplica(); //criada var temporaria inicializada com valor 2
```

No que diz respeito à captura de parâmetros caraterística dos métodos assíncronos e das expressões *Lambda*, o comportamento é precisamente o mesmo que carateriza o uso dos qualificadores `ref` e `out`. Portanto, estes parâmetros:
* não podem ser capturados nas *closures*;
* não podem ser utilizados em *iterators*;
* não são permitidos em métodos assíncronos (`async/await`). 


## *ref readonly* de tipos por valor

A versão 7.2 da linguagem introduz ainda o conceito de devolução por referência de elementos dos chamados tipo por valor: para isso, temos de anotar o tipo de retorno do membro com os termos `ref readonly`. Qualquer tentativa de modificar um valor devolvido por um método cujo tipo de retorno tenha sido anotado com estes qualificadores é automaticamente detetada pelo compilador e transformada em erro de compilação. 


### Devolução de valores a partir de membros anotados com ´ref readonly`

Regressando ao nosso exemplo baseado no tipo `Ponto`, é bem provável que existam várias operações que necessitem de utilizar o chamado ponto de origem, caraterizado pelas coordenadas (0,0). A utilização de uma propriedade que devolve uma referência de leitura para um campo interno é uma boa solução para este cenário. Portanto, vamos adicionar à nossa classe uma nova propriedade designada por `Origem` e anotá-la com os termos `ref readonly`:

```cs
public struct Ponto
{
    public double X;
    public double Y;

    private static Ponto _origem = new Ponto();
    public static ref readonly Ponto Origem => ref _origem;
}
```

Neste exemplo, a propriedade `Origem` devolve uma referência de leitura para o campo privado `_origem`. O leitor atento reparou, com toda a certeza, que o valor devolvido a partir da propriedade é anotado apenas com o termo `ref` e não com os termos `ref readonly`. A equipa de desenho concluíu  que o uso dos termos `ref readonly` aos valores devolvidos resultavam na criação de expressões longas, que acabavam por dificultar a leitura. Uma vez que o contexto de leitura (`readonly`) pode ser obtido a partir da análise da assinatura do membro e como nestes casos o valor devolvido nunca pode ser um valor local ao membro, então decidiu-se utilizar exatamente a mesma sintaxe que é utilizada na devolução de [valores por referência] (3-refs.md).

No que diz respeito aos valores devolvidos a partir de membros, as regras que os regem são semelhantes às que regem os valores devolvidos por referência (e que foram descritos detalhadamente no [capítulo 3(3-refs.md)]). A lista seguinte apresenta as regras que regem os valores que podem ser devolvidos por membros que recorrem aos qualificadores `ref readonly`:
1. referências para variáveis alocadas na *heap*;
2. parâmetros anotados com o qualificador  `in`;
3. parâmetros de saída (`out`);
4. campos de estruturas (`struct`) desde que o recetor também seja seguro para ser devolvido desta forma;
5. um `ref` obtido a partir da invocação de outro método se todos os valores passados aos parâmetros `ref`/`out` desse método também forem seguros para retornar.

Como seria de esperar, o valor `this` não é seguro para devolver a partir de um membro de uma`struct`. Para além disso, os chamados *rvalues* também não podem ser devolvidos a partir deste tipo de membros.


### Invocação de membros que anotados com `ref readonly`

A sintaxe utilizada na atribuição de um valor devolvido por um membro anotado com o `ref readonly` define o comportamento dessa expressão. Voltando ao nosso exemplo inicial, o comportamento associado à recuperação do valor da propriedade `Origem` dependerá sempre da forma como a variável for inicializada:

```cs
var origem = Ponto.Origem;
``` 

No exemplo anterior, a variável `origem` contém uma **cópia** do valor devolvido pela propriedade estática `Origem`. Por outras palavras, a variável `origem` contém uma cópia do valor retornado pela propriedade `Origem`. No excerto seguinte, este comportamento é alterado através da aplicação dos termos `ref readonly` e `ref`. Neste caso, `origem2` referencia diretamente o espaço de memória referenciado pelo campo estático `_origem` (que, por sua vez, foi devolvido a partir da propriedade `Origem`):

```cs
ref readonly var origem2 = ref Ponto.Origem;
```

A partir desta altura, qualquer tentativa de modificar o valor referenciado pela variável `origem2` (ou qualquer um dos campos da `struct` - ex.: `origem2.X = 20;`) resulta num erro de compilação. 


# Tipo `readonly struct`

Como vimos, a devolução de referências de leitura para elementos do tipo por valor é possibilitada pelo uso de `ref readonly` aos tipos de retorno de um membro. Contudo, a criação explícita de métodos para garantir o uso de referências de leitura para elementos destes tipo pode não fazer sentido. Foi a pensar neste (e noutros cenários) que a equipa decidiu permitir a aplicação do termo `readonly` à definição de uma `struct`.

Quando aplicamos este termo a uma `struct`, o compilador garante que todos os seus membros passam a ser de leitura. Por outras palavras, ao aplicarmos este termo, temos a garantia de que a `struct` passa a ser imutável. A aplicação deste termo permite ainda outras otimizações. Por exemplo, podemos aplicar o qualificador `in` em todos os locais onde uma `struct`deste tipo for passada a um método através de um parâmetro. Para além disso, o próprio compilador gera código mais eficiente no acesso aos membros de uma estrutura deste tipo: nestes casos, o valor `this` é mesmo passado por referência ao membro através de um parâmetro *in* (em vez de ser criada uma cópia como acontece normalmente).

Para ilustrarmos a criação deste tipo de estrutura, vamos modificar o tipo `Ponto` que foi introduzido na secção anterior:

```cs
public readonly struct Ponto
{
	public readonly double X;
	public readonly double Y;

	private static Ponto _origem = new Ponto();
	public static ref readonly Ponto Origem => ref _origem;
}
```

Como é possível verifivar, a aplicação deste qualificador a uma `struct`tem algumas implicações. Uma delas (que, aliás, podemos ver no exemplo anterior) passa pela necessidade de todos os campos terem de ser definidos como campos de leitura (note-se o uso do qualificador `readonly` na definição dos campos). Eventuais auto-propriedades que venham a ser definidas em `struct` deste tipo também têm de ser de leitura apenas e não podem conter os chamados *field-like events* (declarados através do termo `event`).


# Tipo `ref struct`

Este novo tipo permite-nos criar um tipo por valor que apenas deve ser alocado na *stack*. Por outras palavras, estes tipos não podem ser definidos como membros dos chamados tipos por referência. A principal razão para a introdução deste novo qualificador foi a introdução do novo tipo [Span<T>](https://blogs.msdn.microsoft.com/dotnet/2017/11/15/welcome-to-c-7-2-and-span/), que promete revolucionar a forma como trabalhamos com porções de memória em  .NET. Internamente, este novo tipo contém apenas dois membros: um apontador e um campo que indica o tamanho da memória gerida pelo *span* (na realidade, este tipo é implementado de forma "mágica", já que a linguagem não permite o uso de apontadores em contextos seguros - isto é, fora de blocos *unsafe*). Ao restringirmos  este elemento à *stack*, passamos a evitar alguns erros problemáticos (ex.: *out od range*, violações de *type safety*, ext.).

A introdução deste novo tipo traz consigo um conjunto de novas regras que contribuem para que o seu uso seja seguro:
* um elemento deste tipo não está sujeito a operações de *boxing*;
* estes elementos não podem ser definidos como membros de tipos por referência ou de tipos por valor "regulares";
* estes elementos não podem ser definidos como variáveis no interior de métodos assíncronos, expressões *lambda*, funções locais ou *iterators*.


## Conclusão

Este capítulo dedicou-se à análise de algumas das novas funcionalidades introduzidas pelo C# 7.2 que nos permitem utilizar a referências para tipos por valor no código C# que escrevemos. Como foi possível verificar, estas novas funcionalidades introduzidas pelo C# 7.2 complementam as introduzidas pela versão 7.1. Portanto, mantém-se a tónica de melhorar a linguagem por forma a otimizar o código escrito em situações onde é necessário reduzir as alocações por questões de performance.

No próximo capítulo, continuamos a analisar as novidades introduzidas pelo C# 7.2 e vamos ver como a versão 7.2 permite o uso de novos modificadores de acesso e define novas regras para inicialização dos valores dos parâmetros passados a um método.


### Bibliografia
["What's new in C# 7.2"](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-2)
["Reference semantics with value types"](https://docs.microsoft.com/en-us/dotnet/csharp/reference-semantics-with-value-types
)
["Readonly references"](https://github.com/dotnet/csharplang/blob/master/proposals/csharp-7.2/readonly-ref.md)