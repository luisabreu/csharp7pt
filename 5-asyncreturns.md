# Tipos de retorno de métodos assíncronos

Os métodos assíncronos revolucionaram a forma como escrevemos código assíncrono em C#. Com a versão 7.0, o programador vê a sua vida melhorada através do levantamento de algumas das restrições que se aplicam a este tipo de métodos e à introdução de novos tipos genéricos que tiram partido do levantamento dessas restrições.


## Introdução

Com o C# 5.0, a linguagem passou a suportar o conceito de método assíncrono. Os métodos deste tipo simplificam a criação e invocação de métodos que realizam tarefas assíncronas. Para nos apercebermos das vantagens inerentes ao uso destes métodos, vamos "recuar" até à versão 4.0 da plataforma .NET para compararmos o código de hoje em dia com o que tínhamos de escrever nessa altura. 

Com o lançamento da versão 4.0 da plataforma .NET, o tipo `Task` passou a assumir um papel fulcral na escrita de código assíncrono, substituíndo mesmo o uso direto de objetos do tipo `Thread` na maioria dos cenários do dia-a-dia. O exemplo seguinte mostra como podemos recorrer a este tipo para criarmos um novo método que efetua uma tarefa assíncrona:

```cs
public Task<int> EfetuaCalculoAssincronoAsync()
{
    // devolver uma tarefa "concluida"
    return Task.FromResult(0);
} 
```

Antes do lançamento do C# 5.0, o consumo típico deste tipo de API passava pelo uso das chamadas continuações. Uma continuação pode ser vista como um método que define as operações que devem ser executadas quando a tarefa assíncrona terminar. O excerto seguinte tenta ilustrar o código utilizado nestes cenários:

```cs
var tarefa = EfetuaCalculoAssincronoAsync();
tarefa.ContinueWith(val => Console.WriteLine(val.Result),
        TaskContinuationOptions.OnlyOnRanToCompletion );
```

Neste exemplo, definimos apenas uma continuação, que é invocada apenas quando a tarefa assíncrona é realizada com sucesso (neste exemplo, não nos vamos preocupar com eventuais cenários de erro).

Com o C# 5, a linguagem passou a suportar os chamados métodos assíncronos. Esta nova funcionalidade contribuiu para simplificar o código usado quando necessitamos de suportar a realização de tarefas assíncronas, fazendo mesmo com que o código utilizado seja muito semelhante ao código síncrono que estamos habituados a ver e a escrever no dia-a-dia. 
Um método "transforma-se" em assíncrono quando o seu tipo de retorno é anotado com o termo `async`. No interior destes métodos, podemos recorrer ao termo `await` sempre que necessitarmos de efetuar uma chamada assíncrona. O exemplo seguinte ilustra estes princípios:

```cs
public async Task<int> ExecutaAsync() 
{
    try 
    {
        var res = await EfetuaCalculoAssincronoAsync();
    }
    catch (Exception ex) 
    {
        Console.WriteLine(ex.ToString());
    }
    return 0;
}
```

Para podermos ilustrar esta funcionalidade, tivemos de colocar as instruções que efetuam a chamada do método assíncrono `EfetuaCalculoAssincronoAsync` no interior de um outro método. Como é possível verificar, o código apresentado no excerto anterior assemelha-se muito ao código típico que escrevemos quando estamos perante operações síncronas. Aliás, as únicas diferenças "visíveis" residem mesmo no uso dos termos `async` e `await` . Do ponto de vista da legibilidade, o código apresentado no exemplo anterior é de muito mais fácil compreensão do que o baseado no uso de continuações explicitas (exemplo apresentado anteriormente). E isto porque a utilização de métodos assíncronos permite-nos delegar todo o trabalho relacionado com a definição de continuações (e com o tratamento de eventuais exceções - cenário este que não foi apresentado no exemplo anterior) no compilador.

