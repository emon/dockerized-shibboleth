# build IdP image

```
docker-compose build idp
docker-compose up -d 
docker run -it -u 0 --rm -v $(pwd)/volumes/idp/conf:/mnt shibboleth_idp:latest bash -c "cd /opt/shibboleth-idp/conf.original; cp -a * /mnt"
vi volumes/idp/conf/*.conf
docker-compose restart
```

コンテナ内の/opt/shibboleth-idp/conf.original/* を、ホスト環境の /volumes/idp/conf/ 配下にコピーする。
設定ファイルに対する修正は /volumes/idp/conf/ 配下で行う。


