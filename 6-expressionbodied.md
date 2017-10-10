# Membros com corpos definidos por expressões

Desde o C# 6.0, a linguagem permite o recurso a expressões para definir o corpo de alguns dos membros definidos por uma classe. Com a versão 7.0, e como veremos ao longo deste capítulo, o número do tipo de membros que podem ser definidos através de expressões deste tipo aumenta. 


## Introdução

O C# 6 introduziu o conceito de membros com corpo definido por expressão (*xpression-bodied members*). O principal objetivo desta funcionalidade era simplificar o código necessário à definição de funções e propriedades de leitura expostas por um tipo. Normalmente, estas expressões contribuem para simplificar a definição de membros cuja implementação se limita a expressões simples (tipicamente, membros cuja implementação se resume a uma linha de código).

O excerto seguinte mostra como podemos simplificar o código utilizado na definição de uma propriedade de leitura através do uso de um membro deste tipo (propriedade `NomeCompleto`):

```cs
public class Pessoa
{
    public string Nome { get; set; }
    public string UltimoNome { get; set; }
    public string NomeCompleto => $"{Nome} {UltimoNome}";
}
```

Com o C# 7, os tipos de membros cujos corpos podem ser definidos à custa de expressões deste tipo é ampliado. Assim, a partir da versão 7, o uso deste tipo de expressões passa a poder ser aplicado a construtores, finalizadores e aos chamados *getters* e *setters* utilizados na implementação de leitura e escrita do valor de propriedades. No excerto seguinte, alteramos o código do exemplo anterior por forma a impedir a atribuição do valor `null` às propriedades `Nome` e `UltimoNome`:

```cs
public class Pessoa
{
    private string nome = string.Empty;
    private string ultimoNome = string.Empty;
    public string Nome
    { 
        get => this.nome; 
        set => this.nome = value ?? string.Empty;
    }
    public string UltimoNome
    { 
        get=> this.ultimoNome;
        set => this.ultimoNome = value ?? string.Empty;
    }
  public string NomeCompleto => $"{Nome} {UltimoNome}";
}
```

Como é possível verificar, o código necessário foi reduzido ao mínimo, sem que isso implique uma perda de legibilidade. A sintaxe utilizada na definição de construtores e finalizadores é muito semelhante à apresentada nos exemplos anteriores, conforme é possível averiguar a partir do excerto seguinte:

```cs
// constructor
Pessoa(string nome)  => this.nome = nome;
// finalizador
~Pessoa() => Console.WriteLine("Finalizado);
```

Na nossa opinião, o uso deste tipo de expressões na definição deste tipo de membros (construtores e finalizadores) será muito pouco frequente. No que diz respeito aos construtores, são raras as situações onde o construtor recebe apenas um parâmetro. E quando isso acontece, a inicialização do campo interno a partir do valor do parâmetro será, com toda a certeza, precedida de uma ou mais instruções que verificam se os valores passados são, ou não, válidos.

Note-se ainda que recomendações existentes para o uso deste tipo de expressões mantêm-se: apenas devem ser utilizadas apenas quando estamos perante membros constituídos por instruções simples. Nos restantes casos, a utilização desta técnica acaba por contribuir apenas para dificultar a legibilidade do código.

Uma observação final: esta é o primeiro conjunto de funcionalidades disponibilizadas pela linguagem que foram implementadas pela comunidade através de contribuições efetuadas no projeto *open source Roslyn*.


## Conclusão

Este capítulo debruçou-se sobre as novidades relacionadas com o uso de expressões na definição dos membros de um tipo. Como vimos, com o C# 7.0 passamos a definir mais membros através deste tipo de expressões.

No próximo capítulo, vamos apresentar as principais caraterísticas relacionadas com a definição e utilização das chamadas funções locais.


### Bibliografia

["C#7: Throw Expressions and More Expression-bodied Members"](http://blog.somewhatabstract.com/2017/01/16/c7-throw-expressions-and-more-expression-bodied-members/)<br> 