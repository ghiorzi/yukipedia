```json
{
    "title": "Sobrecarga de operador",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Introdução

Sobrecarga é um conceito costumeiro no universo da programação, no entanto, estamos mais familiarizados com essa feature conduzida a construtores e métodos.

#### Mas primeiramente, o que é sobrecarga de método e sobrecarga de construtor?

A sobrecarga de método é um artifício que permite uma classe ter mais de um método com o mesmo nome, contanto que seus argumentos sejam dessemelhantes. O mesmo se aplica aos construtores, desde que seus argumentos sejam distintos. Vejamos alguns exemplos no seguinte trecho:

```csharp
public class OverloadSample
{
    public OverloadSample() { }

    // Sobrecarga de construtor #1
    public OverloadSample(string something) { }

    public int Get(int id)
        => 1;

    // Sobrecarga de método #1: Mesma quantidade de parâmetros, porém com tipagem distinta.
    public int Get(string username)
        => 2;
    
    // Sobrecarga de método #2: Retorno com tipagem diferente, mas com quantidade de parâmetros divergente.
    public string Get(string username, string password)
        => "3";
}
```

## 1. Sobrecarga de operador

Sobrecarga é um conceito costumeiro no universo da programação, no entanto, estamos mais familiarizados com essa feature conduzida a construtores e métodos.

Como supramencionado, sobrecarga é um conceito deveras popular, entretanto, nem todos os programadores estão cônscios que sobrecarregar um operador é uma ação plenamente exequível. 

Para aplicar este conceito, iremos trabalhar em cima de um exemplo demasiadamente simples: iremos criar uma classe chamada **NonNullOrEmptyString**, esta classe será responsável por representar uma string que não pode ser nula nem vazia.

```csharp
public class NonNullOrEmptyString 
{
    private string Value { get; }

    public NonNullOrEmptyString(string value)
    {
        if (string.IsNullOrEmpty(value))
            throw new ArgumentException("Value cannot be null or empty.");
        
        Value = value;
    }
}
```

Estamos acostumados a criar um objeto através da palavra-chave **new**, logo, em primeiro momento, pensaríamos em criar nosso objeto da seguinte forma:

```csharp
NonNullOrEmptyString sample = new NonNullOrEmptyString("Hello");
```

Contudo, se eu disser: podemos criar esse objeto de uma maneira mais simplificada, inclusive, atribuindo a string “Hello” diretamente ao objeto. O que você diria?

## 2. Operadores de conversão

Antes de tudo, o que é conversão? 

Conversão é a ação que permite que um valor de um tipo seja transformado em outro tipo qualquer. Uma conversão pode ser categorizada como **implícita*** ou **explícita**. 

A conversão **implícita** ocorre de modo imperceptível sob as condições de haver um operador (de conversão implícita) definido bem como a compatibilidade entre os dois tipos. 

Já a conversão **explícita** intercorre quando os dois tipos não são integralmente compatíveis, portanto, é requerido que o tipo de origem seja prefixado com um operador de conversão (famigerado **casting**).

Então, para começar, que tal sobrecarregarmos nossos primeiros operadores de conversão? 

```csharp
public static implicit operator NonNullOrEmptyString(string @string)
    => new NonNullOrEmptyString(@string);

public static explicit operator NonNullOrEmptyString(NonNullOrEmptyString @string)
    => @string.Value;
```

Muito bem, agora que temos nossas primeiras sobrecargas podemos criar nossos objetos da seguinte maneira:

```csharp
// Implicit conversion
NonNullOrEmptyString firstText = "Hello";
// Explicit conversion
string secondText = (string) new NonNullOrEmptyString("Yahallo");
```

Criamos nossos objetos com conversão implícita, desta forma, é praticável atribuir uma string diretamente para nossos objetos do tipo **NonNullOrEmptyString**. Podemos também transmutar nosso objeto para uma string, através de uma conversão explícita previamente definida em nossa classe. 

> Lembre-se: estamos falando de sobrecargas, portanto, podemos definir diversas sobrecargas de operadores.

Vamos supor que agora quiséssemos comparar se o valor de uma **NonNullOrEmptyString** é igual ou diferente de outra **NonNullOrEmptyString.** Bom, estamos falando de objetos, então se os compararmos diretamente será levado em conta a referência deles e não os seus valores. Logo, a opção que nos resta é fazer essa comparação da seguinte forma, certo?

```csharp
Console.WriteLine(firstText.Value == secondText.Value);
Console.WriteLine(firstText.Value != secondText.Value);
```

**Errou, errou feio, errou rude**. Temos ainda a opção de sobrescrever o método Equals, oriundo da memorável classe **Object**, mas não faremos desta forma, pois o assunto do artigo é sobrecarga, não sobrescrita.

Então o que podemos fazer... Que tal uma sobrecarga nos operadores == e !=?

```csharp
public static bool operator == (NonNullOrEmptyString a, NonNullOrEmptyString b)
    => return a.Value.Equals(b.Value);

public static bool operator != (NonNullOrEmptyString a, NonNullOrEmptyString b)
    => !(a == b);
```

Legal, agora podemos fazer a comparação de valores sem precisar acessar a propriedade Value dos nossos objetos:

```csharp
Console.WriteLine(firstText == secondText);
Console.WriteLine(firstText != secondText); 

Console.WriteLine(firstText == "Hello");
Console.WriteLine(firstText != "Hello");
```

E se quiséssemos concatenar nossos objetos, poderíamos? Certamente, basta 
sobrecarregar nosso operador **+**:

```csharp
public static NonNullOrEmptyString operator + (NonNullOrEmptyString a, NonNullOrEmptyString b)
    => new NonNullOrEmptyString(a.Value + b.Value);
```

Assim, podemos:

```csharp
Console.WriteLine(firstText + secondText);
```

Para ultimar a brincadeira vamos sobrecarregar os seguintes operadores: menor, maior, menor-igual e maior-igual.

```csharp
public static bool operator < (NonNullOrEmptyString a, NonNullOrEmptyString b)
    => a.Value.Length < b.Value.Length;

public static bool operator > (NonNullOrEmptyString a, NonNullOrEmptyString b)
    => a.Value.Length > b.Value.Length;

public static bool operator <= (NonNullOrEmptyString a, NonNullOrEmptyString b)
    => a.Value.Length <= b.Value.Length;

public static bool operator >= (NonNullOrEmptyString a, NonNullOrEmptyString b)
    => a.Value.Length >= b.Value.Length;
```

Portanto:

```csharp
Console.WriteLine(firstText > secondText);
Console.WriteLine(firstText < secondText);

Console.WriteLine(firstText >= secondText);
Console.WriteLine(firstText <= secondText);
```

## Conclusão

Neste artigo foi realizado a demonstração de alguns operadores que permitem a operação de sobrecarga, no entanto, vale enfatizar que é possível também sobrecarregar outros operadores.