Até ao lançamento do C# 7, os valores devolvidos a partir de um método assíncrono estavam limitados aos tipos `Task`, `Task<T>` ou `void` (a utilização deste último valor em métodos deste tipo deve ser feito com muita precaução). Os tipos utilizados em expressões de espera (expressões `await`) também estavam sujeitos a regras que documentam o seu comportamento. Assim, um tipo só pode ser utilizado neste tipo de expressões se implementar o chamado padrão *awaitable*. Na prática, um objeto implementa este padrão se o seu tipo disponibilizar um método (de instância ou de extensão) designado por `GetAwaiter` que, por sua vez, devolve um objeto que implementa a interface `INotifyCompletion` ou `ICriticalNotifyCompletion`. Para além de implementar uma (ou ambas) das interfaces anteriores, este objeto deve ainda disponibilizar:
1.	Uma propriedade designada por `IsCompleted`, que indica se a tarefa assíncrona foi, ou não, concluída;
2.	Um método denominado por `GetResult`, que pode ser utilizado para recuperar o resultado obtido a partir da execução da tarefa assíncrona.

A partir da descrição anterior, facilmente concluímos que a linguagem era um pouco restrita no que diz respeito à definição do tipo de retorno de um método assíncrono. Apesar das vantagens claras inerentes ao uso deste tipo de métodos, a verdade é que a obrigatoriedade de devolução de objetos do tipo `Task` ou `Task<T>` pode acabar por introduzir alguns problemas ao nível da performance devido ao facto de estes tipos serem considerados tipos por referência (*reference types*). Como é do conhecimento geral, o uso de objetos destes tipos podem introduzir alguma penalização na performance de um sistema devido à necessidade de gestão de memória associada ao seu uso (estes objetos são mantidos em blocos de memóra da heap, cuja "limpeza" fica a cargo do chamado *Garbadge Collector*).

Este tipo de problemas tem tendência a agravar-se em alguns cenários. Devolução de valores calculados previamente (e, entretanto, mantidos em *cache*) e obtenção síncrona de valores são exemplos de dois cenários onde a criação de um objeto do tipo `Task` ou `Task<T>` (obrigatória até ao momento neste tipo de métodos!) acaba por ser desnecessária. As penalizações associadas à gestão destes objetos podem piorar ainda mais quando, por exemplo, um método assíncrono que devolve um objeto desse tipo for invocado repetidamente (ex.: no interior de um ciclo).

Com o C# 7, a restrição aplicada ao tipo de retorno dos métodos assíncronos foi alterada, fazendo, assim, com que o tipo de retorno destes métodos deixe de estar limitado ao uso dos tipos `Task`, `Task<T>` ou `void`. Assim, a partir do C# 7, a linguagem passa a suportar o novo conceito de tipo `Task` (*task-like types*), que é utilizado para agrupar os tipos que podem ser utilizados como tipo de retorno de um método assíncrono. Estes tipos podem ser definidos através de classes (`class`) ou estruturas (`struct`) e devem estar associados a um construtor de método assíncrono, que é identificado através do atributo `AsyncMethodBuilderAttribute` que é aplicado ao tipo. Como seria de esperar, para que o tipo usado como tipo retorno de um método assíncrono seja utilizável na expressão `await`, ele tem ainda de implementar o padrão * awaitable*  (que foi descrito no parágrafo anterior). No excerto seguinte, introduzimos um exemplo de um novo tipo que pode ser utilizador como retorno de método assíncrono:

```cs
[AsyncMethodBuilder(typeof(ConstrutorMetodoAssincrono<>))]
class Tarefa<T>
{
    public Espera<T> GetAwaiter();
}
class Espera<T> : INotifyCompletion
{
    public bool IsCompleted { get; }
    public T GetResult();
    public void OnCompleted(Action completion);
}
```

No excerto anterior, começamos por introduzir um novo tipo, designado por `Tarefa<T>`, que pode ser usado como tipo de retorno de um método assíncrono. Como mencionámos, o tipo tem de implementar o padrão *awaitable*  para poder ser utilizado numa expressão de `await`. Neste caso, o método `GetAwaiter` devolve um objeto do tipo `Espera<T>`, que se limita a implementar a interface `INotifyCompletion` e os métodos `IsCompleted` e `GetResult`.

Os métodos assíncronos são transformados pelo compilador numa máquina de estados, que é representada programaticamente por uma classe que implementa a interface `IAsyncStateMachine` (comportamento este que é muito semelhante ao que ocorre quando o compilador está perante iteradores). Esta transformação (numa máquina de estados) fica a cargo de um outro tipo, normalmente designado por construtor de método. O construtor de método assíncrono utilizado é definido pelo programador através da aplicação do atributo `AsyncMethodBuilderAttribute` ao tipo de retorno. Como é possível verificar através do exemplo anterior, este atributo recebe (através de parâmetro) o tipo que deve ser utilizado como construtor de método assíncrono (no exemplo anterior, a transformação numa máquina de estados de um método assíncrono que devolva um objeto do tipo `Tarefa<T>` fica a cargo de uma instância do tipo `ConstrutorMetodoAssincrono`).

