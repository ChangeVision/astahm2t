# memo
　- ステートマシンの構造を変更．TransitionはEventを受けて遷移するように．
 - 出来れば汎用的なUtilとプロジェクトローカルなUtilを分けたい．
 - テンプレートエンジン周りを整理
 - Generateをボタン実装
 - [done] Setting ManagerでGeneratorSettingのXML保存，復帰部を

# astah m2t(Model to Text)

## 概要
Astahで汎用的なコード生成を行うためのプラグインです．
AstahプラグインからGroovyスクリプトを呼び出す形でコード生成を制御します．
また，コードテンプレートはGroovyのGStringTemplateEngineを利用しています．

## 使用方法
### プラグインの追加
 - ヘルプ＞プラグイン一覧
  + ＋インストール
  + →プラグインファイルを選択 (astahm2t/target/～.jar）
  + →ついでに，Astah SDKフォルダの/astah-plugin-SDK-1.2/bundles/console-1.0.1.jarも

Astah再起動

### 設定ファイルの追加

初回の起動時に下記フォルダが生成される．

 - Windows
　C:/Users/{username}/.astah/plugins/m2t/
 - Mac
　UserHome?/.astah/plugins/m2t/

上記のフォルダに，
`astahm2t/groovy/*`
のファイルをすべてコピー

設定ファイル(m2t.properties)のパスを自身の環境のものに変更
`5 : generator_ArduinoCreate=C:\\Users\\hosoai`

　# 今後，自動的にサンプルのファイル一緒に作成するようにします・・．

Astah再起動

### 生成テスト
サンプルのUMLを開く（led-sample.asta等）
AstahM2T Generator Viewで，
Generator Type : Arduino Create
Project : 適当に
Target Dir : 適当に
と設定し，Generate
現状何も反応がないのですが，，ターゲットディレクトリにファイルが生成されてるはず



## 生成の流れ
プラグインを実行すると，プラグインからModelCollector, FileGeneratorのGroovyコードがコールされます．
ModelCollectorはAstah APIで現在のプロジェクトからコード生成に用いるモデルを集め，
FileGeneratorは集めたモデルに応じて，テンプレートを選択しファイルを生成します．
Groovyコードには，bindingを用いてオブジェクトを渡します．
ModelCollectorには，ProjectAccessor(Binding名：projectaccessor)が渡されています．
また，集めたモデルは同じくbinding.putでFileGeneratorに渡します．

両者を分けている理由として，再利用性を高める意図があります．
例えば，クラスとステートマシンからJavaコードを生成する場合と，同じくクラスとステートマシンからCコードを生成する場合では，
ModelCollectorは同様のものが利用出来ます．
一方FileGeneratorでは，各言語に応じてテンプレートの切り替えや，モデルの参照方法が異なるため，言語ごとのものを
作成する必要があります．

テンプレートは，Groovyのテンプレートエンジンを利用しています．
テンプレート単体では，少々機能的に心許ないため，モデル操作のためのUtilクラスも一緒に利用出来るようにしています．
例えばステートマシンで初期状態のモデルを参照したい場合など，テンプレート内に処理を書いてしまうとテンプレートが複雑になってしまう
ため，初期状態を得るためのメソッドをUtilクラスに追加し，利用するようにします．

### 参照ファイル
コード生成用の設定ファイルやスクリプト，テンプレートはデフォルトでは，ユーザディレクトリ/.astah下を指定しています．

- ユーザディレクトリ/.astah/plugins/
- ユーザディレクトリ/.astah/plugins/m2t/
 - ModelCollector.groovy : astahから必要なモデルを集めるスクリプト
 - FileGenerator.groovy ： modelに応じたテンプレートを用いてファイルを生成するスクリプト
 - ClassUtils.groovy : クラス図の要素を扱う際に便利なUtils
 - StateUtils.groovy : ステートマシン図 〃
- ユーザディレクトリ/.astah/plugins/m2t/template
 - cpp.template : C言語のソースファイル用テンプレート
 - header.template : C言語のヘッダファイル用テンプレート
 - arduino.template : Arduinoソースファイル用テンプレート