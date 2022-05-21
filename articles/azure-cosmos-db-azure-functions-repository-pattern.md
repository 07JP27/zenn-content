---
title: "Azure Cosmos DBをRepositoryパターンでAzure FunctionsにDIして使う"
emoji: "🌍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cosmosdb", "azurefunctions"]
published: true
---

Azure Cosmos DBとAzure Functionsを連携させるにはバインディングなどがありますが、そこそこ規模があるような実際のプロジェクトで使えるようにRepositoryパターンでDIする方式を試してみます。

# 完全版のソリューション
今回作成した実行可能なサンプルはここに置いています。
https://github.com/07JP27/CosmosDBFunction-Sample

# プロジェクト構成
- .AzureFunctions：名前の通りAzure Functionsのプロジェクト
- .Core：InterfaceファイルやEntityファイルの置き場（ここにCosmosDBに関するファイルは配置しない）
- .Infrastructure：Interfaceを介したCosmos DBに対する実際の処理を書くプロジェクト
![](https://storage.googleapis.com/zenn-user-upload/4d70b31d4517cf12fc4b5c7c.png)

# コード
## Core -> IRepositoryBase.cs / IProductRepository.cs
Repository抽象化のためのInterfaceを実装します。
``` cs
public interface IRepositoryBase<T> where T : EntityBase
{
    Task<IEnumerable<T>> GetAllItemAsync();
    Task<T> GetByIdAsync(string id);
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
}
```
IProductRepositoryは実装要件によって基底クラス以外のメソッドを実装しますが、今回は基底クラスをそのまま継承します。
``` cs
public interface IProductRepository: IRepositoryBase<Product>
{
}
```

## Infrastructure -> IServiceCollectionExtensions.cs
FunctionsのStartupでIServiceCollectionに対して`AddCosmosDb`でCosmos DBのクライアントをDIできるようにExtensionを作成します。
``` cs
public static class IServiceCollectionExtensions
{
    public static IServiceCollection AddCosmosDb(this IServiceCollection services,
                                                    string endpointUrl,
                                                    string primaryKey,
                                                    string databaseName,
                                                    List<ContainerInfo> containers)
    {
        Microsoft.Azure.Cosmos.CosmosClient client = new Microsoft.Azure.Cosmos.CosmosClient(endpointUrl, primaryKey);
        CosmosDbContainerFactory cosmosDbClientFactory = new CosmosDbContainerFactory(client, databaseName, containers);
        services.AddSingleton<ICosmosDbContainerFactory>(cosmosDbClientFactory);
        return services;
    }
}
```

## Infrastructure -> RepositoryBase.cs / ProductRepository.cs
Coreで抽象化したInterfaceに対して実際にCosmos DBへアクセスするRepositoryを実装します。
``` cs
public abstract class RepositoryBase<T> : IRepositoryBase<T>, IContainerContext<T> where T : EntityBase
{
    private readonly ICosmosDbContainerFactory _cosmosDbContainerFactory;
    private readonly Microsoft.Azure.Cosmos.Container _container;
    public abstract string ContainerName { get; }
    public abstract PartitionKey ResolvePartitionKey(string entityId);
    public virtual string GenerateId(T entity) => Guid.NewGuid().ToString();

    public RepositoryBase(ICosmosDbContainerFactory cosmosDbContainerFactory)
    {
        this._cosmosDbContainerFactory = cosmosDbContainerFactory ?? throw new ArgumentNullException(nameof(ICosmosDbContainerFactory));
        this._container = this._cosmosDbContainerFactory.GetContainer(ContainerName)._container;
    }

    public async Task<IEnumerable<T>> GetAllItemAsync()
    {
        try
        {
            List<T> items = new List<T>();
            FeedIterator<T> feedIterator =  _container.GetItemQueryIterator<T>();
            while(feedIterator.HasMoreResults)
            {
                items.AddRange(await feedIterator.ReadNextAsync());
            }
            return items;
        }
        catch (CosmosException e)
        {
            if (e.StatusCode == HttpStatusCode.NotFound)
            {
                throw new EntityNotFoundException();
            }
            throw;
        }
    }

    public async Task<T> GetByIdAsync(string id)
    {
        try
        {
            var item = await _container.ReadItemAsync<T>(id, ResolvePartitionKey(id));
            return item;
        }
        catch (CosmosException e)
        {
            if (e.StatusCode == HttpStatusCode.NotFound)
            {
                throw new EntityNotFoundException();
            }
            throw;
        }
    }

    public async Task<T> AddAsync(T entity)
    {
        try
        {
            entity.Id = GenerateId(entity);
            var item = await _container.CreateItemAsync(entity);
            return item;
        }
        catch (CosmosException e)
        {
            if (e.StatusCode == HttpStatusCode.Conflict)
            {
                throw new EntityAlreadyExistsException();
            }
            throw;
        }
    }

    public async Task UpdateAsync(T entity)
    {
        try
        {
            await _container.ReplaceItemAsync<T>(entity, entity.Id);
        }
        catch (CosmosException e)
        {
            if (e.StatusCode == HttpStatusCode.NotFound)
            {
                throw new EntityNotFoundException();
            }
            throw;
        }
    }

    public async Task DeleteAsync(T entity)
    {
        try
        {
            await _container.DeleteItemAsync<T>(entity.Id, ResolvePartitionKey(entity.Id));
        }
        catch (CosmosException e)
        {
            if (e.StatusCode == HttpStatusCode.NotFound)
            {
                throw new EntityNotFoundException();
            }
            throw;
        }
    }
}
```
ProductRepositoryでは基本的にRepositoryBaseを継承し、コンテナ名などRepository固有の処理を記述します。
``` cs
public class ProductRepository : RepositoryBase<Product>, IProductRepository
{
    public ProductRepository(ICosmosDbContainerFactory factory) : base(factory) { }
    public override string ContainerName { get; } = "Product";
    public override PartitionKey ResolvePartitionKey(string entityId) => new PartitionKey(entityId.Split(':')[0]);
}
```

## AzureFunction -> Startup.cs
エクステンションで作成した`AddCosmosDb`メソッドを使用してCosmosDBインスタンスを追加して、その下でRepositoryInterfaceを紐付けます。 
``` cs
public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        var configuration = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile($"local.settings.json", optional: true, reloadOnChange: true)
            .AddEnvironmentVariables()
            .Build();
        builder.Services.AddSingleton<IConfiguration>(configuration);
        var cosmosDbConfig = configuration.GetSection("ConnectionStrings:FunctionDB").Get<CosmosDbSettings>();
        builder.Services.AddCosmosDb(cosmosDbConfig.EndpointUrl,
                                cosmosDbConfig.PrimaryKey,
                                cosmosDbConfig.DatabaseName,
                                cosmosDbConfig.Containers);
        builder.Services.AddScoped<IProductRepository, ProductRepository>();
    }
}
```

## AzureFunction -> ProductFunction.cs
DIコンテナから対応するRepositoryを受け取って呼び出します。
```cs
public class ProductFunction
{
    private readonly IProductRepository _repo;

    public ProductFunction(IProductRepository repo)
    {
        this._repo = repo ?? throw new ArgumentNullException(nameof(repo));
    }

    [FunctionName("GetProduct")]
    public async Task<IActionResult> GetProduct(
        [HttpTrigger(AuthorizationLevel.Function, "get", Route = "product")] HttpRequest req,
        ILogger log)
    {
        string id = req.Query["id"];
        if (!String.IsNullOrEmpty(id))
        {
            try
            {
                Product item = await _repo.GetByIdAsync(id);
                return new OkObjectResult(item);
            }
            catch (EntityNotFoundException)
            {
                return new NotFoundResult();
            }
        }

        IEnumerable<Product> items = await _repo.GetAllItemAsync();
        return new OkObjectResult(items);
    }

    [FunctionName("AddProduct")]
    public async Task<IActionResult> AddProduct(
        [HttpTrigger(AuthorizationLevel.Function, "post", Route = "product")] HttpRequest req,
        ILogger log)
    {
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic item = JsonConvert.DeserializeObject<Product>(requestBody);

        await _repo.AddAsync(item);
        return new OkObjectResult(item);
    }
}
```

# 参考
公式のサンプルです（サンプル内で使用されているSDKが古いので注意）
https://github.com/Azure-Samples/PartitionedRepository
