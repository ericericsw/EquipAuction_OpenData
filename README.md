# EquipAuction_OpenData

Results of the **unmanned HentaiVerse equipment auction** run by `ericeric91` on the E-Hentai forums,
published automatically for price-lookup tools and anyone else who wants the data.

HentaiVerse 無人自動裝備拍賣的成交資料，每一期拍賣結清後自動上傳，供價格查詢系統與任何人取用。

- Each auction session is published **once, after it is fully settled** (every item has reached its final
  outcome). Files are not updated afterwards except to correct mistakes.
- Everything here is already public in the session's forum thread (`thread_url`); this repo only puts it in a
  machine-readable form.
- All times are **UTC**. All amounts are **Credits**.

每一期在**全部品項結清之後上傳一次**，之後只有更正錯誤才會改動。這裡的內容在該期的論壇討論串
（`thread_url`）都已公開，只是整理成程式讀得懂的格式。時間一律 **UTC**，金額單位為 **Credits**。

## Layout / 目錄

```
index.json                 list of all published sessions / 所有已公開的期次
<year>/<world>/<NNN>.json  one auction session / 一期拍賣
```

| Part | Meaning |
| :--- | :--- |
| `<year>` | Year of the session's `bidding_end` (UTC) / 截標時刻（UTC）的年份 |
| `<world>` | `main` (persistent world) or `isekai` / 主世界或異世界 |
| `<NNN>` | Session number, 3 digits / 期次編號（三位數） |

Example / 例：`2026/main/001.json`, `2026/isekai/012.json`

## How to crawl / 怎麼抓

1. Read `index.json` and compare its `sessions` with what you already have.
2. Fetch each new `path`, e.g.
   `https://raw.githubusercontent.com/ericericsw/EquipAuction_OpenData/main/2026/main/001.json`

先讀 `index.json`，再依每一期的 `path` 抓取尚未取得的檔案。也可以直接 `git clone` / `git pull` 整個 repo。

## `index.json`

| Field | Type | Description |
| :--- | :--- | :--- |
| `schema_version` | int | Format version (see below) |
| `times` | string | Always `"UTC"` |
| `second_price_note` | string | Reminder of what `second_price` means |
| `sessions[]` | array | One entry per published session, oldest first |
| `sessions[].session_no` | string | Session number as used in item IDs: `001` (main), `I001` (isekai) |
| `sessions[].generation` | int | `1` unless session numbers have wrapped around after 999 |
| `sessions[].world` | string | `main` / `isekai` |
| `sessions[].bidding_start` / `bidding_end` | string | Bidding period, ISO 8601 UTC (`2026-12-04T08:00:00Z`) |
| `sessions[].thread_url` | string \| null | The session's forum thread |
| `sessions[].path` | string | Path of the session file in this repo |

## Session file / 期次檔

Top level: `schema_version`, `session_no`, `generation`, `world`, `bidding_start`, `bidding_end`, `thread_url`,
`second_price_note` (same meaning as in `index.json`) and `items[]`. Items are in catalog order.

Items withdrawn before bidding started are not listed. / 開標前就撤回的品項不列出。

| Field | Type | Description |
| :--- | :--- | :--- |
| `item_id` | string | Auction item ID, e.g. `001One01` |
| `equip_id` | string | HentaiVerse equipment ID |
| `equip_url` | string \| null | Equipment page; stays valid while the equipment exists |
| `equip_name` | string | Full equipment name |
| `category` | string \| null | e.g. `One-handed`, `Staff`, `Cloth` |
| `quality` | string \| null | `Peerless`, `Legendary`, `Magnificent`, `Exquisite`, `Superior`, `Average`, `Fair`, `Crude` |
| `level` | int | Equipment level |
| `pab` | string \| null | Primary attribute bonuses, e.g. `DA`, `IW` |
| `base_average` | number \| null | Average of the base stat values |
| `base_percentage` | number \| null | Average base percentage |
| `seller` | string \| null | Seller's display name |
| `start_price` | int | Starting (reserve) price |
| `valid_bids` | int | Number of valid bids — the `Bidders:` number in the thread |
| `outcome` | string | See below |
| `winner` | string \| null | Winner's display name; `null` when there is no winner |
| `second_price` | int \| null | **2nd-highest valid bid — not the price the winner paid**; `null` with fewer than 2 valid bids |

### `outcome`

| Value | Meaning |
| :--- | :--- |
| `SOLD` | Sold to `winner` / 成交 |
| `NO_BIDS` | No valid bids; returned to the seller / 流標 |
| `DEFAULTED` | `winner` won but did not pay or claim in time; returned to the seller / 得標者逾期未領 |
| `CANCELLED` | Cancelled by the admin after bidding started / 管理者取消 |
| `MANUAL_REVIEW` | Settled by the admin outside the normal flow / 人工處理 |

For price statistics you probably want only `SOLD`. / 做價格統計時通常只取 `SOLD`。

### About `second_price` / 關於第二價格

The auction is a **first-price sealed-bid** auction: the highest bid wins and pays its own bid, and the winning
amount is **never disclosed**. `second_price` is the 2nd-highest valid bid, so the winner paid **at least**
`second_price` (equal amounts are possible: on a tie the earlier bid wins).

本拍賣是**第一價格密封拍賣**：出價最高者得標並支付自己的出價，**得標金額不公開**。`second_price` 是第二高的
有效出價，所以得標者付出的金額**不低於** `second_price`（同價時先出價者得標，可能相等）。

Not published / 不公開：any bid times, the winning amount, the 3rd bid and the identity of anyone except the winner.
所有出價時間、得標金額、第 3 名的金額、得標者以外的任何身分。

## Example / 範例

```json
{
  "item_id": "001One04",
  "equip_id": "123456789",
  "equip_url": "https://hentaiverse.org/equip/123456789/0123456789",
  "equip_name": "Exquisite Rapier of Slaughter",
  "category": "One-handed",
  "quality": "Exquisite",
  "level": 430,
  "pab": "DA",
  "base_average": 144.67,
  "base_percentage": 61.67,
  "seller": "SellerName",
  "start_price": 1000,
  "valid_bids": 2,
  "outcome": "SOLD",
  "winner": "WinnerName",
  "second_price": 5000
}
```

## Format version / 格式版本

`schema_version` is currently `1`. New fields may be added without changing it; it is increased only when the
meaning of an existing field changes. Ignore fields you do not know.

目前為 `1`。新增欄位不會改版本號；既有欄位的意義改變時才加一。讀取時請忽略不認得的欄位。

## Contact / 聯絡

Questions or corrections: send a forum PM to `ericeric91` on the E-Hentai forums.
有疑問或發現錯誤，請在 E-Hentai 論壇私訊 `ericeric91`。

License: see [LICENSE](LICENSE).
