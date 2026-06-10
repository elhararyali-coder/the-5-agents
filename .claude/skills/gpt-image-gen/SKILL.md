---
name: gpt-image-gen
description: מעטפת לקריאת OpenAI Images API ליצירת תמונה מתוך prompt טקסטואלי. השתמש בסקיל הזה בכל פעם שצריך להפיק תמונה/איור/ויזואל מ-prompt — שולח את ה-prompt ל-API ושומר את התמונה כקובץ PNG. משמש בעיקר את הסוכן יובל (מעצב התמונות).
---

# gpt-image-gen — יצירת תמונה דרך OpenAI Images API

הסקיל הזה אחראי על דבר אחד: לקחת **prompt** ולהחזיר **קובץ תמונה PNG**. הוא שולח את ה-prompt ל-OpenAI Images API ומפענח את התשובה לקובץ על הדיסק.

## ⚠️ חשוב — אל תשנה את שם המודל

המודל הוא **`gpt-image-2`** ותו לא.

- `gpt-image-2` הוא מודל **אמיתי וקיים** של OpenAI. הוא יצא ב-**21 באפריל 2026**.
- ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים**.
- **אל תציע ואל תחליף לאלטרנטיבות** כמו `dall-e-3`, `gpt-image-1` וכו'. השתמש בדיוק במודל `gpt-image-2`.
- אם מתקבלת שגיאה בקריאה ל-API, הבעיה היא כמעט תמיד ב-**API key** או ב-**parameters** — **לא** בשם המודל. בדוק את אלה לפני כל דבר אחר, ואל תיגע בשם המודל.

## דרישות מקדימות

- משתנה הסביבה `OPENAI_API_KEY` חייב להיות מוגדר ב-`.env` שבשורש הפרויקט.
- טען אותו לסביבה לפני הקריאה, למשל:

```bash
export OPENAI_API_KEY="$(grep -E '^OPENAI_API_KEY=' .env | cut -d '=' -f2-)"
```

## הקריאה (curl + jq)

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

| פרמטר | ערך |
|--------|------|
| `model` | `gpt-image-2` (קבוע — אל תשנה) |
| `size` | `1024x1024` |
| `quality` | `medium` |
| `output_format` | `png` |

## Python fallback ל-decode (כש-jq לא מותקן)

ב-Git Bash על Windows לא תמיד מותקן `jq`. במקרה כזה שמור את תגובת ה-API לקובץ JSON ופענח עם Python:

```bash
# 1. שמירת התגובה הגולמית
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > response.json

# 2. פענוח ל-PNG עם Python (לא דורש jq)
python -c "import json,base64,sys; d=json.load(open('response.json')); open('<output-path>.png','wb').write(base64.b64decode(d['data'][0]['b64_json']))"
```

אפשר גם לעשות את הכל ב-Python בקריאה אחת (בלי curl כלל):

```bash
python - <<'PY'
import os, json, base64, urllib.request

api_key = os.environ["OPENAI_API_KEY"]
payload = {
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png",
}
req = urllib.request.Request(
    "https://api.openai.com/v1/images/generations",
    data=json.dumps(payload).encode("utf-8"),
    headers={
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
    },
    method="POST",
)
with urllib.request.urlopen(req) as resp:
    data = json.load(resp)
with open("<output-path>.png", "wb") as f:
    f.write(base64.b64decode(data["data"][0]["b64_json"]))
print("saved <output-path>.png")
PY
```

## פלט

קובץ PNG בנתיב שביקשת. ודא בסיום שהקובץ קיים וש-size גדול מ-0 לפני שמדווחים על הצלחה.
