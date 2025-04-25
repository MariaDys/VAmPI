### 1. Dependency Scan (pip-audit)

Инструмент: `pip-audit`  
Результат: Уязвимости не обнаружены. Все используемые зависимости безопасны на момент сканирования.
- **Flask 2.2.2**
  - PYSEC-2023-62 (CVE-2023-30861): Возможна утечка `session`-cookie при определённых условиях кэширования.

- **Werkzeug 2.2.3**
  - PYSEC-2023-221 (CVE-2023-46136): Уязвимость DoS через специально сформированные multipart-запросы.
  - CVE-2024-34069: Возможность удалённого запуска кода через отладчик при вводе PIN.
  - CVE-2024-49766: Ошибка в safe_join() на Windows/Python < 3.11 позволяет доступ к нежелательным путям.
  - CVE-2024-49767: Возможность обойти max_form_memory_size при загрузке форм.


### 2. Static Analysis (Semgrep)

Инструмент: `semgrep` с кастомными (./semgrep) и публичными правилами (p/python https://semgrep.dev/p/python, p/owasp-top-ten https://semgrep.dev/p/owasp-top-ten, p/ci https://semgrep.dev/p/ci)

Semgrep обнаружил **9 потенциальных уязвимостей**, среди которых:
- WARNING: Разные ответы при различном логине/пароле (api_views/users.py:103,106) 
- ERROR: Доступ к объектам с неправильной проверкой прав (api_views/books.py:51)
- ERROR: Возможность несанкционированного изменения пароля (username берется из url, пароль меняется без проверки прав пользователя) api_views/users.py:187,194
- ERROR: Возможность SQL-инъекции (api_views/users.py:187,194)
- WARNING: Риск ReDoS (api_views/users.py:144)

### 3. Secret Scan

Инструмент: `gitleaks`  
Результат: Секреты не обнаружены.


### 4. DAST (OWASP ZAP)

Инструмент: `OWASP ZAP Baseline` (анализ ответов на обычные GET-запросы -- проверка заголовков, метаинформации)
Результат: PASS(62), WARN-NEW(4), FAIL(0)

| Тип                      | URL                         | Описание                                     | Пояснение |
|--------------------------|-----------------------------|----------------------------------------------|------------------------------|
| Header Missing           | `/`                         | Отсутствует `X-Content-Type-Options`         |Браузер может опеределять тип контента самостоятельно|
| Version Disclosure       | `/`                         | `Server` заголовок раскрывает информацию     |Заголовок Server в ответе|
| Cacheable Content        | `/`, `/robots.txt`, `/sitemap.xml` | Возможность кеширования                    | Чувствительные данные могут сохраняться в бразуере|
| Spectre Vulnerability    | `/`                         | Отсутствует site isolation                   |Нужен Cross-Origin-Resource-Policy: same-origin чтобы предотвратить возможную утечку|


---

### Общая сводка

| Категория         | Инструмент     | Найдено потенциальных уязвимостей |
|------------------|----------------|------------|
| Dependencies     | pip-audit      | 5          |
| Static Analysis  | Semgrep        | 9          | 
| Secrets          | Gitleaks       | 0          | 
| DAST             | OWASP ZAP      | 4 (WARN-NEW) |


