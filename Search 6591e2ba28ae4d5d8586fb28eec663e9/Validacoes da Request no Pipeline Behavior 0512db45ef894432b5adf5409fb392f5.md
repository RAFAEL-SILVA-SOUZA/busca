# Validacoes da Request no Pipeline Behavior

Classe que valida a request **SearchProductRequestDtoValidator**

```csharp
public class SearchProductRequestDtoValidator : AbstractValidator<SearchProductRequestDto>
{
    public SearchProductRequestDtoValidator(ICustomerService customerService, ClaimsPrincipal user)
    {
        RuleFor(t => t.CustomersInfo)
            .NotEmpty()
            .WithMessage("Nenhum [CustomersInfo] informado.");

        RuleForEach(t => t.CustomersInfo)
            .SetValidator(new SearchPartnerDetailsDtoValidator())
            .WithMessage("[CustomersInfo] inválido.")
            .DependentRules(() => CustomersParametersValidation(customerService));

        RuleFor(t => t.PageSize)
            .LessThanOrEqualTo(100)
            .WithMessage("O valor máximo de [PageSize] é 100");

        RuleFor(t => t.Cnpj)
            .NotEmpty()
                .WithMessage("CNPJ deve ser informado")
            .Must(cnpj => user.GetCnpjsFromToken().Contains(cnpj))
                .WithMessage("CNPJ informado é inválido");
    }

    private void CustomersParametersValidation(ICustomerService customerService)
    {
        var missing = new List<CustomerInfoDto>();
        RuleFor(t => t.CustomersInfo)
            .MustAsync(async (customerInfo, _) =>
            {
                var result = await customerService.ExistsCustomersAsync(customerInfo);
                missing = result.Missing;
                return result.Exists;
            })
            .When(x => x.CustomersInfo != null && x.CustomersInfo.Any())
            .WithMessage((_) =>
            {
                var customerInfoText = string.Join(", ", missing.Select(x => x.ToString()));
                return "Existem valores inválidos nos dados de cliente enviados. Não foram encontrados clientes relacioados aos códigos {customerInfoText}";
            });
    }
}

public class SearchPartnerDetailsDtoValidator : AbstractValidator<CustomerInfoDto>
{
    public SearchPartnerDetailsDtoValidator()
    {
        RuleFor(t => t.CustomerCodeReceiver)
            .NotEmpty()
            .WithMessage("[CustomerCodeReceiver] é obrigatório.");

        RuleFor(t => t.SellerId)
            .NotEmpty()
            .WithMessage("[SellerId] é obrigatório.");
    }
}
```

### Métodos utilizados no Validator

O método **ExistsCustomersAsync.**

```csharp
public async Task<(bool Exists, List<CustomerInfoDto> Missing)> ExistsCustomersAsync(List<CustomerInfoDto> customerInfos)
{
    var customers = await GetCustomersAsync(customerInfos);

    var missing = customerInfos.Except(customers
									             .Select(x => new CustomerInfoDto
									             {
									                 CustomerCodeReceiver = x.CustomerCode,
									                 SellerId = x.Seller.Id
									             }))
									             .ToList();

    return (!missing.Any(), missing);
}
```

O método **GetCustomersAsync** esta mapeado nas páginas:

> [Consulta dos Customers](Consulta%20dos%20Customers%2024272aa7d80e41438582d5d20c85047b.md)
> 

> [Informacoes do Customer](Analise%20tecnica%20-%20Pontos%20de%20Atencao%20a1741339d4e34669a72979df77771b3e/Informacoes%20do%20Customer%20bc6c376803a841f59ae3d9d2a6c14a4a.md)
>