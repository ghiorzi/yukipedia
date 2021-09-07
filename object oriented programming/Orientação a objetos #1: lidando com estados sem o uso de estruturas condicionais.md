```json
{
    "title": "Lidando com estados sem o uso de estruturas condicionais",
    "author": "Rodrigo Ghiorzi",
    "languages": ["pt-BR"]
}
```

## Introdução

Há quem diga dominar o paradigma orientado a objetos e há quem considere questionável esta assertiva. Em qual destes grupos você se encaixa?

Apesar da orientação a objetos ser imensamente afamada, pode-se exprimir que é relativamente fácil esbarrar em códigos "orientados a objetos" escassos de objetos. Esta escassez é frequentemente preenchida com aspectos imperativos em demasia.

Permita-me, através deste artigo, expor um cenário para reflexionarmos sobre orientação a objetos.

## 1. Cenário

Considere um simulador de banco em que seja possível **depositar**, **sacar**, **fechar**, **ativar** e **congelar** uma **conta**.

- Quando criada, uma conta tem seu estado valorado como ativa e seu saldo é equivalente a zero;
- Quando fechada, uma conta não deve realizar as operaçōes depositar e sacar, entretanto, uma conta fechada pode alterar seu estado para ativa ou congelada;
- Quando congelada, uma conta pode realizar as operaçōes depositar e sacar, no entanto, ao realizar algumas destas operaçōes, seu estado deve alterar para ativa. Naturalmente, uma conta congelada pode alterar seu estado para ativa ou fechada;
- Quando ativa, uma conta pode realizar as operaçōes depositar e sacar. Similarmente, uma conta ativa pode alterar seu estado para congelada ou fechada;
- Não há necessidade de validar o valor requisitado em um saque (não é foco do artigo, manteremos o mais simples possível);
  
**Atenda os requisitos acima sem utilizar flags booleanas e estruturas condicionais**.

Qual a sua solução para este problema?

### 1.1 Resolução

Primeiramente, **esqueça que flags, if, switch existem**. Pense em objetos.

- Há três estados que uma conta pode assumir, iremos representá-los com objetos. Cada estado será um objeto;
- Há dois comportamentos definidos de forma explícita: depositar e sacar. Estes comportamentos variam de acordo com o estado da conta, ou seja, a implementação de cada estado ficará responsável por representar a ação esperada destes comportamentos;
- Há três comportamentos definidos de forma implícita: ativar uma conta, fechar uma conta e congelar uma conta. Outra vez, a implementação de cada estado ficará responsável por reproduzir estas ações;

Seguindo estes princípios, podemos definir um contrato, o qual todo estado deve assinar:

```csharp
internal interface IState
{
    IState Activate();
    IState Close();
    IState Deposit(Action onDeposit);
    IState Freeze();
    IState Withdraw(Action onWithdraw);
}
```

A partir de então, partimos para a implementação do estado fechado:

```csharp
internal class Closed : IState
{
    public IState Activate()
        => new Active();

    public IState Close()
        => this;

    public IState Deposit(Action onDeposit)
        => this;

    public IState Freeze()
        => new Frozen();

    public IState Withdraw(Action onWithdraw)
        => this;
}
```

- Activate: uma conta fechada pode alterar seu estado para ativa. Ao ativar uma conta que estava fechada, um objeto (Active) é retornado;
- Close: Ao "fechar" uma conta já fechada, nada é feito, pois a mesma já estava fechada, portanto, é retornado o próprio objeto;
- Deposit: uma conta fechada não deve realizar a operação de depósito, neste caso nenhum comportamento é executado, além de retornar o próprio objeto;
- Freeze: uma conta fechada pode alterar seu estado para congelada. Ao congelar uma conta que estava fechada, um objeto (Frozen) é retornado;
- Withdraw: uma conta fechada não deve realizar a operação de saque, neste caso nenhum comportamento é executado, além de retornar o próprio objeto.

Até aqui, o que estamos usando mesmo? **Objetos**!

Implementação do estado congelado:

```csharp
internal class Frozen : IState
{
    public IState Activate()
        => new Active();
     
    public IState Close()
        => new Closed();

    public IState Deposit(Action onDeposit)
    {
        onDeposit();

        return Activate();
    }

    public IState Freeze()
        => this;

