# 【手順3】「型（データ型）」と「制約」を決める

1. channels（チャンネル）

| **カラム** | **データ型** | **NULL** | **キー** | **初期値** | **AUTO INCREMENT** |
| --- | --- | --- | --- | --- | --- |
| id | INT |  | PRIMARY |  | YES |
| name | VARCHAR(255) |  |  |  |  | 

2. genres（ジャンル）

| **カラム** | **データ型** | **NULL** | **キー** | **初期値** | **AUTO INCREMENT** |
| --- | --- | --- | --- | --- | --- |
| id | INT |  | PRIMARY |  | YES |
| name | VARCHAR(255) |  | UNIQUE |  |  | 

3. programs（番組）

| **カラム** | **データ型** | **NULL** | **キー** | **初期値** | **AUTO INCREMENT** |
| --- | --- | --- | --- | --- | --- |
| id | INT |  | PRIMARY |  | YES |
| title | VARCHAR(255) |  |  |  |  |
| description | TEXT | YES |  |  |  | 

4. program_genres（番組ジャンル中間テーブル）

| **カラム** | **データ型** | **NULL** | **キー** | **初期値** | **AUTO INCREMENT** |
| --- | --- | --- | --- | --- | --- |
| id | INT |  | PRIMARY |  | YES |
| program_id（外部キー） | INT |  | INDEX (UNIQUE複合) |  |  |
| genre_id（外部キー） | INT |  | INDEX (UNIQUE複合) |  |  |
- 制約補足: `UNIQUE(program_id, genre_id)` で同一番組に対するジャンルの重複登録を防止 

5. seasons（シーズン）

| **カラム** | **データ型** | **NULL** | **キー** | **初期値** | **AUTO INCREMENT** |
| --- | --- | --- | --- | --- | --- |
| id | INT |  | PRIMARY |  | YES |
| program_id（外部キー） | INT |  | INDEX (UNIQUE複合) |  |  |
| season_number（シーズン番号） | INT |  | INDEX (UNIQUE複合) |  |  |
| name（シーズン名） | VARCHAR(255) | YES |  |  |  |
- 制約補足: `UNIQUE(program_id, season_number)` で同一番組内でのシーズン番号重複を防止 

6. episodes（エピソード）

| **カラム** | **データ型** | **NULL** | **キー** | **初期値** | **AUTO INCREMENT** |
| --- | --- | --- | --- | --- | --- |
| id | INT |  | PRIMARY |  | YES |
| program_id（外部キー） | INT |  | INDEX |  |  |
| season_id（外部キー） | INT | YES | INDEX (UNIQUE複合) |  |  |
| episode_number（話数） | INT | YES | INDEX (UNIQUE複合) |  |  |
| title（エピソードタイトル） | VARCHAR(255) |  |  |  |  |
| description（エピソード詳細） | TEXT | YES |  |  |  |
| duration（動画時間/尺:秒） | INT |  |  |  |  |
| release_date（公開日） | DATE |  |  |  |  |
- 修正ポイント: 単発番組を登録できるよう `program_id` を追加し、`season_id` と `episode_number` を NULL OK に変更
- 制約補足: `UNIQUE(season_id, episode_number)` で同一シーズン内での話数重複を防止 

7. channel_schedules（番組枠 / 放送スケジュール中間テーブル）

| **カラム** | **データ型** | **NULL** | **キー** | **初期値** | **AUTO INCREMENT** |
| --- | --- | --- | --- | --- | --- |
| id | INT |  | PRIMARY |  | YES |
| channel_id（外部キー） | INT |  | INDEX (UNIQUE複合) |  |  |
| episode_id（外部キー） | INT |  | INDEX |  |  |
| start_time（放送開始時間） | DATETIME |  | INDEX (UNIQUE複合) |  |  |
| end_time（放送終了時間） | DATETIME |  |  |  |  |
| view_count（★KPI:視聴数） | BIGINT |  |  | 0 |  |
