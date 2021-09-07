```json
{
    "title": "Lazy evaluation",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Introdução

Conhecimento em **LINQ** é mandatório para qualquer pessoa que resolva se aventurar no universo C#. À primeira vista, **LINQ** assemelha ser meramente mais um dos instrumentos da linguagem, contudo, em virtude dele, coleções tornam-se expressivamente profícuas. **LINQ** trás consigo um conjunto de features valorosas, dentre elas: manipular objetos em memória, manipular xml e até mesmo manipular queries em banco de dados. De fato, **LINQ** é encantador! Todavia, como toda abstração, é importante atentar alguns pormenores.

## 1. Enumerable

**Enumerable** é um dos diversos recursos de coleção à disposição no C#. Para compreender o **LINQ** de modo correto é primordial que, antes de tudo, entenda-se mais sobre o **Enumerable**.

**Enumerable** é uma coleção read-only, ou seja, não é possível efetuar operações de escrita como adicionar ou remover um elemento. Porém, este não é seu único aspecto.

**Enumerable is lazy!** Quando alguma operação é produzida em um **Enumerable**, os registros não são carregados na memória. Isto porque as operações não são materializadas de imediato, afinal o **Enumerable** é preguiçoso.

### 1.1. Deferred query execution

Deferred query execution (lazy evaluation) é indubitavelmente um dos conceitos mais significativos do **LINQ**, visto que sem este mecanismo, **LINQ** desempenharia de maneira insatisfatória.

Para ilustrar esta concepção, tomaremos como exemplo o simples trecho de código a seguir:

```csharp
// 1. Dado um conjunto de números de 1 a 10
IEnumerable<int> numbers = Enumerable.Range(1, 10);

// 2. Busque apenas os números pares contidos em numbers
var evenNumbers =
    numbers
        // 3. Onde number obrigatóriamente seja divisível por 2
        .Where(number => number % 2 == 0)
        // 4. Destes, obtenha somente os 3 primeiros
        .Take(3)
        // 5. Materialize em uma lista
        .ToList();
```

As funções **Where** e **Take**, vulgo **query operators**, efetuam ações de consulta na etapa 3 e 4, respectivamente. O enigmático, no entanto, é que estas ações são concretizadas verdadeiramente no quinto estágio, quando há materialização dos dados. Em outras palavras, enquanto não houver materialização, os registros não são conduzidos para a memória.

Para ratificar que o **Enumerable** atua sob demanda, podemos realizar uma experimentação elementar: implementar um **Enumerable**.

## 2. Sequences

**IEnumerable** é uma interface, portanto, é praticável implementá-lo. A implementação de um **IEnumerable** é denominada como **Sequence**. **Sequence**, diferente de **List**, comporta somente operações de leituras. Por derivar-se de um **IEnumerable**, **Sequences** beneficiam-se de **lazy evaluation**.

### 2.1 Experimento

Como experimento, criaremos um **Sequences**. Nosso Sequence será responsável por carregar números pares. Simples, não?

Iremos lidar com números inteiros, logo, implementaremos um IEnumerable<int>:

```csharp
public class EvenNumbers: IEnumerable<int>
{
    private readonly EvenNumbersEnumerator _enumerator;

    public EvenNumbers()
        => _enumerator = new EvenNumbersEnumerator();

    public IEnumerator<int> GetEnumerator()
        => _enumerator;

    IEnumerator IEnumerable.GetEnumerator()
        => _enumerator;
}
```

Perceba que, além do **IEnumerable**, há mais um integrante na jogada. **Enumerator** é um constitutivo primacial, incumbido por gerar uma sequência de valores. Há uma particularidade, entretanto. Em vez de conceber uma coleção com todos os valores de imediato, o **Enumerator** disponibiliza os valores individualmente de forma preguiçosa.

Este mecanismo propicia que os primeiros valores auferidos sejam processados in continenti, sem a compulsoriedade de conter todos os elementos.

Assim sendo, devemos da mesma forma, implementar um **Enumerator**:

```csharp
public class EvenNumbersEnumerator: IEnumerator<int>
{
    public int Current => _current;

    object IEnumerator.Current => _current;

    private int _current = 0;
    
    public bool MoveNext()
    {
        _current = _current + 2;

        return true;
    }

    public void Reset()
        => _current = 0;

