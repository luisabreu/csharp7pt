# Introdução

Desde o início, o C# assumiu sempre o papel oficial de linguagem da plataforma .NET. As novas versões da linguagem têm vindo a contribuir cada vez mais para cimentar esta posição através da introdução de novas funcionalidades que simplificam e reduzem o trabalho relacionado com a criação de aplicações em .NET. Apesar de as funcionalidades introduzidas pela versão 7 não serem tão impressionantes como as disponibilizadas por outras versões, a verdade é que acabaram por introduzir um conjunto de novas funcionalidades que contribuem para aumentar a eficiência do programador em vários cenários.


## O que posso encontrar neste livro?

Neste livro, são apresentadas as principais caraterísticas e funcionalidades introduzidas pela versão 7 da linguagem C#. À semelhança do que aconteceu com o C# 6, as novidades introduzidas pela versão 7 da linguagem estão limitadas a algumas áreas e tentam colmatar algumas das lacunas que foram detetadas ao longo dos últimos anos. 
Este livro apresenta detalhadamente estas novidades, ilustrando-as sempre com vários exemplos práticos que demonstram as vantagens e ganhos inerentes ao seu uso.


## Requisitos

O livro foi escrito para permitir que a aprendizagem das novas funcionalidades da linguagem possa ser feita sem que o leitor tenha de estar sentado ao pé de um computador. Se o leitor assim o desejar, poderá testar os excertos apresentados. Para isso, poderá, por exemplo, recorrer ao [*LINQPad*](https://www.linqpad.net/), um utilitário que, apesar do que se possa depreender a partir do seu nome, não está limitado à escrita de expressões LINQ (*Language Integrated Query*). Na realidade, podemos recorrer a este utilitário para escrever qualquer expressão, instrução ou programa em C#, VB.NET ou F#.
Alternativamente, o leitor poderá ainda recorrer ao *Visual Studio*, nomeadamente, à versão gratuita *Community*, que pode ser obtida a partir do site da [*Microsoft*](https://www.visualstudio.com/downloads/). A partir deste IDE, o leitor pode executar os excertos de código apresentados ao longo do livro através da janela interativa do C# (*C# Interactive Window*) ou através da criação de aplicações do tipo [*Consola*](https://msdn.microsoft.com/library/0wc2kk78.aspx). 
Como é óbvio, o leitor pode sempre recorrer diretamente à linha de comandos, bastando, para tal, garantir que a última versão SDK da [plataforma .NET](https://www.microsoft.com/en-us/download/details.aspx?id=55168) está instalada na máquina . A partir daqui, podemos recorrer ao bloco de notas para escrever aplicações e utilizar o compilador de C# (*csc.exe*) para obter um programa executável.


## A quem se dirige este livro

Este livro é dirigido a todos aqueles que pretendem colocar-se rapidamente a par de todas as novidades introduzidas pela versão 7 da linguagem C#. O livro parte do pressuposto que o leitor já tem alguns conhecimentos e experiência na escrita de código nesta linguagem, pelo que não são apresentadas quaisquer noções básicas ou de introdução à mesma.


## Convenções

Ao longo deste livro, optou-se por seguir um conjunto de convenções que facilitam a interpretação do texto e do código dos vários exemplos apresentados. Assim, todos os excertos de código são apresentados no formato seguinte:

```cs
var a = 10;
```

Por sua vez, as notas ou observações importantes poderão ser encontradas no interior de uma secção semelhante à seguinte:

> **Nota importante**<br>
> Esta é uma nota importante.<br>


## Organização do livro

Este livro agrupa as várias novidades introduzidas pela nova especificação em 9 capítulos, que podem ser lidos sequencialmente ou, se o leitor assim o preferir, alternadamente (isto é, sem respeitar a ordem de capítulos apresentada). A leitura não sequencial dos capítulos é possível devido ao facto de todas as (poucas) dependências entre capítulos estarem devidamente identificadas.


### [Capítulo 1](1-out.md): Declaração de variáveis na lista de parâmetros de saída

O C# sempre suportou o conceito de parâmetro de saída. Tipicamente, estes parâmetros são utilizados por métodos que necessitam de devolver mais do que um valor. Como veremos neste capítulo, o trabalho necessário à utilização deste tipo de parâmetros é reduzido através do suporte à declaração de variáveis na lista de parâmetros de um método.


### [Capítulo 2](2-tuplos.md): Tuplos (*Tuples*)

Outra das novidades introduzidas pela versão 7 da linguagem são os tuplos (também conhecidos por *tuples*). Uma vez mais, estamos perante uma nova funcionalidade que tenta simplificar e reduzir o trabalho do programador em alguns cenários concretos. Com os tuplos, passamos a poder devolver mais do que um valor a partir de um método sem que isso implique o uso de parâmetros de saída ou a definição explícita de novos tipos. O Capítulo 2 analisa as principais caraterísticas associadas ao uso deste tipo de elementos.


### [Capítulo 3](3-refs.md): Devolução de valores por referência

Até ao momento, o C# permitia a passagem de variáveis por referência a métodos através da utilização do qualificador `ref`. Com o lançamento da versão 7, a linguagem passa também a permitir a aplicação deste termo ao resultado devolvido por função. Na prática, isto significa que passamos poder aceder diretamente aos objetos devolvidos a partir de uma função, sem que isso implique a utilização direta de apontadores e a correspondente fixação de memória (*pinned objects*). Neste capítulo, apresentamos os principais aspetos relacionados com esta nova funcionalidade.


### [Capítulo 4](4-patterns.md): Expressões de correspondência de tipos através de padrões (*Patterns*)

A partir do C# 7, a linguagem passa a permitir a definição de padrões mais complexos do que os suportados até ao momento. Assim, para além dos padrões que permitem efetuar comparações com valores primitivos (tipicamente utilizados em conjunto com a instrução `switch`), a linguagem passa ainda a permitir a definição de expressões baseadas nos tipos de um objeto. Apesar de esta funcionalidade não ser muito utilizada quando recorremos a programação orientada a objetos, a verdade é que tenderá a ser muito útil se necessitarmos de consumir bibliotecas criadas com uma aproximação mais funcional.


### [Capítulo 5](5-asyncreturns.md): Tipos de retorno de métodos assíncronos

O C# 5 introduziu o conceito de método assíncrono. Os métodos assíncronos contribuem para simplificar a criação e consumo de métodos que desempenham tarefas assíncronas. Como veremos neste capítulo, a versão 7 da linguagem levanta algumas das restrições aplicadas a este tipo de métodos, permitindo-nos, assim, melhorar a eficiência do código em alguns cenários.


### [Capítulo 6](6-expressionbodied.md): Membros com corpos definidos por expressões

A versão 6 do C# introduziu o conceito de membros com corpos definidos por expressões. Na prática, este conceito reduzia alguma da cerimónia necessária à implementação de alguns tipos de membros que podem ser disponibilizados por uma classe. Com o C# 7, a lista de membros que podem ser definidos à custa deste tipo de expressões é expandida. O Capítulo 6 analisa as novidades introduzidas pela linguagem nesta área.


### [Capítulo 7](7-localfunctions.md): Funções locais

Apesar de contribuírem para um aumento da produtividade do programador, a verdade é que algumas das funcionalidades suportadas pelo C# obrigam a algum cuidado para não produzirem resultados "inesperados". Por exemplo, a implementação de *iterators* baseadas no uso do termo `yield` implica algum cuidado para que o seu consumo não resulte em alguns comportamentos "inesperados". Nestes cenários, a produção do comportamento "esperado" passava pela introdução de um método privado que, tipicamente, era invocado apenas a partir de um único local. Como veremos neste capítulo, o C# 7 permite-nos passar a definir este tipo de métodos no interior de outros.


### [Capítulo 8](8-throwexpressions.md): Expressões *throw*

Até ao lançamento do C# 7, o termo reservado `throw` estava reservado apenas para instruções. Na prática, isso significava que o seu uso estava proibido em determinados locais. Com a versão 7 da linguagem, este termo passa também a poder ser utilizado na construção de expressões. O Capítulo 8 analisa detalhadamente as vantagens introduzidas por esta alteração.


### [Capítulo 9](9-numericliteralssyntax.md): Sintaxe dos literais numéricos

Com o C# 7, passamos a poder representar números inteiros em formato binário. Para além disso, a legibilidade do código é melhorada através da introdução de um carácter que desempenha o papel de separador. Neste capítulo, apresentamos alguns exemplos que justificam a introdução desta funcionalidade e que mostram como a nova sintaxe pode contribuir para melhorar a legibilidade do código que escrevemos.


### [Capítulo 10](10-referencesemanticswithvaluetypes.md): Tipos por valor utilizados como referências

Com o C# 7, a linguagem deu início à introdução de um conjunto de funcionalidades que tentam otimizar o uso dos chamados *value types*. Como veremos neste capítulo, as versões seguintes (7.1 e 7.2) deram continuidade a esta tendência, através da introdução dos conceitos de parâmetro *in* (qualificador `in`), de devolução de uma referência de leitura para um valor (`ref reafonly`), de *structs* de leitura (`readonly struct`) e de *structs* de referência (`ref struct`).


### [Capítulo 11](11-notrailingnamedarguments.md): Outras novidades do C# 7.2

Neste capítulo, encerramos este livro com a apresentação de duas novas funcionalidades introduzidas pelo C# 7.2: o novo acesso `private protected` e as novas regras que regem o uso de parâmetros nomeados. Uma vez mais, estamos perante duas pequenas funcionalidades que vêm colmatar algumas lacunas que foram detetadas com o uso da linguagem ao longo dos últimos anos


## Nota rápida sobre a versão 7.1 (e seguintes) da linguagem

O lançamento da versão 7.1 da linguagem C# constitui um marco importante da linguagem. A partir desta altura, passamos a ter a possibilidade de configurar o compilador para utilizar apenas as funcionalidades introduzidas por uma determinada versão da linguagem através do uso do chamado meacnismo de seleção da linguagem (*language version selection*). A introdução deste mecanismo de configuração permite-nos evoluir a versão da linguagem utilizada, sem que isso implique instalação de novas ferramentas de desenvolvimento.

Na prática, a utilização da versão 7.1 da linguagem C# é suportada no *Visual Studio 2017* (versão 15.3 ou superior) ou em projetos que utilizam o SDK da plataforma .NET 4.7.1 ou da plataforma .NET Core 2.0. Apesar disso, as novas funcionalidades introduzidas pela versão 7.1 estão desligadas por predefinição, pelo que a sua utilização força-nos a efetuar algumas configurações extra. Assim, os leitores que preferem utilizar o *Visual Studio*, terão de começar por aceder às propriedades do Projeto, onde, em seguida, deverão clicar sobre o botão *Advanced* existente no tabulador *Build*. A partir dessa altura, poderão escolher a versão da linguagem C# que desejam utilizar.

Alternativamente,  também podemos obter os mesmos resultados através  da edição direta do ficheiro de projeto (ficheiro de extensão *csproj*). Para isso, torna-se necessário atualizar o valor do elemento `LangVersion` existente no interior do elemento `PropertyGroup`:

```xml
<PropertyGroup>
    <LangVersion>latest</LangVersion>
</PropertyGroup>
```` 

Os leitores que pretendam recorrer a esta estratégia para projetos criados através do *Visual Studio* devem ainda ter em atenção que este IDE cria nós diferentes para as versões de *build* *Debug* e *Release* que são geradas por predefinição. Assim, torna-se necessário proceder à edição de ambos os nós se pretendermos utilizar as funcionalidades mais recentes da linguagem, conforme ilustrado em seguida:

```xml
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|AnyCPU'">
  <LangVersion>latest</LangVersion>
</PropertyGroup>

<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
  <LangVersion>latest</LangVersion>
</PropertyGroup>
```

Nos exemplos anteriores, a utilização do termo `latest` permite-nos utilizar as funcionalidades introduzidas pela última versão *minor* da linguagem. Nesta altura, isso significa que o projeto pode utilizar todas as novidades introduzidas pela versão 7.1 da linguagem. Se, no futuro, instalarmos uma nvoa versão da linguagem, então o uso deste termo garante a utilização dessas novas funcionalidades sem que tenhamos de efetuar qualquer alteração à configuração do projeto.

Para além do valor `latest`, podemos ainda recorrer ao valor especial `default`. A utilização deste valor faz com que apenas sejam utilizadas as funcionalidades introduzidas pela última versão *major* da linguagem. Portanto, se tivessemos utilizado este valor, nesta altura apenas poderíamos utilizar as funcionalidades introduzidas pela versão 7.0 da linguagem.

Outra das novidades relacionadas com o lançamento da versão 7.1 prende-se com a introdução de duas novas *flags* que permitem a geração dos chamados *reference-only assemblies*: [*/refout*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/refout-compiler-option) e [*/refonly*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/refonly-compiler-option). Uma vez que estas funcionalidades não estão diretamente relacionadas com a linguagem, mas sim com a forma como as *assemblies* são criadas e construídas em .NET, então não iremos aprofundar esse tópico neste livro.


## Suporte

Este livro foi escrito com base na versão final da linguagem C# 7. Se, por acaso, o leitor encontrar informação que lhe pareça incorreta, ou se tiver sugestões em relação ao conteúdo de alguma secção do livro, então, não hesite e envie um email com as suas questões para labreu@gmail.com ou paulo@paulomorgado.info.

[Índice](index.md)