# Retorno com Autocomplete

### Detalhamento do método **CreateProductAutocompleteDtoAsync**

Nesse método é feito o loop dos documentos para construi o uma lista de objetos do tipo **ProductShortDtoResult**

```csharp
private static List<ProductShortDtoResult> CreateProductAutocompleteDtoAsync(IEnumerable<CachedProduct> elasticProducts, List<CachedSeller> cachedSellers)
{
      return elasticProducts.Select(x => ProductShortDtoResult.GetProductDto(x, null, cachedSellers, null)).ToList();
}
```

### Detalhamento do método GetProductDto

Neste método os modelos nulos não seram mapeados

```csharp
public static ProductShortDtoResult GetProductDto(CachedProduct model, List<SettingsDto> settings, List<CachedSeller> sellersCache, List<Lot> lot)
{
    if (model == null)
        return null;

    return MapProduct(model, settings, sellersCache, lot);
}
```

### Detalhamento do método MapProduct

```csharp
private static ProductShortDtoResult MapProduct(CachedProduct model, List<SettingsDto> settings, List<CachedSeller> sellersCache, List<Lot> lot)
{
    if (model == null)
        return null;

    var selectedSeller = sellersCache?.FirstOrDefault(x => x.Id == model.SellerId);
    var result = new ProductShortDtoResult();

    result.Id = model.Id;
    result.SellerId = model.SellerId;
    if (selectedSeller != null)
    {
        result.SellerName = selectedSeller.Name;
        result.SellerNameCompany = selectedSeller.NameCompany;
        result.SellerImage = selectedSeller.GUIDImage;
    }
    result.HighlightAsNew = model.HighlightAsNew;
    result.MaterialCode = model.MaterialCode;
    result.Weight = model.Weight;
    result.UnitWeight = model.UnitWeight;
    result.UnitMeasure = model.UnitMeasure;
    result.Name = model.Name;
    result.FullDescription = model.FullDescription;
    result.UrlOrientationApplication = model.UrlOrientationApplication;
    result.CreationDate = model.CreationDate;
    result.UsernameChange = model.UsernameChange;
    result.ChangeDate = model.ChangeDate;
    result.Status = model.Status;
    result.Sector = model.Sector;
    result.TechnicalSpecification = model.TechnicalSpecification;
    result.IsGrouping = false; 
    result.IsCombo = model.Children != null && model.Children.Count > 0;
    result.QuantityMaterialChildren = 0;
    result.Order = 0; 
    result.TrackingUrl = ""; 
    result.Application = model.Application;
    result.Category = (CategoryDto)model.Category;
    result.Indication = model.Indication;
    result.Photos = model.Photos != null && model.Photos.Count > 0 ? model.Photos.Select(x => (ProductPhotoDto)x).ToList() : null;
    result.Children = model.Children != null && model.Children.Count > 0 ? model.Children.Select(x => GetProductComboDto(x, settings, sellersCache, lot)).ToList() : null;
    result.Settings = settings != null && settings.Count > 0 ? settings.FirstOrDefault(x => x.ProductId == model.Id) : null;

    if (result.Settings != null)
    {
        result.Lote = (LotDto)lot?.FirstOrDefault(x =>
            x.CenterCode == result.Settings.CenterCode && x.MaterialCode == model.MaterialCode &&
            x.Incoterms == result.Settings.Incoterm && x.SellerId == model.SellerId);
    }

    result.CategoryId = model.CategoryId;
    result.BriefDescription = model.BriefDescription;

    return result;
}
```

[Metodo **GetProductComboDto**](Metodo%20GetProductComboDto%2047455753da594751886d755cd2848d92.md) 

- Neste método os modelos nulos não seram mapeados.

```csharp
if (model == null)
        return null;
```

- Variáveis do escopo do método

```csharp
var selectedSeller = sellersCache?.FirstOrDefault(x => x.Id == model.SellerId);
var result = new ProductShortDtoResult();
```

- Caso exista um seller para o modelo(**CachedProduct** ), a variável **result** do tipo **ProductShortDtoResult,** recebe os valores de

```csharp
if (selectedSeller != null)
{
    result.SellerName = selectedSeller.Name;
    result.SellerNameCompany = selectedSeller.NameCompany;
    result.SellerImage = selectedSeller.GUIDImage;
}
```

- É atribuido as **Settings** ao result

```csharp
  result.Settings = settings != null && settings.Count > 0 ? settings.FirstOrDefault(x => x.ProductId == model.Id) : null;
```

- Caso não **Settings,** é realizada a atribuição do **Lote** ao **result** baseado em: ***CenterCode , MaterialCode, Incoterms  e SellerId***

```csharp
if (result.Settings != null)
{
  result.Lote = (LotDto) lot?.FirstOrDefault(x =>
      x.CenterCode == result.Settings.CenterCode && x.MaterialCode == model.MaterialCode &&
      x.Incoterms == result.Settings.Incoterm && x.SellerId == model.SellerId);
}
```