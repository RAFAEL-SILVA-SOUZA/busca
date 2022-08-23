# Busca de Lotes (GetBySettings)

**Database Type:** Sql

**Database:** JSMCatalog

T**able:** Lots

A consulta fica assim no repositório da **Catalog**

```csharp
public async Task<List<Lot>> GetBySettings(List<string> centerCode, List<string> materialCode, List<string> incoterm, List<Guid> sellerId)
{
    return await DbSet.Where(x => centerCode.Contains(x.CenterCode) 
               && materialCode.Contains(x.MaterialCode) 
               && incoterm.Contains(x.Incoterms) 
               && sellerId.Contains((Guid)x.SellerId)).ToListAsync();
}
```

O objeto **Lot**

```csharp
public class Lot
{
    public Guid Id { get; set; }
    public Guid? SellerId { get; set; }
    public string CenterCode { get; set; }
    public string MaterialCode { get; set; }
    public string Incoterms { get; set; }
    public double LotMinimum { get; set; }
    public double LotMultiple { get; set; }
}
```