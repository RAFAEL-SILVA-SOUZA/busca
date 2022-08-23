# Consulta dos Produtos no Elastic

Para realizar o build da consulta é preciso informar as permissões dos **Customers** e os **Ids** dos **Customers,** caso contrario uma exception do tipo ArgumentNullException e lançada na aplicação.

```csharp
public Task<ISearchResponse<CachedProduct>> SearchProductsByTermAndRulesAsync(string term,
      ICollection<CustomerPermission> customersPermissions,
      IEnumerable<Guid> customerIds,
      Guid? categoryId,
      ICollection<Guid> attributeValuesIds,
      int @from, int size)
{
    if (customersPermissions is null) throw new ArgumentNullException(nameof(customersPermissions));
    if (customerIds is null) throw new ArgumentNullException(nameof(customerIds));

    return SearchProductsAsync(
        term,
        customersPermissions,
        customerIds,
        categoryId,
        attributeValuesIds,
        Array.Empty<Guid>(),
        from,
        size);
}
```

[Consulta de Produtos](Consulta%20dos%20Produtos%20no%20Elastic%20005314d2bb8643f0ab6474db5e883a6b/Consulta%20de%20Produtos%20edc3a270a0f44b40a126ca3ea18e2769.md)