# 開發環境

---

* 程式語言:

  * Node.js
    * version: 8.10.0

* IDE: [Visual Studio Code](https://code.visualstudio.com/)

  * Pulgin

    * [StandardJS - JavaScript Standard Style](https://marketplace.visualstudio.com/items?itemName=chenxsan.vscode-standardjs)

      * 使用Semi-Standard Style, 可同時驗證.Vue檔案可參考以下設定, 或參考[說明](https://wdd.js.org/vscode-vue-standardjs.html)
        * 安裝相關套件

        ```
        npm i -g semistandard
        npm i -g eslint-plugin-html@3.2.2
        npm i -g eslint
        ```

        * 修改VSCode的Workspace setting.json

        ```
        {
          "vetur.validation.template": false,
          "semistandard.autoFixOnSave": true,
          "javascript.validate.enable": false,
          "standard.semistandard": true,
          "standard.validate": [
            "javascript",
            "javascriptreact",
            {
              "language": "vue",
              "autoFix":true
            }
          ],
          "standard.options": {
            "plugin":["html"]
          },
          "standard.autoFixOnSave": true,
          "files.associations": {
            "*.vue":"vue"
          },  
        }

        ```

    * [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur)

    * [Hyper JavaScript Snippets](https://marketplace.visualstudio.com/items?itemName=t7yang.hyper-javascript-snippets)

    * [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)

* 開發相關工具

  * 程式碼版本管理工具: [Git](https://gitforwindows.org/)
    * GUI Tool
      * [tortoisegit](https://tortoisegit.org/)
      * [SourceTree](https://www.sourcetreeapp.com/)
  * Node.js版本管理工具: [nvm](https://github.com/coreybutler/nvm-windows)
    * [教學](https://oranwind.org/nvm-windows/)
  * command-line工具: [cmder](http://cmder.net/)
    * [教學](https://blog.miniasp.com/post/2015/09/27/Useful-tool-Cmder.aspx)
  * Linux遠端連線工具: [MobaXterm](https://mobaxterm.mobatek.net/)
  * MongoDB GUI 工具: [Robo mongo \(Robo 3T\)](https://robomongo.org/download)
  * PostgreSQL GUI 工具: [PgAdmin 4](https://www.pgadmin.org/download/)
  * Cloud Foundry command-line工具: [cf-cli](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)



