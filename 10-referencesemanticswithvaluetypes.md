# Tipos por valor (*value types*) com comportamento de tipos por referência (*reference types*)

Desdes os seus primórdios, a plataforma .NET caraterizou sempre os tipos em duas categorias, normalmente designados por tipos por valor (também conhecidos por *value types*) e por tipos por referência (*reference types*). Como seria de esperar, cada um possui vantagens e desvantagens. Uma das principais vantagens inerentes ao uso dos tipos por valor reside no facto de podermos criar elementos deste tipo sem que isso resulte em alocações efetuadas na *heap* (a gestão automática de memória efetuada pelo *GC* pode introduzir algumas penalizações a nível de *performance*, pelo que a sua não utilização pode, em alguns cenários, conduzir a melhorias a nível da *performance* de uma aplicação).

Por outro lado, os elementos deste tipo são amazenados na *stack* (que, tradicionalmente, possui uma capacidade bem menor do que a disponibilizada pela *heap*) e são copiados por valor. Por outras palavras, qualquer atribuição de um elemento deste tipo a uma outra variável resulta sempre na criação de uma cópia desse elemento (portanto, uma simples atribuição resulta na duplicação completa da memória utilizada pelo valor original). Este comportamento predefinido faz com que o uso de elementos destes tipos não sejam  recomendados em alguns casos. Por exemplo, a utilização de elementos deste tipo quando estamos perante algoritmos que efetuam cópias sucessivas de valores (ex.: invocação recursiva de métodos) pode conduzir a um esgotamento da *stack* em alguns casos. Felizmente para nós, e como veremos em seguida, algumas destas questões são resolvidas com as novidades introduzidas pelo lançamento do C# 7.2.


## O qualificador `in`

Uma das novidades introduzidas pelo C# 7.2 é a passagem de referências de leitura para tipos por valor. Como é possível depreender a partir da frase anterior, esta nova funcionalidade permite-nos passar valores deste tipo através de referências, que não podem ser modificadas no interior do método que as recebem. Um parâmetro que deseja utilizar este comportamento tem de ser anotado com o qualificador `in`.

Os parâmetros anotados com este qualificador referenciam diretamente o espaço de memória da variável utilizada na sua inicialização (aquando da invocação do método). Nesta altura, o leitor poderá estar a se interrogar acerca da necessidade de introdução deste novo qualificador. Afinal de contas, a passagem por referência já é suportada desde a primeira versão da linguagem, nomeadamente através do uso do qualificador `ref`.

Na verdade, existe uma diferença de comportamento importante associada ao uso deste qualificador (`in`) quando comparado com o qualificador `ref`: como mencionámos, o seu uso impede a modificação do valor referenciado pelo parâmetro no interior do método. Portanto, estamos perante um novo qualificador que complementa os termos reservados `ref` e `out` e que veio colmatar uma lacuna relacionada com a forma como podemos passar valores a métodos através de parâmetros.

Nesta altura, não devem restar dúvidas quanto ao papel do novo qualificador: é usado apenas para passar um parâmetro por referência, não permitindo a sua modificação no interior do método. Para ilustrar o uso deste novo qualificador, vamos começar por introduzir o tipo por valor `Ponto` (note-se o uso do termo reservado `struct`), caraterizado pelas propriedades `X` e `Y`:

```cs
public struct Ponto
{
    public double X;
    public double Y;
}
```

Vamos ainda supor que temos uma classe auxiliar (`Calculadora`), com um método designado por `Calcula` que é responsável por calcular a distância entre dois pontos:

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

O método recebe (através de parâmetros) dois elementos do tipo por valor (uma vez mais, note-se como `Ponto` é representado através de uma `struct`), sendo que cada um deles necessita de (pelo menos) 16 bytes de espaço (recorde-se que cada `double` ocupa 8 bytes). Portanto, sem o qualificador `in`, cada invocação do método `Calcula` resulta numa alocação de 32 bytes. Neste caso, a realização dessas cópias não é necessária e pode ser omitida através do uso do qualificador `in`. A aplicação deste atributo a estes parâmetros permite-nos consumir menos memória. Neste caso concreto, necessitamos apenas de 8 ou 16 bytes, dependendo este valor do tipo de arquitetura onde o código é executado (em sistemas de 32 bits, cada apontador é representado por 4 bytes, pelo que o espaço necessário à passagem dos dois parâmetros resume-se apenas a 8 bytes).