Um construtor de método assíncrono não tem de derivar de uma classe base predefinida ou de implementar uma interface predefinida, mas tem de respeitar o contrato definido neste exemplo:

```cs
class ConstrutorMetodoAssincrono<T>
{
    public static MyTaskMethodBuilder<T> Create();
    public void Start<TStateMachine>(ref TStateMachine stateMachine)
        where TStateMachine : IAsyncStateMachine;
    public void SetStateMachine(IAsyncStateMachine stateMachine);
    public void SetException(Exception exception);
    public void SetResult(T result);
    public void AwaitOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : INotifyCompletion
        where TStateMachine : IAsyncStateMachine;
    public void AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : ICriticalNotifyCompletion
        where TStateMachine : IAsyncStateMachine;
    public MyTask<T> Task { get; }
}
```

O construtor de método assíncrono responsável pela transformação numa máquina de estados também pode ser definido à custa uma estrutura (`struct`). Neste caso, contudo, optámos por recorrer a uma classe para criar o construtor de método assíncrono. Note-se ainda que quando estamos perante tipos não genéricos, o método `SetResult` não possui parâmetros.

O método estático `Create` do construtor da máquina de estados é invocado quando é necessário criar uma nova máquina de estados. Se a máquina de estados utilizada for implementada através de uma estrutura (`struct`), então o método de instância `SetStateMachine` receberá uma referência para um objeto com o *boxed value* da estrutura que representa essa máquina de estados (a invocação deste método permite ao construtor de método assíncronos manter uma referência para a máquina de estados em `cache`).

O método `Start` é utilizado para associar o construtor de método assíncrono a uma instância inicializada da máquina de estados que é gerada pelo compilador. No interior deste método (ou imediatamente após o seu final), o construtor de método assíncrono deve invocar o método `MoveNext` (sobre a instância da máquina de estados). Esta invocação é necessária para fazer avançar a máquina de estados que controla a execução do método assíncrono. Após o método `Start` terminar, o método assíncrono obtém uma referência para a tarefa assíncrona devolvida pela propriedade `Task` do construtor de método assíncrono. Esta tarefa é utilizada como valor devolvido do método assíncrono.

Internamente, a máquina de estados vai transitando entre os vários estados internos até concluir a sua execução. O final da execução controlada pela máquina de estados é sinalizado através da execução de um de dois métodos do construtor de método assíncronos: `SetResult` ou `SetException`. O primeiro é invocado quando a máquina de estados conclui todas as transições internas com sucesso. Por outro lado, o segundo será invocado quando for gerada uma exceção durante as transições internas da máquina de estados.

Quando a máquina de estados atinge o estado associado à expressão `await`, o método `GetAwaiter` do tipo de retorno da tarefa é invocado. Se este tipo implementar a interface `ICriticalNotifyCompletion` e a propriedade `IsCompleted` devolver o valor `false`, então o método `AwaitUnsafeOnCompleted` do construtor de método assíncronos acabará por ser invocado. Este método é responsável por executar o método `OnComplete` disponibilizado pelo tipo de retorno da tarefa, passando-lhe uma `Action` que, por sua vez, invoca o método da máquina de estados. O comportamento descrito neste parágrafo é semelhante quando o tipo de retorno do método assíncrono implementa a interface `INotifyCompletion`, mas, neste caso, o método executado é o método `AwaitOnCompleted` disponibilizado pelo construtor de método assíncronos.

