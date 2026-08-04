# ステップ3：データ抽出クエリ


```jsx
ステップ3問題文
以下のデータを抽出するクエリを書いてください。

1.よく見られているエピソードを知りたいです。エピソード視聴数トップ3のエピソードタイトルと視聴数を取得してください
2.よく見られているエピソードの番組情報やシーズン情報も合わせて知りたいです。エピソード視聴数トップ3の番組タイトル、シーズン数、エピソード数、エピソードタイトル、視聴数を取得してください
3.本日の番組表を表示するために、本日、どのチャンネルの、何時から、何の番組が放送されるのかを知りたいです。本日放送される全ての番組に対して、チャンネル名、放送開始時刻(日付+時間)、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を取得してください。なお、番組の開始時刻が本日のものを本日方法される番組とみなすものとします
4.ドラマというチャンネルがあったとして、ドラマのチャンネルの番組表を表示するために、本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を本日から一週間分取得してください
5.(advanced) 直近一週間で最も見られた番組が知りたいです。直近一週間に放送された番組の中で、エピソード視聴数合計トップ2の番組に対して、番組タイトル、視聴数を取得してください
6.(advanced) ジャンルごとの番組の視聴数ランキングを知りたいです。番組の視聴数ランキングはエピソードの平均視聴数ランキングとします。ジャンルごとに視聴数トップの番組に対して、ジャンル名、番組タイトル、エピソード平均視聴数を取得してください。
```
----------------------------------------------------------

Q1.よく見られているエピソードを知りたいです。エピソード視聴数トップ3のエピソードタイトルと視聴数を取得してください

```jsx
1. エピソード視聴数トップ3のエピソードタイトルと視聴数
放送枠ごとの視聴数をエピソード単位で集計（SUM）し、上位3件を取得します。

SELECT 
    e.title AS episode_title,
    SUM(cs.view_count) AS total_view_count
FROM channel_schedules AS cs
INNER JOIN episodes AS e ON cs.episode_id = e.id
GROUP BY e.id, e.title
ORDER BY total_view_count DESC
LIMIT 3;
```
<img width="2048" height="778" alt="image" src="https://github.com/user-attachments/assets/e31338c6-cded-4a7f-a327-8dd37e6093d8" />


----------------------------------------------------------
Q2.よく見られているエピソードの番組情報やシーズン情報も合わせて知りたいです。エピソード視聴数トップ3の番組タイトル、シーズン数、エピソード数、エピソードタイトル、視聴数を取得してください

```jsx
2. エピソード視聴数トップ3の番組情報やシーズン情報
1の集計に加えて、programs および seasons テーブルを結合して情報を取得します。
SELECT 
    p.title AS program_title,
    s.season_number,
    e.episode_number,
    e.title AS episode_title,
    SUM(cs.view_count) AS total_view_count
FROM channel_schedules AS cs
INNER JOIN episodes AS e ON cs.episode_id = e.id
INNER JOIN programs AS p ON e.program_id = p.id
LEFT JOIN seasons AS s ON e.season_id = s.id
GROUP BY p.title, s.season_number, e.episode_number, e.title, e.id
ORDER BY total_view_count DESC
LIMIT 3;

//※ episode,programs,seasonsのなかでseasonsだけは映画などには含まれていないためINNER JOIN にするとデータが反映されないのでLEFT JOIN使っている
```
<img width="2048" height="1100" alt="image" src="https://github.com/user-attachments/assets/93f66c8e-bf90-4319-84ff-8d5a9b50a47b" />

<details>
<summary>疑問：LEFT JOIN ってINNER JOINと何が違うの？</summary>

一言でいうと、「データが片方にしかない時に、切り捨てるか（INNER）、残すか（LEFT）」 の違い

### 💡 違いを一瞬で掴むイメージ

今回の「エピソード」と「放送履歴（視聴数）」で考えてみましょう。
まだ一度も放送されていない「新作エピソード（視聴数データが0件）」が存在するとする