    public IState Withdraw(Action onWithdraw)
    {
        onWithdraw();

        return Activate();
    }
}
```

- Activate: uma conta congelada pode alterar seu estado para ativa. Ao ativar uma conta que estava congelada, um objeto (Active) é retornado;
- Close: uma conta congelada pode alterar seu estado para fechada. Ao fechar uma conta que estava congelada, um objeto (Closed) é retornado;
- Deposit: uma conta congelada pode realizar a operação de depósito, entretanto, ao realizar esta operação, seu estado deve alterar para ativa. Portanto, após a ação ser feita, um objeto (Active) é retornado;
- Freeze: Ao "congelar" uma conta já congelada, nada é feito, pois a mesma já estava congelada, portanto, é retornado o próprio objeto;
- Withdraw: uma conta congelada pode realizar a operação de saque, entretanto, ao realizar esta operação, seu estado deve alterar para ativa. Portanto, após a ação ser feita, um objeto (Active) é retornado;

Implementação do estado ativo:

```csharp
internal class Active : IState
{
    public IState Activate()
        => this;

    public IState Close()
        => new Closed();

    public IState Deposit(Action onDeposit)
    {
        onDeposit();

        return this;
    }

    public IState Freeze()
        => new Frozen();

    public IState Withdraw(Action onWithdraw)
    {
        onWithdraw();

        return this;
    }
}
```

- Activate: Ao "ativar" uma conta já ativa, nada é feito, pois a mesma já estava ativa, portanto, é retornado o próprio objeto;
- Close: uma conta ativa pode alterar seu estado para fechada. Ao fechar uma conta que estava ativa, um objeto (Closed) é retornado;
- Deposit: uma conta ativa pode realizar a operação de depósito;
- Freeze: uma conta ativa pode alterar seu estado para congelada. Ao congelar uma conta que estava ativa, um objeto (Frozen) é retornado;
- Withdraw: uma conta congelada pode realizar a operação de saque;

Enfim, isto feito, podemos codificar a nossa conta:

```csharp
public readonly struct Amount
{
    public readonly decimal Value 
        => _value; 

    private readonly decimal _value;

    private Amount(decimal amount)
        => _value = amount;
    
    public static implicit operator Amount(decimal amount) 
        => new Amount(amount);
    
    public static explicit operator decimal(Amount amount) 
        => amount._value;

    public bool LessThanOrEqualTo(decimal value)
        => _value.LessThanOrEqualTo(value);
} 

public readonly struct Balance
{
    public readonly decimal Value { get; }

    private Balance(decimal balance)
        => Value = balance;

    public static implicit operator Balance(decimal balance)
        => new Balance(balance);

    public static explicit operator decimal(Balance balance)
        => balance.Value;

    public Balance Increase(Amount amount)
        => new Balance(Value + amount.Value);

    public Balance Decrease(Amount amount)
        => new Balance(Value - amount.Value);

    public bool GreaterOrEqualTo(Amount amount)
        => amount.LessThanOrEqualTo(Value);
}

public class Account
{
    public Balance Balance { get; private set; }
    
    private IState _state;
    private Action<Amount> _deposit;
    private Action<Amount> _withdraw;

    public Account()
    {
        this._state = new Active();

        this._deposit = amount => Balance = Balance.Increase(amount);
        this._withdraw = amount => Balance = Balance.Decrease(amount);
    } 
    
    public Account Activate()
        => Update(() => _state.Activate());

    public Account Close()
        => Update(() => _state.Close());

    public Account Freeze()
        => Update(() => _state.Freeze());

    public Account Deposit(Amount amount)
        => Update(() => _state.Deposit(() => _deposit(amount)));

    public Account Withdraw(Amount amount)
        => Update(() => _state.Withdraw(() => _withdraw(amount)));

    private Account Update(Func<IState> onChange)
    {
        _state = onChange();

        return this;
    }
}
```
A magia está presente na orquestração de estado, onde um objeto do tipo Account se comunica com um objeto do tipo IState, através de uma abordagem polimórfica.

Considerando que cada estado (objeto) mantém os próprios comportamentos, é necessário, portanto, apenas instrumentar a troca de estado (objeto) mediante ao objeto do tipo Account.

Na prática o uso deste objeto poderá ser feito da seguinte maneira:

```csharp
decimal balance =
    new Account() // starts as active account
    .Deposit(200) // 200
    .Deposit(300) // 500
    .Close() // it turns to close, so it cannot deposit or withdraw
    .Deposit(5000) // do nothing
    .Withdraw(2000) // do nothing
    .Freeze() // it turns to freeze, so it will turn to active if deposit or withdraw 
    .Deposit(500) // 1000 then turn to active 
    .Freeze() // it turns to freeze, so it will turn to active if deposit or withdraw 
    .Close() // it turns to close, so it cannot deposit or withdraw
    .Deposit(1000) // do nothing
    .Activate()
    .Deposit(2000) // 3000 
    .Balance
    .Value;

WriteLine(balance); // 3000
```

## Palavras finais

Quão orientado a objetos é o código que tu escreves? Você acredita realmente tirar um bom proveito dos recursos que a orientação a objetos prove para ti?