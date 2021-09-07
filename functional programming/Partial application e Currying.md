```json
{
    "title": "Partial application e Currying",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Partial application

Partial application é um conceito empregue no momento em que uma função é aplicada parcialmente em alguns dos parâmetros que a mesma expecta.

Para entender o conceito vamos escrever uma função que representa uma multiplicação. Segue o código em JS:

```javascript
const multiply = (a, b) => a * b;

console.log(multiply(2, 4)); // 8
console.log(multiply(3, 7)); // 21
```

Até ai nada de novo. Mas agora vamos pensar no seguinte cenário: se eu precisar fazer mais três multiplicações por 2, sendo elas: 2 * 2, 2* 7 e 2* 9. O que eu teria que fazer?

- multiply(2, 2);
- multiply(2, 7);
- multiply(2, 9);

Em todos esses três casos o número dois se repete, mas será que realmente precisaria ser desta forma?

Uma alternativa seria deixar o número dois fixo na função multiply, entretanto, não seria mais possível reaproveitar a função para os cenários onde a multiplicação fosse efetuada por um valor diferente.

Outra forma, obviamente, é através de partial application. Eu acredito que seja o tema mais conveniente para começar a visualizar sentindo para os casos onde uma função retorna outra.

Segue o código com partial application:

```javascript
// função que retorna outra função
const multiplyBy = factor => number => factor * number;

// partial application
const multiplyBy2 = multiplyBy(2);

[2, 7, 9]
    .map(multiplyBy2)
    .forEach(value => console.log(value));

// Ainda conseguimos aplicar os parâmetros um por vez
console.log(multiplyBy(2)(9));
```

- multiplyBy é função de alta ordem que retorna outra função. Desta forma, conseguimos aplicar parcialmente nossa função;
- multiplyBy2 aplica a função multiplyBy parametrizando o valor dois. Portanto, um dos parâmetros já foi aplicado, como retorno temos outra função que fica responsável por receber o parâmetro restante;
- Seguindo em frente, usamos a função multiplyBy2 parametrizando apenas o valor que queremos multiplicar por dois;

Ou seja, ao invés de deixarmos o valor dois fixo na função de multiplicação, recebemos outra que fica encarregada de fixar este comportamento.

## Currying

Agora vamos para a segunda parte deste artigo: currying (na verdade, nós já vimos currying no exemplo passado).

Basicamente, currying é a arte de transformar uma função n-ária em um enumerado de funções unárias.

Veremos agora, a mesma lógica em F#:

```fsharp
let multiplyBy factor number = factor * number

let multiplyBy2 = multiplyBy 2

[2;7;9] |> List.map multiplyBy2 |> printfn "%A" // 4, 14, 18

multiplyBy 2 9 |> printfn "%d" // 18
```

O primeiro ponto que gostaria de destacar é que no F# não precisamos nos preocupar em retornar funções intermediárias, pois isto é feito automaticamente pelo compilador. Veja que apesar de termos declarado multiplyBy como uma função binária sem retornar explicitamente outra função, ainda é possível aplicá-la parcialmente.

O segundo ponto que gostaria de expor é que no F# operadores também são categorizados como funções. Portanto, podemos efetuar partial application em operadores. Segue a segunda versão:

```fsharp
let multiplyBy = (*)

let multiplyBy2 = multiplyBy 2

[2;7;9] |> List.map multiplyBy2 |> printfn "%A" // 4, 14, 18

multiplyBy 2 9 |> printfn "%d" // 18
```

Como supradito, operadores são funções. Entretanto, como a multiplicação não está aplicada a nenhum parâmetro, multiplyBy recebe a função de multiplicação como valor. A partir dai, seguimos a mesma ideia de antes.

Eu tenho quase certeza que ainda não ficou claro qual a diferença, não é verdade?

## O que difere partial application de currying?

Currying é uma função que retorna outra função que pode retornar outra função que pode retornar outra função …. onde cada função **é unária**.

Partial application é uma função que retorna outra função que pode retornar outra função que pode retornar outra função …. onde cada função **pode ser n-ária**.

Para ficar mais claro, criaremos uma função para multiplicar três valores ao invés de dois.

Com partial application:

```javascript
const multiply = a => (b, c) => a * b * c;

const multiplyBy2 = multiply(2);

console.log(multiplyBy2(4, 6)); // 48
```

- Neste exemplo, ao aplicarmos o primeiro parâmetro na função de alta ordem multiply, recebemos como valor uma função binária;
- Logo, ao aplicarmos a função multiplyBy2 temos que passar os dois parâmetros restantes.

Como a função retornada não é unária, o código acima não aplica o conceito de currying.

Muito bem, com currying, o código fica assim:

```javascript
const multiply = a => b => c => a * b * c;

const multiplyBy2 = multiply(2);
const multiply2By4 = multiplyBy2(4);

console.log(multiply2By4(6)); // 48
```

- Veja que, na função multiply, é retornado uma função intermediária b, que por sua vez, retorna outra função intermédia c, para então a multiplicação ocorrer de fato;
- Assim, multiplyBy2 recebe uma função uná;
- E multiply2By4 recebe também uma função unária;
- Por fim, podemos aplicar o ultimo parâmetro restante.

Na prática esta é a diferença, com currying temos um conjunto de funções exclusivamente unárias. Enquanto com partial application temos uma coleção de funções que não são necessariamente unárias.

## Para finalizar: Por que usar estes conceitos?

É mais fácil lidar com funções unárias, pois geralmente elas assumirão uma responsabilidade especifica. Não limitado apenas a isso, fica tranquilo aplicar pipelines em funções que recebem apenas um valor.

A meu ver, estes conceitos são essenciais para evitar replicação de código. Funções genéricas (como multiplyBy) servem para reuso de funções especificas (como multiply2). Indo além, imagine que você precisa inserir um registro no banco de dados, mas você não quer passar a conexão do banco toda vez que usar a função para efetuar o registro. Estes conceitos poderiam te ajudar a evadir esse problema, não acha?

Ainda na questão de funções genéricas e especificas, eu particularmente vejo mais expressividade na função multiplyBy2 do que na função multiply. É claro que ambas deixam claro que uma multiplicação será feita, no entanto, multiplyBy2 já expressa um dos valores usado, logo preciso me preocupar apenas com o outro. Mas isso não significa que a função multiply deva deixar de existir, afinal podemos reutilizá-la.

Você deve ter reparado que com funções intermediarias a multiplicação acontece somente depois de termos aplicados todos os parâmetros. Ou seja, podemos ir "preparando o terreno" até termos todos os valores para aplicar.

## Palavras finais

Não é muito difícil enxergar sentido em uma função que recebe outra função como valor. Porém, é um pouco mais confuso entender porque deveríamos ter uma função que retorna outra. E esta é ideia deste artigo: introduzir os conceitos de partial application, currying e mostrar também que uma função retornar outra não é algo despropositado.

Até a próxima! :)