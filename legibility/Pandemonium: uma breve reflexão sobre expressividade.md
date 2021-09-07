```json
{
    "title": "Pandemonium: uma breve reflexão sobre expressividade",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Introdução

Inteligibilidade é comumente negligenciada quando estamos dando nossos primeiros passos no mundo da programação. Manifeste-se, por favor, quem em nenhuma circunstância declarou variáveis desprovidas de semântica.

Há muito tempo debate-se sobre código de fácil entendimento. No entanto, sempre houveram pormenores que me incomodavam, dentre eles a negação (!).

A princípio, declarações com negação parecem abnóxias. Contudo, não é preciso ir muito longe para transformar o nosso código em um pandemônio. Deste preceito, surgiu a minha vontade de desenvolver uma biblioteca .NET: [Pandemonium](https://github.com/ghiorzi/Pandemonium).

## 1. Livre-se da negação

Há alguns meses eu havia escrito um artigo sobre métodos de extensão, nele eu exponho meu ponto de vista sobre negação: "Acredito que, por questão semântica, toda verificação booleana deva ser auferida tanto por um comportamento que ratifica o caso verdadeiro quanto por um comportamento que certifica o caso falso".

Pois bem, vamos por em pratica outro exemplo, o famigerado Any negado.

```csharp
var names = new List<string>()
{
    "Hachiman",
    "Yukino"
};

// from !Any
!names.Any(name => name == "Yui");
// to None
names.None(name => name == "Yui");

bool value = false;
```

É comum que não haja comportamentos antagônicos para ratificações booleana, o que nos induz a negar o código. Ao meu ver, None (disponivel no Pandemonium) é notóriamente mais explícito que !Any.

Ok, porém creio que expressividade pode ir além disso...

## 2. Amplie a expressividade através de variáveis

Parece ser uma pratica universal evadir o uso de variáveis booleanas quando lida-se com estruturas condicionais. Para refletirmos, imagine que precisamos verificar se um cliente pode pagar o preço promocional em um produto qualquer, assumindo ao menos um dos seguintes critérios para tal:

- O cliente tem menos de 10 anos;
- O cliente tem mais de 60 anos;
- O cliente faz aniversário na data da compra.

Não ligue para os critérios, apenas se importe que temos três condicionais aqui.

Uma forma para representar a condicional é a seguinte:

```csharp
var customer = new Customer("Ichinose", new DateTime(2000, 1, 1));

if (
    customer.CalculateAge() < 10 ||
    customer.CalculateAge() > 60 ||
    customer.Birthday.Date == DateTime.Now.Date)
)
{

}
```

Poderíamos mudar para:

```csharp
var customer = new Customer("Ichinose", 1.January(2000));

int customerAge = customer.CalculateAge();

bool customerIsUnder10YearsOld = customerAge < 10;
bool customerIsOver60YearsOld = customerAge > 60;
bool isCustomerBirthdayToday = customer.Birthday.Date.Today();

bool applyPromotionalPrice =
    customerIsUnder10YearsOld or
    customerIsOver60YearsOld or
    isCustomerBirthdayToday;

if (applyPromotionalPrice) 
{

}
```

Tudo bem, eu admito que o código ficou consideravelmente maior, mas preste atenção na expressividade deste trecho:

```csharp
bool applyPromotionalPrice =
    customerIsUnder10YearsOld or
    customerIsOver60YearsOld or
    isCustomerBirthdayToday;

