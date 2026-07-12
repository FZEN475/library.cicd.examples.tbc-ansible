# tbc-ansible

# Общая логика
* **job** не срабатывают на **TAG**
* `destroy` - ручное выполнение или по таймеру `on_stop` `auto_stop_in`
  * production не имеет этапа `destroy`
* `review != PROD_REF || INTEG_REF`
* `integration = INTEG_REF`
* `staging = PROD_REF + destroy`
* `production = PROD_REF - destroy`

| Имя переменной  | Значение    | Действие GitLab                    |
|-----------------|-------------|------------------------------------|
| *_DISABLED      | "true"      | Исключить запуск job (when: never) |
| *_PLAYBOOK_FILE | null или "" | Исключить запуск job (when: never) |

---

# LINT

### ansible-lint-review
Автоматическая проверка кода (lint) в ветках разработчиков.

| № | Условие (if)                                              | Результат (если ИСТИНА) | Описание логики                                   |
|---|-----------------------------------------------------------|-------------------------|---------------------------------------------------|
| 1 | `$CI_COMMIT_TAG`                                          | when: never             | Исключить запуск на тегах                         |
| 2 | `$ANSIBLE_LINT_DISABLED == "true"`                        | when: never             | Исключить запуск, если линтер выключен            |
| 3 | `$ANSIBLE_REVIEW_PLAYBOOK_FILE == null \|\| ... == ""`    | when: never             | Исключить, если не задан плейбук для review       |
| 4 | `$CI_COMMIT_REF_NAME =~ $INTEG_REF \|\| ... =~ $PROD_REF` | when: never             | Исключить запуск в ветках интеграции и продакшена |
| 5 | `!reference [.test-policy, rules]`                        | (наследуется)           | Подключение общих правил тестирования             |

### ansible-lint-integration
Автоматическая проверка кода (lint) в интеграционной ветке.

| № | Условие (if)                                          | Результат (если ИСТИНА) | Описание логики                                 |
|---|-------------------------------------------------------|-------------------------|-------------------------------------------------|
| 1 | `$CI_COMMIT_TAG`                                      | when: never             | Исключить запуск на тегах                       |
| 2 | `$ANSIBLE_LINT_DISABLED == "true"`                    | when: never             | Исключить запуск, если линтер выключен          |
| 3 | `$ANSIBLE_INTEG_PLAYBOOK_FILE == null \|\| ... == ""` | when: never             | Исключить, если не задан плейбук для интеграции |
| 4 | `$CI_COMMIT_REF_NAME =~ $PROD_REF`                    | when: never             | Исключить запуск в ветке продакшена             |
| 5 | `!reference [.test-policy, rules]`                    | (наследуется)           | Подключение общих правил тестирования           |

### ansible-lint-staging
Автоматическая проверка кода (lint) для окружения staging.

| № | Условие (if)                                            | Результат (если ИСТИНА) | Описание логики                              |
|---|---------------------------------------------------------|-------------------------|----------------------------------------------|
| 1 | `$CI_COMMIT_TAG`                                        | when: never             | Исключить запуск на тегах                    |
| 2 | `$ANSIBLE_LINT_DISABLED == "true"`                      | when: never             | Исключить запуск, если линтер выключен       |
| 3 | `$ANSIBLE_STAGING_PLAYBOOK_FILE == null \|\| ... == ""` | when: never             | Исключить, если не задан плейбук для staging |
| 4 | `!reference [.test-policy, rules]`                      | (наследуется)           | Подключение общих правил тестирования        |

### ansible-lint-production
Автоматическая проверка кода (lint) для финального продакшена.

| № | Условие (if)                                         | Результат (если ИСТИНА) | Описание логики                                 |
|---|------------------------------------------------------|-------------------------|-------------------------------------------------|
| 1 | `$CI_COMMIT_TAG`                                     | when: never             | Исключить запуск на тегах                       |
| 2 | `$ANSIBLE_LINT_DISABLED == "true"`                   | when: never             | Исключить запуск, если линтер выключен          |
| 3 | `$ANSIBLE_PROD_PLAYBOOK_FILE == null \|\| ... == ""` | when: never             | Исключить, если не задан плейбук для продакшена |
| 4 | `!reference [.test-policy, rules]`                   | (наследуется)           | Подключение общих правил тестирования           |

---

# CHECKOV

### ansible-checkov
Статический анализ безопасности (Checkov).

| № | Условие (if)                          | Результат (если ИСТИНА) | Описание логики                         |
|---|---------------------------------------|-------------------------|-----------------------------------------|
| 1 | `$ANSIBLE_CHECKOV_DISABLED == "true"` | when: never             | Исключить запуск, если checkov выключен |
| 2 | `!reference [.test-policy, rules]`    | (наследуется)           | Подключение общих правил тестирования   |

