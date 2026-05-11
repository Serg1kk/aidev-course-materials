# Расширение Feature Dashboard для WF1

> Как добавить блок «Auto-Pilot Controls» в Dashboard из M4 для домашки M5 (WF1 manual trigger).

---

## Что добавляем

В `FeatureDashboardScreen.js` (или ваш аналог из M4) — отдельная карточка с кнопками управления выбранной фичей. Клик → fetch на webhook n8n → ответ → обновление UI.

### 3 кнопки (минимум)

- **«Запустить ручную проверку»** — POST с `{"feature_id": "search_v2", "action": "check"}`
- **«Включить тестовый режим»** — POST с `{"feature_id": "search_v2", "action": "test", "target_state": "Testing"}`
- **«Откатить фичу»** — POST с `{"feature_id": "search_v2", "action": "rollback", "target_state": "Disabled"}`

Можно расширить — добавить slider для `traffic_percentage` (тогда POST с `traffic_percentage: 50`).

---

## Env-переменные

Добавьте в `.env` фронтенда:

```bash
VITE_N8N_WEBHOOK_URL=https://your-n8n-instance.com/webhook
```

Для локальной разработки используйте n8n cloud или self-host через ngrok / Cloudflare Tunnel чтобы webhook был доступен снаружи.

---

## React snippet (proshop_mern stack)

```jsx
// frontend/src/screens/FeatureDashboardScreen.js (расширение)

import { useState } from 'react';

function AutoPilotControls({ feature, onUpdate }) {
  const [loading, setLoading] = useState(null);  // null | 'check' | 'test' | 'rollback'
  const [error, setError] = useState(null);

  async function callAutoPilot(action, extras = {}) {
    setLoading(action);
    setError(null);

    try {
      const response = await fetch(
        `${import.meta.env.VITE_N8N_WEBHOOK_URL}/feature-control`,
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            feature_id: feature.id,
            action,
            ...extras,
          }),
        }
      );

      const result = await response.json();

      if (!response.ok || result.success === false) {
        setError(result.message || `HTTP ${response.status}`);
        return;
      }

      // Обновляем UI с новым state из ответа агента
      onUpdate(result.current_state);
    } catch (e) {
      setError(`Network error: ${e.message}`);
    } finally {
      setLoading(null);
    }
  }

  return (
    <div className="card auto-pilot-controls" aria-label="Auto-Pilot Controls">
      <h3>Auto-Pilot для {feature.name}</h3>

      <button
        onClick={() => callAutoPilot('check')}
        disabled={loading !== null}
      >
        {loading === 'check' ? 'Проверяем…' : 'Запустить ручную проверку'}
      </button>

      <button
        onClick={() => callAutoPilot('test', { target_state: 'Testing' })}
        disabled={loading !== null}
      >
        {loading === 'test' ? 'Включаем…' : 'Включить тестовый режим'}
      </button>

      <button
        onClick={() => callAutoPilot('rollback', { target_state: 'Disabled' })}
        disabled={loading !== null}
        className="btn-danger"
      >
        {loading === 'rollback' ? 'Откатываем…' : 'Откатить фичу'}
      </button>

      {error && (
        <div className="error" role="alert">
          ⚠️ {error}
        </div>
      )}
    </div>
  );
}
```

### Использование в основном экране

```jsx
function FeatureDashboardScreen() {
  const [features, setFeatures] = useState([]);
  // ... ваш существующий код из M4 ...

  function handleFeatureUpdate(featureId, newState) {
    setFeatures(prev =>
      prev.map(f => (f.id === featureId ? { ...f, ...newState } : f))
    );
  }

  return (
    <div>
      {/* ваша существующая таблица фич из M4 */}
      <FeaturesTable features={features} />

      {/* новый блок для управления выбранной */}
      {selectedFeature && (
        <AutoPilotControls
          feature={selectedFeature}
          onUpdate={(newState) => handleFeatureUpdate(selectedFeature.id, newState)}
        />
      )}
    </div>
  );
}
```

---

## Vue / Svelte эквиваленты

Если ваш Dashboard на Vue или Svelte — паттерн тот же:

1. Composable / store для состояния `loading`, `error`, `currentState`
2. Async-функция `callAutoPilot(action, extras)` с fetch на `VITE_N8N_WEBHOOK_URL`
3. На ответ — обновить store / state

Замените React-snippets на ваш стек, логика идентична.

---

## Ожидаемый ответ от n8n

n8n workflow возвращает JSON по схеме (см. `prompts/gcao-templates.md` Template 1):

```json
{
  "success": true,
  "message": "Feature search_v2 переведена в режим Testing.",
  "current_state": {
    "id": "search_v2",
    "name": "Новый алгоритм поиска",
    "status": "Testing",
    "traffic_percentage": 0,
    "last_modified": "2026-05-15T10:00:00Z"
  }
}
```

Или при отказе:

```json
{
  "success": false,
  "message": "Процент трафика должен быть в диапазоне 0-100. Получено: -50",
  "rejected_at": "input-validation"
}
```

---

## Локальная разработка с self-host n8n

Webhook должен быть доступен снаружи (если фронтенд на Vercel / Netlify / production). Варианты:

### Вариант 1: n8n cloud
```
VITE_N8N_WEBHOOK_URL=https://your-account.app.n8n.cloud/webhook
```

### Вариант 2: ngrok (туннель к локальному n8n)
```bash
ngrok http 5678
# получаете https://abcd-1234.ngrok-free.app
# в .env:
VITE_N8N_WEBHOOK_URL=https://abcd-1234.ngrok-free.app/webhook
```

### Вариант 3: Cloudflare Tunnel
```bash
cloudflared tunnel --url http://localhost:5678
```

### Вариант 4: и фронт и n8n локально
Если оба на `localhost` (фронт `:3000`, n8n `:5678`) — CORS должен быть настроен на n8n стороне.

---

## CORS gotcha

n8n webhook по умолчанию **не отвечает с CORS-заголовками**. Если фронтенд на другом origin — добавьте в Webhook Trigger ноде → Options → `Response Headers`:

```
Access-Control-Allow-Origin: https://your-frontend.com
Access-Control-Allow-Methods: POST, OPTIONS
Access-Control-Allow-Headers: Content-Type
```

Или поставьте перед n8n nginx-прокси с настроенным CORS.

---

## Test на галлюцинации (важно для сдачи)

В Dashboard **не нужно** делать защиту от `-50%` — это работа Switch-ноды в n8n. Дашборд просто:

1. Шлёт POST как есть
2. Получает ответ `{success: false, message: "..."}`
3. Показывает error через `<div className="error">`

Это и есть демонстрация Algorithm-before-AI: дашборд может позволить себе быть «глупым», потому что валидация — на стороне workflow, в Switch-ноде до AI Agent + JSON Schema на MCP.

> Можете руками протестировать через DevTools → Network → отправить POST с `traffic_percentage: -50` и убедиться что ответ — отказ.

---

## Проверка перед PR

- [ ] Блок «Auto-Pilot Controls» виден в Feature Dashboard
- [ ] Кнопки disabled пока идёт запрос
- [ ] Loading state на кнопке во время запроса
- [ ] При успешном ответе — UI обновляется (badge, slider) без перезагрузки страницы
- [ ] При `success: false` — показывается error message (можно через toast / inline div)
- [ ] При network error — graceful fallback («попробуйте позже»)
- [ ] ARIA labels на кнопках
- [ ] Keyboard navigation работает (Tab, Enter)

---

*M5 HSS AI-dev L1. Stack — proshop_mern (React + Vite). Ваш стек может отличаться — паттерн остаётся.*
