```uml
@startuml
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response
 
Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
@enduml
```
```uml
@startuml{news-after-login.png}
'画像変換するときにここで指定したファイル名になる。
 
title ニュース一覧取得API（ログイン後）
 
hide footbox
'シーケンスの縦線の下部にシーケンスボックスを出すかどうか指定する。よほど長いものでない限りいらないだろう
 
actor Webアプリ as user
'actorにすると人っぽい見かけになる
 
participant API as api
'participantにすると四角になる
 
database Redis as redis
'databaseにするとストレージっぽい見かけになる
 
database DynamoDB as dynamo
'as の左部が表示名、右部が変数名
 
participant NewsAPI as news
 
user -> api : ニュース一覧取得リクエスト
' -> 同期メッセージという意味になる。
 
activate api
' オブジェクト生成的な意味合いになる。
 
api -> redis : AccessToken
note right : db:9, ttl:60s
redis --> api : userId
api -> dynamo : userId
note right : table:metadata
'矢印の右側にコメントボックスを表示してそこにコメントを表示する、という意味
 
dynamo --> api : ユーザープロフィール
 
group 公開ニュースの取得
'区切りがわかりにくくなってしまう場合はgroup - endで囲うとよい
 
api -> redis  : 公開ニュース取得
note right : db:11,ttl:60s
redis --> api : 公開ニュースのJSON
 
alt キャッシュなし
'分岐を表現したいときは分岐ケースをalt - endで囲う
 
api -> news : 公開ニュース取得
news --> api : 公開ニュース
api -> redis : 公開ニュースキャッシュ
end
end
 
group ユーザニュースの取得
api -> redis  : ユーザニュース取得
note right : db:11,ttl:60s
redis --> api : ユーザニュースのJSON
alt キャッシュなし
api -> news : ユーザニュース取得
news --> api : ユーザニュース
api -> redis : ユーザニュースキャッシュ
end
end
api -> api : プロフィールから配信ニュース抽出、\n公開開始日時でソート
api --> user : ニュース一覧
@enduml