---

# APPLY

### ansible-review
Применение кода (apply) в ветках разработчиков (feature-branches).

| № | Условие (if)                                            | Результат (если ИСТИНА)      | Описание логики                             |
|---|---------------------------------------------------------|------------------------------|---------------------------------------------|
| 1 | `$CI_COMMIT_TAG`                                        | when: never                  | Исключить запуск на тегах                   |
| 2 | `$ANSIBLE_REVIEW_PLAYBOOK_FILE == null \|\| ... == ""`  | when: never                  | Исключить, если не задан плейбук для review |
| 3 | `$CI_COMMIT_REF_NAME !~ $PROD_REF && ... !~ $INTEG_REF` | when: on_success (по умолч.) | Запуск только в не-прод и не-интег ветках   |

### ansible-integration
Применение кода (apply) после слияния в интеграционную ветку.

| № | Условие (if)                                          | Результат (если ИСТИНА)      | Описание логики                                 |
|---|-------------------------------------------------------|------------------------------|-------------------------------------------------|
| 1 | `$ANSIBLE_INTEG_PLAYBOOK_FILE == null \|\| ... == ""` | when: never                  | Исключить, если не задан плейбук для интеграции |
| 2 | `$CI_COMMIT_REF_NAME =~ $INTEG_REF`                   | when: on_success (по умолч.) | Запуск только в интеграционных ветках           |

### ansible-staging
Применение кода (apply) для окружения staging из продакшн-ветки.

| № | Условие (if)                                            | Результат (если ИСТИНА)      | Описание логики                              |
|---|---------------------------------------------------------|------------------------------|----------------------------------------------|
| 1 | `$ANSIBLE_STAGING_PLAYBOOK_FILE == null \|\| ... == ""` | when: never                  | Исключить, если не задан плейбук для staging |
| 2 | `$CI_COMMIT_REF_NAME =~ $PROD_REF`                      | when: on_success (по умолч.) | Запуск только в продакшн-ветках              |

### ansible-production
Применение кода (apply) в продакшн-инфраструктуру.

| № | Условие (if)                                         | Результат (если ИСТИНА)      | Описание логики                                         |
|---|------------------------------------------------------|------------------------------|---------------------------------------------------------|
| 1 | `$CI_COMMIT_REF_NAME !~ $PROD_REF`                   | when: never                  | Исключить запуск во всех ветках, кроме ветки продакшена |
| 2 | `$ANSIBLE_PROD_PLAYBOOK_FILE == null \|\| ... == ""` | when: never                  | Исключить, если не задан плейбук для продакшена         |
| 3 | `$ANSIBLE_PROD_DEPLOY_STRATEGY == "manual"`          | when: manual                 | Ручной запуск деплоя                                    |
| 4 | `$ANSIBLE_PROD_DEPLOY_STRATEGY == "auto"`            | when: on_success (по умолч.) | Автоматический запуск деплоя                            |

---

# DELETE (CLEANUP)

| Настройка параметра | Значение в коде     | Описание логики                       | Действие GitLab                              |
|---------------------|---------------------|---------------------------------------|----------------------------------------------|
| Влияние на пайплайн | allow_failure: true | Ошибка удаления считается некритичной | Пайплайн остается зеленым/варнингом при сбое |

### ansible-cleanup-review
Очистка инфраструктуры в ветках разработчиков.

| № | Условие (if)                                                                                                            | Результат (если ИСТИНА)           | Описание логики                                                           |
|---|-------------------------------------------------------------------------------------------------------------------------|-----------------------------------|---------------------------------------------------------------------------|
| 1 | `$CI_COMMIT_TAG`                                                                                                        | when: never                       | Исключить появление кнопки уничтожения на Git-тегах                       |
| 2 | `$CI_COMMIT_REF_NAME =~ $PROD_REF \|\| ... =~ $INTEG_REF`                                                               | when: never                       | Исключить запуск в ветках интеграции и продакшена                         |
| 3 | `($ANSIBLE_REVIEW_CLEANUP_PLAYBOOK_FILE != null && ... != "") \|\| ($ANSIBLE_REVIEW_CLEANUP_TAGS != null && ... != "")` | when: manual, allow_failure: true | Показывает кнопку удаления только если задан плейбук или теги для очистки |

### ansible-cleanup-integration
Очистка инфраструктуры в интеграционной ветке.

