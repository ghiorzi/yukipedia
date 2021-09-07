```json
{
    "title": "Pipeline",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Introdução

Tire as crianças de perto, pois hoje aprenderemos um pouquinho sobre programação funcional.

No artigo anterior, para categorizar funções, optei por utilizar JavaScript. Mas neste eu intendo usar uma linguagem de verdade: F#. (humor)

Para começar a brincadeira, diga-me: o que o código abaixo faz?

```fsharp
5 |> fun x -> x * x |> printf "%d"
```

Calcula e imprime o quadrado de 5. Em resumo, 5 é o valor inputado em nossa função anônima unária, a qual retorna o quadrado de x, sendo ele 5. Em seguida, o output desta operação é usado como input da função printf. Desta forma, %d é substituído por 25, dado o valor 5 como input inicial.

Caso ainda tenha ficado obscuro, relaxa que aos poucos ficará fácil de entender!

## Declarando funções

Vamos entender melhor como isso funciona com um exemplo bem simples. Desenvolveremos, passo a passo, um código que:

- Receberá uma lista de números inteiros (de 1 a 10);
- Então filtraremos apenas os números pares;
- Em seguida, calcularemos o quadrado de cada numero presente na lista;
- Isto feito, somaremos estes números e retornaremos um valor como output.

Antes de escrever o fluxo completo deste cenário, vamos extrair os comportamentos necessários de forma isolada:

- Primeiro, declararemos uma lista para termos um ponto de partida;
- Criaremos uma função para ratificar se um número é par;
- Criaremos uma função para calcular o quadrado de um número;
- E no caso da soma, não precisaremos criar a função, pois a mesma já estará disponível nativamente.

Seguindo estes passos nosso código ficará assim:

```fsharp
let numbers = [1..10]
let isEven number = number % 2 = 0
let square number = number * number
```

Vamos entender a sintaxe do código acima:

- Não usamos parênteses, simplesmente temos um espaço após o nome da função para declarar seus respectivos parâmetros;
- Neste caso, temos apenas funções unárias, mas caso tivéssemos funções poliádicas não usaríamos virgulas para separar os parâmetros, ao invés disso, separaríamos com um espaçamento entre cada;
- Note que, isEven não usa o operador lambda e nem faz uso da palavra return, ainda sim, esta função retorna um booleano. Por que? Porque toda função retorna algo, mesmo quando seu código não retorna nada (nestes casos, Unit é usado como retorno);
- Unit é um tipo usado para representar a ausência de retorno, se você já programou em linguagens como C#, Java ou C++, provavelmente associará Unit com Void.

## Desenvolvendo a primeira versão do nosso código

```fsharp
let numbers = [1..10]
let isEven number = number % 2 = 0
let square number = number * number

let sumEventNumbers values =
    let evenNumbers = List.filter isEven values
    let squaredNumbers = List.map square evenNumbers
    let sum = List.sum squaredNumbers
    sum

numbers |> sumEvenNumbers |> printf "%d"
```

Para filtrar os números pares, usamos a função filter (similar ao filter do JS). A função filter é uma função de alta ordem pura que retorna uma nova lista contendo os elementos que atendem o critério de filtro.

Após realizar a filtragem e receber uma lista com os números pares, usamos a função List.map (novamente, similar ao map do JS) para obter o quadrado de cada número. A função map é uma função de alta ordem pura que mapeia um valor para outro, neste caso, retorna uma nova lista.

Tendo nossos números ao quadrado, podemos, portanto, realizar a soma dos valores através da função sum. A função sum é uma função unária pura.

Em seguida, apenas retornamos o valor da soma. Até aqui continua simples, certo?

Muito bem, agora que a lógica está pronta, podemos, enfim, abordar o tema do artigo: pipeline.

## Pipeline

Vamos relembrar do primeiro trecho de código visto neste artigo:

```fsharp
5 |> fun x -> x * x |> printf "%d"
```

Note que, temos o seguinte operador: |>. Este operador é conhecido como pipe, sendo mais especifico: forward pipe, pois há também: <|, este é chamado de backward pipe.

### Forward piping

Podemos simplificar o fluxo do nosso código através de pipelines, veja abaixo:

```fsharp
let numbers = [1..10]
let isEven number = number % 2 = 0
let square number = number * number

let sumEventNumbers values =
    values
    |> List.filter isEven
    |> List.map square
    |> List.sum

numbers |> sumEvenNumbers |> printf "%d"
```
Para simplificar a linha de raciocínio, imagine que cada pipe (|>) é literalmente um tubo. Cada "tubo" é conectado em outro, quando um "tubo"chega ao fim, chega-se ao inicio de outro. Dentro dos "tubos" temos funções e o output de cada função, isto é, final de um "tubo", é o começo de outro, em outras palavras, o input da próxima função. Portanto:

- ([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]) values |> List.filter isEven: [2, 4, 6, 8, 10];
- ([2, 4, 6, 8, 10]) |> List.map square: [4, 16, 36, 64, 100];
- ([4, 16, 36, 64, 100]) |> List.sum: 220

Mesmo que você nunca tenha usado uma linguagem primariamente funcional, verá que esse fluxo não é muito distante de linguagens mainstreams. Segue minha versão em JS:

```javascript
const sum = 
    [...Array(10).keys()]
        .map(i => i + 1)
        .filter(number => number % 2 === 0)
        .map(number => number * number)
        .reduce((a,b) => a + b);

console.log(sum);
```

E pra não dar barda, em C#:

```csharp
Enumerable
    .Range(1, 10)
    .Where(number => number % 2 == 0)
    .Select(number => number * number)
    .Aggregate((a,b) => a + b);
```

Aspectos funcionais estão presentes em diversas linguagens, mas nem todo mundo se da conta disso. Alias, perceba que nosso código está puramente imutável. Eu poderia falar um pouco sobre, entretanto, acredito que essa tema mereça um artigo dedicado.

### Backward piping

Forward piping conecta o valor vigente com a função que o sucede. Enquanto no backward piping o valor é conectado com a função que o antecede.

Com forward piping:

```fsharp
"father" |> printf "I'm your %s"
```

Com backward piping:

```fsharp
printf "I'm your %s" <| "father"
```

## “finais” |> printf “Palavras %s”

Este artigo visa introduzir o conceito de pipeline, bem como demonstrar que, alguns conceitos de programação funcional, na verdade, já estão presentes em linguagens conhecidas.