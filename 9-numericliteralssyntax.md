# Sintaxe dos literais numéricos e utilização de literais *default*

A versão 7.0 da linguagem introduz algumas novidades na sintaxe utilizada na representação de valores literais numéricos. Assim, a partir desta altura, passamos a poder representar valores inteiros em formato binários e a poder utilizar um novo carácter como separador de dígitos. Estas alterações na representação de literais numéricos procuram reduzir os erros associados à leitura deste tipo de valores, especialmente quando necessitamos de trabalhar com valores binários ou com enumerações do tipo *flag*.


## Representação de inteiros em formato binário

Desde o seu lançamento, o C# sempre nos permitiu representar valores numéricos inteiros em base decimal (base esta que é utilizada por predefinição) e em base 16 (hexadecimal). Um dos casos mais comuns associados ao uso de literais em hexadecimal é a definição de valores utilizados como *flags* em enumerações (elementos definidos através do termo reservado `enum`).
Uma vez que cada digito em hexadecimal é definido à custa de 4 bits, então, nestas situações, os valores da enumeração resumiam-se aos valores 1, 2, 4, 8, etc. O excerto seguinte apresenta a forma usual de definirmos este tipo de elemento em C# em base decimal:

```cs
[Flags]
public enum Opcoes
{
    Op1   =   0,
    Op2   =   1,
    Op3   =   2,
    Op4   =   4,
    Op5   =   8,
    Op6   =  16,
    Op7   =  32,
    Op8   =  64,
    Todos = 255,
}
```

Alguns programadores preferem utilizar a representação em base hexadecimal, conforme ilustrado através do excerto seguinte;

```cs
[Flags]
public enum Opcoes
{
    Op1   = 0x00,
    Op2   = 0x01,
    Op3   = 0x02,
    Op4   = 0x04,
    Op5   = 0x08,
    Op6   = 0x10,
    Op7   = 0x20,
    Op8   = 0x40,
    Todos = 0xff,
}
```

A partir do C# 7, podemos representar este tipo de valores através da nova sintaxe de representação de valores inteiros binários. O excerto seguinte recorre a esta nova sintaxe para representar a mesma enumeração que foi apresentada anteriormente:

```cs
[Flags]
public enum Opcoes
{
    Op1   = 0b00000000,
    Op2   = 0b00000001,
    Op3   = 0b00000010,
    Op4   = 0b00000100,
    Op5   = 0b00001000,
    Op6   = 0b00010000,
    Op7   = 0b00100000,
    Op8   = 0b01000000,
    Todos = 0b11111111,
}
```

Se optarmos pelo uso da sintaxe binária, então cada valor está limitado a uma combinação de dígitos 0 e 1. Para além disso, o uso do prefixo `0b` é obrigatório para definir a base usada na representação. O uso de valores binários nestes casos aumenta a legibilidade, especialmente quando temos mais do que uma *flag* selecionada. No excerto seguinte, é fácil comprovarmos os aumentos de legibilidade decorrentes do uso desta nova forma de representação:

```cs
var o = 0b00001001; //Op5 & Op2
```


## Separador de dígitos

A representação de números em binário pode obrigar-nos à utilização de muitos dígitos. Nestes casos, a legibilidade pode acabar por ser comprometida. Pior: podemos mesmo acabar por introduzir alguns erros quando estamos perante números com alguma dimensão. Foi para tentar solucionar este tipo de problemas que a linguagem C# passa a permitir o uso do carácter `_` como separador de dígitos. 

O excerto seguinte ilustra o uso deste carácter:

```cs
[Flags]
public enum Opcoes
{
    Op1   = 0b0000_0000,
    Op2   = 0b0000_0001,
    Op3   = 0b0000_0010,
    Op4   = 0b0000_0100,
    Op5   = 0b0000_1000,
    Op6   = 0b0001_0000,
    Op7   = 0b0010_0000,
    Op8   = 0b0100_0000,
    Todos = 0b1111_1111,
}
```

