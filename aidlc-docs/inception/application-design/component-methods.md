# Component Methods

## 1. BFF API Routes (Next.js)

### Auth Routes
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| POST | `/api/auth/register` | `{ email, password, displayName }` | `{ userId, token }` | ユーザー登録 |
| POST | `/api/auth/login` | `{ email, password }` | `{ token, user }` | ログイン |
| POST | `/api/auth/logout` | `(session)` | `{ success }` | ログアウト |
| POST | `/api/auth/reset-password` | `{ email }` | `{ success }` | パスワードリセットメール送信 |

### Closet Routes
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| GET | `/api/closet` | `(query: filters)` | `{ items[] }` | 服一覧取得 |
| POST | `/api/closet` | `{ name, type, color, season, formality, photo? }` | `{ item }` | 服登録 |
| PUT | `/api/closet/:id` | `{ ...updateFields }` | `{ item }` | 服情報更新 |
| DELETE | `/api/closet/:id` | `(param: id)` | `{ success }` | 服削除 |
| POST | `/api/closet/upload` | `FormData(image)` | `{ uploadUrl, basicTags }` | 画像アップロード＋同期タグ |
| GET | `/api/closet/:id/tags` | `(param: id)` | `{ detailedTags }` | 詳細タグ取得（非同期結果） |

### Daily Context Routes
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| GET | `/api/calendar/events` | `{ date }` | `{ events[] }` | Google Calendar予定取得 |
| POST | `/api/calendar/connect` | `{ authCode }` | `{ success }` | Google Calendar OAuth連携 |
| GET | `/api/weather` | `{ lat?, lon?, city? }` | `{ weather }` | 天気情報取得 |

### Outfit Recommendation Routes
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| POST | `/api/outfit/recommend` | `{ date, events[], location? }` | `{ persona, outfit, reason, alternative }` | 服装提案生成 |
| POST | `/api/outfit/regenerate` | `{ date, previousOutfitId, feedback? }` | `{ persona, outfit, reason, alternative }` | 別の提案を再生成 |
| POST | `/api/outfit/accept` | `{ outfitId }` | `{ success }` | 提案を採用（着用履歴記録） |

---

## 2. Core API (FastAPI)

### Auth Service
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| `register_user()` | `POST /users/register` | `UserCreate` | `UserResponse + JWT` | ユーザー作成 |
| `login_user()` | `POST /users/login` | `LoginRequest` | `TokenResponse` | JWT発行 |
| `verify_token()` | `(middleware)` | `JWT header` | `UserContext` | トークン検証 |
| `reset_password()` | `POST /users/reset` | `ResetRequest` | `SuccessResponse` | パスワードリセット |

### Closet Service
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| `list_items()` | `GET /closet` | `filters, userId` | `List[ClothingItem]` | 服一覧取得 |
| `create_item()` | `POST /closet` | `ClothingItemCreate` | `ClothingItem` | 服登録 |
| `update_item()` | `PUT /closet/{id}` | `ClothingItemUpdate` | `ClothingItem` | 服更新 |
| `delete_item()` | `DELETE /closet/{id}` | `itemId` | `SuccessResponse` | 服削除 |
| `generate_upload_url()` | `POST /closet/upload-url` | `{ fileName, contentType }` | `{ presignedUrl, key }` | S3プリサインドURL生成 |
| `tag_image_sync()` | `POST /closet/tag-sync` | `{ s3Key }` | `{ basicTags }` | 同期: 基本タグ（色・種別） |
| `tag_image_async()` | `POST /closet/tag-async` | `{ s3Key, itemId }` | `{ jobId }` | 非同期: 詳細タグ投入 |
| `get_tags()` | `GET /closet/{id}/tags` | `itemId` | `{ detailedTags }` | 詳細タグ結果取得 |

### Calendar Service
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| `connect_calendar()` | `POST /calendar/connect` | `{ authCode, userId }` | `{ success }` | OAuth認可コード交換 |
| `get_events()` | `GET /calendar/events` | `{ date, userId }` | `List[CalendarEvent]` | 指定日の予定取得 |
| `refresh_token()` | `(internal)` | `userId` | `{ accessToken }` | トークンリフレッシュ |

### Weather Service
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| `get_weather()` | `GET /weather` | `{ lat, lon } or { city }` | `WeatherData` | 天気情報取得（キャッシュ付き） |
| `_fetch_from_api()` | `(internal)` | `location` | `WeatherData` | OpenWeatherMap API呼び出し |
| `_get_cached()` | `(internal)` | `cacheKey` | `WeatherData?` | キャッシュ確認（15分TTL） |

### Outfit Recommendation Service
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| `recommend_outfit()` | `POST /outfit/recommend` | `RecommendRequest` | `OutfitRecommendation` | 服装提案生成 |
| `regenerate_outfit()` | `POST /outfit/regenerate` | `RegenerateRequest` | `OutfitRecommendation` | 別の提案を再生成 |
| `accept_outfit()` | `POST /outfit/accept` | `{ outfitId, userId }` | `SuccessResponse` | 提案採用→着用履歴記録 |
| `_build_prompt()` | `(internal)` | `context` | `str` | Bedrockプロンプト構築 |
| `_call_bedrock()` | `(internal)` | `prompt` | `BedrockResponse` | Bedrock API呼び出し |
| `_parse_response()` | `(internal)` | `BedrockResponse` | `OutfitRecommendation` | レスポンスパース |

### Wear History Service
| Method | Endpoint | Input | Output | Purpose |
|--------|----------|-------|--------|---------|
| `record_wear()` | `POST /history` | `{ userId, itemIds[], date }` | `SuccessResponse` | 着用記録 |
| `get_recent()` | `GET /history/recent` | `{ userId, days? }` | `List[WearRecord]` | 直近の着用履歴 |

**Note**: 詳細なビジネスルール（TPO判定ロジック、プロンプト設計等）はFunctional Design（CONSTRUCTION Phase）で定義します。
