# Informacoes do Customer

Página da alanise: [Informacoes do Customer](../Informacoes%20do%20Customer%20ba2f702c3656438f920b44dc7cedd1e0.md) 

### CustomerInfos

> Se o Id do seller for informado no objeto root da request, devemos obter os dados dos customers do seller informado.
Se não, serão utilizados os dados de todos os customers passados em um array no corpo da request.
> 

### Consulta de customers

> A consulta na collection de customers deve buscar por todos os códigos de customer passados por parâmetro e todos os selers passados por parâmetro?
> 

## Ponto de atenção:

***Acredito que o código a baixo esta incoerente, pois a segunda query sobrescreve a primeira e a terceita query sobrescreve a segunda.***

```csharp
public async Task<List<CachedCustomer>> GetCustomersAsync(List<CustomerInfoDto> customerInfos)
{
    var customerCodes = customerInfos.Select(x => x.CustomerCodeReceiver);
    var sellerIds = customerInfos.Select(x => x.SellerId);

    var customers = (await _cachedCustomerRepository
  **1ª**  => .GetAllAsync(x => customerCodes.Contains(x.CustomerCode) && sellerIds.Contains(x.Seller.Id)))
  2ª  => .Where(customer => customerInfos.Any(x => x.SellerId == customer.Seller.Id && x.CustomerCodeReceiver == customer.CustomerCode))
  3ª  => .Where(customer => _user.GetCnpjsFromToken().Contains(customer.Cnpj) && _user.GetSellersGuidIdFromToken().Contains(customer.Seller.Id))
         .ToList();

    return customers;
}
```

O método **GetAllAsync** vai ao banco, logo em seguida, em memória, é realizada outra consulta baseada no parâmetro **customerInfos** onde é validado o **SellerId** e o **CustomerCodeReceiver**, por fim, mais uma vez em memória, um filtro é aplicado com base nas informações do token.