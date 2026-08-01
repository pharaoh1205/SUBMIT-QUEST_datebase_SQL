# ステップ２【手順書】MySQL環境構築（Mac / Homebrew）、データベースとテーブルの作成、データ挿入

### 概要

本ドキュメントは、Mac端末にてHomebrewを用いてMySQL環境を構築し、インターネットTVサービス（ABEMA風）のデータベース・テーブル作成およびサンプルデータの投入までを再現性高く実施するための手順書です。

### 1. Homebrewの動作確認・アップデート

ターミナルを開き、以下のコマンドを実行してHomebrewが正しく動作するか確認します。

```bash
brew -v
```

出力例:Homebrew 4.x.x

のようにバージョンが表示されれば問題ありません。

※ コマンドが見つからない場合は、事前にHomebrewの公式サイトを参考にインストールを行ってください。

最新のパッケージ情報を取得するため、Homebrewをアップデートします。

```bash
brew update
```

### 2. MySQLのインストール

a. Homebrewを使用してMySQLをインストールします。

```bash
brew install mysql
```

b. インストール完了後、以下のコマンドでバージョンを確認し、正常にインストールされたことを確認します。

```bash
mysql --version
```

### 3. MySQLサービスの起動と初期設定

a. MySQLサービスの起動バックグラウンドでMySQLサーバーを起動します。

```bash
brew services start mysql
```

b. 初期セキュリティ設定（任意・推奨）rootユーザーのパスワード設定などを行う場合は、以下のコマンドを実行し、対話形式で設定を進めます。

```bash
mysql_secure_installation
```

### 4. MySQLへのログイン

管理者（root）ユーザーでMySQLに接続します。

```bash
mysql -u root -p
```

- ※ パスワードを設定していない場合は `mysql -u root` のみでログイン可能です。
- 成功すると、プロンプトが `mysql>` に変わります。

### 5. データベースの作成

学習用・本課題用のデータベースを作成し、使用を宣言します。

a. データベースの作成

```sql
CREATE DATABASE abema_tv CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

b. 作成されたことの確認

```sql
SHOW DATABASES;
```

c. 使用するデータベースの切り替え

```sql
USE abema_tv;
```

### 6. テーブルの作成

インターネットTVサービスの仕様に基づき、計7つのテーブルを構築します。

```sql
-- 1. チャンネルテーブル
CREATE TABLE channels (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);

-- 2. ジャンルテーブル
CREATE TABLE genres (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE
);

-- 3. 番組テーブル
CREATE TABLE programs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT
);

-- 4. 番組ジャンル中間テーブル
CREATE TABLE program_genres (
    id INT AUTO_INCREMENT PRIMARY KEY,
    program_id INT NOT NULL,
    genre_id INT NOT NULL,
    CONSTRAINT fk_program_genres_program FOREIGN KEY (program_id) REFERENCES programs(id) ON DELETE CASCADE,
    CONSTRAINT fk_program_genres_genre FOREIGN KEY (genre_id) REFERENCES genres(id) ON DELETE CASCADE,
    CONSTRAINT uq_program_genre UNIQUE (program_id, genre_id)
);

-- 5. シーズンテーブル
CREATE TABLE seasons (
    id INT AUTO_INCREMENT PRIMARY KEY,
    program_id INT NOT NULL,
    season_number INT NOT NULL,
    name VARCHAR(255),
    CONSTRAINT fk_seasons_program FOREIGN KEY (program_id) REFERENCES programs(id) ON DELETE CASCADE,
    CONSTRAINT uq_program_season UNIQUE (program_id, season_number)
);

-- 6. エピソードテーブル
CREATE TABLE episodes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    program_id INT NOT NULL,
    season_id INT NULL,
    episode_number INT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    duration INT NOT NULL,
    release_date DATE NOT NULL,
    CONSTRAINT fk_episodes_program FOREIGN KEY (program_id) REFERENCES programs(id) ON DELETE CASCADE,
    CONSTRAINT fk_episodes_season FOREIGN KEY (season_id) REFERENCES seasons(id) ON DELETE CASCADE,
    CONSTRAINT uq_season_episode UNIQUE (season_id, episode_number)
);

-- 7. 番組枠 / 放送スケジュール中間テーブル
CREATE TABLE channel_schedules (
    id INT AUTO_INCREMENT PRIMARY KEY,
    channel_id INT NOT NULL,
    episode_id INT NOT NULL,
    start_time DATETIME NOT NULL,
    end_time DATETIME NOT NULL,
    view_count BIGINT NOT NULL DEFAULT 0,
    CONSTRAINT fk_schedules_channel FOREIGN KEY (channel_id) REFERENCES channels(id) ON DELETE CASCADE,
    CONSTRAINT fk_schedules_episode FOREIGN KEY (episode_id) REFERENCES episodes(id) ON DELETE CASCADE,
    CONSTRAINT uq_channel_start_time UNIQUE (channel_id, start_time)
);
```

テーブルが正しく作成されたか確認します。

```sql
SHOW DATABASES;
SHOW TABLES;
```

### 7. サンプルデータの投入

ステップ3でデータ抽出クエリ（SELECT文）を検証できるよう、各ジャンル（ドラマ・アニメ・映画・ニュース）に対応する有名作品を含めたサンプルデータを投入します。

- 映画: 『君の名は。』（※単発作品のためシーズン数・エピソード数はNULL設定）
- ニュース: 『ABEMA Prime』

```sql
-- 1. チャンネルテーブル
INSERT INTO channels (id, name) VALUES
(1, 'ドラマ1'),
(2, 'ドラマ2'),
(3, 'アニメ1'),
(4, 'アニメ2'),
(5, 'ニュース');

