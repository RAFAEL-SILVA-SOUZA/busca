# Informacoes do Customer

O código a seguir verifica se foi passado um **SellerId** por parâmetro no scopo do objeto root(**SearchProductRequestDto**) da requisição.

 - Se o **SellerId** possuir ****valor, é realizado um filtro para obter todos os **CustomersInfo** onde o SellerId é igual ao valor do **SellerId** passado por parâmetro no scopo do objeto root(**SearchProductRequestDto**)

 - Se o **SellerId** não posuir valor, a variável **customerInfos** receberá a lista de  **CustomersInfo,** passados no objeto root(**SearchProductRequestDto**).

```csharp
 var customerInfos = request.SellerId.HasValue
        ? request.CustomersInfo.Where(x => x.SellerId == request.SellerId.Value).ToList()
        : request.CustomersInfo;
```