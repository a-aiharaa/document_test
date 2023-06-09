# Dockerの流れ
イメージを`build`して
`run`する

# Dockerイメージを作成する
DockerイメージはDockerfileから`docker build`コマンドで作成できる

`-t`オプションでイメージ名を変えられる

```sh
docker build ./ -t example
```
このコマンドを行うと、実行ディレクトリと同じ場所にあるDockerfileを使って`example`という名前でDockerイメージを作成できる

# Dockerイメージを実行する
作成したイメージ名で下記コマンドを実行するとコンテナが作成される
```sh
docker run example
```

# Dockerfileの作成
下記のような内容で拡張子のない`Dockerfile`という名前でファイルを作成するとそれがDockerイメージを作るDockerfileとなる

```dockerfile
FROM node:20

CMD ["echo", "hello world"]
```
上記のみのコマンドのDockerfileで作成されたDockerイメージは起動したらコンソールに`hello world`と表示されてコンテナが終了する

終了させないためには下記の形でDockerfileを作成し、`-it`オプションをつけた次のコマンドを入力する

```sh
docker run -it example
```

```dockerfile
FROM node:20

CMD ["/bin/bash"]
```

なぜコンテナが終了しないかというと
`CMD ["/bin/bash"]`この部分がそれ単体で終了するコマンドではなく、エラーも起きなかったから

コンテナの終了条件はCMDやENTRYPOINTで記載した処理が終わったら、である

そして、`CMD ["/bin/bash"]`は`docker run`コマンドのみだと必要なものが見つからずに終了してしまう

この必要なものは`terminal`となっていて、何もオプションを渡さないと`terminal`がコンテナに渡されない

`-it`オプションは`-i`と`-t`オプションに別れる`-i`は標準入力を渡し、`-t`は`terminal`を渡す

なので、上記コマンドとDockerfileでコンテナを実行すると終了しない


# DockerfileのARGについて
Dockerfileをビルドする際に引数を渡すことができる
```sh
docker build ./ --build-arg hoge=hello
```
渡した引数はDockerfile内の一部で使える
```dockerfile
ARG hoge
RUN echo ${hoge}
USER ${hoge}
```

# DockerとWSL
Docker内にサーバーを立てる場合、ソースなどをどこかに入れてDockerで実行するということがあるが、そのソースをWindowsなどローカルに直接置くとファイルシステムの違いで動作がかなり遅くなる
> `yarn install`など

なので、UbuntuなどのWSLのディストリビューションを使用してその中にソースを入れて、そこから実行するとよい

ただし、そのまま実行するとユーザー権限の問題が発生する

Dockerが作成したファイルはUser指定をしないと`root`ユーザーで作成されるため、権限がおかしなことになりファイルにアクセスできずにエラーが発生したりする

なので、コンテナを作成するときにユーザー指定をローカルユーザーと同じにする必要がある

WSLでの操作ユーザーは下記コマンドで確認できる
```sh
id
```

そして、Dockerでのユーザー指定は`1000:1000`のような形でDockerfileで行う

```dockerfile
USER 1000:1000
```

ただし、このままだとローカルユーザーの変更があった時に対応が面倒なので、対応できるようにする

```dockerfile
ARG UID_GID
USER ${UID_GID}
```

こうすることでDockerイメージを作成する際に引数を渡せばユーザーを指定して作成することができる

渡し方は下記
```sh
docker build ./ -t example --build-arg UID_GID=$(id -u):$(id -g)
```
