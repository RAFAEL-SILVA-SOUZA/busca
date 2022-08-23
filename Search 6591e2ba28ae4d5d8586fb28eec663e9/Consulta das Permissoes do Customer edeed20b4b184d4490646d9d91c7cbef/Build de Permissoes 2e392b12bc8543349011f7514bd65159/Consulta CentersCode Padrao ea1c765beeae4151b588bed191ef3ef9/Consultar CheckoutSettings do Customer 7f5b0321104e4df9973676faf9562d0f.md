# Consultar CheckoutSettings do Customer

**Database Type:** NoSql

**Database:** JsmSales

**Collection:** CheckoutSettings

```csharp
private async Task<IEnumerable<CheckoutSettings>> GetCustomerCheckoutSettings(IEnumerable<Guid> offlineSellersId, string customerCnpj = null, IEnumerable<string> cnpjs = null)
{
    FilterDefinition<CheckoutSettings> checkoutSettingsInCnpjsFilter;
    var checkoutSettingsInSellersFilter = Builders<CheckoutSettings>.Filter.In(checkout => checkout.SellerId, offlineSellersId);

    if (cnpjs?.Any() ?? false)
        checkoutSettingsInCnpjsFilter = Builders<CheckoutSettings>.Filter.In(checkout => checkout.Cnpj, cnpjs);
    else
        checkoutSettingsInCnpjsFilter = Builders<CheckoutSettings>.Filter.Eq(checkout => checkout.Cnpj, customerCnpj);

    var checkoutFilter = Builders<CheckoutSettings>.Filter.And(checkoutSettingsInSellersFilter, checkoutSettingsInCnpjsFilter);
    return await _checkoutSettingsRepository.Find(checkoutFilter);
}
```

- No trecho de código a seguir é realizado o Build de uma Query  onde se obtem os sellers offline passados por parâmetro.

```csharp
var checkoutSettingsInSellersFilter = Builders<CheckoutSettings>.Filter.In(checkout => checkout.SellerId, offlineSellersId);
```

- Se os CNPJs passados por parâmetro possuirem valor, é realizado um Build de uma Query com todos os CNPJs, caso contrario, é realizado um Build de uma Query com o CNPJ do **Customer**

```csharp
if (cnpjs?.Any() ?? false)
   checkoutSettingsInCnpjsFilter = Builders<CheckoutSettings>.Filter.In(checkout => checkout.Cnpj, cnpjs);
else
   checkoutSettingsInCnpjsFilter = Builders<CheckoutSettings>.Filter.Eq(checkout => checkout.Cnpj, customerCnpj);
```

- O objeto **CheckoutSettings**

```csharp
public class CheckoutSettings
{
    public string Incoterm { get; set; }
    public string CenterCode { get; set; }
    public string Cnpj { get; set; }
    public Guid SellerId { get; set; }
}
```