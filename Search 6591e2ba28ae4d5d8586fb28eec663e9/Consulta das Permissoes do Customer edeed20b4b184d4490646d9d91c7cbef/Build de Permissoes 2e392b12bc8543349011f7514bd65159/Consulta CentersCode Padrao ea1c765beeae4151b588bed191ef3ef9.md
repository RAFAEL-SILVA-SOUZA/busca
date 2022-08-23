# Consulta CentersCode Padrao

### O Método

```csharp
public async Task<IEnumerable<string>> GetDefaultCentersCode(IEnumerable<CachedCustomer> cachedCustomer, string customerCnpj, ICollection<Guid> offlineSellersId, IEnumerable<string> cnpjs = null)
{
    var checkoutSettings = await GetCustomerCheckoutSettings(offlineSellersId, customerCnpj, cnpjs);

    if (checkoutSettings.Any())
    {
        return checkoutSettings.Select(checkout => checkout.CenterCode);
    }
    else
    {
        var customerPostalCode = cachedCustomer.First().PostalCode;
        var defaultCenters = await _sphinxClient.GetDefaultCenters(customerPostalCode, offlineSellersId);
        return defaultCenters.Select(center => center.CenterCode);
    }
}
```

[Consultar CheckoutSettings do Customer](Consulta%20CentersCode%20Padrao%20ea1c765beeae4151b588bed191ef3ef9/Consultar%20CheckoutSettings%20do%20Customer%207f5b0321104e4df9973676faf9562d0f.md)

Caso exista **CheckoutSettings**, é retornada uma lista de Centros de Distribuição contidas nos **CheckoutSettings.**

```csharp
if (checkoutSettings.Any())
{
    return checkoutSettings.Select(checkout => checkout.CenterCode);
}
```

Caso **NÃO** exista **CheckoutSettings,** uma chamada http é realizada para a API `SPHINX`  para objter os Centros de Distrubuição padrão.

```csharp
else
{
    var customerPostalCode = cachedCustomer.First().PostalCode;
    var defaultCenters = await _sphinxClient.GetDefaultCenters(customerPostalCode, offlineSellersId);
    return defaultCenters.Select(center => center.CenterCode);
}
```

### Detalhamento do método GetDefaultCenters

 Este é um método simples utilizado para obter os **Centros de Distribuição** padrão através dos **SellerIds** para um determinado **CEP(PostalCode)**

```csharp
public async Task<IEnumerable<Center>> GetDefaultCenters(string cep, IEnumerable<Guid> sellersId)
{
    var sellersIdToString = sellersId.Select(id => $"sellerIds={id}").Append($"cep={cep.Replace("-", string.Empty)}");
    var parsedParameters = string.Join("&", sellersIdToString);
    var response = await CreateClient().GetAsync(CreateUrl($"{_getDefaultCenters}?{parsedParameters}"));
    return await Transform(response);
}
```

### Request exemplo

```csharp
[https://dev-api-loja.juntossomosmais.com.br/sphinx/v1/centers/default?sellerIds=2cb3e721-2e5c-4ea9-bdaa-a79954629cd3&cep=03559030](https://hml-api-loja.juntossomosmais.com.br/sphinx/v1/centers/default?sellerIds=2cb3e721-2e5c-4ea9-bdaa-a79954629cd3&cep=03559030)
```

### Response exemplo

```json
{
    "success": true,
    "data": [{
            "centerCode": "ET33",
            "cnpjCpf": "00000000000000",
            "socialName": "S.A EM RECUPERACAO JUDICIAL",
            "address": "Rua Presidente ",
            "postalCode": "83411-000",
            "city": "Colombo",
            "region": "PR",
            "billingTerm": 15,
            "maximumCifCapacity": 9999999999999.0,
            "maximumFobCapacity": 32000.0,
            "unitMeasureMaximumCapacity": "peso(KG)",
            "minimumOrderCif": 0.0,
            "measureUnitMinimumOrderCif": null,
            "sellerId": "2cb3e721-2e5c-4ea9-bdaa-dsfdsfsdf",
            "minimumOrderType": 0,
            "minimumOrderValue": 5500.0000,
            "stock": null,
            "freight": [{
                    "centerCode": "ET33",
                    "initialPostalCode": "565464564",
                    "finalPostalCode": "432423424",
                    "uf": "SP",
                    "mesoRegion": null,
                    "microRegion": null,
                    "city": null,
                    "deadline": 30,
                    "value": 650.0000,
                    "maxWeight": 3750.0000,
                    "minWeight": 3501.0000,
                    "id": "258530b1-7cee-419c-a604-sdfsdfsdf"
                }
            ],
            "id": "abb0e9b5-3c08-4bf2-abd9-d385d988e08c"
        }
    ]
}
```

### **Objeto Center**

```csharp
public class Center
{
    public Guid Id { get; set; }
    public string CenterCode { get; set; }
    public string CnpjCpf { get; set; }
    public string SocialName { get; set; }
    public string Address { get; set; }
    public string PostalCode { get; set; }
    public string City { get; set; }
    public string Region { get; set; }
    public int BillingTerm { get; set; }
    public double MaximumCifCapacity { get; set; }
    public double MaximumFobCapacity { get; set; }
    public string UnitMeasureMaximumCapacity { get; set; }
    public double MinimumOrderCif { get; set; }
    public string MeasureUnitMinimumOrderCif { get; set; }
}
```