- INNER JOIN（内部結合）
⚬「両方に存在するデータだけ」 を残す
⚬放送履歴がない新作エピソードは バッサリ消える
- LEFT JOIN（左外部結合）
⚬「左側（FROMに書いた方）のテーブルのデータは全部」 残す
⚬放送履歴がない新作エピソードも消さずに残し、視聴数のところは NULL（空っぽ） として表示
</details>
----------------------------------------------------------
Q3.本日の番組表を表示するために、本日、どのチャンネルの、何時から、何の番組が放送されるのかを知りたいです。本日放送される全ての番組に対して、チャンネル名、放送開始時刻(日付+時間)、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を取得してください。なお、番組の開始時刻が本日のものを本日方法される番組とみなすものとします


```jsx
3. 本日の番組表
CURRENT_DATE()（本日）に放送開始される全チャンネルの番組スケジュールを取得します。
SELECT 
    c.name AS channel_name,
    cs.start_time,
    cs.end_time,
    s.season_number,
    e.episode_number,
    e.title AS episode_title,
    e.description AS episode_description
FROM channel_schedules AS cs
INNER JOIN channels AS c ON cs.channel_id = c.id
INNER JOIN episodes AS e ON cs.episode_id = e.id
LEFT JOIN seasons AS s ON e.season_id = s.id
WHERE DATE(cs.start_time) = CURRENT_DATE()
ORDER BY c.id, cs.start_time;
```

<img width="2048" height="1106" alt="image" src="https://github.com/user-attachments/assets/ff1f698b-b01c-42f1-9ae4-eace19193d75" />




----------------------------------------------------------
Q4.ドラマというチャンネルがあったとして、ドラマのチャンネルの番組表を表示するために、本日から一週間分、何日の何時から何の番組が放送されるのかを知りたいです。ドラマのチャンネルに対して、放送開始時刻、放送終了時刻、シーズン数、エピソード数、エピソードタイトル、エピソード詳細を本日から一週間分取得してください

```jsx
4. 「ドラマ1」チャンネルの1週間分の番組表
特定のチャンネル（例: 'ドラマ1'）について、本日〜1週間（7日間）の放送予定を取得します。
SELECT 
    cs.start_time,
    cs.end_time,
    s.season_number,
    e.episode_number,
    e.title AS episode_title,
    e.description AS episode_description
FROM channel_schedules AS cs
INNER JOIN channels AS c ON cs.channel_id = c.id
INNER JOIN episodes AS e ON cs.episode_id = e.id
LEFT JOIN seasons AS s ON e.season_id = s.id
WHERE c.name = 'ドラマ1'
  AND cs.start_time >= CURRENT_DATE()
  AND cs.start_time < DATE_ADD(CURRENT_DATE(), INTERVAL 7 DAY)
ORDER BY cs.start_time;

```

<img width="2048" height="1113" alt="image" src="https://github.com/user-attachments/assets/ae7e643f-fd15-46dc-89a0-342a1eb15890" />


----------------------------------------------------------
Q5.(advanced) 直近一週間で最も見られた番組が知りたいです。直近一週間に放送された番組の中で、エピソード視聴数合計トップ2の番組に対して、番組タイトル、視聴数を取得してください

```jsx
5. (advanced) 直近一週間で最も見られた番組トップ2
直近7日間内に放送開始された枠の合計視聴数を番組ごとに集計します。
SELECT 
    p.title AS program_title,
    SUM(cs.view_count) AS total_view_count
FROM channel_schedules AS cs
INNER JOIN episodes AS e ON cs.episode_id = e.id
INNER JOIN programs AS p ON e.program_id = p.id
WHERE cs.start_time >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
  AND cs.start_time <= NOW()
GROUP BY p.id, p.title
ORDER BY total_view_count DESC
LIMIT 2;

```
<img width="2048" height="1108" alt="image" src="https://github.com/user-attachments/assets/eb4e27d8-4b91-4d97-bb47-3499b1fb33da" />


