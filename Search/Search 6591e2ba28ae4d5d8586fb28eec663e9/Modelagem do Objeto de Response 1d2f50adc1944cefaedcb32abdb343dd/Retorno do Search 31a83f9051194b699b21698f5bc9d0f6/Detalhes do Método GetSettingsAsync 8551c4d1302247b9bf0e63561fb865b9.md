# Detalhes do Método GetSettingsAsync

No trecho de a seguir é realizada uma requisição htto na api de **Sale** para obter as **Settings.**

Para esta request é necessárioutilizar o token de integração do tipo **Bearer** contido no appsettings.json da aplicação **Catalog(***Settings:IntegrationServicesToken)*

Por fim é retornado uma lista de objetos do tipo **SettingsDto**

```csharp
public async Task<List<SettingsDto>> GetSettingsAsync(ICollection<Guid> productIds, string cnpj, string sellerId,
            string customerCode)
{
    var httpClient = _httpClientFactory.CreateClient();
    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", _integrationToken);

    var requestModel = new
    {
        sellerId,
        cnpj,
        productIds,
        searchOnSAP = false,
        customerCodeReceiver =  customerCode
    };

    HttpResponseMessage response;
    try
    {
        _logger.LogInformation($"Requesting settings from sale. Payload: {JsonConvert.SerializeObject(requestModel)}");
        response = await httpClient.PostAsJsonAsync($"{_saleUrlBase}/Settings/lote", requestModel);
    }
    catch (HttpRequestException ex)
    {
        _logger.LogError(ex, "Error when getting lot from Sale integration.");
        return null;
    }
    
    var stringResponse = await response.Content.ReadAsStringAsync();
    if (!response.IsSuccessStatusCode)
    {
        _logger.LogError($"Error when getting lot from Sale integration. {response.StatusCode}: {stringResponse}");
        return null;
    }

    var settingsDto = JsonConvert.DeserializeObject<DataSettingsDto>(stringResponse)?.Data;
    if (settingsDto is null)
    {
        _logger.LogWarning($"Got null settings from Sale Integration. Seller: {sellerId}, cnpj: {cnpj}, " +
                           $"productIds: {productIds}, customerCode: {customerCode}.");
        return null;
    }

    return settingsDto;
}
```

### Path

```json
https://dev-api-loja.juntossomosmais.com.br/sale/v1/Settings/lote
```

### Exemplo de Request

```json
{
    "sellerId": "2cb3e721-2e5c-4ea9-bdaa-a79954629cd3",
    "cnpj": "00938601000140",
    "productIds": ["8c7f01d2-d90e-434a-9c1e-b70392868653"],
    "searchOnSAP": false,
    "customerCodeReceiver": "528857"
}
```

### Exemplo de Response

```json
{
    "success": true,
    "data": [
        {
            "productId": "8c7f01d2-d90e-434a-9c1e-b70392868653",
            "incoterm": "CIF",
            "centerCode": "ET33",
            "centerDescription": "ETERNIT S.A EM RECUPERACAO JUDICIAL",
            "paymentConditionCode": "B001",
            "paymentConditionDescription": "Á vista",
            "paymentFormCode": "B",
            "paymentFormDescription": "Boleto",
            "shipmentLine": ""
        }
    ]
}
```