No exemplo anterior, os ganhos a nível de espaço são reduzidos. Contudo, a situação mudaria rapidamente de figura se, por exemplo, estivermos perante invocações sucessivas de um método no interior de um ciclo. 


### Regras para a utilização do qualificador `in`

Os parâmetros anotados com o qualificador `in` seguem regras semelhantes às aplicadas aos campos de leitura (`readonly`). Por exemplo, quando este qualificador é aplicado a um parâmetro de um tipo por valor, então todos os campos desse valor são automaticamente transformados em campos de leitura:

```cs
public static double Calcula(in Ponto pt1, in Ponto pt2)
{
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

O qualificador `in` pode ser utilizado em qualquer tipo de membro que espera parâmetros (portanto, este qualificador também pode ser utilizado para anotar parâmetros de *delegates*, expressões *Lambda*, funções locais, *indexers* e operadores).

À semelhança do que acontecia com os qualificadores `out`e `ref`, também não podemos definir *overloads* de métodos que diferem apenas na aplicação deste novo qualificador. Como seria de esperar, também não podemos aplicar este qualificador a um parâmetro que tenha sido anotado com o qualificador `out` ou `ref`. No que diz respeito à variância que carateriza o uso de genéricos, os parâmetros anotados com o qaulificador `in` são considerados não variantes (*invariant*).

A utilização do qualificador `in` para caraterizar parâmetros que recebem elementos dos chamados tipos por referência é possível, mas não traz grandes benefícios (excluíndo, claro, o facto de aplicação deste qualificador impedir a modificação do valor do objeto referenciado pelo parâmetro no interior do método).


### Invocação de métodos com parmâmetros anotados com o qualificador `in`

A aplicação do termo `in` ao valor passado a um parâmetro `in` aquando da invocação não é obrigatória. Note-se, contudo, que a sua aplicação a um valor passado a um parâmetros que não é deste tipo resulta num erro de compilação. Para ilustrar este ponto, vamos começar por adicionar um novo método à classe `Calculadora`:

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
Calculadora.Calcula(pt1, pt2); //ok
Calculadora.Calcula2(in pt1, in pt2); //erro
```

A primeira invocação compila sem qualquer erro e, em *runtime*, resulta na passagem por referência (de leitura) dos valores armazenados nas variáveis `pt1` e `pt2`. Por outro lado, a segunda invocação não pode ser efetuada, já que o compilador recusa-se a compilá-la. Neste caso, o problema reside na aplicação do qualificador `in` ao valor passado ao segundo parâmetro do método `Calcula2`. Esta aplicação não é possível, já que este parâmetro não foi anotado com o qualificador `in`. 

Outra restrição aplicada aos valores passados aos parâmetros prende-se com o facto de apenas ser possível passar os chamados valores do tipo *LValue* (onde *LValue*  refere-se a expressões que representam localizações de memória que podem ser referidas diretamente). Na prática, e supondo que temos um método designado por `Duplica` que recebe um número inteiro através de um parâmetro `in`:

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

No exemplo anterior, `Guid.Empty` produz um tipo cuja identidade não é convertível no tipo `object`. Ao detetar este problema, o compilador acaba por sinalizá-lo através da geracão de um erro de compilação.

As regras anteriores foram introduzidas para garantir a passagem de uma referência direta para o espaço de memória do valor passado através de parâmetro ao método aquando da sua invocação. 

Antes de terminarmos esta secção, resta-nos referir que, se ainda o desejarmos, podemos alimentar parâmetros *in* com valores "literais" (desde que estes não sejam anotados com o qualificador `in`):

O mesmo acontece com a invocação seguinte:

```cs
Calculadora.Duplica(in 2); //oops
Calculadora.Duplica(2); //ok
```

