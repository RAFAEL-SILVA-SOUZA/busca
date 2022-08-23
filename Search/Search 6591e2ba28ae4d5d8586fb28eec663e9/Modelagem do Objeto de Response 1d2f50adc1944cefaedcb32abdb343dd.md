# Modelagem do Objeto de Response

No caso do Autocomplete o médoto **CreateProductAutocompleteDtoAsync**  é executado, o método recebe os documentos retornados do **Elsatic** e os **Sellers**

```csharp
var products = isAutocomplete
        ? CreateProductAutocompleteDtoAsync(elasticProducts.Documents, sellers)
        : await CreateProductDtoWithSettingsAndLotAsync(elasticProducts.Documents.ToList(), customers, sellers);
```

[Retorno com Autocomplete](Modelagem%20do%20Objeto%20de%20Response%201d2f50adc1944cefaedcb32abdb343dd/Retorno%20com%20Autocomplete%208be181e74a844d1292c2a1a7b741c555.md)

[Retorno do Search](Modelagem%20do%20Objeto%20de%20Response%201d2f50adc1944cefaedcb32abdb343dd/Retorno%20do%20Search%2031a83f9051194b699b21698f5bc9d0f6.md)

[Método **GetProductComboDto**](Modelagem%20do%20Objeto%20de%20Response%201d2f50adc1944cefaedcb32abdb343dd/Me%CC%81todo%20GetProductComboDto%2047455753da594751886d755cd2848d92.md)