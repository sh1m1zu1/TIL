# GitHubにおけるPR分析 演習メモ

---

## 0. 演習目標とGitHubについて
Gitとはソフトウェアの変更履歴を記録し、過去のバージョンに戻したり、複数人での作業による変更衝突を防ぐためのシステムを指す。  
そのGitをベースにクラウド上で提供されているサービスが **GitHub** である。  

GitHubを利用することで、インターネット経由でどこからでもプロジェクトにアクセスでき、共同開発に役立つ機能（Gitの基本機能に加えてコードレビュー、SNS的機能、使いやすいUIなど）を利用できる。  

さらに、GitHub上のデータを収集・分析してソフトウェア開発に役立つ知見を得る研究は **Mining Software Repository (MSR)** と呼ばれる。  

MSRの主な目的例：
- PR（Pull Request）のレビュー時間の傾向を調べる  
- IssueとPRの関連を分析する  
- 貢献者の活動パターンを可視化する  

本演習の目標：
- Mining Software Repositoryのプロセス理解  
- マイニング（データ収集）ツールの利用  

---

## 1. 環境設定
- **WSL2** 上で行うことを前提とする  
  参考: https://qiita.com/zakoken/items/61141df6aeae9e3f8e36  
- 演習用ディレクトリを作成し、作業はすべてその下で実施  
- `venv`（Pythonの仮想環境）を作成し、`PyGithub` をインストール  

```bash
$ sudo apt update
$ sudo apt install python3.10-venv
$ python3 -m venv venv
$ ls
venv
$ source venv/bin/activate
(venv)$ pip install PyGithub

---

## 2. GithubAPIに接続しよう
Github APIとは、githubで記録されているデータにアクセスしたり、gitの機能をプログラムで操作することが出来る橋渡し的な機能(インターフェース)を果たす。

- create_instance.pyを作成・実行(コードに自分のトークンを記述する必要あり)

ここで生成するgを通して、ユーザー情報の取得(g.get_user())やリポジトリの取得(g.get_repo())などを行うことが出来る。
要するに、gがpythonを用いてGitHubと会話する窓口となる。
※トークンに関するサイト(https://qiita.com/RyutoYoda/items/c6908025c0aef4b35cdd)


## 3. 気になるリポジトリを取得しよう
リポジトリとはソースコードやドキュメントなどのファイルを保存・管理するための場所を指す。
githubでは様々なユーザがいて、それぞれのユーザは複数のリポジトリを保有している。

- get_repos.py を作成・実行

ここで定義したrepoは今後指定したリポジトリ上のpullの情報やissue、commitなどの情報を呼び出す際に使える。


## 4. PR(Pull Request)を取得しよう
PR(Pull Request)とはGitリポジトリにおける変更を他の開発者に通知し、コードレビューを依頼するための機能を指す。
具体的には、開発者がローカルブランチで行った変更を、他の開発者がいる共有リポジトリのブランチにマージするよう提案する手段を指す。
利用例として、「機能追加」「バグ修正」「コード改善」などが挙げられる。﻿

- get_prs.pyを作成・実行

今後はこのprsを用いて各PRにに含まれるコミットや変更行数、変更ファイル数、コミットメッセージ長などの情報(今後メトリクスと呼ぶ)を取得することが出来る。
また、PRの取得数は任意だが、全てを取得すると今後行うメトリクスの情報の取得の際に膨大な時間が取られてしまう一方、数が少なすぎてもよい結果が得られないので適切な数値設定が求められる。
※400コだと15分くらいかかった(4のコードで取得したメトリクスの場合)


## 5. 取得したPRから得られる必要なメトリクスをCSV形式でファイルに出力しよう
得られたPRに含まれるメトリクスを用いて分析を行うためにはデータを効率的に解析できるように整える必要がある。
なので今回はCSV形式のファイルに出力することでデータで分析しやすい形にする。

- create_prs_data.pyを作成・実行

プログラム中で呼び込まれているpandasとはPythonでデータ操作、解析を効率的に行うためのライブラリを指す。
このプログラムを実行することで、同じディレクトリ内にメトリクスが記録されたpr_data.csvが生成される。

出力例
34925,False,Make `ProblemDetailJacksonXmlMixin` compatible with Jackson 3,1,2,14,9,23
34921,False,Trung | Added StringUtils::resizeConsecutiveChars() and related Unit Tests,1,2,93,0,93
34917,False,Set the access modifier of the MultipartFileResource class to public,1,1,1,1,2
34914,True,Polish HttpRequestValues,1,1,6,3,9
34900,True,Upgrade to Kotlin 2.1.21,1,2,3,3,6


## 6. 生成したメトリクスから分析しよう
csvで分析したメトリクス