A passagem de um valor literal ao método `Duplica` segue os mesmos princípios de uma passagem por valor tradicional. Ao contrário do que se possa pensar, isto não implica uma alteração à regras apresentadas até ao momento (ou mesmo a adição de uma nova regra). E isto porque, ao encontrar código semelhante ao anterior, o compilador efetua uma transformação semelhante à apresentada no excerto seguinte:

```cs
var aux = 2;
Calculadora.Duplica(in aux); //ok
```

Portanto, o compilador começa por criar uma variável que é inicializada com uma cópia do valor 2. Em seguida, essa cópia é passada por referência de leitura ao método `Duplica`. Portanto, a passagem do valor literal não invalida nenhuma das regras apresentadas até ao momento (note-se como o compilador transforma o valor literal num *lvalue* antes de passá-lo ao método). Na realidade, ao permitir a passagem direta do valor literal, o compilador acaba apenas por nos poupar algum trabalho extra.

A discussão anterior permite-nos concluir que  criação de variáveis temporárias é necessária é alguns casos para garantir que os parâmetros anotados com o qualificador `in` recebem sempre *lvalues*. Para além do cenário anterior (onde um valor literal foi passado ao método sem ser anotado com o qualificador `in`), a aplicação de um valor predefinido a um parâmetros deste tipo também pode resultar na criação de um valor temporário:

```cs
public static int Duplica(in int x  = 2)
{
    return x * 2;
}

Calculadora.Duplica(); //criada var temporaria inicializada com valor 2
```

No que diz respeito à captura de parâmetros caraterística dos métodos assíncronos e das expressões *Lambda*, o comportamento é precisamente o mesmo que carateriza o uso dos qualificadores `ref` e `out´. Portanto, estes parâmetros:
* não podem ser capturados nas *closures*;
* não podem ser utilizados em *iterators*;
* não são permitidos em métodos assíncronos (`async/await`). 


## *ref readonly* de tipos por valor

A versão 7.2 da linguagem introduz ainda o conceito de devolução por referência de elementos dos chamados tipo por valor: para isso, temos de anotar o tipo de retorno do membro com os termos `ref readonly`. Qualquer tentativa de modificar um valor devolvido por um método cujo tipo de retorno tenha sido anotado com estes qualificadores é automaticamente detetada pelo compilador e transformada em erro de compilação. 

### Devolução de valores a partir de membros anotados com ´ref readonly`

Regressando ao nosso exemplo baseado no tipo `Ponto`, é bem provável que existam várias operações que necessitem de utilizar o chamado ponto de origem, caraterizado pelas coordenadas (0,0). Nestes casos, podíamos anotar o tipo de retorno de um método ou propriedade utilizada para este objetivo com os termos `ref readonly`:

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

Como vimos, a devolução de referências de leitura para elementos do tipo por valor é possibilitada pelo uso de `ref readonly` aos tipos de retorno de um membro. Contudo, a definição de métodos para garantir o uso de referências de leitura para elementos destes tipo pode não fazer sendto. Foi a pensar neste (e noutros cenários) que a equipa decidiu permitir a aplicação do termo `readonly` à definição de uma `struct`.

Quando aplicamos este termo a uma `struct`, o compilador garante que todos os seus membros passam a ser de leitura. Por outras palavras, ao aplicarmos este termo, temos a garantia de que a `struct` passa a ser imutável. A aplicação deste termo permite ainda outras otimizações. Por exemplo, podemos aplicar o qualificador `in` em todos os locais onde uma `struct`deste tipo for passada a um método através de parâmetro. O tipo `Ponto`introduzido na secção anterior é um bom candidado a esta alteração:

```cs
public readonly struct Ponto
{
	public readonly double X;
	public readonly double Y;

	private static Ponto _origem = new Ponto();
	public static ref readonly Ponto Origem => ref _origem;
}
```

A aplicação deste qualificador a uma `struct`tem algumas implicações. Uma delas (que, aliás, podemos ver no exemplo anterior) passa pela necessidade de todos os campos terem de ser de leitura. Para além disso, eventuais auto-propriedades também têm de ser de leitura apenas e não podem conter os chamados *field-like eventos* (declarados através do termo `event`).