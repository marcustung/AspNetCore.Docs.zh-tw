---
title: ASP.NET Core Blazor 版面配置
author: guardrex
description: 瞭解如何為 Blazor 應用程式建立可重複使用的版面配置元件。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 07/02/2019
uid: blazor/layouts
ms.openlocfilehash: 2d652e149381f0a93e3135da978ab5737d47c6f1
ms.sourcegitcommit: 0b9e767a09beaaaa4301915cdda9ef69daaf3ff2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2019
ms.locfileid: "68948218"
---
# <a name="aspnet-core-blazor-layouts"></a>ASP.NET Core Blazor 版面配置

依[Rainer Stropek](https://www.timecockpit.com)

某些應用程式專案 (例如功能表、著作權訊息和公司標誌) 通常是應用程式整體版面配置的一部分, 並由應用程式中的每個元件所使用。 每當其中一個專案需要更新時, 將這些專案的程式碼複製到應用程式&mdash;的所有元件都不是有效的方法, 每個元件都必須更新。 這種複製很容易維護, 而且可能會在一段時間後導致不一致的內容。 *版面*配置會解決此問題。

就技術上而言, 版面配置只是另一個元件。 版面配置定義于 Razor 範本或程式碼中C# , 而且可以使用[資料](xref:blazor/components#data-binding)系結、相依性[插入](xref:blazor/dependency-injection)和其他元件案例。

若要將*元件*轉換成*版面*配置, 元件:

* 繼承自`LayoutComponentBase`, 其`Body`定義配置內呈現之內容的屬性。
* 會使用 Razor 語法`@Body` , 在版面配置標記中, 指定轉譯內容的位置。

下列程式碼範例顯示*MainLayout*版面配置元件的 razor 範本。 版面配置會`LayoutComponentBase`繼承並設定`@Body`導覽列和頁尾之間的:

[!code-cshtml[](layouts/sample_snapshot/3.x/MainLayout.razor?highlight=1,13)]

## <a name="specify-a-layout-in-a-component"></a>在元件中指定版面配置

使用 Razor `@layout`指示詞將版面配置套用至元件。 編譯器會將`@layout`轉換`LayoutAttribute`成, 並將其套用至元件類別。

下列元件 ( *MasterList*) 的內容會插入至`MainLayout`的位置: `@Body`

[!code-cshtml[](layouts/sample_snapshot/3.x/MasterList.razor?highlight=1)]

## <a name="centralized-layout-selection"></a>集中式版面配置選取

應用程式的每個資料夾都可以選擇性地包含名為 *_Imports*的範本檔案。 編譯器包含相同資料夾中所有 Razor 範本的匯入檔案中所指定的指示詞, 並且會以遞迴方式在其所有子資料夾中進行。 因此, 包含`@layout MainLayout`的 *_Imports razor*檔案可確保資料夾中的所有元件都使用。 `MainLayout` 不需要重複新增`@layout MainLayout`至資料夾和子資料夾內的所有*razor*檔案。 `@using`指示詞也會以同樣的方式套用至元件。

下列 *_Imports razor*檔案匯入:

* `MainLayout`.
* 相同資料夾和任何子資料夾中的所有 Razor 元件。
* `BlazorApp1.Data` 命名空間。
 
[!code-cshtml[](layouts/sample_snapshot/3.x/_Imports.razor)]

*_Imports* razor 檔案類似[razor 視圖和頁面的 _ViewImports](xref:mvc/views/layout#importing-shared-directives)檔案, 但特別適用于 razor 元件檔案。

Blazor 範本會使用 *_Imports*檔案進行版面配置選擇。 從 Blazor 範本建立的應用程式包含專案根目錄和*Pages*資料夾中的 *_Imports razor*檔案。

## <a name="nested-layouts"></a>嵌套版面配置

應用程式可以包含嵌套的版面配置。 元件可以參考配置, 而該配置會轉而參考另一個版面配置。 例如, 用來建立多層級功能表結構的嵌套配置。

下列範例顯示如何使用嵌套的配置。 *EpisodesComponent. razor*檔案是要顯示的元件。 元件會參考`MasterListLayout`:

[!code-cshtml[](layouts/sample_snapshot/3.x/EpisodesComponent.razor?highlight=1)]

*MasterListLayout razor*檔案提供`MasterListLayout`。 版面配置會參考另一個`MasterLayout`版面配置, 也就是其呈現位置。 `EpisodesComponent`會在出現`@Body`的位置呈現:

[!code-cshtml[](layouts/sample_snapshot/3.x/MasterListLayout.razor?highlight=1,9)]

最後, `MasterLayout`在*MasterLayout*中, 包含最上層的版面配置元素, 例如頁首、主功能表和頁尾。 `MasterListLayout`會`EpisodesComponent`在出現的`@Body`處呈現:

[!code-cshtml[](layouts/sample_snapshot/3.x/MasterLayout.razor?highlight=6)]

## <a name="additional-resources"></a>其他資源

* <xref:mvc/views/layout>
