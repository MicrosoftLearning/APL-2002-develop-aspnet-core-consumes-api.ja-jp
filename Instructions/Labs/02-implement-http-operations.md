---
lab:
  title: '演習: ASP.NET Core Razor Pages で HTTP 操作を実装する'
  module: 'Module: Implement HTTP operations in ASP.NET Core Razor Pages'
---

この演習では、必要なコードを ASP.NET Core Razor Pages アプリに追加して HTTP クライアントを作成し、`GET`、`POST`、`PUT`、`DELETE` 操作を実行する方法について学習します。 このコードは *.cshtml.cs* 分離コード ファイルに追加されます。 *.cshtml* ファイル内のデータをレンダリングするコードが完了しました。

## 目標

この演習を終了すると、次のことができるようになります。

* HTTP クライアントとして `IHttpClientFactory` を実装する
* ASP.NET Core Razor Pages で HTTP 操作を実装する

## 前提条件

演習を完了するには、次がシステムにインストールされている必要があります。

* [Visual Studio Code](https://code.visualstudio.com)
* [最新の .NET 7.0 SDK](https://dotnet.microsoft.com/download/dotnet/7.0)
* Visual Studio Code 用の [C# 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

**この演習の推定所要時間**: 30 分

## 演習のシナリオ

この演習には、次の 2 つのコンポーネントがあります。

* API に HTTP 要求を送信するアプリ。 `http://localhost:5010` で実行されるアプリ
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

    * [Fruit Web アプリの分離コード プロジェクト コード](https://raw.githubusercontent.com/MicrosoftLearning/APL-2002-develop-aspnet-core-consumes-api/master/Allfiles/Downloads/FruitWebApp-codebehind.zip)

1. **エクスプローラー**を起動し、ファイルが保存された場所に移動します。

1. ファイルを独自のフォルダーに解凍します。

1. Visual Studio Code を起動して、**[ファイル]** を選択してから、メニュー バーの **[フォルダーを開く]** を選びます。

1. プロジェクト ファイルを解凍した場所に移動し、*FruitWebApp-codebehind* フォルダーを選択します。

1. **[エクスプローラー]** ペインのプロジェクト構造は、次のスクリーンショットのようになります。 **[エクスプローラー]** ペインが表示されない場合は、メニュー バーで **[表示]** を選択し、**[エクスプローラー]** を選択します。

    ![Fruit Web アプリのプロジェクト構造を示すスクリーンショット。](media/02-web-app-cb-struture.png)

>**注:** この演習全体を通して編集されている各ファイルのコードについては、時間をかけて確認してください。 このコードには多数のコメントがあるため、コード ベースを理解するのに役立ちます。

## HTTP クライアントと `GET` 操作のコードを実装する

Fruit Web アプリは、ホーム ページに API サンプル データを表示します。 HTTP クライアントと `GET` 操作の両方を実装するコードを追加して、Web アプリを最初にビルドして実行するときにホーム ページに表示するデータを取得する必要があります。

### タスク 1: HTTP クライアントを実装する

1. **[エクスプローラー]** ペインの *Program.cs* ファイルを選択し、編集用に開きます。

1. `// Begin HTTP client code` と `// End of HTTP client code` のコメント間に次のコードを追加します。

    ```csharp
    // Add IHttpClientFactory to the container and set the name of the factory
    // to "FruitAPI". The base address for API requests is also set.
    builder.Services.AddHttpClient("FruitAPI", httpClient =>
    {
        httpClient.BaseAddress = new Uri("http://localhost:5050/fruitlist/");
    });
    ```

1. *Program.cs* に対する変更を保存します。

### タスク 2: GET 操作を実装する

1. **[エクスプローラー]** ペインで *Index.cshtml.cs* ファイルを選択して、編集用に開きます。

1. `// Begin GET operation code` と `// End GET operation code` のコメント間に次のコードを追加します。

    ```csharp
    // OnGet() is async since HTTP requests should be performed async
      public async Task OnGet()
      {
          // Create the HTTP client using the FruitAPI named factory
          var httpClient = _httpClientFactory.CreateClient("FruitAPI");

          // Perform the GET request and store the response. The empty parameter
          // in GetAsync doesn't modify the base address set in the client factory 
          using HttpResponseMessage response = await httpClient.GetAsync("");

          // If the request is successful deserialize the results into the data model
          if (response.IsSuccessStatusCode)
          {
              using var contentStream = await response.Content.ReadAsStreamAsync();
              FruitModels = await JsonSerializer.DeserializeAsync<IEnumerable<FruitModel>>(contentStream);
          }
      }
    ```

1. 変更を *Index.cshtml.cs* に保存します。

1. *Index.cshtml.cs* ファイルのコードを確認します。 依存関係の挿入を使用してページに追加される `IHttpClientFactory` の場所に注意してください。 また、データ モデルは `[BindProperty]` 属性を使用してページにバインドされることにも注意してください。

### タスク 9: Web アプリを実行する

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、次のスクリーンショットに示すように、Web アプリが実行され、API サンプル データが表示された状態でブラウザー ウィンドウが起動します。

    ![サンプル データを表示する Web アプリのスクリーンショット。](media/02-web-app-get-sample-data.png)

    >**注:** 演習の後半で、Web アプリの追加、編集、削除機能を有効にするコードを追加します。 

    >**注:** アプリを実行する際、以下のプロンプトを無視しても問題ありません。

    ![自己署名証明書のインストール時に表示されるプロンプトのスクリーンショット。](media/install-cert.png)

1. 演習を続行するには、ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** の順に選択するか、**Shift + F5** キーを押します。

## POST、PUT、DELETE 操作のコードを実装する

このセクションでは、Web アプリで **[リストに追加]**、**[編集]**、および **[削除]** 機能を有効にするコードをプロジェクトに追加します。 

### タスク 1: POST 操作を実装する

1. **[エクスプローラー]** ペインで *Add.cshtml.cs* ファイルを選択して、編集用に開きます。

1. `// Begin POST operation code` と `// End POST operation code` のコメント間の前に次のコードを追加します。

    ```csharp
    public async Task<IActionResult> OnPost()
    {
        // Serialize the information to be added to the database
        var jsonContent = new StringContent(JsonSerializer.Serialize(FruitModels),
            Encoding.UTF8,
            "application/json");
    
        // Create the HTTP client using the FruitAPI named factory
        var httpClient = _httpClientFactory.CreateClient("FruitAPI");
    
        // Execute the POST request and store the response. The parameters in PostAsync 
        // direct the POST to use the base address and passes the serialized data to the API
        using HttpResponseMessage response = await httpClient.PostAsync("", jsonContent);
    
        // Return to the home (Index) page and add a temporary success/failure 
        // message to the page.
        if (response.IsSuccessStatusCode)
        {
            TempData["success"] = "Data was added successfully.";
            return RedirectToPage("Index");
        }
        else
        {
            TempData["failure"] = "Operation was not successful";
            return RedirectToPage("Index");
        }
    }
    ```

1. *Add.cshtml.cs* に加えた変更を保存し、コード内のコメントを確認します。

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、Web アプリを実行した状態でブラウザー ウィンドウが起動します

1. **[リストに追加]** ボタンを選択し、生成されたフォームに入力します。 続いて、**[作成]** ボタンを選択します。

1. 追加した項目がリストの下部に表示されることを確認します。 問題が発生した場合は、ページの上部付近で成功/失敗メッセージの通知が表示されます。

1. 演習を続行するには、ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** の順に選択するか、**Shift + F5** キーを押します。

### タスク 1: PUT 操作を実装する

1. **[エクスプローラー]** ペインで *Edit.cshtml.cs* ファイルを選択して、編集用に開きます。

1. `// Begin PUT operation code` と `// End PUT operation code` のコメント間に次のコードを追加します。

    ```csharp
    public async Task<IActionResult> OnPost()
        {
            // Serialize the information to be edited in the database
            var jsonContent = new StringContent(JsonSerializer.Serialize(FruitModels),
                Encoding.UTF8,
                "application/json");
    
            // Create the HTTP client using the FruitAPI named factory
            var httpClient = _httpClientFactory.CreateClient("FruitAPI");
    
            // Execute the PUT request and store the response. The parameters in PutAsync 
            // appends the item Id to the base address and passes the serialized data to the API
            using HttpResponseMessage response = await httpClient.PutAsync(FruitModels.id.ToString(), jsonContent);
    
            // Return to the home (Index) page and add a temporary success/failure 
            // message to the page.
            if (response.IsSuccessStatusCode)
            {
                TempData["success"] = "Data was edited successfully.";
                return RedirectToPage("Index");
            }
            else
            {
                TempData["failure"] = "Operation was not successful";
                return RedirectToPage("Index");
            }
    
        }
    ```

1. *Edit.cshtml.cs* に加えた変更を保存し、コード内のコメントを確認します。

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、Web アプリを実行した状態でブラウザー ウィンドウが起動します

1. 編集するリスト内の項目を選択し、**[編集]** ボタンを選択します。 
1. **[フルーツ名]** と **[使用できる項目]** フィールドを編集して、**[更新]** を選択します。

1. 変更がリストに表示されることを確認します。 問題が発生した場合は、ページの上部付近で成功/失敗メッセージの通知が表示されます。

1. 演習を続行するには、ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** の順に選択するか、**Shift + F5** キーを押します。

### タスク 1: DELETE 操作を実装する

1. *[エクスプローラー]* ペインで **Delete.cshtml.cs** ファイルを選択して、編集用に開きます。

1. `// Begin DELETE operation code` と `// End DELETE operation code` のコメント間に次のコードを追加します。

    ```csharp
    public async Task<IActionResult> OnPost()
    {
        // Create the HTTP client using the FruitAPI named factory
        var httpClient = _httpClientFactory.CreateClient("FruitAPI");
    
        // Appends the data Id for deletion to the base address and performs the operation
        using HttpResponseMessage response = await httpClient.DeleteAsync(FruitModels.id.ToString());
    
        // Return to the home (Index) page and add a temporary success/failure 
        // message to the page.
        if (response.IsSuccessStatusCode)
        {
            TempData["success"] = "Data was deleted successfully.";
            return RedirectToPage("Index");
        }
        else
        {
            TempData["failure"] = "Operation was not successful";
            return RedirectToPage("Index");
        }
    
    }
    ```

1. *Delete.cshtml.cs* に加えた変更を保存し、コード内のコメントを確認します。

1. Visual Studio Code の上部のメニューから **[実行] \| [デバッグの開始]** の順に選択するか、**F5** キーを押します。 プロジェクトのビルドが完了すると、Web アプリを実行した状態でブラウザー ウィンドウが起動します

1. 削除する項目をリストから選択し、**[削除]** ボタンを選択します。 

1. **[フルーツ名]** と **[使用できる項目]** フィールドを編集して、**[更新]** を選択します。

1. リストにこの項目が表示されていないことを確認します。 問題が発生した場合は、ページの上部付近で成功/失敗メッセージの通知が表示されます。

演習を終了する準備ができたら、次の手順を実行します。

* ブラウザーまたはブラウザー タブを閉じ、Visual Studio Code で **[実行] \| [デバッグの停止]** を選択するか、**Shift + F5** キーを押します。 

* 実行中のターミナルで **Ctrl + C** キーを押して Fruit API を停止します。

## 確認

この演習では、以下の方法を学習しました。

* HTTP クライアントとして `IHttpClientFactory` を実装する
* ASP.NET Core Razor Pages コードの分離ファイに HTTP 操作を実装する