-- 2. ジャンルテーブル
INSERT INTO genres (id, name) VALUES
(1, 'ドラマ'),
(2, 'アニメ'),
(3, '映画'),
(4, 'ニュース');

-- 3. 番組テーブル
INSERT INTO programs (id, title, description) VALUES
(1, '半沢直樹', '規格外の倍返し！バブル期に東京中央銀行に入行した銀行員・半沢直樹の戦いを描くドラマ。'),
(2, '呪術廻戦', '驚異的な身体能力を持つ高校生・虎杖悠仁が、呪いから人々を救うため呪術師の道へ進む物語。'),
(3, '鬼滅の刃', '家族を鬼に殺された少年・竈門炭治郎が、鬼に変貌した妹を人間に戻すために戦う和風ファンタジー。'),
(4, '君の名は。', '出会うはずのない2人の出逢い。少女と少年の恋と奇跡の物語。'),
(5, 'ABEMA Prime', '変わる報道番組。夜9時からの報道リアリティーショー。');

-- 4. 番組ジャンル中間テーブル
-- (君の名は。はアニメかつ映画の複数ジャンルに指定)
INSERT INTO program_genres (id, program_id, genre_id) VALUES
(1, 1, 1), -- 半沢直樹: ドラマ
(2, 2, 2), -- 呪術廻戦: アニメ
(3, 3, 2), -- 鬼滅の刃: アニメ
(4, 4, 2), -- 君の名は。: アニメ
(5, 4, 3), -- 君の名は。: 映画
(6, 5, 4); -- ABEMA Prime: ニュース

-- 5. シーズンテーブル
INSERT INTO seasons (id, program_id, season_number, name) VALUES
(1, 1, 1, '2013年版'),
(2, 1, 2, '2020年版'),
(3, 2, 1, '第1期'),
(4, 2, 2, '懐玉・玉折／渋谷事変'),
(5, 3, 1, '竈門炭治郎 立志編'),
(6, 3, 2, '遊郭編');

-- 6. エピソードテーブル
-- ※ 単発映画（君の名は。）や帯ニュース（ABEMA Prime）は season_id, episode_number を NULL で登録可能
INSERT INTO episodes (id, program_id, season_id, episode_number, title, description, duration, release_date) VALUES
(1, 1, 1, 1, '第1話 やられたらやり返す！倍返しだ！！', '5億円融資事故が発生。半沢は回収に向け奔走する。', 3240, '2013-07-07'),
(2, 1, 1, 2, '第2話 上司のミスは部下の責任！', '国税局の暗躍により、半沢は窮地に追い込まれる。', 2700, '2013-07-14'),
(3, 2, 3, 1, '第1話 両面宿儺', '呪物の封印が解かれ、学校に危険が迫る。', 1440, '2020-10-03'),
(4, 2, 3, 2, '第2話 自分のために', '呪術高専へ移籍した虎杖に下された判決とは。', 1440, '2020-10-10'),
(5, 3, 5, 1, '第1話 残酷', '炭治郎の家族が鬼に襲われ、生活が一変する。', 1440, '2019-04-06'),
(6, 3, 5, 2, '第2話 育手・鱗滝左近次', '炭治郎は狭霧山を目指し、育手のもとへ向かう。', 1440, '2019-04-13'),
(7, 4, NULL, NULL, '君の名は。', '田舎町に暮らす女子高生・三葉と東京で暮らす男子高校生・瀧の身に起きた入れ替わり現象。', 6420, '2016-08-26'),
(8, 5, NULL, NULL, '8月1日放送回', '最新ニュースと独自の視点による議論を展開。', 7200, '2026-08-01');

-- 7. 番組枠 / 放送スケジュール中間テーブル
INSERT INTO channel_schedules (id, channel_id, episode_id, start_time, end_time, view_count) VALUES
(1, 1, 1, '2026-08-01 20:00:00', '2026-08-01 20:54:00', 15200),
(2, 1, 2, '2026-08-01 21:00:00', '2026-08-01 21:45:00', 12800),
(3, 3, 3, '2026-08-01 19:00:00', '2026-08-01 19:24:00', 25400),
(4, 3, 5, '2026-08-01 19:30:00', '2026-08-01 19:54:00', 31000),
(5, 4, 5, '2026-08-02 12:00:00', '2026-08-02 12:24:00', 8500),
(6, 3, 7, '2026-08-01 21:00:00', '2026-08-01 22:47:00', 45000),
(7, 5, 8, '2026-08-01 21:00:00', '2026-08-01 23:00:00', 9800);
```

### 8. データ投入の確認

正常にデータが挿入されたことを確認するために、SELECT文を実行します。

a. 番組とジャンルの連携確認

```sql
SELECT p.title AS 番組名, g.name AS ジャンル
FROM programs p
JOIN program_genres pg ON p.id = pg.program_id
JOIN genres g ON pg.genre_id = g.id;
```

b. 単発作品（シーズンNULL）を含むエピソード一覧の確認

```sql
SELECT id, title, season_id, episode_number, duration FROM episodes;
```

c. 放送スケジュールと視聴数の確認

```sql
SELECT s.id, c.name AS チャンネル, e.title AS 放送エピソード, s.start_time, s.view_count
FROM channel_schedules s
JOIN channels c ON s.channel_id = c.id
JOIN episodes e ON s.episode_id = e.id;
```

### 9. 補足：環境の初期化・再実施手順

検証をやり直したい場合は、以下のコマンドでデータベースを一度削除し、手順5から再実行することで簡単に初期状態へ戻すことができます。

```sql
DROP DATABASE abema_tv;
```

MySQLの接続を終了するには、以下のコマンドを実行します。

```sql
EXIT;
```
