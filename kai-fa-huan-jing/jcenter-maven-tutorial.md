# Java - 發佈maven專案到JCenter

## Step1. 在Bintray portal上新建空的repo和package

repo name我設成`wisepaas-datahub`
packagename根據JCenter上慣例, 規則為groupId:artifactId, 所以我的pkg name為`wisepaas.datahub:cloud-sdk`

## Stelp2. 根據Portal auto-gen提示去修改project pom.xml和maven setting.xml
在portal建完pkg後, 會順便產生設定讓你複製貼上, 點擊 `Set me up`
![](/assets/bintrayhint.PNG)
接著再點擊`Uploading` > `Deploying with Maven`
![](/assets/bintrayhint2.PNG)
可以看到有兩段cfg,上面那一段是要放在全域maven setting.xml的, 下面那段是要放在專案pom.xml的

pom.xml
```
...
	<distributionManagement>
		<repository>
			<id>bintray-advwacloud-wisepaas-datahub</id>
			<name>advwacloud-wisepaas-datahub</name>
			<url>https://api.bintray.com/maven/advwacloud/wisepaas-datahub/wisepaas.datahub:cloud-sdk/;publish=1</url>
		</repository>
	</distributionManagement>
</project>
```
接著要改setting.xml, 不知道它放哪, 可以下mvn -X, 找到下面兩行, 任選一個來改就行
```
[DEBUG] Reading global settings from xxxx\settings.xml
[DEBUG] Reading user settings from xxxx\settings.xml
```