# Consulta dos Sellers

**Database Type:** NoSql

**Database:** JsmCache

**Collection:** Sellers

- O código a seguir obtem todos os sellers enviados na request na lista de objetos do tipo **CustomerInfoDto**

```csharp
private Task<List<CachedSeller>> GetSellersAsync(IEnumerable<Guid> sellerIds)
{
    return _sellerCacheRepository.GetAllAsync(x => sellerIds.Contains(x.Id));
}
```

- Objeto **CachedSeller**

```csharp
public class CachedSeller : CacheEntity
{
    public string CodeSeller { get; set; }
    public string NameCompany { get; set; }
    public string Name { get; set; }
    public string CNPJ { get; set; }
    public string Email { get; set; }
    public string Telephone { get; set; }
    public string Segment { get; set; }
    public string City { get; set; }
    public string Region { get; set; }
    public string PostalCode { get; set; }
    public string Address { get; set; }
    public bool AllowActiveDirectory { get; set; }
    public string GUIDImage { get; set; }
    public bool HighlightAsNew { get; set; }
    public string PartnerCodeReceiver { get; set; }
    public bool ShowLocalCarousel { get; set; }
    public bool ShowLinxCarousel { get; set; }
    public string Hub2bSellerCode { get; set; }
    public bool AllowLotePriceSearch { get; set; }
    public int DaysConsideredOrder { get; set; }
    public bool NonCustomerAllowed { get; set; }
    public List<CachedSellerParameter> Parameters { get; set; }
    public List<CachedSellerPaymentForm> PaymentForms { get; set; }
    public List<CacheSellerTax> SellerTaxes { get; set; }
    public List<CachedSellerDistributor> DistributorTo { get; set; }
    public List<CachedPriceVariation> PriceVariations { get; set; }
}
```

- Objetos que fazem parte da composição do objeto **CachedSeller**

```csharp
public class CachedSellerParameter : CacheEntity
{
    public string Code { get; set; }
    public string Value { get; set; }
    public string Description { get; set; }
}

public class CachedSellerPaymentForm : CacheEntity
{
    public string CodePayment { get; set; }
    public string DescriptionPayment { get; set; }
    public int? DaysRemovedFromSchedule { get; set; }
    public string Incoterm { get; set; }
    public bool Default { get; set; }
    public List<CachedSellerPaymentCondition> PaymentConditions { get; set; }
}

public class CachedSellerPaymentCondition
{
    public string CodePaymentTerm { get; set; }
    public string DescriptionPaymentTerm { get; set; }
    public string Portion { get; set; }
}

public class CacheSellerTax : CacheEntity
{
    public double TaxPercent { get; set; }
    public string Brand { get; set; }
    public string SalesPlan { get; set; }
    public string PaymentPlan { get; set; }
}

public class CachedSellerDistributor : CacheEntity
{
    public Guid SellerId { get; set; }
    public string SellerDistributorId { get; set; }
}

public class CachedPriceVariation
{
    public string SellerId { get; set; }
    public CachedPriceVariationType Type { get; set; }
    public string PaymentForm { get; set; }
    public string PaymentFormDescription { get; set; }
    public string PaymentTermCode { get; set; } 
    public string Supplier { get; set; }
    public int Condition { get; set; }
    public string ConditionDescription { get; set; }
    public string ParameterType { get; set; }
    public object Parameter { get; set; }
    public int Installment { get; set; }
    public decimal PriceVariation { get; set; }

    public static string GetPriceVariationEnumTypeDescription(CachedPriceVariationType type)
    {
        switch (type)
        {
            case CachedPriceVariationType.PAYMENT_CONDITION:
                return "payment-condition";
            
            case CachedPriceVariationType.CATEGORY:
                return "category";

            case CachedPriceVariationType.SKU:
                return "sku";

            case CachedPriceVariationType.CNPJ:
                return "cnpj";

            case CachedPriceVariationType.RULE:
                return "rule";

            default:
                throw new Exception("Tipo de variação de preço inválido.");
        }
    }

    public static CachedPriceVariationType GetPriceVariationEnumType(string type)
    {
        switch (type)
        {
            case "payment-condition":
                return CachedPriceVariationType.PAYMENT_CONDITION;

            case "category":
                return CachedPriceVariationType.CATEGORY;

            case "sku":
                return CachedPriceVariationType.SKU;

            case "cnpj":
                return CachedPriceVariationType.CNPJ;
            
            case "rule":
                return CachedPriceVariationType.RULE;

            default:
                throw new Exception("Tipo de variação de preço inválido.");
        }
    }
}
```

- Classe base para todos os objetos do Mongo

```csharp
public class CacheEntity
{
    [MongoDB.Bson.Serialization.Attributes.BsonId]
    public Guid Id { get; set; } = Guid.NewGuid();
    public int Status { get; set; } = 1;
    public DateTime CreationDate { get; set; } = DateTime.Now.ToBrazilianTimezone();
}
```

- Método de extenção **ToBrazilianTimezone**

```csharp
public static DateTime ToBrazilianTimezone(this DateTime dateTime)
{
    TimeZoneInfo targetTimeZone;

    if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
    {
        targetTimeZone = TimeZoneInfo.FindSystemTimeZoneById("E. South America Standard Time");
    }
    else
    {
        targetTimeZone = TimeZoneInfo.FindSystemTimeZoneById("America/Sao_Paulo");
    }

    var targetDatetime = TimeZoneInfo.ConvertTime(dateTime, targetTimeZone);

    return targetDatetime;
}
```