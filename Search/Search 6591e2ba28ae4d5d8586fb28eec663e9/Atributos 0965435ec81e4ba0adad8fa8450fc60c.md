# Atributos

Se a consulta for do tipo Autocomplete ou a lista de produtos estiver vazia o parâmetro **attributeResponse** recebe uma lista vazia de objetos do tipo **AttributeResponse,** caso contrário os atributos do produtos serão recuperados através do Elsatic no método **GetProductsAttributesAsync**

```csharp
var attributeResponse = isAutocomplete || !products.Any()
                ? new List<AttributeResponse>()
                : await GetProductsAttributesAsync(request, permissions, customers);
```

O método **GetProductsAttributesAsync**

```csharp
private async Task<List<AttributeResponse>> GetProductsAttributesAsync(SearchProductRequestDto request, ICollection<CustomerPermission> permissions,IEnumerable<CachedCustomer> customers)
{
    var attributesResult = await _elasticServiceV2.SearchProductsAttributesAsync(
        request.Filter,
        permissions,
        customers.Select(x => x.Id),
        request.CategoryId);

    if (attributesResult.Aggregations.Count == 0)
        return new List<AttributeResponse>();

    var attributesValuesIds = attributesResult
        .Aggregations.Terms("attributes")
        .Buckets.ToDictionary(x => Guid.Parse(x.Key), x => x.DocCount);

    var attributes = await _attributeKeyRepository.GetByValueIds(attributesValuesIds.Keys);

    var attributeList = attributes.Select(key => new AttributeResponse
    {
        Label = key.Label,
        IsBeginOpen = key.IsBeginOpen,
        Key = key.Key,
        Type = key.Type,
        Attributes = key.Values.OrderBy(value => value.Label).Select(value => new AttributeValueResponse
        {
            Label = value.Label,
            Value = value.Value,
            Id = value.Id,
            Status = value.Status,
            IsSelected = request.AttributeValues.Contains(value.Id),
            Total = attributesValuesIds.ContainsKey(value.Id)
                 ? (int)(attributesValuesIds[value.Id] ?? 0)
                 : 0
        }).ToList()
    }).ToList();

    var soldBy = attributeList.FirstOrDefault(m => m.Key == "vendido-por");

    if (soldBy == null)
        return attributeList;

    attributeList.Remove(soldBy);
    attributeList.Insert(0,soldBy);
    
    return attributeList;
}
```

### Consulta de Atributos no Elsatic

[Metodos que compõem a Consulta de Atributos](Atributos%200965435ec81e4ba0adad8fa8450fc60c/Metodos%20que%20compo%CC%83em%20a%20Consulta%20de%20Atributos%20927d2081ac274182b9a23b3010eec46d.md)

Caso o resultado da consulta de atributos no Elastic não contenha **Aggregations,** é retornada uma lista vazia do objeto **AttributeResponse.**

```csharp
if (attributesResult.Aggregations.Count == 0)
    return new List<AttributeResponse>();
```

No trecho de código a seguir, é criado um dicionário com as chaves dos atributos

```csharp
var attributesValuesIds = attributesResult.Aggregations.Terms("attributes").Buckets.ToDictionary(x => Guid.Parse(x.Key), x => x.DocCount);
```

No trecho de código a seguir, é realizada a consulta no banco utilizando as chaves dos atributos como parâmetro.

```csharp
var attributes = await _attributeKeyRepository.GetByValueIds(attributesValuesIds.Keys);
```

[Detalhes do Repositório de Consulta dos Atributos](Atributos%200965435ec81e4ba0adad8fa8450fc60c/Detalhes%20do%20Reposito%CC%81rio%20de%20Consulta%20dos%20Atributos%200472c25c045a4093a770d9cb349083df.md)

No trecho de código a seguir é criada uma lista de objetos do tipo **AttributeResponse,** baseado no retorno da consulta ao banco.

```csharp
var attributeList = attributes.Select(key => new AttributeResponse
{
    Label = key.Label,
    IsBeginOpen = key.IsBeginOpen,
    Key = key.Key,
    Type = key.Type,
    Attributes = key.Values.OrderBy(value => value.Label).Select(value => new AttributeValueResponse
    {
        Label = value.Label,
        Value = value.Value,
        Id = value.Id,
        Status = value.Status,
        IsSelected = request.AttributeValues.Contains(value.Id),
        Total = attributesValuesIds.ContainsKey(value.Id)
             ? (int)(attributesValuesIds[value.Id] ?? 0)
             : 0
    }).ToList()
}).ToList();
```

No trecho de código a seguir é feita uma verificação para verificar se existe alguma chave do tipo “**vendido-por**”, caso não contenha o metodo retorna a lista de objetos do tipo  **AttributeResponse** atual**,**  caso contrário, é aplicado um tratamento para reordenar a posição das chaves colocando as chaves “**vendido-por**” no indice 0 da lista.

```csharp
var soldBy = attributeList.FirstOrDefault(m => m.Key == "vendido-por");

if (soldBy == null)
    return attributeList;

attributeList.Remove(soldBy);
attributeList.Insert(0,soldBy);
```

O objeto **AttributeResponse**

```csharp
public class AttributeResponse
{
    public string Label { get; set; }
    public bool IsBeginOpen { get; set; }
    public FilterType Type { get; set; }
    public string Key { get; set; }
    public List<AttributeValueResponse> Attributes { get; set; }

    public static AttributeResponse MapAttributeResponse(AttributeKey model, Guid[] selectedValues)
    {
        if (model == null)
            return null;

        return new AttributeResponse
        {
            Type = model.Type,
            IsBeginOpen = model.IsBeginOpen,
            Key = model.Key,
            Label = model.Label,
            Attributes =  model.Values?
                .Where(x => x.Total > 0)?
                .OrderBy(attribute => attribute.Label)
                .Select(x => AttributeValueResponse.MapAttributeValueResponse(x, selectedValues))
                .ToList()
        };
    }
}
```

O objeto **AttributeValueResponse**

```csharp
public class AttributeValueResponse
{
    public Guid Id { get; set; }
    public bool IsSelected { get; set; }
    public string Label { get; set; }
    public string Value { get; set; }
    public int Status { get; set; }
    public int Total { get; set; }

    public static AttributeValueResponse MapAttributeValueResponse(AttributeValue model, Guid[] selectedValues)
    {
        if (model == null)
            return null;

        return new AttributeValueResponse
        {
            Id = model.Id,
            Label = model.Label,
            Total = model.Total,
            Status = model.Status,
            Value = model.Value,
            IsSelected = selectedValues != null && selectedValues.Contains(model.Id)
        };
    }
}
```