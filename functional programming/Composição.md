```json
{
    "title": "Composição",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Introdução

Compor funções é como montar lego. Cada função é um bloco, assim podemos encaixar uma na outra até chegar a forma que nos almejamos.

No F#, funções podem ser compostas por outras e para isso podemos usar os operadores de composição: >> (foward composition) e << (backward composition). Veja o seguinte exemplo com foward composition:

```fsharp
let square number = number * number

let print value = value |> printf "%d"

let squareThenPrint = square >> print

5 |> squareThenPrint
```

Temos três funções:

- square: para calcular o quadrado do número parametrizado;
- print: para exibir o valor parametrizado no console;
- squareThenPrint: para compor as funções square e print.

Deste modo, parametrizamos o valor 5 para a função squareThenPrint e assim não precisamos conectar os valores (de forma explícita) as funções square e print.

Ok, mas não é exatamente o mesmo que piping? É similar, no entanto, não é igual.

## Piping <> composition

Com foward piping, o output da função anterior é o input da função posterior. Porém, o piping não é mantido como valor. Isto impossibilita a reutilização do trecho em questão. Além disso, o piping performa as instruções retornando o valor já transformado.

Com foward composition, também seguimos a ideia de repassar o output de uma função como input de outra. Entretanto, quando aplicamos composição temos uma função como retorno, consequentemente, podemos reutilizar essa função se necessário.

Para fixar a ideia: consegue elencar uma diferença entre uma função anônima unitária e uma função unitária?
Vamos entender a sintaxe do código acima:

```fsharp
let square number = number * number

let print value = value |> printf "%d"

let squareThenPrint = square >> print

[1..5] |> List.map squareThenPrint // 1, 4, 9, 16, 25

[1..5] 
    |> List.map (fun number -> number * number |> print) // 1, 4, 9, 16, 25
```

Na primeira listagem usamos composição, enquanto na segunda usamos piping com uma função anônima que calcula o quadrado de um número.

Ambos auferem o mesmo resultado, entretanto, perceba alguns pontos:

- Se a gente precisar reaproveitar o fluxo do piping em outras partes do nosso código, conseguiremos? Não, mas conseguiremos com composição;
- Se a gente precisar reaproveitar a função anônima que calcula o quadrado de um número, conseguiremos? Não, mas caso tenhamos esta função como valor, conseguiremos. Como é o caso da função square;

Isto quer dizer que piping não presta e que devemos usar apenas composição? Não, pois terá casos em que piping servirá muito bem e outros em que a composição será uma alternativa mais adequada. O mesmo vale para funções anônimas.


## Backward composition

Assim como backward piping, temos também backward composition. A ideia é a mesma, basicamente invertemos a ordem. Segue o mesmo fluxo com backward piping e backward composition:


```fsharp
let square number = number * number

let print value = printf "%d" <| value

let squareThenPrint = print << square

List.map
    <| squareThenPrint
    <| [1..5]  // 1, 4, 9, 16, 25
```

## Palavras finais

Este artigo visa introduzir o conceito de composição (foward composition e backward composition) de uma forma simples.