<details>
<summary>疑問：INNER JOIN episodes AS e ON cs.episode_id = e.idに関して、csとpをつなげられないから中継としてepisodesをいれてるのか？</summary>
→YES  データベースの設計上、cs（放送スケジュール）と p（番組）が直接繋がっていないため、間に e（エピソード） を「中継役（橋渡し）」として挟んでいる

</details>



----------------------------------------------------------
Q6.(advanced) ジャンルごとの番組の視聴数ランキングを知りたいです。番組の視聴数ランキングはエピソードの平均視聴数ランキングとします。ジャンルごとに視聴数トップの番組に対して、ジャンル名、番組タイトル、エピソード平均視聴数を取得してください。


```jsx
6. (advanced) ジャンルごとの番組視聴数ランキング（トップ1のみ）
1つの番組が複数エピソードを持つため、「番組のエピソード平均視聴数」を算出し、ウィンドウ関数（ROW_NUMBER）を使ってジャンル内で1位の番組を抽出します。

WITH program_views AS ( //$a =みたいなこと   $はWITH、aはprofram_... =はAS ()が内容
    -- 1. 番組ごとのエピソード平均視聴数を計算
    SELECT 
        e.program_id,
        AVG(cs.view_count) AS avg_view_count
    FROM channel_schedules AS cs
    INNER JOIN episodes AS e ON cs.episode_id = e.id
    GROUP BY e.program_id
),
genre_program_ranking AS (
    -- 2. ジャンルごとに平均視聴数でランキングを付与
    SELECT 
        g.name AS genre_name,//ジャンル名
        p.title AS program_title,//番組名
        pv.avg_view_count,//番組ごとのエピソード平均視聴数
        ROW_NUMBER() OVER (PARTITION BY g.id ORDER BY pv.avg_view_count DESC) AS rk
    FROM program_views AS pv
    INNER JOIN programs AS p ON pv.program_id = p.id
    INNER JOIN program_genres AS pg ON p.id = pg.program_id
    INNER JOIN genres AS g ON pg.genre_id = g.id
)
-- 3. 各ジャンルの1位のみを取得
SELECT 
    genre_name,
    program_title,
    ROUND(avg_view_count, 1) AS avg_view_count
FROM genre_program_ranking
WHERE rk = 1
ORDER BY genre_name;

```
## 💡 全体のながれ

このクエリは、大きな処理を 3段階のステップ で進めています。

```
【ステップ1】番組ごとに「平均視聴数」を計算する
     ▼
【ステップ2】ジャンルごとに番組を並べて「ランキング（1位、2位…）」をつける
     ▼
【ステップ3】各ジャンルの「1位」だけを抜き出す
```

<details>
<summary>【ステップ1】WITH program_views AS (...)</summary>

👉 “番組ごとに”平均視聴数を計算する下準備

```sql
SELECT
    e.program_id,
    AVG(cs.view_count) AS avg_view_count
FROM channel_schedules AS cs
INNER JOIN episodes AS e ON cs.episode_id = e.id
GROUP BY e.program_id //※これを行うためにSELECT e.program_idをした
```

- やっていること
⚬「放送履歴（`channel_schedules`）」と「エピソード（`episodes`）」を合体
⚬AVG(cs.view_count) で1つの番組に複数あるエピソードの視聴数を平均（AVG）します
⚬「番組ID：平均視聴数」という一時的な計算結果表（`program_views`）を作ります

※ GROUP BY e.program_idを行うためにSELECT e.program_idをした


</details>

<img width="2048" height="920" alt="image" src="https://github.com/user-attachments/assets/f832a541-8d45-4861-a5e1-1cc1ed59ba0c" />

<img width="1920" height="856" alt="image" src="https://github.com/user-attachments/assets/c18b1b9f-a24e-41d7-bd99-e88546767d32" /> 




