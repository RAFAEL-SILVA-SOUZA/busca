# Consulta dos Customers

**Database Type:** NoSql

**Database:** JsmCache

**Collection:** Customers

O código a seguir obtem todos os **Customers** baseadosem uma lista de **CustomerInfoDto.**

### O Método

```csharp
public async Task<List<CachedCustomer>> GetCustomersAsync(List<CustomerInfoDto> customerInfos)
{
    var customerCodes = customerInfos.Select(x => x.CustomerCodeReceiver);
    var sellerIds = customerInfos.Select(x => x.SellerId);

    var customers = (await _cachedCustomerRepository
        .GetAllAsync(x => customerCodes.Contains(x.CustomerCode) && sellerIds.Contains(x.Seller.Id)))
        .Where(customer => customerInfos.Any(x => x.SellerId == customer.Seller.Id && x.CustomerCodeReceiver == customer.CustomerCode))
        .Where(customer => _user.GetCnpjsFromToken().Contains(customer.Cnpj) && _user.GetSellersGuidIdFromToken().Contains(customer.Seller.Id))
        .ToList();

    return customers;
}
```

### Variaveis do método

- São obtidos todos os **CustomerCodeReceiver** da lista de **CustomerInfoDto.**
- São obtidos todos os **SellerIds** da lista de **CustomerInfoDto.**

```csharp
 var customerCodes = customerInfos.Select(x => x.CustomerCodeReceiver);
 var sellerIds = customerInfos.Select(x => x.SellerId);
```

### É criada uma query em três etapas

- É realizado um filtro onde a collection deve conter todos os **CustomerCodes** e todos os **SellerIds.**

```csharp
 x => customerCodes.Contains(x.CustomerCode) && sellerIds.Contains(x.Seller.Id))
```

- O Custumer da collection do banco deve estar contido na lista de **CustomerInfoDto**, onde o `Seller.Id` do customer deve é Igual a o `SellerId` do objeto **CustomerInfoDto** e o `CustomerCode` do customer deve ser igual ao `CustomerCodeReceiver` do **CustomerInfoDto.**

```csharp
customerInfos.Any(x => x.SellerId == customer.Seller.Id && x.CustomerCodeReceiver == customer.CustomerCode))
```

- O Customer da collection do banco de possuir o `Cnpj` e o  `Seller.Id`   devem estar contidos no token da Http-Request nas Claims

```csharp
customer => _user.GetCnpjsFromToken().Contains(customer.Cnpj) && _user.GetSellersGuidIdFromToken().Contains(customer.Seller.Id)
```

- O objeto **CachedCustomer**

```csharp
public class CachedCustomer : CacheEntity
{
    public string Cnpj { get; set; }
    public string Cpf { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Telephone { get; set; }
    public string Fax { get; set; }
    public string City { get; set; }
    public string PostalCode { get; set; }
    public string Region { get; set; }
    public string Address { get; set; }
    public string District { get; set; }
    public string TaxJurisiction { get; set; }
    public string TransportationZone { get; set; }
    public string DatabaseSchema { get; set; }
    public string CustomerCode { get; set; }
    public CachedCustomerSeller Seller { get; set; }
    public List<CachedCenter> DistCenters { get; set; }
    public List<CachedVendor> Vendors { get; set; }
    public List<CachedParameter> Parameters { get; set; }
    public List<CachedPartner> Partners { get; set; }
    public List<CachedSalesArea> SalesAreas { get; set; }
    public List<CachedPaymentForm> PaymentForms { get; set; }
    public List<CachedPaymentCondition> PaymentConditions { get; set; }
    public int CnaeCode { get; set; }
}

```

- Objetos que fazem parte da composição do objeto **CachedCustomer**

```csharp

public class CachedCustomerSeller : CacheEntity
{
    public string NameCompany { get; set; }
    public string Name { get; set; }
}

public class CachedCenter : CacheEntity
{
    public string Code { get; set; }
    public string Name { get; set; }
    public List<CachedSalesArea> SalesAreas { get; set; }
    public bool Default { get; set; }
}

public class CachedVendor : CacheEntity
{
    public string PersonnelNumber { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Telephone { get; set; }
}

public class CachedParameter : CacheEntity
{
    public string Description { get; set; }
    public string Code { get; set; }
    public string Value { get; set; }
    public Guid ParameterId { get; set; }
}

public class CachedPartner : CacheEntity
{
    public string Code { get; set; }
    public string CustomerCode { get; set; }
    public string Name { get; set; }
    public List<CachedSalesArea> SalesAreas { get; set; }
    public Guid? PartnerCustomerId { get; set; }
}

public class CachedSalesArea : CacheEntity
{
    public string Organization { get; set; }
    public string OrganizationName { get; set; }
    public string DistributionChannel { get; set; }
    public string DistributionChannelName { get; set; }
    public string Division { get; set; }
    public string DivisionName { get; set; }
    public string CustomerGroup { get; set; }
    public string Incoterm { get; set; }
    public string SalesOffice { get; set; }
    public string PaymentTerm { get; set; }
}

public class CachedPaymentForm : CacheEntity
{
    public string Enterprise { get; set; }
    public string CodePayment { get; set; }
    public string DescriptionPayment { get; set; }
    public string Bank { get; set; }
    public int DaysRemovedFromSchedule { get; set; }
    public List<CachedSalesArea> SalesAreas { get; set; }
}

public class CachedPaymentCondition : CacheEntity
{
    public string CodePayment { get; set; }
    public string CodePaymentTerm { get; set; }
    public string DescriptionPaymentTerm { get; set; }
    public List<CachedSalesArea> SalesAreas { get; set; }
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