    public void Dispose()
        => GC.SuppressFinalize(this);
}
```

**O ponto-chave é a função MoveNext**. Em síntese, enquanto houver valores para serem iterados, o cursor se moverá **sequencialmente** para o elemento sucessor. No tocante ao nosso cenário, não há delimitação para o final da sequência. Isto posto, o incremento ocorrerá sem restringimento.

Contudo, algo a se observar é que enquanto o MoveNext não for invocado, não haverá valor para ser iterado. Por conseguinte, o "valor inicial" será 0, mas para legitimar essa declaração iremos percorrer a sequência de modo inusual.

### 2.2 Iterator

Habitualmente, iteramos sequências através do foreach. Porém, para entendermos o fluxo do **Enumerator**, iremos seguir uma abordagem "mais manual":

```csharp
var evenNumbers = new EvenNumbers();

// 1. Obtemos o Enumerator
using (var enumerator = evenNumbers.GetEnumerator())
{
    // 2. Retornará 0, pois o MoveNext ainda não foi chamado
    WriteLine(enumerator.Current);
    // 3. Moveremos o cursor para o próximo valor
    enumerator.MoveNext();
    // 4. Exibirá o número 2
    WriteLine(enumerator.Current);
    // 5. Moveremos o cursor para o próximo valor
    enumerator.MoveNext();
    // 6. Exibirá o número 4 e assim sucessivamente
    WriteLine(enumerator.Current);
}
```

Note que, a partir da nossa **Sequence** (EvenNumbers), obtemos o **Enumerator** (EvenNumbersEnumerator). E dai a diante temos acesso aos valores da sequência. Enquanto o MoveNext não é chamado, o valor 2 não será retornado, da mesma forma que o valor 4 será retornado somente após o MoveNext ser chamado após o cursor estar apontado para o valor 2.

Por baixo dos panos, o que ocorre ao usarmos o foreach é o seguinte:

```csharp
using (var enumerator = evenNumbers.GetEnumerator())
{
    while (enumerator.MoveNext())
    {
        // Trecho dentro do foreach
    }
}
```

Entretanto, como não demarcamos um valor final para a nossa Sequence, cairíamos em um loop infinito.

Enfim, para ratificar o **lazy evalution**, usaremos o **Sequence** que acabamos de criar:

```csharp
var evenNumbers = new EvenNumbers();

evenNumbers
    .Take(10)
    .ToList();
```

Isto feito, podemos fazer o teste:

- Na função MoveNext, deixe um breakpoint na linha em que o valor é atribuído para _current;
- Em seguida, remova o ToList do chaining. Ao fazer isso, não haverá materialização dos dados, consequentemente, o MoveNext não será chamado;
- Adicione novamente o ToList e perceba que a partir de então o MoveNext será chamado.

Ou seja, podemos configurar como queremos filtrar o nosso **Sequence** e apenas quando precisarmos realmente dos dados, eles serão carregados na memória. **Isto, é claro, se nenhuma materialização ser efetuada de modo indevido**. Fantástico, concorda?

## 3. Query operators

Como já visto, podemos efetuar operações de leitura em **Sequences**, mas como os operadores satisfazem o preceito de **lazy evaluation**? 

Para entendermos melhor, que tal implementarmos um operador chamado **Filter** que fará essencialmente o mesmo que o **Where**?

```csharp
// Método de extensão com comportamento similar ao Where
public static IEnumerable<T> Filter<T>(this IEnumerable<T> source, Predicate<T> predicate)
{
    foreach (var element in source)
    {
        if (predicate(element))
            yield return element;
    }
}
```

Em suma, iteramos um Sequence elemento por elemento, validamos se o elemento vigente corresponde ao predicado e caso o mesmo satisfaça a condição, o elemento é então retornado.

Como está sendo usado um foreach, os dados, portanto, estão sendo materializados, correto? Nope. Primeiramente, repare que, apesar do operador Filter retornar um IEnumerable, "não está sendo retornado" uma sequência de elemento dentro de seu escopo. Na verdade, está. Porém, aos poucos.

### 3.1 Yield return

Sumariamente, **yield return** é um **syntax sugar** designado para orquestrar a execução do fluxo mediante um estado alusivo ao cursor vigente em um **Enumerable**. Em outros termos, **yield** torna sua aplicação "inteligente" suficientemente para saber em que parte do **Enumerable** o cursor parou e que em algum momento a execução deve retomar a partir deste mesmo ponto. Desta forma, não há problema em retornar um elemento por vez.

## Palavras finais

Para finalizar, é importante recapitular o que vimos aqui:

- **Lazy evaluation** permite que a execução seja efetuada apenas quando os dados são requisitados;
- **LINQ** só é viável graças ao **lazy evaluation**;
- Criar **Sequences** customizados promove flexibilidade, afinal seu objeto desfrutará de lazy evalution e, por consequência, terá acesso aos recursos do **LINQ**;
- Materializar um ou mais **Sequences** de modo indevido seguramente trará dor de cabeça.