>***Overload* de métodos**<br>
>A introdução novos tipos de retorno de métodos assíncronos obrigou à alteração do comportamento de resolução quando estamos perante overloads de métodos. Assim, a resolução de *overloads* foi alterada para passar a ter em conta os novos tipos deste género. Uma expressão lambda sem tipo de retorno corresponde diretamente a um candidato não genérico de um tipo de retorno de um método assíncrono. Por sua vez, uma expressão *lambda* com um tipo de retorno `T` corresponde diretamente a um candidato que possui um parâmetro genérico de um tipo de retorno de método assíncrono. Se a expressão *lambda* usada não corresponder a nenhum dos candidatos anteriores de um tipo de retorno de método assíncrono, e se existir uma conversão explícita do tipo utilizado num candidato para o outro, então esse candidato é considerado vencedor. Caso contrário, procede-se à evolução recursiva dos tipos `A` e `B` a partir de `Task<A>` e `Task<B>` para obtermos a melhor correspondência. Finalmente, se uma expressão *lambda* não for diretamente convertível num dos candidatos de um tipo *task* mas se um dos candidatos for mais especializado que o outro, então o mais especializado ganha. <br>


## O tipo *ValueTask<T>*

O desejo de obtenção de melhor performance foi o principal catalisador responsável pela introdução do novo conceito de generalização do tipo de retorno de método assíncrono. Na prática, a introdução das novas regras vieram beneficiar o uso do novo tipo `ValueTask<T>`. Este tipo já se rege pelas regras apresentadas nos parágrafos anteriores e pode ser visto como uma união (*discriminated union*) entre os tipos `T` (normalmente obtido à custa de processamentos síncronos) e `Task<T>` (utilizado em processamentos assíncronos). Este tipo é implementado através de uma estrutura (`struct`), o que contribui para reduzir a pressão sobre a memória quando o valor é calculado de forma síncrona. Este tipo é muito útil, especialmente quando estamos perante cenários onde as operações encapsuladas são efetuadas sincronamente na maior parte das situações. 

O excerto seguinte foi retirado da documentação oficial e mostra como podemos utilizar este tipo para devolver um valor imediatamente quando ele existe ou uma tarefa quando ainda não dispomos do valor:

```cs
public ValueTask<int> CachedFunc()     
{
    return (cache) ? 
        new ValueTask<int>(cachedResult) : 
        new ValueTask<int>(LoadCache());
}
private bool cache = false;
private int cachedResult;
private async Task<int> LoadCache()     
{
    // simulate async work:
    await Task.Delay(100);
    cache = true;
    cachedResult = 100;
    return cacheResult;
}
```

No exemplo anterior, `LoadCache` é um método assíncrono privado, que é responsável por realizar uma tarefa longa (razão pela qual essa operação é efetuada assincronamente). Note-se como o método `CachedFunc` encapsula essa chamada por forma a reutilizar um eventual valor que foi calculado previamente. Quando isso não é possível, então recorremos ao construtor que espera um objeto do tipo `Task<T>`, que, por sua vez, referencia a tarefa que deve ser realizada assincronamente.

Antes da introdução dos novos tipos de retorno de métodos estáticos, teríamos de recorrer sempre ao tipo `Task<int>` para representar o valor devolvido no cenário anterior. Nesses casos, a representação do valor calculado sincronamente obrigava-nos sempre a criar uma nova instância do tipo `Task<int>` (tipicamente obtida a partir da execução do método `Task.FromResult`). A devolução de uma instância deste tipo implicaria sempre uma eventual limpeza de memória (* garbadge collection*) que fica a cargo do GC. Com a introdução das novas regras e do novo tipo `ValueTask<T>`, podemos evitar este uso desnecessário da * heap* (e obter alguns ganhos a nível da performance da aplicação).


## Conclusão

O C# 7.0 introduz algumas novidades interessantes relacionadas com a criação e utilização de métodos assíncronos. Como vimos neste capítulo, foram levantadas algumas restrições e os métodos deste tipo já podem devolver qualquer tipo que seja considerado como *task-like type*. 

No próximo capítulo, analisamos as novidades introduzidas pela linguagem no que diz respeito ao uso de expressões para definir o corpo dos membros de um tipo.


### Bibliografia

["Generalized async return types"](http://blog.somewhatabstract.com/2017/02/06/c7-better-performance-with-ref-locals-and-ref-and-async-returns/) <br>
["Understanding C# async/await compilation"](https://weblogs.asp.net/dixin/understanding-c-sharp-async-await-1-compilation) <br>
["Return Any (Task-Like) Type From An Async Method"](http://blog.i3arnon.com/2016/07/25/arbitrary-async-returns/) <br>
["Async Task Types in C#"](https://github.com/dotnet/roslyn/blob/master/docs/features/task-types.md) 