Nesta altura, o leitor pode sentir-se tentado a utilizar o separador de dígitos para separar o prefixo da base do número de forma a melhorar ainda mais a legibilidade do código. Inicialmente, isto não era possível, já que a primeira versão da linguagem que introduziu esta funcionalidade não permitia a colocação do separador de dígitos imediatamente a seguir ao prefixo nem no final do número. Com o lançamento da versão 7.2, esta regra foi mudada, sendo que o separador de digitos já pode passar a ser colocado imediatamente a seguir ao prefixo.

O excerto seguinte ilustra estes pontos: 

```cs
var aux1 = 0b_0000_0001; // ERRO antes 7.2; OK a partir de 7.
var aux2 = 0b0000_0001_; // ERRO
```

Por outro lado, e se assim o desejarmos, podemos mesmo utilizar consecutivamente o separador de dígitos. Por exemplo, no excerto seguinte recorremos a esta estratégia para fazer com que os grupos de dígitos de um número sejam bem legíveis:

```cs
var aux = 0b__0000________0001;
```

Pessoalmente, parece-nos que uma boa utilização do separador de dígitos contribuirá sobremaneira para melhorar a legibilidade do valor final.

Nesta altura, resta-nos apenas referir que este separador não está limitado à representação de números em binário e pode ser utilizado com qualquer valor numérico. O excerto seguinte apresenta alguns exemplos que ilustram o seu uso com números de outros tipos:

```cs
public const long contas = 100_000_500_000;
public const double Avogrado = 6.022_140_857_747_474e23
```

## Utilização de literais *default*

Os literais *default* podem ser vistos como uma simplificação das expressões *default value*. As expressões *default value* são utilizadas para inicializar variáveis com o valor predefinido de um determinado tipo. No exemplo seguinte, recorremos a uma expressão deste tipo para inicializar a variável `i`:

```cs
int i = default(int); //inicializado a 0
```

A partir do C# 7.1, podemos obter o mesmo resultado através do uso de um literal *default*. Assim, o excerto seguinte produz exatamente o mesmo resultado que o exemplo anterior:

```cs
int i = default //inicializado a 0
```

Na prática, podemos ver o literal `default` como possuindo um comportamento muito semelhante ao `null`. Por outras palavras, estamos a falar de um elemento sem tipo predefinido, que pode adquirir um tipo através de uma conversão implícita ou explícita. Contudo, ao contrário do que acontece com `null`, `default` pode mesmo ser convertido em qualquer tipo T (incluindo-se aqui os tipos não *nullable*), sendo equivalente a `default(T)`. Quando estamos perante tipos onde o uso de `default`e `null` é permitido, então `default` possui o valor `null`. 


## Conclusão

Ao longo deste capítulo, vimos como a representação e números inteiros em formato binário e como o uso de um carácter separador contribuem para melhorar a legibilidade do código em alguns cenários. Uma vez mais, e à semelhança do que acontece com as restantes novidades introduzidas pela versão 7.0 da linguagem, estamos perante um conjunto de novas funcionalidades que estão longe de revolucionar por completo a linguagem.

Parece-nos, contudo, que se somarmos todas novidades introduzidas por esta versão, o resultado final é uma linguagem mais forte e mais flexível do que a que tínhamos anteriormente. Portanto, rapidamente concluímos que estamos perante um conjunto de novidades que serão muito bem recebidas pelos programadores.


### Bibliografia

["C# 7: Binary Literals and Numeric Literal Digit Separators"](http://blog.somewhatabstract.com/2017/01/02/c7-binary-literals-and-numeric-literal-digit-separators/) <br>
["What's new in C# 7"](https://docs.microsoft.com/en-us/dotnet/articles/csharp/csharp-7#numeric-literal-syntax-improvements) <br>
[Allow digit separator after 0b or 0x](https://github.com/dotnet/csharplang/blob/master/proposals/csharp-7.2/leading-separator.md)

[Anterior](9-numericliteralssyntax.md) [Índice](index.md) [Próximo](10-referencesemanticswithvaluetypes.md)
