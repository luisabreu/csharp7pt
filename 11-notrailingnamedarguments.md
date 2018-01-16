# Outras novidades do C# 7.2

Este capítulo condensa as restantes novidades do C# 7.2 que ainda não foram apresentadas nos restantes capítulos do livro. Assim, começamos por analisar o novo tipo de acesso definido à custa dos termos `private protected`, para, em seguida, analisarmos as novidades introduzidas pela versão 7.2 da linguagem no que diz respeito à passagem de valoresa métodos através de parâmetros.


## Acesso `private protected`

O novo tipo de acesso `private protected` pode ser aplicado a um membro de um tipo e indica que esse membro pode ser acedido pela própria classe ou por outras classes derivadas que tenham sido definidas na mesma *assembly* da classe que disponibiliza esse membro. Convém não confundir este tipo de acesso com o acesso `protected internal`. Um membro anotado com este último pode ser acedido por outras classes que tenham sido declaradas na mesma *assembly* que contém o tipo que disponibiliza esse membro.