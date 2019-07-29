# 開發環境 - Node.js

---

* Version: 10.16.0
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
    * Follow [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)

      * Master 主要是用來放穩定、隨時可上線的版本。這個分支的來源只能從別的分支合併過來，開發者不會直接 Commit 到這個分支。因為是穩定版本，所以會在這個分支上的 Commit 上打上版本號標籤。
      * Develop 這個分支主要是所有開發的基礎分支，當要新增功能的時候，所有的 Feature 分支都是從這個分支切出去的。而 Feature 分支的功能完成後，也都會合併回來這個分支。

      * 當開發新需求時，就是使用 Feature \(名稱可自定或是定為需求名稱\)分支。Feature 分支都是從 Develop 分支來的，完成之後會再併回 Develop 分支，合併後即可將該分支移除。

  * Node.js版本管理工具: [nvm](https://github.com/coreybutler/nvm-windows)

    * [教學](https://oranwind.org/nvm-windows/)

  * command-line工具: [cmder](http://cmder.net/)

    * [教學](https://blog.miniasp.com/post/2015/09/27/Useful-tool-Cmder.aspx)

  * Linux遠端連線工具: [MobaXterm](https://mobaxterm.mobatek.net/)
  * MongoDB GUI 工具: [Robo mongo \(Robo 3T\)](https://robomongo.org/download)
    * Local測試環境
      * host: PC031206
      * port: 27017
      * username: wisepaas
      * password: wisepaas
      * database: WISE-PaaS
  * PostgreSQL GUI 工具: [PgAdmin 4](https://www.pgadmin.org/download/)
    * Local測試環境
      * host: PC031206
      * port: 5432
      * username: postgres
      * password: admin
      * database: wisepaas
  * Cloud Foundry command-line工具: [cf-cli](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)



