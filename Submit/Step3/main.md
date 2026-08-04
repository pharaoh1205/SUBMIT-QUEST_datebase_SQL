# ステップ3：データ抽出クエリ

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

<img width="2048" height="759" alt="image" src="https://github.com/user-attachments/assets/7cc9a3b4-30be-4d5b-890b-e2e2be1e3ccb" />

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


