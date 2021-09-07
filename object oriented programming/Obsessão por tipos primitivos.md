```json
{
    "title": "Obsessão por tipos primitivos",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Introdução

O conceito de obsessão por tipos primitivos é auto-explicativo: uso excessivo de tipos primitivos. Parece ser algo notório, mas apenas parece. 

A obsessão por tipos primitivos nasce em momentos de obscurantismo e somente nos cientificamos de sua existência após cometer os mesmos equívocos incomensuráveis vezes.

## Cenário

Para ilustrar o tema criaremos uma classe chamada Account para simbolizar uma conta de acesso.

```csharp
public class Account { }
```

Para cadastrar uma conta no nosso sistema hipotético definiu-se que:

- Uma conta de acesso deve conter um nome de usuário;
- Uma conta de acesso deve conter um email;
- Por fim, uma conta de acesso deve conter uma senha.

Em seguida, nós temos a fabulosa ideia de criar todas essas propriedades como strings.

```csharp
public class Account 
{ 
    public Account(string username, string email, string password)
    {
        Username = username;
        Email = email;
        Password = password;
    }

    public string Username { get; private set; }
    public string Email { get; private set; }
    public string Password { get; private set; }
}
```

Até aí tudo bem, disse o frango na porta do forno \_(ヅ)_/

“Nome de usuário, email e senha são representações textuais, logo são strings”. Acontece que o mundo gira e vacilão roda. Senha não é uma representação textual qualquer, existem critérios que podem categorizá-la como valida ou invalida, segura ou insegura, da mesma forma que existem premissas que um email deve seguir.

Veja só, a bola de lama que a nossa classe está se tornando:

```csharp
public class Account 
{ 
    public Account(string username, string email, string password)
    {
        // validate username

        if (string.IsNullOrEmpty(email))
            throw new ArgumentException("Email is invalid.");

        if (!Regex.IsMatch(email, "Too lazy to write a regex"))
            throw new ArgumentException("Email is invalid.");

        // validate password

        Username = username;
        Email = email;
        Password = password;
    }
}
```

Ops, começamos a ter código duplicado, afinal precisamos validar o email novamente caso haja alguma tentativa de alterá-lo.

```csharp
public void ChangeEmail(string email)
{
    if (string.IsNullOrEmpty(email))
        throw new ArgumentException("Email is invalid.");

    if (!Regex.IsMatch(email, "Too lazy to write a regex"))
        throw new ArgumentException("Email is invalid.");

    Email = email;
}
```

Parece que email não é simplesmente um texto, não é mesmo? 

### Então, o que podemos mudar? 

Eu vejo algumas alternativas:

- Criar um helper para centralizar a lógica de validação de email e senha em métodos específicos (nope);
- Criar métodos de extensão para que a string passe a conter os métodos de validação de email e senha (nope);
- Criar uma classe chamada AccountValidator a qual será responsável por validar uma Account (nope).

E o que todas essas abordagens têm em comum? Elas assumem que não é responsabilidade da classe Account validar um email. E neste ponto eu concordo, realmente não acho que seja responsabilidade dela validar um email, porém algo muito importante está passando despercebido: 

> Desde quando é responsabilidade de uma string representar um email?

Estamos usando uma linguagem que trabalha com o paradigma orientado a objetos, então por que estamos tão obcecados em usar os tipos primitivos? Bora orientar o nosso código! 

Tudo o que precisamos fazer é criar um modelo de Email para dispor desta responsabilidade, além disso, podemos usufruir dos operadores implícitos para facilitar a criação dos objetos do tipo Email.

```csharp
public struct Email 
{
    private readonly string _value;

    private Email (string value)
        => _value = value;
    
    public static implicit operator Email (string value)
        => Create(value);
    
    public static Email Create(string email)
    {
        if (string.IsNullOrEmpty(email))
            throw new ArgumentException("Email is invalid.");

        if (!Regex.IsMatch(email, "Too lazy to write a regex"))
            throw new ArgumentException("Email is invalid.");
        
        return new Email(email);
    }
}
```

E assim preservamos a simplicidade para criar um email.

```csharp
Email email = "operator@implicito.com";
```

Pronto, temos uma representação de um email que se preocupa com as responsabilidades de um email. Doravante, a classe Account encarrega-se somente em ratificar se o email é nulo.

```csharp
public Account (Username username, Email email, Password password)
{   
    Username = username ?? throw new ArgumentException("Username cannot be null");
    Email = email ?? throw new ArgumentException("Email cannot be null");
    Password = password ?? throw new ArgumentException("Password cannot be null");
}
```

## Palavras finais

A classe Account passa a ficar menos carregada e nosso código começa a ganhar mais semântica.

Outros exemplos comuns de obsessão por tipos primitivos: CPF, CNPJ, IP, CEP e por ai vai.