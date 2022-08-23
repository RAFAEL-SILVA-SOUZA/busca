# Método GetProductComboDto

O método

```csharp
public static ProductShortDtoResult GetProductComboDto(ProductCombo model, List<SettingsDto> settings, List<CachedSeller> sellersCache, List<Lot> lot)
{
    if (model == null)
        return null;

    var result = MapProduct(model.Children, settings, sellersCache, lot);

    if (result == null)
        return null;

    result.Position = model.Position;
    result.QuantityMaterialChildren = model.QuantityMaterial;

    return result;
}
```

O objeto **ProductCombo**

```csharp
public class : Entity
{
    public virtual Product Parent { get; set; }
    public virtual Product Children { get; set; }
    public Guid ParentId { get; set; }
    public Guid ChildrenId { get; set; }
    public string Position { get; set; }
    public int QuantityMaterial { get; set; }  
    [NotMapped]
    public string MaterialCode { get; set; }
}
```

Objeto que compõem o **ProductCombo**

```csharp
public class Product : Entity
{
    public Guid SellerId { get; set; }
    public bool HighlightAsNew { get; set; }
    public string MaterialCode { get; set; }
    public decimal Weight { get; set; }
    public string UnitWeight { get; set; }
    public string UnitMeasure { get; set; }
    public string Name { get; set; }
    public string BriefDescription { get; set; }
    public string JsmBriefDescription { get; set; }
    public string FullDescription { get; set; }
    public string UrlOrientationApplication { get; set; }
    public Guid? IdUserChange { get; set; }
    public string UsernameChange { get; set; }
    public DateTime? ChangeDate { get; set; }
    public Guid? ParentProductId { get; set; }

    public string Application { get; set; }
    public string Indication { get; set; }
    public string Sector { get; set; }
    public string TechnicalSpecification { get; set; }

    public List<ProductAttribute> Attributes { get; set; }
    public List<ProductDataSheet> DataSheets { get; set; }
    public List<ProductDisplayRule> DisplayRules { get; set; }
    public string Ean { get; set; }
    public string Brand { get; set; }

    public Guid? ProducerId { get; set; }
    public Producer Producer { get; set; }

    // Just for mapping
    public ProductGrouping Grouping { get; set; }
    public List<CarouselProduct> Carousels { get; set; }

    public Guid? CategoryId { get; set; }
    public Category Category { get; set; }

    public List<ProductSalesArea> SalesArea { get; set; }
    public List<ProductKeyword> Keywords { get; set; }
    public List<ProductPhoto> Photos { get; set; }
    public List<ProductVideo> Videos { get; set; }

    public List<RestrictionProduct> Restrictions { get; set; }

    public List<ProductCombo> Children { get; set; }
    public List<Price> Prices { get; set; }

    public virtual List<ProductAttributeValue> ProductAttributes { get; set; } 

    //For children
    [NotMapped]
    public int QuantityMaterial { get; set; }

    [NotMapped]
    public List<string> CategoriesIds { get; set; }

    [NotMapped]
    public int Postition { get; set; }

    public void UpdateCategory(Guid categoryId)
    {
        CategoryId = categoryId;
    }
}

[Obsolete("Entity is going to be removed in the future. Use AttributeKey instead.")]
public class ProductAttribute: Entity
{
    public Guid ProductId { get; set; }
    public string Key { get; set; }
    public string Value { get; set; }

    public virtual Product Product { get; set; }
}

public class ProductDataSheet : Entity
{

    public Guid ProductId { get; set; }
    public string Url { get; set; }

    public virtual Product Product { get; set; }

}

public class ProductDisplayRule : Entity
{
    public Guid ProductId { get; set; }
    public Guid ParameterId { get; set; }
    public string Code { get; set; }
    public string Description { get; set; }
    public string Value { get; set; }

    public virtual Product Product { get; set; }
}

public class ProductGrouping
{
    public Guid ProductId { get; set; }
    public Guid GroupingId { get; set; }

    public virtual Product Product { get; set; }
    public virtual Grouping Grouping { get; set; }
}

public class CarouselProduct : Entity
{
    public Guid CarouselId { get; set; }
    public Guid ProductId { get; set; }
    public int? Order { get; set; }

    public Product Product { get; set; }
    public Carousel Carousel { get; set; }
}

public class Category : Entity
{
    public Guid SellerId { get; set; }
    public Guid? ParentId { get; set; }
    public string Name { get; set; }
    public string Slug { get; set; }
    public int Order { get; set; }
    public string ResponsibleUser { get; set; }
    public DateTime ModificationDate { get; set; }
    public string BannerUrl { get; set; }
    public List<Category> Children { get; set; }
    public Category ParentLevel { get; set; }
    public List<Product> Products { get; set; }
}

public class ProductSalesArea : Entity
{
    public Guid ProductId { get; set; }
    public string Organization { get; set; }
    public string OrganizationName { get; set; }
    public string DistributionChannel { get; set; }
    public string DistributionChannelName { get; set; }
    public string Division { get; set; }
    public string DivisionName { get; set; }
    public virtual Product Product { get; set; }
}

public class ProductKeyword : Entity
{

    public Guid ProductId { get; set; }
    public string Value { get; set; }
    public virtual Product Product { get; set; }

}

public class ProductPhoto : Entity, IMedia
{

    public Guid ProductId { get; set; }
    public string Url { get; set; }
    public virtual Product Product { get; set; }

}

public class ProductVideo : Entity, IMedia
{
    public Guid ProductId { get; set; }
    public string Url { get; set; }
    public virtual Product Product { get; set; }

}

public class RestrictionProduct : Entity
{
    public Guid RestrictionId { get; set; }
    public Guid ProductId { get; set; }
    public virtual Restriction Restriction { get; set; }
    public virtual Product Product { get; set; }
}

public class ProductCombo : Entity
{
    public virtual Product Parent { get; set; }
    public virtual Product Children { get; set; }
    public Guid ParentId { get; set; }
    public Guid ChildrenId { get; set; }
    public string Position { get; set; }
    public int QuantityMaterial { get; set; }
    [NotMapped]
    public string MaterialCode { get; set; }
}

public class Price : Entity
{
    public Guid ProductId { get; set; }
    public string CenterCode { get; set; }
    public double PriceValue { get; set; }
    public Product Product { get; set; }
    public IList<PriceParameter> Parameters { get; set; }
}

public class PriceParameter : Entity
{
    public Guid PriceId { get; set; }
    public Guid ParameterId { get; set; }
    public string Code { get; set; }
    public string Value { get; set; }
    public Price Price { get; set; }
}

public class ProductAttributeValue
{
    public Guid ProductId { get; set; }
    public Guid AttributeValueId { get; set; }
    public virtual Product Product { get; set; }
    public virtual AttributeValue AttributeValue { get; set; }
}

public class AttributeValue : Entity
{
    public Guid AttributeKeyId { get; set; }
    public string Label { get; set; }
    public string Value { get; set; }
    public virtual AttributeKey AttributeKey { get; set; }
    [NotMapped]
    public int Total { get; set; }
}

public class AttributeKey : Entity
{
    public string Key { get; set; }
    public string Label { get; set; }
    public string Description { get; set; }
    public bool IsBeginOpen { get; set; }
    public FilterType Type { get; set; }
    public virtual List<AttributeValue> Values { get; set; }
}

public abstract class Entity
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public int Status { get; set; } = 1;
    public DateTime CreationDate { get; set; } = DateTime.Now.ToBrazilianTimezone();

    protected Entity()
    {
        CreationDate = DateTime.Now.ToBrazilianTimezone();
    }
}
```

Método de extensão **ToBrazilianTimezone**

```csharp
public static DateTime ToBrazilianTimezone(this DateTime dateTime)
{
    TimeZoneInfo targetTimeZone;

    if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
    {
        targetTimeZone = TimeZoneInfo.FindSystemTimeZoneById("E. South America Standard Time");
    }
    else
    {
        targetTimeZone = TimeZoneInfo.FindSystemTimeZoneById("America/Sao_Paulo");
    }

    var targetDatetime = TimeZoneInfo.ConvertTime(dateTime, targetTimeZone);

    return targetDatetime;
}
}
```