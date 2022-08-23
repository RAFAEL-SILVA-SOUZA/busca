# Consulta de Produtos

### O Método

```csharp
private async Task<ISearchResponse<CachedProduct>> SearchProductsAsync(string term,
            ICollection<CustomerPermission> customersPermissions,
            IEnumerable<Guid> customerIds,
            Guid? categoryId,
            ICollection<Guid> attributeValuesIds,
            ICollection<Guid> productsIds,
            int @from, int size)
{
    if (!customersPermissions.Any())
        return new SearchResponse<CachedProduct>();

    var searchDescriptor = new SearchDescriptor<CachedProduct>()
        .Index(ProductSearchIndex)
        .From(from)
        .Size(size)
        .Query(q =>
        {
            var query = q.ConstantScore(cs => cs.Filter(
                filter =>
                {
                    var filterQuery =
                        filter.Term(t => t.Value(StatusProduct.Active.ToString()).Field(f => f.Status)) &&
                        MustFilterByCustomersPermissions(customersPermissions, filter) &&
                        MustNotContainsRestrictedCustomers(customerIds, filter) &&
                        ShouldMatchCategoryIdQuery(categoryId?.ToString(), filter);

                    if (attributeValuesIds.Any())
                    {
                        filterQuery = filterQuery && filter.Terms(t =>
                            t.Field(f => f.ProductAttributeValues[0].AttributeValueId)
                                .Terms(attributeValuesIds));
                    }

                    if (productsIds.Any())
                    {
                        filterQuery = filterQuery && q.Ids(c => c.Values(productsIds));
                    }

                    return filterQuery;
                }));

            if (!string.IsNullOrEmpty(term))
            {
                query = query && MustMatchTerm(term, q);
            }

            return query;
        })
        .Sort(s => s
            .Descending(SortSpecialField.Score)
            .Ascending(p => p.Name.Suffix("keyword")));

    var products = await Client.SearchAsync<CachedProduct>(searchDescriptor);

    foreach (var item in products.Hits)
    {
        item.Source.Id = Guid.Parse(item.Id);
    }

    return products;
}
```

### Detalhes do método

Caso o Customer não tenha permissões, o método retorna um objeto vazio do tipo **SearchResponse<CachedProduct>**

```csharp
  if (!customersPermissions.Any())
        return new SearchResponse<CachedProduct>();
```

### Composição da Query

```csharp
 var searchDescriptor = new SearchDescriptor<CachedProduct>()
        .Index(ProductSearchIndex)
        .From(from)
        .Size(size)
        .Query(q =>
        {
            var query = q.ConstantScore(cs => cs.Filter(
                filter =>
                {
                    var filterQuery =
                        filter.Term(t => t.Value(StatusProduct.Active.ToString()).Field(f => f.Status)) &&
                        MustFilterByCustomersPermissions(customersPermissions, filter) &&
                        MustNotContainsRestrictedCustomers(customerIds, filter) &&
                        ShouldMatchCategoryIdQuery(categoryId?.ToString(), filter);

                    if (attributeValuesIds.Any())
                    {
                        filterQuery = filterQuery && filter.Terms(t =>
                            t.Field(f => f.ProductAttributeValues[0].AttributeValueId)
                                .Terms(attributeValuesIds));
                    }

                    if (productsIds.Any())
                    {
                        filterQuery = filterQuery && q.Ids(c => c.Values(productsIds));
                    }

                    return filterQuery;
                }));

            if (!string.IsNullOrEmpty(term))
            {
                query = query && MustMatchTerm(term, q);
            }

            return query;
        })
        .Sort(s => s
            .Descending(SortSpecialField.Score)
            .Ascending(p => p.Name.Suffix("keyword")));
```

### Exemplo de output string da Query do Elastic

```json
{
    "from": 0,
    "query": {
        "bool": {
            "must": [{
                    "constant_score": {
                        "filter": {
                            "bool": {
                                "must": [{
                                        "term": {
                                            "Status": {
                                                "value": "1"
                                            }
                                        }
                                    }, {
                                        "term": {
                                            "SellerId": {
                                                "value": "2cb3e721-2e5c-4ea9-bdaa-a79954629cd3"
                                            }
                                        }
                                    }, {
                                        "terms": {
                                            "DisplayRules.ParameterId": ["289357b8-64bc-4791-2461-08d94dfa654c", "f6eee518-a6b6-4445-2462-08d94dfa654c", "5df6efa2-1433-456a-a6bf-08d9364deab2", 
																																				 "0381024e-ca7f-461b-2463-08d94dfa654c", "758b4aff-bfe0-4d66-a6bd-08d9364deab2", "d19d689b-01c5-47e5-a6c6-08d9364deab2"]
                                        }
                                    }, {
                                        "nested": {
                                            "path": "Centers",
                                            "query": {
                                                "bool": {
                                                    "must": [{
                                                            "match_phrase": {
                                                                "Centers.CenterCode": {
                                                                    "query": "ET33"
                                                                }
                                                            }
                                                        }, {
                                                            "range": {
                                                                "Centers.Stock": {
                                                                    "gt": 0.0
                                                                }
                                                            }
                                                        }
                                                    ]
                                                }
                                            }
                                        }
                                    }
                                ],
                                "must_not": [{
                                        "terms": {
                                            "Restrictions.ParameterIds": ["289357b8-64bc-4791-2461-08d94dfa654c", "f6eee518-a6b6-4445-2462-08d94dfa654c", "5df6efa2-1433-456a-a6bf-08d9364deab2", 
																																					"0381024e-ca7f-461b-2463-08d94dfa654c", "758b4aff-bfe0-4d66-a6bd-08d9364deab2", "d19d689b-01c5-47e5-a6c6-08d9364deab2"]
                                        }
                                    }, {
                                        "terms": {
                                            "Restrictions.CustomerIds": ["da41d53e-a6d9-48da-8578-ac292f465c6c"]
                                        }
                                    }
                                ]
                            }
                        }
                    }
                }, {
                    "match": {
                        "search_results": {
                            "fuzziness": "AUTO:4,6",
                            "query": "Telha"
                        }
                    }
                }
            ]
        }
    },
    "size": 30,
    "sort": [{
            "_score": {
                "order": "desc"
            }
        }, {
            "Name.keyword": {
                "order": "asc"
            }
        }
    ]
}
```

[Metodos que compoem a Consulta](Consulta%20de%20Produtos%20edc3a270a0f44b40a126ca3ea18e2769/Metodos%20que%20compoem%20a%20Consulta%201e44df6a142f4d8f98e5c2003d1a49be.md)