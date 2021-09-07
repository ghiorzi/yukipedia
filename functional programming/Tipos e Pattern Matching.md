```json
{
    "title": "Tipos e Pattern Matching",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Tipos

Em resumo, tipos são uma forma de categorizar dados. Essa categorização serve para expressar elementos do mundo real.

No Haskell, umas das formas de declarar um novo tipo, isto é, um tipo modelado por nós, é através da palavra reservada data:

```haskell
data JobTitle = 
  Intern 
  | Programmer 
  | Engineer 
  | Manager 
  deriving Show

data Employee = Employee {jobTitle :: JobTitle, name :: String} deriving Show
```

Aqui declaramos dois tipos: JobTitle e Employee. Sendo que Employee é composto por um JobTitle e uma String. String também é um tipo, porém não foi modelos por nós.

JobTitle pode assumir quatro valores: Intern, Programmer, Engineer e Manager. Em linguagens orientadas a objetos costumamos ter o Enum, que é similar neste caso. No entanto, na programação funcional não chamamos de Enum, na verdade, há diversos nomes para definir essa estrutura, tais como: tagged union, variant record, choice type, discriminated union, disjoint union, sum type, coproduct, etc.

Na programação funcional, os tipos são responsáveis por modelar as características. Enquanto os comportamentos são modelados pelas funções.

## Pattern Matching

Como o nome sugere, pattern matching é uma forma de encontrar um padrão pré-determinado.

Para exemplificar, vamos criar uma função que irá obter o salário de um funcionário, baseado no cargo que o mesmo possui:

```haskell
getSalary :: Employee -> Double
getSalary (Employee Intern _) = 1000
getSalary (Employee Programmer _) = 1001
getSalary (Employee Engineer _) = 1002
getSalary (Employee Manager _) = 10000
```

Temos uma função pura unária que recebe um funcionário como input e retorna um double como output. E nela usamos pattern matching.

Ok, vamos entender melhor a primeira linha : getSalary (Employee Intern) = 1000. Dado um Employee com JobTitle equivalente a Intern e nome equivalente a qualquer String, retorne 1000. Underscore (_) serve como um valor coringa, não estamos aplicando nenhum padrão para o nome do funcionário, apenas para o cargo.

Caso também quisséssemos encontrar um padrão no nome, bastaria fazer algo assim:

```haskell
getSalary :: Employee -> Double
getSalary (Employee Intern _) = 1000
getSalary (Employee Programmer "Rodrigo Ghiorzi") = -1001
getSalary (Employee Engineer _) = 1002
getSalary (Employee Manager _) = 10000
```

Neste caso, se o funcionário tiver o cargo de programador e tiver o meu nome, o salário retornado será -1001, caso contrário, se o funcionário tiver o cargo de programador, mas não tiver o meu nome, então é retornado 1001. E claro, é possível criar diversas variações desta forma.

É importante prestar atenção na ordem dos casos quando usar o underscore para algum campo. Não faria sentido o trecho "getSalary (Employee Programmer "Rodrigo Ghiorzi") = -1001" ser declarado após "getSalary (Employee Programmer _) = 1001", não faz sentido, pois nunca cairia neste caso. Mas provavelmente o próprio compilador te dará um puxão de orelha se tu fizeres isso.

Para variar um pouquinho, vamos usar pattern matching mais uma vez, porém, desta vez, será uma função para promover um funcionário:

```haskell
promote :: Employee -> Employee
promote (Employee Intern name) = Employee Programmer name
promote (Employee Programmer name) = Employee Engineer name
promote (Employee Engineer name) = Employee Manager name
promote (Employee _ name) = Employee Manager name
```

Não muito diferente da outra função, temos os casos para cada cargo, o que fazemos aqui, no entanto, é retornar outro valor de funcionário, reaproveitando o nome do valor anterior e "substituindo" o cargo com o próximo da hierarquia. Digo “substituindo”, pois estamos trabalhando com imutabilidade. No ultimo caso, usamos o underscore no cargo, afinal, se não "cair" nos casos anteriores, o único cargo possível será o de Gerente, o qual não há promoção.

Eu disse que estamos trabalhando com imutabilidade, então vamos ver como o nosso código se comporta, veja o resultado final:

```haskell
data JobTitle = 
  Intern 
  | Programmer 
  | Engineer 
  | Manager 
  deriving Show

data Employee = Employee {jobTitle :: JobTitle, name :: String} deriving Show

-- pattern matching
getSalary :: Employee -> Double
getSalary (Employee Intern _) = 1000
getSalary (Employee Programmer _) = 1001
getSalary (Employee Engineer _) = 1002
getSalary (Employee Manager _) = 10000

-- pattern matching
promote :: Employee -> Employee
promote (Employee Intern name) = Employee Programmer name
promote (Employee Programmer name) = Employee Engineer name
promote (Employee Engineer name) = Employee Manager name
promote (Employee _ name) = Employee Manager name

showInfo :: Employee -> String
showInfo employee = 
    "{name: \"" ++ (name employee) ++ 
    "\", job title: \"" ++ show (jobTitle employee) ++ 
    "\", salary: " ++ show (getSalary employee) ++ "}"

me :: Employee
me = Employee Programmer "Rodrigo Ghiorzi"

promoted :: Employee
promoted = promote me

main =
  print (showInfo me) >> 
  print (showInfo promoted)
```

Note que temos o valor me que recebe um valor com JobTitle igual a Programmer e nome equivalente a Rodrigo Ghiorzi. Abaixo, temos o valor promoted que recebe o retorno da função promote tendo como input o valor me.

O que esse código irá exibir no console?

```
{name: \"Rodrigo Ghiorzi\", job title: \"Programmer\", salary: 1001.0}
{name: \"Rodrigo Ghiorzi\", job title: \"Engineer\", salary: 1002.0}
```

Como os valores são imutáveis, mesmo o valor me sendo parametrizado para função promote, não haverá side-effect. Pois a função promote não altera o valor me, ela retorna outro valor do tipo funcionário. Portanto, o valor me não fica com o mesmo valor do promoted, pois nunca foi alterado.

## Palavras finais
A ideia do artigo é introduzir o conceito de tipo e pattern matching, desta vez usando Haskell como ferramenta.

E ai, conseguiu pegar a ideia?