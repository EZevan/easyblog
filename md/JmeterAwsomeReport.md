# <center>Jmeter Awsome Report 工具快速使用

基于（Python + Pandas + Beautifulsoup4） 开发的一款通过解析 `Jmeter`的*.jtl或*.csv文件创建精美的HTML（仪表板视图）报告的轻量化工具

---

 - __示例报告__： ![链接](../assets/img/jmeter%20awsome%20report/jmeter-awsome-report.png)
---

#### 如何工作:

1. 通过`pandas`读取并解析 *.jtl or *.csv 文件 (using pandas)

2. 获取 'label', 'success', 'elapsed', 'failureMessage', 'responseCode', 'threadName' 等指标的值

3. 基于Beautifulsoup4将以上数据转换为HTML报告

---

#### 快速开始:

> 安装 Python 3.8+

1. 安装 jmeter-awsome-report

   ```
    # install
      pip install -i http://testhub.taimei.com/pypi/simple jmeter-awsome-report --trusted-host testhub.taimei.com
      
      # upgrade
      pip install -i http://testhub.taimei.com jmeter-awsome-report  --upgrade
      # or
      pip install -i http://testhub.taimei.com jmeter-awsome-report  -U
   ```

2. 执行 `jmereport` 命令生成 html 报告

    > 方式1: 默认情况jmeter命令行运行完成后生成的结果文件名称为：result.jtl，直接运行命令：`jmereport`
    ```
    jmereport
    ```

    > 方式2: 假如`result.jtl`文件在`Result`目录下
    ```
    jmereport --inputpath ./Result/ --output result1.jtl
    # 或
    jmereport -i ./Result/ -o result1.jtl
    ```

    > 查看帮助:

    ```
    jmereport --help
    ```

3. 默认会在当前目录生成 __jmeter-awsome-report.html__ 文件

    > Note: jmereport support *.csv file as well

---

### 自定义报告名称

 - 命令行自定义报告文件名称，如：api_test.html
    ```
    jmereport -n api_test.html
    ```
---

#### 报告中不包含`Table Results`

 - 可以在 html 报告中排除 `table results` 通过使用`--ignoretableresult` 或 `-I` 参数

    ```
    jmereport -I True
    ```

    > `--ignoretableresult` 默认值为 `False`

---

#### 解析多个`csv/jtl`结果文件

 - Jmeter awsome report 支持同时解析多个 `csv/jtl` 文件

    ```
    jmereport -o "result.jtl,result1.csv,result2.jtl,result3.csv"
    ```
---

#### CSV/JTL文件中使用自定义分隔符 (delimiter)

 -  Jmeter awsome report 支持使用自定义分隔符解析 `csv/jtl` 文件 。（如: | 或 _)

    ```
    jmereport -s "|"
    ```
---

#### 自定义报告

Jmeter awsome report中使用自定义 logo:

 - __ 自定义 logo__ :  通过使用 `--logo` 参数自定义 logo

     ```
     --logo "https://custom/logo.png"
     ```
 - 默认使用：![默认logo](../assets/img/jmeter%20awsome%20report/testhub.png)
---

#### 生成jmeter-awsome-report报告

在运行玩 jmeter 脚本后生成 Jmeter awsome report 报告:

 - 使用如下代码（jmereport 后带上需要的参数）创建 .bat (or) .sh 文件

    ```
    jmeter -n -t test.jmx -l result.jtl &&
    jmereport [:options]
    ```

    > && 符号在命令行中表示拼接多个命令，上面的命令表示：先执行 jmeter 脚本，然后执行 jmereport 命令

  - 按照需求修改 jmereport 命令后运行 .bat或.sh 文件

  - 运行完成后即可生成 jmeter-awsome-report 报告

---

如果你对`Jmeter Awsome Report`有任何问题/建议/意见，请随时与我联系

 - 邮箱: <a href="mailto:wenqiang.zhang@taimei.com?Subject=Jmeter%20Awsome%20Report" target="_blank">`wenqiang.zhang@taimei.com`</a> 
 - git仓库: <a href="http://git.taimei.com/testing/domore/jmeter-awsome-report" target="_blank">`jmeter-awsome-report`</a>

---

:star: 喜欢的话就点个赞吧
