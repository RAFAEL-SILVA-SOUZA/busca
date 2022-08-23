# Retorno do Search

### Detalhamento do método CreateProductDtoWithSettingsAndLotAsync

Nesse método é feito o loop dos documentos para construi o uma lista de objetos do tipo **ProductShortDtoResult**

```csharp
public async Task<List<ProductShortDtoResult>> CreateProductDtoWithSettingsAndLotAsync(
            IReadOnlyCollection<CachedProduct> products,
            IReadOnlyCollection<CachedCustomer> customers, List<CachedSeller> cachedSellers)
{

    var productsDto = new List<ProductShortDtoResult>();
    var sellerIds = products.Select(x => x.SellerId).Distinct();

    foreach (var sellerId in sellerIds)
    {
        var groupedProductsBySeller = products.Where(x => x.SellerId == sellerId).ToList();
        var productIds = groupedProductsBySeller.Select(x => x.Id).ToList();
        var selectedCustomerInfo = customers.FirstOrDefault(x => x.Seller.Id == sellerId);

        if (selectedCustomerInfo is null)
        {
            _logger.LogWarning($"Customer {customers.First().Cnpj} not found for seller {sellerId}");
            continue;
        }

        var customerCodeReceiver = selectedCustomerInfo.CustomerCode ?? customers.First().CustomerCode;
        var settings = await _saleSettingsService.GetSettingsAsync(productIds, selectedCustomerInfo.Cnpj,
            sellerId.ToString(), customerCodeReceiver);

        List<Lot> productLots = null;
        if (settings != null)
        {
            productLots = await _lotRepository.GetBySettings
                (
                    settings.Select(x => x.CenterCode).Distinct().ToList(),
                    groupedProductsBySeller.Select(x => x.MaterialCode).ToList(),
                    settings.Select(x => x.Incoterm).Distinct().ToList(),
                    new List<Guid> { sellerId }
                );
        }

        var result = groupedProductsBySeller.Select(x => ProductShortDtoResult.GetProductDto(x, settings, cachedSellers, productLots));
        productsDto.AddRange(result);
    }

    return productsDto;
}
```

### Variáveis do scopo do método

```csharp
var productsDto = new List<ProductShortDtoResult>();
var sellerIds = products.Select(x => x.SellerId).Distinct();
```

Para cada **sellerId** são obtidos os **produtos agrupados** e os **customerInfos** caso não exista **customerInfo** o loop continua para o proximo sellerId.

```csharp
foreach (var sellerId in sellerIds)
{
    var groupedProductsBySeller = products.Where(x => x.SellerId == sellerId).ToList();
    var productIds = groupedProductsBySeller.Select(x => x.Id).ToList();
    var selectedCustomerInfo = customers.FirstOrDefault(x => x.Seller.Id == sellerId);
    if (selectedCustomerInfo is null)
    {
        _logger.LogWarning($"Customer {customers.First().Cnpj} not found for seller {sellerId}");
        continue;
    }
...
```

No trecho de código a seguir um serviço da **Sale** recupera as **Settings,** passando como parâmetro os **Ids dos produtos**, **cnpj do customer** e o **customerCode**

```csharp
var customerCodeReceiver = selectedCustomerInfo.CustomerCode ?? customers.First().CustomerCode;
var settings = await _saleSettingsService.GetSettingsAsync(productIds, selectedCustomerInfo.Cnpj, sellerId.ToString(), customerCodeReceiver);
```

[Detalhes do Método GetSettingsAsync](Retorno%20do%20Search%2031a83f9051194b699b21698f5bc9d0f6/Detalhes%20do%20Me%CC%81todo%20GetSettingsAsync%208551c4d1302247b9bf0e63561fb865b9.md)

Caso exista **Settings,** é feita uma consulta no banco de dados na tabela **Lots** passando os parâmetros em listas distintas de centerCode, materialCode, incoterm e sellerId

```csharp
List<Lot> productLots = null;
if (settings != null)
{
    productLots = await _lotRepository.GetBySettings
        (
            settings.Select(x => x.CenterCode).Distinct().ToList(),
            groupedProductsBySeller.Select(x => x.MaterialCode).ToList(),
            settings.Select(x => x.Incoterm).Distinct().ToList(),
            new List<Guid> { sellerId }
        );
}
```

[Busca de Lotes (GetBySettings)](Retorno%20do%20Search%2031a83f9051194b699b21698f5bc9d0f6/Busca%20de%20Lotes%20(GetBySettings)%205b5b2ea74ba0488ea65d1f79a2b0604d.md)

No final no loop o objeto **ProductShortDtoResult** é mapeado e adicionado a lista

```csharp
 var result = groupedProductsBySeller.Select(x => ProductShortDtoResult.GetProductDto(x, settings, cachedSellers, productLots));
 productsDto.AddRange(result);
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

[Método **GetProductComboDto**](Me%CC%81todo%20GetProductComboDto%2047455753da594751886d755cd2848d92.md) 

Neste método os modelos nulos não seram mapeados.

```csharp
if (model == null)
        return null;
```

Variáveis do escopo do método

```csharp
var selectedSeller = sellersCache?.FirstOrDefault(x => x.Id == model.SellerId);
var result = new ProductShortDtoResult();
```

Caso exista um seller para o modelo(**CachedProduct** ), a variável **result** do tipo **ProductShortDtoResult,** recebe os valores de

```csharp
if (selectedSeller != null)
{
    result.SellerName = selectedSeller.Name;
    result.SellerNameCompany = selectedSeller.NameCompany;
    result.SellerImage = selectedSeller.GUIDImage;
}
```

É atribuido as **Settings** ao result

```csharp
  result.Settings = settings != null && settings.Count > 0 ? settings.FirstOrDefault(x => x.ProductId == model.Id) : null;
```

Caso não **Settings,** é realizada a atribuição do **Lote** ao **result** baseado em: ***CenterCode , MaterialCode, Incoterms  e SellerId*** 

```csharp
if (result.Settings != null)
{
	  result.Lote = (LotDto)lot?.FirstOrDefault(x =>
		      x.CenterCode == result.Settings.CenterCode && x.MaterialCode == model.MaterialCode &&
		      x.Incoterms == result.Settings.Incoterm && x.SellerId == model.SellerId);
}
```