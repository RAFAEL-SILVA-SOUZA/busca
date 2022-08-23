# Metodos que compoem a Consulta de Atributos

```csharp
public async Task<ISearchResponse<CachedProduct>> SearchProductsAttributesAsync(string term,ICollection<CustomerPermission> customersPermissions,IEnumerable<Guid> customerIds,
Guid? categoryId,int maxAttributes = 1000)
{
    return await Client.SearchAsync<CachedProduct>(s => s
        .Index(ProductSearchIndex)
        .Size(0)
        .Query(q =>
        {
            var query = q.Term(t => t.Value(StatusProduct.Active.ToString()).Field(f => f.Status)) &&
                   MustFilterByCustomersPermissions(customersPermissions, q) &&
                   MustNotContainsRestrictedCustomers(customerIds, q) &&
                   ShouldMatchCategoryIdQuery(categoryId?.ToString(), q);

            if (!string.IsNullOrEmpty(term))
                query = query && MustMatchTerm(term, q);

            return query;
        })
        .Aggregations(a => a
            .Terms("attributes", t => t
                .Size(maxAttributes).Field(f => f.ProductAttributeValues[0].AttributeValueId))));
}
```

### Deve filtrar por permissões de clientes

```csharp
private static QueryContainer MustFilterByCustomersPermissions(ICollection<CustomerPermission> customersPermissions, QueryContainerDescriptor<CachedProduct> q)
{
    QueryContainer query = null;
    foreach (var sellerPermission in customersPermissions)
    {
        query = query || MustFilterByCustomerPermission(sellerPermission, q);
    }

    return query;
}
```

### Deve filtrar por permissão do Customer

```csharp
private static QueryContainer MustFilterByCustomerPermission(CustomerPermission customerPermission, QueryContainerDescriptor<CachedProduct> q)
{
    var (sellerId, parameters, isOffline, centers, restrictedProducerIds) = customerPermission;
    var sellerQuery = q.Term(t => t.Field(f => f.SellerId).Value(sellerId.ToString()));

    if (!parameters.Any() || (isOffline && !centers.Any()))
        return sellerQuery && !sellerQuery;

    var query =
            sellerQuery &&
            !q.Terms(t => t.Field(f => f.Restrictions[0].ParameterIds).Terms(parameters)) &&
             q.Terms(t => t.Field(f => f.DisplayRules[0].ParameterId).Terms(parameters));

    QueryContainer producerQuery = null;
    foreach (var producerId in restrictedProducerIds)
    {
        producerQuery = producerQuery && !q.MatchPhrase(m => m.Field(t => t.ProducerId).Query(producerId.ToString()));
    }

    query = query && producerQuery;

    if (isOffline)
    {
        QueryContainer centerQuery = null;
        foreach (var center in centers)
        {
            centerQuery = centerQuery || q
                .Nested(c => c.Path(p => p.Centers)
                    .Query(nq => nq.MatchPhrase(m => m.Query(center).Field(f => f.Centers[0].CenterCode)) &&
                                 nq.Range(r => r.GreaterThan(0).Field(f => f.Centers[0].Stock))));
        }

        query = query && centerQuery;
    }

    return query;
}
```

### Deve corresponder a consulta CategoryId

```csharp
private static QueryContainer ShouldMatchCategoryIdQuery(string categoryId, QueryContainerDescriptor<CachedProduct> q)
{
    QueryContainer query = null;
    if (!string.IsNullOrWhiteSpace(categoryId))
    {
        query = query || q.MatchPhrase(m => m.Query(categoryId).Field(f => f.CategoriesIds));
    }

    return query;
}
```

### Deve corresponder ao termo

```csharp
private QueryContainer MustMatchTerm(string term, QueryContainerDescriptor<CachedProduct> q)
{
  return int.TryParse(term, out _)
         ? q.Match(m => m.Query(term).Field(ProductSearchField))
         : q.Match(m => m.Query(term).Field(ProductSearchField).Fuzziness(Fuzziness.AutoLength(4, 6)));
}
```