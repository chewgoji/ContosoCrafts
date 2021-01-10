## Asp.net
* 由微軟支持並開發的一套跨平臺web框架
* Razor Page 運用 HTML 與 C#

建立 Asp.net Core Web App 
```
dotnet new webapp -o NAMETHEWEBAPP
```
建立 解決方案 sln
```
dotnet new sln
```
sln 中加入 webapp

```
dotnet sln add NAMETHEWEBAPP
```

根目錄```wwwroot, Pages, Program.cs, Startup.cs ....```
![根目錄](https://i.imgur.com/SZDUsnB.png)

## Add DataBase
SQL Server or NOSQL(Not Only SQL)
or even a FILE
在這練習中使用的是[products.json](https://gist.github.com/bradygaster/3d1fcf43d1d1e73ea5d6c1b5aab40130#file-products-json)![products.json](https://i.imgur.com/CH1MSof.png)

* 在```wwwroot```下創建```data``` folder放入```products.json```
* 在根目錄創建```Models``` (creat a c# class that describe what a product is)
* 使用 ```[JsonPropertyName("img")]```連接c# 與 json(like a post-it note)
* 最後使用ToString去serialize將所有property組合起來
```
public override string ToString() => JsonSerializer.Serialize<Product>(this);
```
## Adding a Service
:::info 
Web Service及 Web API的差別，整理後的結論：
1. 兩者都提供資料的方式，差異WebService是使用SOAP協定，WebAPI使用HTTP協定。
2. Window開發WebServie會有跨平台資料存取問題，而且資料操作複雜，WebAPI使用原生的HTTP協定，並無跨平台資料存取問題
3. Web API可以藉由 GET, POST等method，由使用者自由進行自訂的查詢等工作。
4. 符合RESTful的Web API為目前趨勢，優點是易讀易懂，故目前Google與Line所提供服務幾乎都是使用WebAPI的方式。
5. SAOP的Webservice有完整的WS-*安全認證及簽章標準，但Web API只能仰賴https。
:::

:::info
何謂REST
REST全名 Resource Representational State Transfer ，可譯為具象狀態傳輸。
簡單的說，就是一個單從發出的HTTP要求裡面所包含的資訊，就可以直接預期這要求會收到怎樣類型的資料。再更白話一點，就是人眼看得懂。

:::

**Single responsibility principle**

* 建立```service``` folder 建立c# class
* 這裡使用的是已經建立好的[service](https://gist.github.com/bradygaster/3d1fcf43d1d1e73ea5d6c1b5aab40130#file-jsonfileproductservice-cs)
* 利用collapsing text editor 方便閱讀
* 功能是去定位database也就是```products.json```
* 定位後deserialize成```Models```讀得懂的object
* 在```startup.cs```中分別在```ConfigureServices```與```UseEndpoints```啟用
```
services.AddTransient<JsonFileProductService>();
```
舉例：![](https://i.imgur.com/Bht7SEX.png)

## Data in a Razor Page

[Reference](https://www.youtube.com/watch?v=aP02__gMLtw&list=PLdo4fOcmZ0oW8nviYduHq7bmKode-p8Wy&index=5&ab_channel=dotNET)

* 在```Index.cshtml.cs```中，為上一章的service請求一個```logger```


## Making a API
Application Programming Interface

* Make dir Controllers
* under that ```dotnet new webapi -n YOURAPINAME```
* 呼叫之前建立的service
```
public ProductsController(JsonFileProductService productService)
        {
            this.ProductService = productService;
        }
```
* 取得service中的product
```
[HttpGet] //why httpget
public IEnumerable<Product> Get()
        {
            return ProductService.GetProducts();
        }
```
* 在```startup.cs```中分別在```ConfigureServices```與```UseEndpoints```啟用 Controllers

:::info
[route, endpoint, resource](https://stackoverflow.com/questions/56075017/difference-between-route-and-endpoint)

3 different concepts here:

Resource: ```{id: 42, type: employee, company: 5}```
Route: ```localhost:8080/employees/42```
Endpoint: ```GET localhost:8080/employees/42```
You can have different endpoints for the same route, such as ```DELETE localhost:8080/employees/42```. So endpoints are basically actions.

Also you can access the same resource by different routes such as ```localhost:8080/companies/5/employees/42```. So a route is a way to locate a resource.
:::

## Enhancing your Web API
* add rating section in database
```
/JsonFileProductService.cs

public void AddRating(string productId, int rating)
    {
        var products = GetProducts();

        var query = products.First(x => x.Id == productId);

        if(query.Ratings == null)
        {
            query.Ratings = new int[]{rating};
        }
        else
        {
            var ratings = query.Ratings.ToList();
            ratings.Add(rating);
            query.Ratings = ratings.ToArray();
        }

        using(var outputStream = File.OpenWrite(JsonFileName))
        {
            JsonSerializer.Serialize<IEnumerable<Product>>(
                new Utf8JsonWriter(outputStream, new JsonWriterOptions
                {
                    SkipValidation = true,
                    Indented = true,
                }),
                products
            );
        }
    }
```
```

/ProductsController.cs

[Route("Rate")]
[HttpGet]
public ActionResult Get([FromQuery]string productId, [FromQuery]int Rating)
{
    ProductService.AddRating(productId, Rating);
    return Ok();
}
```







## Add Blazer Page
Blazor 應用程式是以 元件(Components)
為基礎。 

```dotnet new razorcomponent -n YOUBLAZORPAGENAME```










## Publish an ASP.NET Core app to Azure with Visual Studio Code
### Generate the deployment package locally

[Reference](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/publish-to-azure-webapp-using-vscode?view=aspnetcore-5.0)

```
dotnet publish -c Release -o ./publish
```
### Publish to Azure App Service

:::info
Install the Azure App Service Extension FIRST!!!
:::

* Right click the ```publish``` folder and select ```Deploy to Web App...```
* Select the subscription you want to create the Web App
* Select ```Create New Web App```
* Enter a name for the Web App