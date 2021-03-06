# Engine Extensionのセットアップ
Engine ExtensionはEngine Libraryの機能を拡張するための機構です。  
詳細は[Personium Engine](../app-developer/Personium-Engine.md)をご覧ください。  

ここでは、Personium Open Sourceで公開しているEngine Extensionのセットアップ方法を記載します。  

## リポジトリ  
* [personium-ex-httpclient](https://github.com/personium/personium-ex-httpclient)
* [personium-ex-mailsender](https://github.com/personium/personium-ex-mailsender)
* [personium-ex-slack-messenger](https://github.com/personium/personium-ex-slack-messenger)
* [personium-ex-ew-services](https://github.com/personium/personium-ex-ew-services)

## 事前確認  
同じExtensionのバージョンが複数ある場合、Personiumは正常に起動しない可能性があります。Extensionをインストールする前に、下記を実施してください。  

### 手順  
1. 同じExtensionの古いバージョンが既に存在するかどうかを確認します。  
```# ls -l /personium/personium-engine/extensions  ```  
1. **同じExtensionの古いバージョンが存在しない場合、事前確認手順はここまでです。**  
そうでない場合は、Extensionファイルを安全に削除するために次の手順を実行してください。  
1. Tomcat処理を停止します。  
```# systemctl stop tomcat```  
1. Extensionファイルを削除します。  
```# cd /personium/personium-engine/extensions```  
```# rm personium-ex-${COMPONENT}-${VERSION}-libs.jar```   

## ビルド済Extensionのインストール  
JARファイルをAPサーバに配備し、再起動してください。  
デフォルトの配備先Pathは"/personium/personium-engine/extensions"です。  
```
# cd /personium/personium-engine/extensions
# curl -O https://personium.io/mvnrepo/io/personium/personium-ex-${COMPONENT}/${VERSION}/personium-ex-${COMPONENT}-${VERSION}-libs.jar
# systemctl restart tomcat
```

## 最新版Extensionのビルドとインストール  
最新のEngine ExtensionをbuildしてAPサーバーに配備したい場合は下記を実施してください。

### 前提条件 (Maven)  
Engine ExtensionのビルドにはMavenが必要です。  
http://maven.apache.org/install.html を参考にインストールを行ってください。  

### 手順  
対象のリポジトリ(personium-ex-xxxxx)をチェックアウトし、以下を行ってください。  
"personium-ex-xxxxx/target"配下にJARファイルが作成されます。  
```
# cd personium-ex-xxxxx
# mvn clean package -DskipTests
# cp personium-ex-xxxxx/target/personium-ex-${COMPONENT}-${VERSION}-libs.jar /personium/personium-engine/extensions
# systemctl restart tomcat
```
