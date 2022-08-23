# Consulta das Permissões do Customer

### O Método

```csharp
public async Task<List<CustomerPermission>> GetCustomersPermissionsAsync(string cnpj, IReadOnlyCollection<CachedCustomer> customers, IReadOnlyCollection<CachedSeller> sellers)
{
    var restrictedProducerIds = await _producerService.GetRestrictedProducers(new GetRestrictedProducersParameters
    {
        CustomerCnpj = cnpj,
        CustomerPostalCode = customers.Select(c => c.PostalCode).FirstOrDefault(),
        SellersIds = sellers.Select(s => s.Id)
    });

    var permissions = new List<CustomerPermission>(customers.Count);
    foreach (var customer in customers)
    {
        var seller = sellers.Single(s => s.Id == customer.Seller.Id);

        IEnumerable<string> centers = null;
        if (seller.NonCustomerAllowed) //isOffline
        {
            centers = await _offlineProductService.GetDefaultCentersCode(
                new[] { customer },
                cnpj,
                new[] { seller.Id });
            if (!centers.Any())
                _logger.LogWarning($"No centers was found for customer {customer.Id} and seller {seller.Id}");
        }

        customer.Parameters = customer.Parameters ?? new List<Crosscutting.Cache.Models.Customer.CachedParameter>();

        if (!customer.Parameters.Any())
        {
            _logger.LogWarning($"No parameters was found for customer {customer.Id.ToString()} and seller {seller.Id.ToString()}");
        }

        permissions.Add(new CustomerPermission
        {
            SellerId = customer.Seller.Id,
            IsOffline = sellers.Single(s => s.Id == customer.Seller.Id).NonCustomerAllowed,
            CenterCodes = centers?.ToList() ?? new List<string>(),
            ParametersIds = customer.Parameters.Select(x => x.ParameterId).ToList(),
            RestrictedProducerIds = restrictedProducerIds.Where(r => r.SellerId == customer.Seller.Id).Select(r => r.ProducerId).ToArray()
        });
    }
    return permissions;
}
```

[Consulta de Fabricantes Restritos](Consulta%20das%20Permisso%CC%83es%20do%20Customer%20edeed20b4b184d4490646d9d91c7cbef/Consulta%20de%20Fabricantes%20Restritos%2071866df466f943ecb0e8036414b3a680.md)

[Build de Permissões](Consulta%20das%20Permisso%CC%83es%20do%20Customer%20edeed20b4b184d4490646d9d91c7cbef/Build%20de%20Permisso%CC%83es%202e392b12bc8543349011f7514bd65159.md)