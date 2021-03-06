---
title: ASP.NET Core Razor 元件類別庫
author: guardrex
description: 探索如何將元件包含在來自外部元件程式庫的 Blazor 應用程式中。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 07/02/2019
uid: blazor/class-libraries
ms.openlocfilehash: 402b7b072554f63f85e7cf5e55336104d235a071
ms.sourcegitcommit: 776367717e990bdd600cb3c9148ffb905d56862d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2019
ms.locfileid: "68948438"
---
# <a name="aspnet-core-razor-components-class-libraries"></a>ASP.NET Core Razor 元件類別庫

依[Simon Timms](https://github.com/stimms)

元件可以在跨專案的[Razor 類別庫 (RCL)](xref:razor-pages/ui-class)中共用。 *Razor 元件類別庫*可以包含在:

* 方案中的另一個專案。
* NuGet 套件。
* 參考的 .NET 程式庫。

就像元件是一般的 .NET 類型, RCL 所提供的元件是一般的 .NET 元件。

## <a name="create-an-rcl"></a>建立 RCL

遵循<xref:blazor/get-started>文章中的指導方針, 設定您的環境以進行 Blazor。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

1. 建立新的專案。
1. 選取 [ASP.NET Core Web 應用程式]。 選取 [下一步]。
1. 在 [專案名稱] 欄位中提供專案名稱，或接受預設專案名稱。 本主題中的範例會使用專案名稱`MyComponentLib1`。 選取 [建立]。
1. 在 [建立新的 ASP.NET Core Web 應用程式] 對話方塊中，確認選取 [.NET Core] 和 [ASP.NET Core 3.0]。
1. 選取 [ **Razor 類別庫**] 範本。 選取 [建立]。
1. 將 RCL 新增至方案:
   1. 以滑鼠右鍵按一下方案。 選取 [**加入** > **現有專案**]。
   1. 流覽至 RCL 的專案檔。
   1. 選取 RCL 的專案檔 ( *.csproj*)。
1. 從應用程式新增參考 RCL:
   1. 以滑鼠右鍵按一下應用程式專案。 選取 [**新增** > **參考**]。
   1. 選取 [RCL] 專案。 選取 [確定]。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

1. 使用**Razor 類別庫**範本 (`razorclasslib`) 搭配命令 shell 中的[dotnet new](/dotnet/core/tools/dotnet-new)命令。 在下列範例中, 會建立名為`MyComponentLib1`的 RCL。 執行命令時, `MyComponentLib1`會自動建立保存的資料夾:

   ```console
   dotnet new razorclasslib -o MyComponentLib1
   ```

1. 若要將程式庫新增至現有的專案, 請在命令 shell 中使用[dotnet add reference](/dotnet/core/tools/dotnet-add-reference)命令。 在下列範例中, 會將 RCL 新增至應用程式。 從應用程式的專案資料夾中, 使用程式庫的路徑執行下列命令:

   ```console
   dotnet add reference {PATH TO LIBRARY}
   ```

---

## <a name="rcls-not-supported-for-client-side-apps"></a>用戶端應用程式不支援 RCLs

在目前的 ASP.NET Core 3.0 Preview 中, Razor 類別庫與 Blazor 用戶端應用程式不相容。 針對 Blazor 用戶端應用程式, 請在命令介面中使用範本所`blazorlib`建立的 Blazor 元件程式庫:

```console
dotnet new blazorlib -o MyComponentLib1
```

使用範本的`blazorlib`元件庫可以包含靜態檔案, 例如影像、JavaScript 和樣式表單。 在建立期間, 靜態檔案會內嵌到建立的元件檔 ( *.dll*) 中, 以允許取用元件, 而不必擔心如何包含其資源。 包含在`content`目錄中的任何檔案都會標示為內嵌資源。

## <a name="consume-a-library-component"></a>使用程式庫元件

若要使用另一個專案中的程式庫中所定義的元件, 請使用下列其中一種方法:

* 使用命名空間的完整類型名稱。
* 使用 Razor 的[ \@using](xref:mvc/views/razor#using)指示詞。 個別元件可以依名稱新增。

在下列範例中, `MyComponentLib1`是`SalesReport`包含元件的元件庫。

`SalesReport`元件可以使用其完整型別名稱和命名空間來加以參考:

```cshtml
<h1>Hello, world!</h1>

Welcome to your new app.

<MyComponentLib1.SalesReport />
```

如果使用`@using`指示詞將程式庫帶入範圍, 也可以參考此元件:

```cshtml
@using MyComponentLib1

<h1>Hello, world!</h1>

Welcome to your new app.

<SalesReport />
```

將指示詞包含在最上層的 *_Import razor*檔案中, 以將程式庫的元件提供給整個專案。 `@using MyComponentLib1` 將指示詞新增至任何層級的 *_Import razor*檔案, 以將命名空間套用至資料夾內的單一頁面或一組頁面。

## <a name="build-pack-and-ship-to-nuget"></a>組建、封裝和寄送至 NuGet

因為元件程式庫是標準的 .NET 程式庫, 所以封裝和傳送至 NuGet 的方式與將任何程式庫封裝和傳送至 NuGet 的方式並無不同。 封裝是使用命令 shell 中的[dotnet pack](/dotnet/core/tools/dotnet-pack)命令來執行:

```console
dotnet pack
```

使用命令 shell 中的[dotnet NuGet publish](/dotnet/core/tools/dotnet-nuget-push)命令, 將封裝上傳至 NuGet:

```console
dotnet nuget publish
```

使用`blazorlib`範本時, 靜態資源會包含在 NuGet 套件中。 程式庫取用者會自動接收腳本和樣式表單, 因此取用者不需要手動安裝資源。

## <a name="create-a-razor-components-class-library-with-static-assets"></a>建立具有靜態資產的 Razor 元件類別庫

RCL 可以包含靜態資產。 使用該程式庫的任何應用程式都可以使用靜態資產。 如需詳細資訊，請參閱 <xref:razor-pages/ui-class#create-an-rcl-with-static-assets>。

## <a name="additional-resources"></a>其他資源

* <xref:razor-pages/ui-class>