| № | Условие (if)                                                                                                          | Результат (если ИСТИНА)           | Описание логики                                                           |
|---|-----------------------------------------------------------------------------------------------------------------------|-----------------------------------|---------------------------------------------------------------------------|
| 1 | `$CI_COMMIT_REF_NAME !~ $INTEG_REF`                                                                                   | when: never                       | Исключить запуск, если это не интеграционная ветка                        |
| 2 | `($ANSIBLE_INTEG_CLEANUP_PLAYBOOK_FILE != null && ... != "") \|\| ($ANSIBLE_INTEG_CLEANUP_TAGS != null && ... != "")` | when: manual, allow_failure: true | Показывает кнопку удаления только если задан плейбук или теги для очистки |

### ansible-cleanup-staging
Очистка инфраструктуры staging из продакшн-ветки.

| № | Условие (if)                                                                                                              | Результат (если ИСТИНА)           | Описание логики                                                           |
|---|---------------------------------------------------------------------------------------------------------------------------|-----------------------------------|---------------------------------------------------------------------------|
| 1 | `$CI_COMMIT_REF_NAME !~ $PROD_REF`                                                                                        | when: never                       | Исключить запуск, если это не продакшн-ветка                              |
| 2 | `($ANSIBLE_STAGING_CLEANUP_PLAYBOOK_FILE != null && ... != "") \|\| ($ANSIBLE_STAGING_CLEANUP_TAGS != null && ... != "")` | when: manual, allow_failure: true | Показывает кнопку удаления только если задан плейбук или теги для очистки |


### variables
```yaml
variables:
  # default production ref name (pattern)
  PROD_REF: /^(master|main)$/
  # default integration ref name (pattern)
  INTEG_REF: /^develop$/
  # Включение тестов вне зависимости от типа события (.test-policy:)
  ADAPTIVE_PIPELINE_DISABLED: "true"
```

### stages
```yaml
stages:
  - build           # ansible-lint-review, ansible-lint-integration, ansible-lint-staging, ansible-lint-production
  - test            # ansible-checkov
  - package-build   # Нет job
  - package-test    # Нет job
  - infra           # Нет job
  - deploy          # ansible-review, ansible-integration, ansible-staging,
                    # ansible-cleanup-review, ansible-cleanup-integration, ansible-cleanup-staging
  - acceptance      # Нет job
  - publish         # Нет job
  - infra-prod      # Нет job
  - production      # ansible-production
```

### extends
```yaml
extends:
  .ansible-base:
    ansible-checkov:
    .ansible-lint:
      ansible-lint-review:
      ansible-lint-integration:
      ansible-lint-staging:
      ansible-lint-production:
    .ansible-env-base:
      .ansible-deploy:
        ansible-review:
        ansible-integration:
        ansible-staging:
        ansible-production:
      .ansible-cleanup:
        ansible-cleanup-review:
        ansible-cleanup-integration:
        ansible-cleanup-staging:
```

### workflow.rules

```yaml
.tbc-workflow-rules:
  skip-back-merge:                            # Не запускать при обратном MR (из прод в feature)
  prefer-mr-pipeline:                         # => when: never
    - это обычный commit в ветку
    - и для этой ветки уже есть открытый MR
    - и ветка не prod и не integration
  extended-skip-ci:                           # Поиск в CI_COMMIT_MESSAGE паттерна [skip ci on ...]
    - "*tag" && $CI_COMMIT_TAG                # Не запускать на тэге
    - "*branch" && $CI_COMMIT_BRANCH          # Не запускать при обычном коммите
    - "*mr" && $CI_MERGE_REQUEST_ID           # Не запускать при MR
    - "*default" && $CI_COMMIT_REF_NAME =~ $CI_DEFAULT_BRANCH  # Не запускать на ветке по умолчанию
    - "*prod" && $CI_COMMIT_REF_NAME =~ $PROD_REF  # Не запускать на продакшене
    - "*integ" && $CI_COMMIT_REF_NAME =~ $INTEG_REF # Не запускать на интеграции
    - "*dev" && $CI_COMMIT_REF_NAME !~ $PROD_REF && $CI_COMMIT_REF_NAME !~ $INTEG_REF # Не запускать на продакшене и интеграции

```

### .test-policy
Правила применения тестирования
```yaml
.test-policy:
  - Это обычный commit -> on_success
  - ADAPTIVE_PIPELINE_DISABLED == "true" -> on_success
  - Ветка интеграции или релиза -> on_success
  - Это не MR и нет открытых -> manual && allow_failure=true
  - '$CI_MERGE_REQUEST_TITLE =~ /^Draft:.*/' ->  on_success && allow_failure=true
  - on_success
```