if (applyPromotionalPrice) 
{

}
```

O quanto de conhecimento de programação esse trecho envolve? Praticamente nada, é possível entender que o bloco do if será somente executado para o caso de preço promocional. Por que? Porque as variáveis estão carregando semântica, está próximo da **linguagem humana**.

O cliente tem menos de 10 anos **ou** o cliente tem mais de 60 anos **ou** é aniversario do cliente. **Está literalmente escrito desta forma**. Da maneira anterior, temos que "resolver" uma álgebra booleana, enquanto do modo atual temos que "interpretar" uma "frase", graças as variáveis booleanas.

> Não sou favorável a deixar as regras "soltas" como nos códigos acima, poderíamos seguir por outras linhas.

## 3. Aproxime-se da linguagem humana

Você pode programar em C#, JavaScript, Dart, Ruby, Python, Java, etc. Ainda que a gente não use a mesma linguagem de programação, temos um ponto em comum bem forte: **falamos a mesma língua**.

Tomarei liberdade para interpretar o papel de Mr. Obvious aqui. Se um código deve ser entendido por todos, por que não refletimos um pouquinho sobre como nos comunicamos com a nossa língua?

Peça para um desenvolvedor C# criar a seguinte data: 31 de dezembro de 2020. Você terá algo similar a este código:

```csharp
var date = new DateTime(2020, 12, 31);
```

Não é difícil de entender qual é data olhando para o código, mas será que uma pessoa sem conhecimento prévio em programação conseguiria entender? Talvez sim, talvez não.

Agora, vejamos como podemos deixar isso pouquinho mais claro:

```csharp
var date = 31.December(2020);
```

E ai, não está mais próximo da linguagem humana? Eu acredito que está claro o suficiente para qualquer pessoa entender, mesmo que nunca tenha programado.

Ainda se tratando sobre tempo:

```csharp
// from
new DateTime(2020, 12, 31);
// to
31.December(2020);

// from 
TimeSpan.FromMilliseconds(1);
// to
1.Milliseconds();

// from
TimeSpan.FromSeconds(30);
// to
30.Seconds();

// from
TimeSpan.FromMinutes(15);
// to
15.Minutes();

// from
TimeSpan.FromHours(2);
// to
2.Hours();

// from
TimeSpan.FromDays(7);
// to
7.Days();
```

## 4. Evite declarações condicionais triviais com funções de alta ordem

"De vez em sempre" nos deparamos com fluxos mais simples que resultam em basicamente um if-else:

```csharp
string text = null;

if (text != null)
    WriteLine(text)
else 
    WriteLine("text is required")
```

Nestes casos de condicionais triviais, eu prefiro outra abordagem:

```csharp
string text = null;

text
    .NotNull()
    .Then(() => WriteLine(text))
    .Otherwise(() => WriteLine("text is required"))
```

## 5. Use funções de alta ordem para remover valores de uma string

Ah, quem nunca foi ingênuo de usar o replace sem dó ou concatenar strings como se não houvesse amanhã?

Lembro como se fosse ontem, um trabalho de faculdade em que meu código estava demorando cerca de 10 minutos para guardar uma serie de valores de ordenação, simplesmente por não usar String builder. Após entender o motivo e concertar a cagada, o código passou a ser executado instantaneamente. Não concatene strings em laços com milhões de interações, talvez não seja performático, talvez.

Em resumo, caso você não esteja ciente, **strings são imutáveis**. Isso não quer dizer que você não possa reatribuir um valor numa variável do tipo string, ou que você não possa concatenar ou substituir parte de uma string. Quer dizer que sempre que você "alterar" uma string, você está alocando mais uma em memória, essa era a causa do meu código estar lento, milhões de instancias estavam sendo alocadas durante a execução.

Esse cenário ocorria para o caso de concatenação, entretanto, o mesmo pode ocorrer quando usamos o replace, pois o replace retorna uma nova string, afinal string é imutável.

Então ao invés de fazer isso:

```csharp
string text =
    "123 text"
        .Replace("123", "")
        .Replace(" ", "");
```

Podemos fazer isso:

```csharp
"123 text"
    .Remove(value => value.Letter() || value.Whitespace());
```

Na pratica será feita apenas uma alocação de string, mesmo que ocorra mais de uma substituição, pois um predicado pode conter mais de um critério de remoção e a instância só é criada após o predicado ser aplicado, portanto um critério de remoção ou 300 irão resultar em uma instância da mesma forma, diferente do replace.

Além disso, desta forma é mais dinâmico, com replace você precisa saber exatamente o que você quer substituir, value.Number() vai valer para qualquer número, não ficará limitado apenas para 123.

Ainda assim, **se você já sabe que a sua string irá sofrer mutações, compensa mais usar String builder, portanto, use a estrutura correta**.

## Palavras finais

O Pandemonium trás diversos outros métodos de extensão não citados aqui, porém o objetivo deste artigo é trazer uma reflexão sobre como escrevemos o nosso código, não apresentar a biblioteca em si.

Como reflexão, deixo duas indagações:

Quão distante está uma linguagem de programação de uma língua (como a língua inglesa)?
Será que escrever um texto com clareza é muito diferente de escrever um código claro?

