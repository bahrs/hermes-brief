# Развернутые ответы на вопросы, заданные во время презентации

---

## 1. Skill curator: безопасен? отключаем?

Skills безопасны настолько, насколько безопасен сам hermes и его sandbox.
Настоятельная рекомендация: разворачивайте его в Docker-контейнере (см [README](https://github.com/bahrs/hermes-brief#развёртывание-на-vps-через-docker-compose) и [docs](https://hermes-agent.nousresearch.com/docs/user-guide/docker))

В отличие от OpenClaw, где упор на community skills, [Hermes фиксирует сложные паттерны сам](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills#when-the-agent-creates-skills).

Инструмент `skills.guard_agent_created` сканирует загружаемые ([Skill hub](https://hermes-agent.nousresearch.com/docs/skills)) скилы на вредоносные паттерны (credential harvesting, obvious prompt injection, exfil instructions). При этом он отключен для создаваемых скилов (частые False positives). ([docs](https://hermes-agent.nousresearch.com/docs/user-guide/configuration#guard-on-agent-created-skill-writes))

Отключается либо (для одной сессии) через `hermes curator pause`,
либо (для одного профиля) записью в конфиг `~/.hermes/config.yaml` `curator.enabled: false`.

Либо включается `/skills approval on` - подтверждение пользователя при создании нового скила.
Из интересного: команда `/learn <prompt>` позволяет создать скилл по мотивам того процесса, который вы опишете в промпте. ([docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills#learning-a-skill-from-sources-learn))

## 2. Безопасность harness в целом и Hermes в частности.

Harness - в первую очередь оболочка для agent loop, которая отправляет данные в интернет либо как запрос к модели, либо при помощи инструмента (tool: `web_scraping`) / MCP. В Hermes есть встроенные инструменты для защиты от prompt injection, запрет на чтение .env для MCP-серверов (кроме строго необходимых переменных) и другие механизмы.
Если не предполагать скрытых в harness backdoors (в случае OSS их наличие может проверить наша собственная КБ), то уровень опасности harness зависит от используемой модели (все вендоры используют запросы пользователей для дообучения).

Материал для дополнительного чтения: 
- безопасность Hermes ([official docs](https://hermes-agent.nousresearch.com/docs/user-guide/security)) 
- дополнительная защита secrets при помощи `egress_proxy` ([docs](https://hermes-agent.nousresearch.com/docs/user-guide/egress/iron-proxy))
В банке, видимо, пока аудит КБ не пройдет - не одобрят для внутреннего пользования.

## 3.  Стоимость использования?

Как и для других harness, зависит от стоимости используемой модели.
При этом разумно предположить, что использование "вертикально интегрированных harness" с их собственными моделями будет чуть более эффективным.

Пример: Opus лучше использовать с Claude Code, GPT 5.6 Luna - с Codex (либо hermes + codex-runtime)

Недорогая подписка - [OpenCode Go (5-10$)](https://opencode.ai/go)
Оптимизация затрат на сложную агентную разработку - [Sakana Fugu](https://sakana.ai/fugu/) (20$ либо OpenRouter API по цене Opus 5 API)

[Свежая статья-сравнение harness](https://composio.dev/content/best-ai-agent-harnesses)
Kimi K3 max resoning через OpenRouter, 25 задач, одинаковый сетап для 8 harness: Kimi Code, **Hermes Agent**, __Claude Code__, Pi Agent, OpenCode, __Codex__, Oh My Pi, Grok Build. Оценка: число выполненных задач, время выполнения, расход токенов, цена успешного выполнения.
Результаты: Hermes оказался одним из самых быстрых и дешевых в использовании harness, ТОП-3 по выполненным задачам.

## 4. Hermes core: langgraph?

Нет, у них собственный движок. ([docs](https://hermes-agent.nousresearch.com/docs/developer-guide/architecture))
