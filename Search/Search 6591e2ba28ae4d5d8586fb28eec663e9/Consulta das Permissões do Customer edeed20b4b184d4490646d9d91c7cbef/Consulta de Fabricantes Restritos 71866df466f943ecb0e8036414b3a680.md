# Consulta de Fabricantes Restritos

**Database Type:** Sql

**Database:** JSMCatalog

T**able:** ProducerRestrictionRule

### Objeto GetRestrictedProducersParameters

```csharp
public class GetRestrictedProducersParameters
{
    public IEnumerable<Guid> SellersIds { get; set; }
    public string CustomerPostalCode { get; set; }
    public string CustomerCnpj { get; set; }
}
```

### O Método

```csharp
public Task<List<ProducerRestrictedDto>> GetRestrictedProducers(GetRestrictedProducersParameters parameters)
{
    int.TryParse(parameters.CustomerPostalCode.OnlyNumbers(), out var postalCode);
    var cnpj = parameters.CustomerCnpj.OnlyNumbers();

    return _sqlContext.ProducerRestrictionRule
        .Where(rule => parameters.SellersIds.Contains(rule.SellerId) && 
                      (rule.Cnpjs.Any(c => c.Cnpj == cnpj) || rule.Regions.Any(reg => postalCode >= reg.InitialPostalCode && postalCode <= reg.FinalPostalCode)))
        .SelectMany(rule => rule.Producers, (rule, producer) => new ProducerRestrictedDto(producer.ProducerId, rule.SellerId))
        .Distinct()
        .ToListAsync();
}
```

### Detalhamento

No trecho de código a seguir, se obtem os parametros `postalCode` e `cnpj` para realizar a consulta(**apenas números**).

```csharp
 int.TryParse(parameters.CustomerPostalCode.OnlyNumbers(), out var postalCode);
 var cnpj = parameters.CustomerCnpj.OnlyNumbers();
```

No trecho de código a seguir é realidada a consulta no banco na tabela **ProducerRestrictionRule**

```csharp
_sqlContext.ProducerRestrictionRule
                .Where(rule => parameters.SellersIds.Contains(rule.SellerId) 
                            && rule.Cnpjs.Any(c => c.Cnpj == cnpj) 
                            || rule.Regions.Any(reg => postalCode >= reg.InitialPostalCode && postalCode <= reg.FinalPostalCode)))
                .SelectMany(rule => rule.Producers, (rule, producer) => new ProducerRestrictedDto(producer.ProducerId, rule.SellerId))
                .Distinct()
                .ToListAsync();
```

No filtro que é aplicado na tabela **ProducerRestrictionRule** o seller da **Rule** deve estar contido na lista de sellers passados por parâmetro ****e a **Rule** deve conter o `cnpj` do **Customer,**

Ou nas **Regions** da **Rules,** o `postalCode` do **Customer** deve estar entre o `InitialPostalCode` e o `FinalPostalCode` (**Beetwen**).

```csharp
    parameters.SellersIds.Contains(rule.SellerId) && rule.Cnpjs.Any(c => c.Cnpj == cnpj) 
 || rule.Regions.Any(reg => postalCode >= reg.InitialPostalCode && postalCode <= reg.FinalPostalCode))
```

- No trecho de código a seguir é criada uma lista distinta do objeto **ProducerRestrictedDto** agrupando por `producer` e suas respectivas **Rules**

```csharp
.SelectMany(rule => rule.Producers, (rule, producer) => new ProducerRestrictedDto(producer.ProducerId, rule.SellerId))
.Distinct()
.ToListAsync();
```

- O objeto **ProducerRestrictedDto**

```csharp
public class ProducerRestrictedDto
{
    public ProducerRestrictedDto(Guid producerId, Guid sellerId)
    {
        ProducerId = producerId;
        SellerId = sellerId;
    }

    public Guid SellerId { get; private set; }
    public Guid ProducerId { get; private set; }
}
```

- O objeto **ProducerRestrictionRule**

```csharp
public class ProducerRestrictionRule : Entity
{
    public string Name { get; set; }
    public Guid SellerId { get; set; }
    public ICollection<ProducerRestrictionRuleProducer> Producers { get; set; }
    public ICollection<ProducerRestrictionRuleRegion> Regions { get; set; }
    public ICollection<ProducerRestrictionRuleCnpj> Cnpjs { get; set; }
}
```

- Objetoc que compõem o objeto **ProducerRestrictionRule**

```csharp
public class ProducerRestrictionRuleProducer
{
    public Guid RuleId { get; set; }
    public Guid ProducerId { get; set; }
    public ProducerRestrictionRule Rule { get; set; }
    public Producer Producer { get; set; }
}

public class Producer : Entity
{
    public string Name { get; set; }
    public string Slug { get; set; }
}

public class ProducerRestrictionRuleRegion
{
    public Guid RuleId { get; set; }
    public int InitialPostalCode { get; set; }
    public int FinalPostalCode { get; set; }
    public string Name { get; set; }
    public ProducerRestrictionRule Rule { get; set; }
}

public class ProducerRestrictionRuleCnpj
{
    public Guid RuleId { get; set; }
    public string Cnpj { get; set; }
    public ProducerRestrictionRule Rule { get; set; }
}
```

- Classe base para dos objetos do SqlServer

```csharp
public abstract class Entity
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public int Status { get; set; } = 1;
    public DateTime CreationDate { get; set; } = DateTime.Now.ToBrazilianTimezone();

    protected Entity()
    {
        CreationDate = DateTime.Now.ToBrazilianTimezone();
    }
}
```