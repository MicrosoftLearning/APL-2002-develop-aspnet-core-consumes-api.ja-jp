---
lab:
  title: ASP.NET Core Razor Pages で API 応答をレンダリングする
  module: 'Module: Render API responses in ASP.NET Core Razor Pages'
---

この演習では、ASP.NET Core Razor Pages アプリで API からの応答をレンダリングする方法について学びます。 このコードは .cshtml* ファイルに*追加されます。 .cshtml.cs ファイルで操作を *実行する* コードは完了です。

## 目標

この演習を終了すると、次のことができるようになります。

* アプリに Razor キーワード (keyword)を実装する
* C# コードを Razor Pages 構文と統合する

## 前提条件

演習を完了するには、次がシステムにインストールされている必要があります。

* [Visual Studio Code](https://code.visualstudio.com)
* [最新の .NET 7.0 SDK](https://dotnet.microsoft.com/download/dotnet/7.0)
* Visual Studio Code 用の [C# 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

**この演習の推定所要時間**: 30 分

## 演習のシナリオ

この演習には、次の 2 つのコンポーネントがあります。

* API に HTTP 要求を送信する Web アプリ。 `http://localhost:5010` で実行されるアプリ
* HTTP 要求に応答する API。 `http://localhost:5050` で実行される API

![装飾](media/02-architecture.png)

## コードのダウンロード

このセクションでは、Fruit Web アプリと Fruit API のコードをダウンロードします。 また、Fruit API をローカルで実行して、Web アプリで使用できるようにします。

### タスク 1: API コードをダウンロードして実行する

1. 次のリンクを右クリックし、**[名前を付けてリンク先を保存]** オプションを選択します。 

    * [FruitAPI プロジェクト コード](https://raw.githubusercontent.com/MicrosoftLearning/APL-2002-develop-aspnet-core-consumes-api/master/Allfiles/Downloads/FruitAPI.zip) コード

1. **エクスプローラー**を起動し、ファイルが保存された場所に移動します。

1. ファイルを独自のフォルダーに解凍します。

1. **Windows ターミナル**または**コマンド プロンプト**を開き、API のコードを抽出した場所に移動します。

1. **Windows ターミナル** ペインで `dotnet` コマンドを実行します。

    ```
    dotnet run
    ```

1. 生成された出力の例を次に示します。 出力内の `Now listening on: http://localhost:5050` 行をメモします。 API のホストとポートを識別します。

    ```
    info: Microsoft.EntityFrameworkCore.Update[30100]
          Saved 3 entities to in-memory store.
    info: Microsoft.Hosting.Lifetime[14]
          Now listening on: http://localhost:5050
    info: Microsoft.Hosting.Lifetime[0]
          Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
          Hosting environment: Development
    info: Microsoft.Hosting.Lifetime[0]
          Content root path: 
          <project location>
    ```

>**注:** Fruit API は、演習の残りの部分で実行したままにします。 

### タスク 2: Web アプリ プロジェクトをダウンロードして開く

1. 次のリンクを右クリックし、**[名前を付けてリンク先を保存]** オプションを選択します。 

    * [Fruit Web アプリのレンダリング プロジェクト コード](https://raw.githubusercontent.com/MicrosoftLearning/APL-2002-develop-aspnet-core-consumes-api/master/Allfiles/Downloads/FruitWebApp-render.zip)

1. **エクスプローラー**を起動し、ファイルが保存された場所に移動します。

1. ファイルを独自のフォルダーに解凍します。

1. Visual Studio Code を起動して、**[ファイル]** を選択してから、メニュー バーの **[フォルダーを開く]** を選びます。

1. プロジェクト ファイルを解凍した場所に移動し、FruitWebApp-render* フォルダーを*選択します。

1. **[エクスプローラー]** ペインのプロジェクト構造は、次のスクリーンショットのようになります。 **[エクスプローラー]** ペインが表示されない場合は、メニュー バーで **[表示]** を選択し、**[エクスプローラー]** を選択します。

    ![Fruit Web アプリのプロジェクト構造を示すスクリーンショット。](media/03-web-app-render-structure.png)

>**注:** この演習全体を通して編集されている各ファイルのコードについては、時間をかけて確認してください。 このコードには多数のコメントがあるため、コード ベースを理解するのに役立ちます。

## [インデックス] ページにデータをレンダリングするコードを実装する

Fruit Web アプリは、ホーム ページに API サンプル データを表示します。 分離コード ファイルで実行された HTTP `GET` 操作によって返されるサンプル データを反復処理するコードを追加する必要があります。

### タスク 1: テーブルにデータをレンダリングするコードを追加する

1. [エクスプローラー] ペインの Program.cs を選択し、エディターでファイルを開きます。

1.  コメントの前に次のコードを追加します。

    ```csharp
    <tbody>
        
        @*  The Razor keyword @foreach is used to iterate through the
            data returned to the data model from the HTTP operations. *@
        @foreach (var obj in Model.FruitModels)
        {
            <tr>
                @* Display the name of the fruit. *@
                <td width="50%">@obj.name</td>
                @*  The following if statment is a Razor code block that changes the color 
                    and icon of the available indicator in the page rendering. *@
                @{
                    if (@obj.instock)
                    {
                        <td width="20%" class="text-md-center">
                            <i class="bi bi-check-circle" style="font-size: 1rem; color: green;"></i>&nbsp;Yes
                        </td>
                    }
                    else
                    {
                        <td width="20%" class="text-md-center">
                            <i class="bi bi-dash-circle" style="font-size: 1rem; color:red;"></i>&nbsp;No
                        </td>
                    }
                }
                <td width="30%" class="text-center">
                    @*  The following div contains information to handle the edit and delete functions. *@
                    <div class="w-75 btn-group btn-group-sm" role="group" style="text-align:center">
                        @* Routes to the Edit page and passes the id of the record. *@
                        <a asp-page="Edit" asp-route-id="@obj.id" class="btn btn-primary  mx-2">
                            <i class="bi bi-pencil-square"></i> Edit
                        </a>
                        @* Routes to the Delete page and passes the id of the record. *@
                        <a asp-page="Delete" asp-route-id="@obj.id" class="btn btn-danger mx-2">
                            <i class="bi bi-trash"></i> Delete
                        </a>
                    </div>
                </td>
            </tr>
        }
    </tbody>
    ```

1. 変更を Index.cshtml* に*保存し、コード内のコメントを確認します。

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、Web アプリを実行した状態でブラウザー ウィンドウが起動します

1. [インデックス] ページに API のサンプル データが表示されていることを確認します。

    >**注:** リスト**に**追加、編集 **、** および**削除**の各関数は、この演習の後半でコードを追加するまで機能しません。

    >**注:** アプリを実行する際、以下のプロンプトを無視しても問題ありません。

    ![自己署名証明書のインストール時に表示されるプロンプトのスクリーンショット。](media/install-cert.png)

1. 演習を続行するには、ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** の順に選択するか、**Shift + F5** キーを押します。

## リスト**に追加機能を**処理するコードを実装する

追加、編集、削除の各操作は、プロジェクト内の個別 *の .cshtml* ページで処理されます。 このセクションでは、Add.cshtml* ファイルにフォームを作成するコードを*追加して、リストにデータを追加できるようにします。

### タスク 1: データ追加フォームを作成するコードを追加する

1. [エクスプローラー] ペインの Program.cs を選択し、エディターでファイルを開きます。

1.  コメントの前に次のコードを追加します。

    ```csharp
    <form method="post">
        @*  The FruitModels.id is here so the full data model is represented on the page.
            The database behind the API will assign the id. *@
        <input hidden asp-for="FruitModels.id" />
        <div class="border p-3 mt-4" style="width:50%">
            <div class="row pb-2">
                <h2 class="text-primary pl-3">Add Fruit</h2>
                <hr />
            </div>
            <div class="mb-3">
                <label asp-for="FruitModels.name" class="h5"></label><br/>
                @* Empty text box for the name of the fruit to be added. *@
                <input type="text" asp-for="FruitModels.name" />
                <span asp-validation-for="FruitModels.name" class="text-danger"></span>
            </div>
            <div class="mb-3">
                <label asp-for="FruitModels.instock" class="h5"></label><br/>
                @* Render the true/false instock state from the record in an editable checkbox. *@
                <input type="checkbox" asp-for="FruitModels.instock" style="width:20px; height:20px" />
                <label class="h7"><i class="bi bi-arrow-left"></i>  Check the box if it's available.</label>
                <span asp-validation-for="FruitModels.instock" class="text-danger"></span>
            </div>
            @* Submit the addition or return to the Index page if the Add is cancelled.*@
            <button type="submit" class="btn btn-primary" style="width:150px;">Create</button>
            <a asp-page="Index" class="btn btn-secondary" style="width:150px;">Cancel</a>
        </div>
    </form>
    ```

1. 変更を Add.cshtml* に*保存し、コード内のコメントを確認します。

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、Web アプリを実行した状態でブラウザー ウィンドウが起動します

1. リスト ページで**新規**を選択します。

1. リストに追加するフルーツの名前を入力し、チェックボックスを選択して使用可能であることを示します。

1. [作成]** を選択**して一覧にエントリを追加すると、ホーム ページにルーティングされます。 エントリがリストに追加されたことを確認します。

1. 演習を続行するには、ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** の順に選択するか、**Shift + F5** キーを押します。

## 編集**機能を処理するコードを実装する**

このセクションでは、Edit.cshtml* ファイルにフォームを*作成して、リストへのデータの編集を有効にするコードを追加します。

### タスク 1: 編集フォームのコードを追加する

1. *エクスプローラー** ペインで Edit.cshtml* ファイルを**選択して、編集用に開きます。

1.  コメントの前に次のコードを追加します。

    ```csharp
    <form method="post">
        @*  The id for the data record is hidden because it needs to be available to the 
            code-behind processing, but it's not displayed. *@
        <input hidden asp-for="FruitModels.id" />
        <div class="border p-3 mt-4" style="width:50%">
            <div class="row pb-2">
                <h2 class="text-primary pl-3">Edit Fruit</h2>
                <hr />
            </div>
            <div class="mb-3">
                <label asp-for="FruitModels.name" class="h5"></label><br/>
                @* Render the current name of the fruit in an editable text box. *@
                <input type="text" asp-for="FruitModels.name" />
                <span asp-validation-for="FruitModels.name" class="text-danger"></span>
            </div>
            <div class="mb-3">
                <label asp-for="FruitModels.instock" class="h5"></label><br/>
                @* Render the true/false instock state from the record in an editable checkbox. *@
                <input type="checkbox" asp-for="FruitModels.instock" style="width:20px; height:20px" />
                <label class="h7"><i class="bi bi-arrow-left"></i>  Check the box if available.</label>
                <span asp-validation-for="FruitModels.instock" class="text-danger"></span>
            </div>
            @* Submit the changes or return to the Index page if the edit is cancelled.*@
            <button type="submit" class="btn btn-primary" style="width:150px;">Update</button>
            <a asp-page="Index" class="btn btn-secondary" style="width:150px;">Cancel</a>
        </div>
    </form>
    ```

1. Edit.cshtml* への変更を*保存し、コード内のコメントを確認します。

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、Web アプリを実行した状態でブラウザー ウィンドウが起動します

1. 変更するリスト内の項目を選択し、その行で [編集]** を選択**します。

1. フルーツの名前を編集し、チェックボックスを選択して、その可用性の状態を変更します。

1. [更新]** を選択**して変更を保存すると、ホーム ページにルーティングされます。 変更が一覧に表示されていることを確認します。

1. 演習を続行するには、ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** の順に選択するか、**Shift + F5** キーを押します。

## Delete** 機能を処理するコードを実装する**

このセクションでは、Delete.cshtml* ファイルにフォームを*作成するコードを追加して、リストからデータを削除できるようにします。

### タスク 1: 削除フォームのコードを追加する

1. *エクスプローラー** ペインで Delete.cshtml* ファイルを**選択して、編集用に開きます。

1.  コメントの前に次のコードを追加します。

    ```csharp
    <form method="post">
        @*  The id for the data record is hidden because it needs to be avaialable to the 
            code-behind processing, but it's not displayed. *@
        <input hidden asp-for="FruitModels.id" />
        <div class="border p-3 mt-4" style="width:50%">
            <div class="row pb-2">
                <h2 class="text-primary pl-3">Delete Fruit</h2>
                <hr />
            </div>
            <div class="mb-3">
                <label asp-for="FruitModels.name" class="h5"></label><br/>
                @* Render the name of the fruit in a non-editable text box. *@
                <input type="text" asp-for="FruitModels.name" disabled/>
                <span asp-validation-for="FruitModels.name" class="text-danger"></span>
            </div>
            <div class="mb-3">
                <label asp-for="FruitModels.instock" class="h5"></label><br/>
                @* Render the true/false instock state from the record in a non-editable checkbox. *@
                <input type="checkbox" asp-for="FruitModels.instock" style="width:20px; height:20px" disabled  />
                <span asp-validation-for="FruitModels.instock" class="text-danger"></span>
            </div>
            @* Submit the changes or return to the Index page if the delete is cancelled.*@
            <button type="submit" class="btn btn-danger " style="width:150px;">Delete</button>
            <a asp-page="Index" class="btn btn-secondary" style="width:150px;">Cancel</a>
        </div>
    </form>
    ```

1. Delete.cshtml* への変更を*保存し、コード内のコメントを確認します。

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、Web アプリを実行した状態でブラウザー ウィンドウが起動します

1. 削除するリスト内の項目を選択し、その行の [削除 **] を選択**します。

1. [削除] を**選択**すると、ホーム ページに戻ります。 削除したアイテムが一覧に表示されなくなったかどうかを確認します。

演習を終了する準備ができたら、次の手順を実行します。

* ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** を選択するか、**Shift + F5** キーを押します。 

* 実行中のターミナルで **Ctrl + C** キーを押して Fruit API を停止します。

## 確認

この演習では、以下の方法を学習しました。

* アプリに Razor キーワード (keyword)を実装する
* C# コードを Razor Pages 構文と統合する
