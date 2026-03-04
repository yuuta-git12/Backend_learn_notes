#docker

Docker Composeは .env                                                                    
  ファイルの変数を自動的に読み込む機能があるため、${変数名:-デフォルト値}
  の組み合わせがよく使われます。                                                           
  例えば Laravel Sail（Laravel公式のDocker環境）の docker-compose.yml     
  でもまさにこの書き方が多用されています:                                                  
  # Laravel Sail の例
  ```
  services:
    laravel.test:
      ports:
        - '${APP_PORT:-80}:80'
    mysql:
      ports:
        - '${FORWARD_DB_PORT:-3306}:3306'
    mailpit:
      ports:
        - '${FORWARD_MAILPIT_PORT:-1025}:1025'
        - '${FORWARD_MAILPIT_DASHBOARD_PORT:-8025}:8025'

  ```
  
  実際、今回のプロジェクトの mailpit
  の記述はSailの設定をそのまま持ってきたものだと思われます。
  メリットは、チームメンバーごとにポートを変えたい場合に、docker-compose.yml を編集せず
  .env だけ変更すれば済む点です。例えば別のプロジェクトで8025番を使っていたら:
  # .env
  FORWARD_MAILPIT_DASHBOARD_PORT=8026

  と書くだけでポートを変更できます。