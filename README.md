1. Criar projeto web:
dotnet new web -o [PROJECT]

2. Abrir pasta do projeto no Visual Studio Code

3. Criar pasta Hubs.

4. inicializar npm:
```
npm init
```

5. Alterar "package.json":

5.1. Incluir dev dependencies
```javascript
"devDependencies": {
  "clean-webpack-plugin": "1.0.1",
  "css-loader": "2.1.0",
  "html-webpack-plugin": "4.0.0-beta.5",
  "mini-css-extract-plugin": "0.5.0",
  "ts-loader": "5.3.3",
  "typescript": "3.3.3",
  "webpack": "4.29.3",
  "webpack-cli": "3.2.3"
}
```

5.2. Incluir dependencies:
```javascript
"dependencies": {
  "@aspnet/signalr": "^1.1.4"
}
```

6. Instalar npm:
```
npm install
```

7. Criar arquivo "Hubs/ChatHub.cs"
```csharp
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace [DEFAULT NAMESPACE].Hubs
{
    public class ChatHub : Hub
    {
        public async Task NewMessage(long username, string message)
        {
            await Clients.All.SendAsync("messageReceived", username, message);
        }
    }
}
```

8. Alterar Program.cs para incluir o serviço SignalR:
```csharp
using SignalRWebPack.Hubs;
...
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR();
}
```

9. Incluir recurso para acessar arquivos estáticos e definir arquivos padrão:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...
    app.UseRouting();
    app.UseDefaultFiles();
    app.UseStaticFiles();

    ...
}
```

10. Substituir Endpoint para mapear para o ChatHub:
```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHub<ChatHub>("/hub");
});
```

11. Criar nova pasta "src"

12. Criar arquivo "src/index.html":
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>ASP.NET Core SignalR</title>
</head>
<body>
    <div id="divMessages" class="messages">
    </div>
    <div class="input-zone">
        <label id="lblMessage" for="tbMessage">Message:</label>
        <input id="tbMessage" class="input-zone-input" type="text" />
        <button id="btnSend">Send</button>
    </div>
</body>
</html>
```

13. Criar arquivo "src/index.ts":
```typescript
import "./css/main.css";
import * as signalR from "@aspnet/signalr";

const divMessages: HTMLDivElement = document.querySelector("#divMessages");
const tbMessage: HTMLInputElement = document.querySelector("#tbMessage");
const btnSend: HTMLButtonElement = document.querySelector("#btnSend");
const username = new Date().getTime();

const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hub")
    .build();

connection.on("messageReceived", (username: string, message: string) => {
    let messageContainer = document.createElement("div");

    messageContainer.innerHTML =
        `<div class="message-author">${username}</div><div>${message}</div>`;

    divMessages.appendChild(messageContainer);
    divMessages.scrollTop = divMessages.scrollHeight;
});

connection.start().catch(err => document.write(err));

tbMessage.addEventListener("keyup", (e: KeyboardEvent) => {
    if (e.keyCode === 13) {
        send();
    }
});

btnSend.addEventListener("click", send);

function send() {
    connection.send("newMessage", username, tbMessage.value)
              .then(() => tbMessage.value = "");
}
```

14. Criar pasta "src\css".

15. Criar arquivo "src\css\main.css":
```css
*, *::before, *::after {
    box-sizing: border-box;
}

html, body {
    margin: 0;
    padding: 0;
}

.input-zone {
    align-items: center;
    display: flex;
    flex-direction: row;
    margin: 10px;
}

.input-zone-input {
    flex: 1;
    margin-right: 10px;
}

.message-author {
    font-weight: bold;
}

.messages {
    border: 1px solid #000;
    margin: 10px;
    max-height: 300px;
    min-height: 300px;
    overflow-y: auto;
    padding: 5px;
}
```

16. Criar o arquivo ".\src\tsconfig.json":
```javascript
{
  "compilerOptions": {
    "target": "es5"
  }
}
```

17. Criar arquivo .\webpack.config.js:
```javascript
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const CleanWebpackPlugin = require("clean-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
    entry: "./src/index.ts",
    output: {
        path: path.resolve(__dirname, "wwwroot"),
        filename: "[name].[chunkhash].js",
        publicPath: "/"
    },
    resolve: {
        extensions: [".js", ".ts"]
    },
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: "ts-loader"
            },
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, "css-loader"]
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(["wwwroot/*"]),
        new HtmlWebpackPlugin({
            template: "./src/index.html"
        }),
        new MiniCssExtractPlugin({
            filename: "css/[name].[chunkhash].css"
        })
    ]
};
```

18. Incluir scripts no "package.json":
```javascript
"scripts": {
  "build": "webpack --mode=development --watch",
  "release": "webpack --mode=production",
  "publish": "npm run release && dotnet publish -c Release"
}
```

19. Executar o script publish:
```
npm run publish
```

20. Executar o dotnet:
```
dotnet run
```