<details>
<summary>【ステップ2】genre_program_ranking AS (...)</summary>
👉 ジャンルごとに番組を集めて、順位（ランキング）をつける

```sql
SELECT
    g.name AS genre_name,
    p.title AS program_title,
    pv.avg_view_count,
    ROW_NUMBER() OVER (PARTITION BY g.id ORDER BY pv.avg_view_count DESC) AS rk
FROM ...
```
ROW_NUMBER() OVER (...)とは
- PARTITION BY g.id ➔ 「ジャンルごと（アニメ、ドラマ、ニュースなど）に部屋を分ける」 という意味
※PARTITION(パーテーション)「データをグループごとに区切る（仕切る）」
- OVER は「対象（範囲）を指定するための橋渡し（〜に跨がって、〜に対して）」 という役割　英語の “over” には 「〜全体に」「〜に渡って」 という意味がありますよね。まさにそのイメージです！
- ORDER BY pv.avg_view_count DESC ➔ 「その部屋の中で、平均視聴数が高い順に並べる」 という意味です。
- ROW_NUMBER() ➔ 上から順番に 1, 2, 3... と背番号（順位） を振ってくれます。これを `rk`（ランク）という列名にしています。

つまり、アニメ部屋の1位・2位…、ドラマ部屋の1位・2位… という一覧表（genre_program_ranking）がここで完成します。

</details> 

<details>
<summary>疑問：FROM program_views AS pv
    INNER JOIN programs AS p ON pv.program_id = p.id
    INNER JOIN program_genres AS pg ON p.id = pg.program_id
    INNER JOIN genres AS g ON pg.genre_id = g.id
これってNNER JOINでprogram_viewsと他三つを繋げてるの？</summary>
→YES  program_views（1つ目のCTEで作った「番組ごとの平均視聴数テーブル」）をベースにして、そこに他のテーブルを芋づる式でガッチャンコしている

### 💡 どうやって繋がっているのか？（合体のリレー）

それぞれのJOINが何をしているのか、順番に見てみましょう！

```
① program_views (pv)
       │
       ├── [ON pv.program_id = p.id]
       ▼
② programs (p)  ── 番組タイトル(p.title) をゲット！
       │
       ├── [ON p.id = pg.program_id]
       ▼
③ program_genres (pg)  ── 中間テーブル（番組とジャンルをつなぐ橋）
       │
       ├── [ON pg.genre_id = g.id]
       ▼
④ genres (g)  ── ジャンル名(g.name) をゲット！
```

### 🛠 なぜこんなにたくさんJOINするの？

最終的にほしい情報は次の3つ

1. ジャンル名 (`g.name`)
2. 番組タイトル (`p.title`)
3. 平均視聴数 (`pv.avg_view_count`)

ですが、元のprogram_viewsテーブルには 「番組ID」と「平均視聴数」 しかありません。

- 番組タイトルを知るために ➔ programs テーブルを結合
- ジャンル名を知るために ➔ genres テーブルを結合したい！
※ただし、番組とジャンルは直接繋がっておらず、間に program_genres（中間テーブル） が挟まっているため、2段階で結合している。

このように、「足りない情報（列）を別のテーブルから補うために、どんどん結びつけて1つの大きな表にしている」 

</details>

<img width="2048" height="683" alt="image" src="https://github.com/user-attachments/assets/a85ee227-8cac-452d-96cf-c7fc51a0200b" />

<details>
<summary>【ステップ3】最後のSELECT</summary>
👉 各ジャンルの「1位（トップ）」だけを切り取る

```sql
SELECT
    genre_name,
    program_title,
    ROUND(avg_view_count, 1) AS avg_view_count
FROM genre_program_ranking
WHERE rk = 1
ORDER BY genre_name;
```
- WHERE rk = 1：ステップ2で作ったランキング表から、順位（rk）が「1」の番組だけをフィルターして抜き出す
- ROUND(..., 1)：平均視聴数の少数点が見づらくないよう、小数第1位までに四捨五入してキレイに整理

</details>
