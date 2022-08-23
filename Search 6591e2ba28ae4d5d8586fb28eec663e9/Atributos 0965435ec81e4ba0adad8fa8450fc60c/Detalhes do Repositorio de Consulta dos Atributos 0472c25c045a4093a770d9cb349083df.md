# Detalhes do Repositorio de Consulta dos Atributos

**Database Type:** Sql

**Database:** JSMCatalog

T**able:** AttributeValues

```csharp
public async Task<IEnumerable<AttributeKey>> GetByValueIds(IEnumerable<Guid> attributesValuesIds)
{
    var attributeValues = await _context.AttributeValue
        .Include(x => x.AttributeKey)
        .AsSplitQuery()
        .Where(x => attributesValuesIds.Contains(x.Id))
        .ToListAsync();

    var attributeKeys = attributeValues.Select(x => x.AttributeKey).Distinct();

    return attributeKeys.ToList();
}
```