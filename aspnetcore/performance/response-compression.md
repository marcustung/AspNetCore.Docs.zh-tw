---
title: ASP.NET Core 中的回應壓縮
author: guardrex
description: 了解回應壓縮及如何使用 ASP.NET Core 應用程式中的回應壓縮中介軟體。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/09/2019
uid: performance/response-compression
ms.openlocfilehash: e320e87179f9f1b9773a55c380684a3f3f712632
ms.sourcegitcommit: 89fcc6cb3e12790dca2b8b62f86609bed6335be9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2019
ms.locfileid: "68993465"
---
# <a name="response-compression-in-aspnet-core"></a>ASP.NET Core 中的回應壓縮

作者：[Luke Latham](https://github.com/guardrex)

[檢視或下載範例程式碼](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples) \(英文\) ([如何下載](xref:index#how-to-download-a-sample))

網路頻寬是有限的資源。 減少回應的大小通常應用程式的回應速度通常會大幅增加。 減少承載大小的其中一種方法是壓縮應用程式的回應。

## <a name="when-to-use-response-compression-middleware"></a>使用回應壓縮中介軟體的時機

在 IIS、Apache 或 Nginx 中使用以伺服器為基礎的回應壓縮技術。 中介軟體的效能可能與伺服器模組不相符。 [Http.sys 伺服器](xref:fundamentals/servers/httpsys)伺服器和[Kestrel](xref:fundamentals/servers/kestrel)伺服器目前未提供內建的壓縮支援。

當您是時, 請使用回應壓縮中介軟體:

* 無法使用下列以伺服器為基礎的壓縮技術:
  * [IIS 動態壓縮模組](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Apache mod_deflate 模組](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Nginx 壓縮和解壓縮](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* 直接裝載于:
  * Http.sys[伺服器](xref:fundamentals/servers/httpsys)(先前稱為 WebListener)
  * [Kestrel 伺服器](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>回應壓縮

通常, 任何未原生壓縮的回應都可以因回應壓縮而受益。 原本不壓縮的回應通常包括:CSS、JavaScript、HTML、XML 和 JSON。 您不應該壓縮原生壓縮的資產, 例如 PNG 檔案。 如果您嘗試進一步壓縮原生壓縮的回應, 則在處理壓縮所花費的時間內, 任何小型額外的大小和傳輸時間縮減可能會失色。 不要壓縮小於150-1000 個位元組的檔案 (視檔案的內容和壓縮效率而定)。 壓縮小型檔案的額外負荷可能會產生比未壓縮檔案更大的壓縮檔案。

當用戶端可以處理壓縮內容時, 用戶端必須透過要求傳送`Accept-Encoding`標頭, 以通知伺服器其功能。 當伺服器傳送壓縮的內容時, 它必須在`Content-Encoding`標頭中包含有關壓縮回應編碼方式的資訊。 下表顯示中介軟體支援的內容編碼方式。

::: moniker range=">= aspnetcore-2.2"

| `Accept-Encoding`標頭值 | 支援的中介軟體 | 描述 |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | 是 (預設值)        | [Brotli 壓縮資料格式](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | 否                   | [DEFLATE 壓縮資料格式](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | 否                   | [W3C 有效率的 XML 交換](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | 是                  | [GZIP 檔案格式](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | 是                  | 「沒有編碼」識別碼:回應不得編碼。 |
| `pack200-gzip`                  | 否                   | [JAVA 封存的網路傳輸格式](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | 是                  | 未明確要求任何可用的內容編碼 |

::: moniker-end

::: moniker range="< aspnetcore-2.2"

| `Accept-Encoding`標頭值 | 支援的中介軟體 | 描述 |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | 否                   | [Brotli 壓縮資料格式](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | 否                   | [DEFLATE 壓縮資料格式](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | 否                   | [W3C 有效率的 XML 交換](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | 是 (預設值)        | [GZIP 檔案格式](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | 是                  | 「沒有編碼」識別碼:回應不得編碼。 |
| `pack200-gzip`                  | 否                   | [JAVA 封存的網路傳輸格式](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | 是                  | 未明確要求任何可用的內容編碼 |

::: moniker-end

如需詳細資訊, 請參閱[IANA 官方內容編碼清單](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry)。

中介軟體可讓您新增自訂`Accept-Encoding`標頭值的其他壓縮提供者。 如需詳細資訊, 請參閱下面的[自訂提供者](#custom-providers)。

中介軟體能夠在用戶端傳送時, 回應品質值`q`(qvalue,) 加權, 以設定壓縮配置的優先順序。 如需詳細資訊，請參閱[RFC 7231：接受編碼](https://tools.ietf.org/html/rfc7231#section-5.3.4)。

壓縮演算法會受到壓縮速度與壓縮效率之間的取捨。 此內容中的*有效性*是指壓縮之後的輸出大小。 最大的大小是透過*最佳*壓縮來達成。

下表說明有關要求、傳送、快取和接收壓縮內容的標頭。

| 頁首             | 角色 |
| ------------------ | ---- |
| `Accept-Encoding`  | 從用戶端傳送到伺服器, 以指示用戶端可接受的內容編碼配置。 |
| `Content-Encoding` | 從伺服器傳送到用戶端, 以指出內容在承載中的編碼方式。 |
| `Content-Length`   | 當進行`Content-Length`壓縮時, 會移除標頭, 因為當回應壓縮時, 本文內容會變更。 |
| `Content-MD5`      | 當進行`Content-MD5`壓縮時, 會移除標頭, 因為本文內容已變更, 而且雜湊已不再有效。 |
| `Content-Type`     | 指定內容的 MIME 類型。 每個回應都應該`Content-Type`指定其。 中介軟體會檢查此值, 以判斷是否應該壓縮回應。 中介軟體會指定一組可編碼的[預設 MIME 類型](#mime-types), 但您可以取代或加入 MIME 類型。 |
| `Vary`             | 當伺服器將的值`Accept-Encoding`傳送給用戶端和 proxy 時`Vary` , 標頭會向用戶端或 proxy 指出它應該根據要求的`Accept-Encoding`標頭值來快取 (改變) 回應。 傳回具有`Vary: Accept-Encoding`標頭之內容的結果是, 會分別快取壓縮和未壓縮的回應。 |

使用[範例應用程式](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples)探索回應壓縮中介軟體的功能。 此範例說明:

* 使用 Gzip 和自訂壓縮提供者來壓縮應用程式回應。
* 如何將 MIME 類型新增至 MIME 類型的預設清單以進行壓縮。

## <a name="package"></a>套件

::: moniker range=">= aspnetcore-3.0"

回應壓縮中介軟體是由[AspNetCore. ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/)套件所提供, 它會隱含地包含在 ASP.NET Core 應用程式中。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

若要在專案中包含中介軟體, 請新增[AspNetCore 應用程式中繼套件](xref:fundamentals/metapackage-app)的參考, 其中包含[AspNetCore. ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/)封裝。

::: moniker-end

## <a name="configuration"></a>組態

::: moniker range=">= aspnetcore-2.2"

下列程式碼示範如何針對預設 MIME 類型和壓縮提供者 ([Brotli](#brotli-compression-provider)和[Gzip](#gzip-compression-provider)) 啟用回應壓縮中介軟體:

::: moniker-end

::: moniker range="< aspnetcore-2.2"

下列程式碼示範如何為預設 MIME 類型和[Gzip 壓縮提供者](#gzip-compression-provider)啟用回應壓縮中介軟體:

::: moniker-end

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

附註：

* `app.UseResponseCompression` 必須在 `app.UseMvc` 之前呼叫。
* 使用[Fiddler](https://www.telerik.com/fiddler)、 [Firebug](https://getfirebug.com/)或`Accept-Encoding` [Postman](https://www.getpostman.com/)之類的工具來設定要求標頭, 並研究回應標頭、大小和主體。

不使用`Accept-Encoding`標頭將要求提交至範例應用程式, 並觀察回應是否已解壓縮。 `Content-Encoding` 和`Vary`標頭不存在於回應中。

![顯示要求不含接受編碼標頭之結果的 Fiddler 視窗。 回應不會壓縮。](response-compression/_static/request-uncompressed.png)

::: moniker range=">= aspnetcore-2.2"

使用`Accept-Encoding: br`標頭 (Brotli 壓縮) 將要求提交至範例應用程式, 並觀察回應是否已壓縮。 `Content-Encoding` 和`Vary`標頭出現在回應中。

![Fiddler 視窗, 顯示具有接受編碼標頭和值為 br 之要求的結果。 Vary 和內容編碼標頭會新增至回應。 回應已壓縮。](response-compression/_static/request-compressed-br.png)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

使用`Accept-Encoding: gzip`標頭將要求提交至範例應用程式, 並觀察回應是否已壓縮。 `Content-Encoding` 和`Vary`標頭出現在回應中。

![Fiddler 視窗, 顯示具有接受編碼標頭和 gzip 值之要求的結果。 Vary 和內容編碼標頭會新增至回應。 回應已壓縮。](response-compression/_static/request-compressed.png)

::: moniker-end

## <a name="providers"></a>提供者

::: moniker range=">= aspnetcore-2.2"

### <a name="brotli-compression-provider"></a>Brotli 壓縮提供者

使用壓縮[Brotli 壓縮資料格式](https://tools.ietf.org/html/rfc7932)的回應。 <xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProvider>

如果未將任何壓縮提供者明確新增<xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>至:

* Brotli 壓縮提供者預設會加入至壓縮提供者陣列, 以及[Gzip 壓縮提供者](#gzip-compression-provider)。
* 當用戶端支援 Brotli 壓縮資料格式時, 壓縮會預設為 Brotli 壓縮。 如果用戶端不支援 Brotli, 當用戶端支援 Gzip 壓縮時, 壓縮會預設為 Gzip。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

明確新增任何壓縮提供者時, 必須新增 Brotoli 壓縮提供者:

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

使用<xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProviderOptions>設定壓縮等級。 Brotli 壓縮提供者預設為最快速的壓縮層級 ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)), 這可能不會產生最有效率的壓縮。 如果需要最有效率的壓縮, 請設定中介軟體以獲得最佳壓縮。

| 壓縮等級 | 描述 |
| ----------------- | ----------- |
| [CompressionLevel.Fastest](xref:System.IO.Compression.CompressionLevel) | 即使產生的輸出未以最佳方式壓縮, 壓縮也應該儘快完成。 |
| [CompressionLevel. NoCompression](xref:System.IO.Compression.CompressionLevel) | 不應該執行壓縮。 |
| [CompressionLevel.Optimal](xref:System.IO.Compression.CompressionLevel) | 回應應以最佳方式壓縮, 即使壓縮需要較長的時間才能完成也一樣。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<BrotliCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

::: moniker-end

### <a name="gzip-compression-provider"></a>Gzip 壓縮提供者

使用來壓縮具有[GZIP 檔案格式](https://tools.ietf.org/html/rfc1952)的回應。 <xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider>

如果未將任何壓縮提供者明確新增<xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>至:

::: moniker range=">= aspnetcore-2.2"

* Gzip 壓縮提供者預設會加入壓縮提供者的陣列, 以及[Brotli 壓縮提供者](#brotli-compression-provider)。
* 當用戶端支援 Brotli 壓縮資料格式時, 壓縮會預設為 Brotli 壓縮。 如果用戶端不支援 Brotli, 當用戶端支援 Gzip 壓縮時, 壓縮會預設為 Gzip。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* Gzip 壓縮提供者預設會加入至壓縮提供者陣列。
* 當用戶端支援 Gzip 壓縮時, 壓縮會預設為 Gzip。

::: moniker-end

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

當明確新增任何壓縮提供者時, 必須加入 Gzip 壓縮提供者:

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

::: moniker-end

使用<xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions>設定壓縮等級。 Gzip 壓縮提供者預設為最快速的壓縮層級 ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)), 這可能不會產生最有效率的壓縮。 如果需要最有效率的壓縮, 請設定中介軟體以獲得最佳壓縮。

| 壓縮等級 | 描述 |
| ----------------- | ----------- |
| [CompressionLevel.Fastest](xref:System.IO.Compression.CompressionLevel) | 即使產生的輸出未以最佳方式壓縮, 壓縮也應該儘快完成。 |
| [CompressionLevel. NoCompression](xref:System.IO.Compression.CompressionLevel) | 不應該執行壓縮。 |
| [CompressionLevel.Optimal](xref:System.IO.Compression.CompressionLevel) | 回應應以最佳方式壓縮, 即使壓縮需要較長的時間才能完成也一樣。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>自訂提供者

使用<xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider>建立自訂壓縮。 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*>表示此`ICompressionProvider`產生的內容編碼。 中介軟體會使用這項資訊, 根據要求`Accept-Encoding`標頭中所指定的清單來選擇提供者。

使用範例應用程式時, 用戶端會提交含有`Accept-Encoding: mycustomcompression`標頭的要求。 中介軟體會使用自訂壓縮實作為, 並傳回具有`Content-Encoding: mycustomcompression`標頭的回應。 用戶端必須能夠解壓縮自訂編碼, 才能讓自訂壓縮實行正常執行。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/3.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/2.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

::: moniker-end

使用`Accept-Encoding: mycustomcompression`標頭將要求提交至範例應用程式, 並觀察回應標頭。 `Vary` 和`Content-Encoding`標頭出現在回應中。 範例不會壓縮回應主體 (未顯示)。 範例的`CustomCompressionProvider`類別中沒有壓縮的執行。 不過, 此範例會顯示您將在哪裡執行這種壓縮演算法。

![Fiddler 視窗, 顯示具有接受編碼標頭和值為 mycustomcompression 之要求的結果。 Vary 和內容編碼標頭會新增至回應。](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>MIME 類型

中介軟體會針對壓縮指定一組預設的 MIME 類型:

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

以回應壓縮中介軟體選項取代或附加 MIME 類型。 請注意, `text/*`不支援萬用字元 MIME 類型, 例如。 範例應用程式會為新增 MIME 類型`image/svg+xml` , 並壓縮並提供 ASP.NET Core 的橫幅影像 (*橫幅. svg*)。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

::: moniker-end

## <a name="compression-with-secure-protocol"></a>使用安全通訊協定進行壓縮

透過安全連線的`EnableForHttps`壓縮回應可以使用選項來控制, 預設為停用。 以動態產生的頁面使用壓縮, 可能會導致安全性問題, 例如[犯罪](https://wikipedia.org/wiki/CRIME_(security_exploit))和[入侵](https://wikipedia.org/wiki/BREACH_(security_exploit))攻擊。

## <a name="adding-the-vary-header"></a>新增 Vary 標頭

根據`Accept-Encoding`標頭壓縮回應時, 可能會有多個壓縮版本的回應和未壓縮的版本。 若要指示用戶端和 proxy 快取有多個版本存在且應該儲存, `Vary`則標頭會加`Accept-Encoding`上值。 在 ASP.NET Core 2.0 或更新版本中, 中介軟體`Vary`會在回應壓縮時自動新增標頭。

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Nginx 反向 proxy 後方發生中介軟體問題

當要求由 Nginx proxy 時, `Accept-Encoding`會移除標頭。 `Accept-Encoding`移除標頭可防止中介軟體壓縮回應。 如需詳細資訊，請參閱 [NGINX：壓縮和解壓縮](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)。 此問題是藉由[瞭解 Nginx (aspnet/BasicMiddleware \#123) 的傳遞壓縮](https://github.com/aspnet/BasicMiddleware/issues/123)來追蹤。

## <a name="working-with-iis-dynamic-compression"></a>使用 IIS 動態壓縮

如果您在想要針對應用程式停用的伺服器層級上設定作用中的 IIS 動態壓縮模組, 請停用包含 web.config 檔案新增的模組。 如需詳細資訊，請參閱[停用 IIS 模組](xref:host-and-deploy/iis/modules#disabling-iis-modules)。

## <a name="troubleshooting"></a>疑難排解

使用[Fiddler](https://www.telerik.com/fiddler)、 [Firebug](https://getfirebug.com/)或[Postman](https://www.getpostman.com/)等工具, 可`Accept-Encoding`讓您設定要求標頭, 並研究回應標頭、大小和主體。 根據預設, 回應壓縮中介軟體會壓縮符合下列條件的回應:

::: moniker range=">= aspnetcore-2.2"

* 標頭存在, 其值為`br`、 `gzip`、 `*`或自訂編碼, 其符合您所建立的自訂壓縮提供者。 `Accept-Encoding` 此值不得為`identity`或具有 0 (零) 的品質值 (qvalue, `q`) 設定。
* 必須設定 mime 類型`Content-Type`(), 而且必須符合中設定<xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>的 mime 類型。
* 要求不能包含`Content-Range`標頭。
* 除非在回應壓縮中介軟體選項中設定了安全通訊協定 (HTTPs), 否則要求必須使用不安全的通訊協定 (HTTP)。 *請注意, 在啟用安全內容壓縮時,[上述](#compression-with-secure-protocol)的危險。*

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* 標頭存在, 其值為`gzip`、 `*`或符合您所建立之自訂壓縮提供者的自訂編碼。 `Accept-Encoding` 此值不得為`identity`或具有 0 (零) 的品質值 (qvalue, `q`) 設定。
* 必須設定 mime 類型`Content-Type`(), 而且必須符合中設定<xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>的 mime 類型。
* 要求不能包含`Content-Range`標頭。
* 除非在回應壓縮中介軟體選項中設定了安全通訊協定 (HTTPs), 否則要求必須使用不安全的通訊協定 (HTTP)。 *請注意, 在啟用安全內容壓縮時,[上述](#compression-with-secure-protocol)的危險。*

::: moniker-end

## <a name="additional-resources"></a>其他資源

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Mozilla 開發人員網路:Accept-Encoding](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 區段 3.1.2.1:Content Codings](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230 區段 4.2.3:Gzip 編碼](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [GZIP 檔案格式規格版本4。3](https://www.ietf.org/rfc/rfc1952.txt)
