# Подготовка релиза FreeUnit 1.36.2

- Ветка: `pre-1.36.2`, создана от `master` на `eb1767c9`.
- Образец — цикл 1.36.1: `bb35d17d` и `cd1efd93` (changelog), `4484b540`
  (version и openapi), `bcf560a8` (unitctl), `212ea040` (Dockerfile'ы).
  Общий порядок описан в `RELEASE-PROCESS.md`.
- Состояние на 2026-09-24. Всё, что помечено «проверено», прогнано в чистом
  worktree этой ветки.

## 1. Текущее состояние

| Файл | Сейчас | К релизу |
|---|---|---|
| `version` | `1.36.1` / `13601` | `1.36.2` / `13602` |
| `docs/unit-openapi.yaml`, строка 3 | `FreeUnit 1.36.1 (ex NGINX Unit)` | `FreeUnit 1.36.2 (ex NGINX Unit)` |
| `docs/changes.xml`, два блока `ver="1.36.2"` (unit и модули) | есть, `date="2026-08-30"` (заглушка) | дата релиза в обоих блоках |
| `CHANGES`, секция 1.36.2 | 40 записей, `30 Aug 2026`, **не совпадает** с `changes.xml` | перегенерировать из `changes.xml` |
| `tools/unitctl`: 3 × `Cargo.toml`, `Cargo.lock`, `openapi-config.json`, `unit-openapi/README.md` | `1.36.1` | `1.36.2` |
| `tools/unitctl/CHANGELOG.md` | `[Unreleased]` пуст, хотя с 1.36.1 было 12 коммитов в unitctl | секция `[1.36.2]` и ссылки |
| `pkg/docker/Dockerfile.*` (27 файлов) | `image.version` и `git clone -b` = `1.36.1` | перегенерировать |
| `RELEASE-PROCESS.md`, пример в appendix | `1.36.1` / `13601` | `1.36.2` / `13602` |

Вручную не правятся:

- `pkg/{deb,rpm,npm}/Makefile`: берут версию из `version`.
- `src/nodejs/unit-http/package.json`: `%%VERSION%%` подставляется при сборке.
- Changelog'и deb и rpm генерируются из `docs/changes.xml`. Модульный блок
  1.36.2 уже покрывает все `MODULE_SUFFIX_*`. Проверено: сверху стоит
  `1.36.2-1` для `unit` и для `unit-php8.3`.
- `info.version: 0.2.0` в OpenAPI — это версия API; в 1.36.1 её не меняли.

## 2. Changelog (самый большой шаг, делать первым)

### 2.1 Свести `CHANGES` и `docs/changes.xml`

Проверено: `make -C docs changes` выдаёт секцию 1.36.2, которая отличается от
закоммиченной:

- записи стоят в другом порядке;
- у 4 записей разный текст:
  - Security: экранирование строкового `"access_log"` `"format"` (оба текста
    из `0f83875a`);
  - Feature: byte-range requests for static files (оба текста из `a8ddab24`);
  - Feature: conditional requests for static files (`CHANGES` из `f288ee89`,
    `changes.xml` дописан позже в `60ee106f`);
  - Change: weak entity-tag и last-modified внутри секунды mtime (`2edd375f`;
    потом оба файла правил `49c8a544`, но тексты всё равно разные);
- в `changes.xml` слово `TLS-terminating` разбито по строкам, поэтому в выводе
  получается `TLS- terminating` (запись о том, что unitctl больше не
  предлагает HTTP/2);
- xslt пишет заголовок `Changes with Unit`, а в репозитории принят
  `Changes with FreeUnit`.

Тело секции 1.36.1 совпадает с генерацией построчно. Секции 1.36.0 и старше
расходятся исторически (в 1.36.0 закоммичено 42 записи, а генерация даёт 14),
поэтому `mv build/CHANGES .` из `RELEASE-PROCESS.md` **не использовать**: он
перепишет старые секции и все заголовки `FreeUnit`.

- [ ] Выбрать итоговый текст для 4 записей и внести его в `docs/changes.xml`
      (это источник истины).
- [ ] Собрать `TLS-terminating` на одной строке.
- [ ] Задать порядок записей в `changes.xml`: `CHANGES` берёт порядок оттуда.

### 2.2 Дописать недостающие записи

Проверено поиском по обоим файлам (0 совпадений): записей нет для этих
изменений.

| Коммит | Что изменилось |
|---|---|
| `f491e004` | новая опция `"limits": {"start_timeout"}`: ограничение на время старта процесса приложения |
| `54a6a490` | новая опция `"execution_timeout"` у `wasm-wasi-component` |
| `3bc72e19` | contrib wasmtime 43.0.1 → 47.0.4 |
| `62e10c59`, `f49f39ca`, `90614984`, `a42376a6` | wasm: некорректный или неудавшийся запрос больше не роняет воркер; ошибка чтения тела не сбивает длину буфера; поля запроса декодируются как lossy UTF-8 |
| `362f584f` | WebSocket: text-фреймы и close reason проверяются как UTF-8 (код 1007) |
| `7234df62`, `7776a124` | конфигурация не в UTF-8 отвергается (сейчас упомянуто только внутри записи про unitctl) |
| `279e1bb5` | object-формат `access_log`: теперь экранируется каждое значение |
| `4c79840d` | PROCESS_READY проверяется по pid отправителя, который сообщает ядро |
| `420f4b2d` | port-сообщение с усечёнными control data отвергается |
| `44a10ff3` | `"rootfs"` вместе с credential ns без mount ns отвергается уже при валидации |
| `4b6a77e6` | Java: server name берётся у роутера, а не разбирается из `Host` (IPv6) |
| `99086536` | static: `"index"` с нулевым байтом отвергается |
| `e27b3e50` | static: обработка файла, который усекли во время отдачи |
| `fd2d1501` | static: `Last-Modified` отдаётся в GMT, а не в локальном времени |
| `4aaede27`, `af4f0445` | тело ответа отбрасывается у статусов, которым тело не положено; proxy: ответ upstream без тела завершается на заголовках |
| `94553347` | listener: сокет закрывается до подтверждения его удаления |
| `9c0ab65c`, `bdbdba86` | state-файлы пишутся атомарно; короткая запись pid-файла дописывается до конца |
| `b10d89f7` | discovery: список модулей экранируется как JSON |
| `7a2bb597` | php: монтирование `automount.language_deps` |
| `77a4528e` | unitctl: `execute` с JS-модулем падал с panic |
| `8157a805` | wasm-wasi-component: wasmtime 47.0.2 → 47.0.4 (RUSTSEC-2026-0269, -0268, -0223, -0222; одна из них — выход за пределы файловой песочницы) |
| `8157a805` | otel: h2 → 0.4.19 (RUSTSEC-2026-0258), anyhow → 1.0.104 (RUSTSEC-2026-0190) |
| `8157a805` | unitctl: rustls 0.23.42 → 0.23.45 (RUSTSEC-2026-0285) вместе с aws-lc-rs, aws-lc-sys и rustls-webpki |

Второй эшелон: решить, видно ли изменение пользователю.

- Старт процессов: `7cda8d6d`, `2906191e`, `ec20d6af`, `1596376a`, `6c3578be`,
  `72036943`, `9e67c46e`, `cc39416c`, `f0fb2eb1`.
- Порт и роутер: `fe6df715`, `0c0f9811`, `beb54cbd`, `5ff0492a`, `feeeea37`.
- Сборка: `aa032974`, `630616c3`, `d44a18e3`, `5e51d937`.

Правило из 1.36.1 (`bb35d17d`): не включать тесты, CI, Dependabot и
внутренние рефакторинги без наблюдаемого эффекта.

- [ ] Внести записи в блок `apply="unit"` в `docs/changes.xml`. Модульный блок
      (`FreeUnit updated to 1.36.2.`) не трогать.

### 2.3 Дата и перегенерация `CHANGES`

- [ ] В обоих блоках `ver="1.36.2"` поставить `date="YYYY-MM-DD"` — день
      релиза.
- [ ] Перегенерировать только секцию 1.36.2. Проверено: всё, начиная с 1.36.1,
      остаётся байт-в-байт, а заголовок получается в принятом формате.

```sh
make -C docs changes     # xmllint --valid и xsltproc → build/CHANGES; нужен пакет xsltproc
{ sed -n '1,/^Changes with Unit 1\.36\.1 /p' build/CHANGES | sed '$d' \
    | sed 's/^Changes with Unit \(1\.36\.2\)    /Changes with FreeUnit \1/'
  sed -n '/^Changes with FreeUnit 1\.36\.1 /,$p' CHANGES
} > CHANGES.new && mv CHANGES.new CHANGES
```

Коммит: `docs(changes): complete the 1.36.2 entries`.

## 3. Версия

- [ ] `version`: `NXT_VERSION=1.36.2`, `NXT_VERNUM=13602`.
- [ ] `docs/unit-openapi.yaml`, строка 3: `title: "FreeUnit 1.36.2 (ex NGINX Unit)"`.

Коммит: `1.36.2`. Проверено: после `./configure --openssl && make`
`build/sbin/unitd --version` печатает `unit version: 1.36.2`.

## 4. unitctl

- [ ] `version = "1.36.2"` в `tools/unitctl/{unitctl,unit-client-rs,unit-openapi}/Cargo.toml`.
- [ ] `cd tools/unitctl && cargo update --workspace`. Проверено: в `Cargo.lock`
      меняются только три пакета workspace.
- [ ] `openapi-config.json`: `"packageVersion": "1.36.2"`;
      `unit-openapi/README.md`: `Package version: 1.36.2`.
- [ ] `CHANGELOG.md`: секция `## [1.36.2] - YYYY-MM-DD`, ссылка `[Unreleased]`
      на `compare/unitctl/1.36.2...HEAD` и ссылка `[1.36.2]`. Содержание —
      записи про unitctl из `CHANGES`:
  - TLS вынесен в Cargo-фичу `tls`; сборка под musl идёт без TLS;
  - JSON-файл отправляется как написан;
  - JSON5 и hjson больше не читаются, YAML не читается и не пишется;
  - unitctl работает с конфигурацией, которую не удаётся декодировать как UTF-8;
  - порядок ключей конфигурации сохраняется;
  - rustls-pemfile заменён на rustls-pki-types, HTTP/2 убран
    (RUSTSEC-2025-0134, RUSTSEC-2026-0258);
  - rustls 0.23.45 (RUSTSEC-2026-0285, `8157a805`);
  - плюс `77a4528e` (JS-модуль) и `abe1d52d` (подсказка про curl).
- [ ] `RELEASE-PROCESS.md`: в примере в appendix — `1.36.2` / `13602`.

Коммит: `chore(unitctl): bump crate versions to 1.36.2`. Клиент OpenAPI (в
спецификации уже есть `start_timeout`, `execution_timeout` и `detached`)
генерируется в CI (`make openapi-generate`) и в git не коммитится.

## 5. pkg/docker

```sh
make -C pkg/docker clean && make -C pkg/docker dockerfiles   # нужен jq
```

Проверено: меняются ровно 27 файлов `Dockerfile.*`, в каждом 2 строки
(`image.version` и `git clone -b 1.36.2`). Набор вариантов из
`pkg/eol.json` тот же, что в 1.36.1. `Dockerfile.builder-*` не трогаются.

Коммит: `chore(docker): regenerate Dockerfiles for 1.36.2`.

## 6. Проверка перед merge

```sh
grep -nE 'NXT_VERSION=|NXT_VERNUM=' version
grep -m1 'Changes with FreeUnit' CHANGES
grep -m2 -A1 'ver=' docs/changes.xml       # в обоих блоках одна и та же date=
sed -n '3p' docs/unit-openapi.yaml
# вывод должен быть пустым:
grep -rn --exclude-dir=.git -I '1\.36\.1' . \
  | grep -vE '^\./(CHANGES|docs/changes\.xml|tools/unitctl/CHANGELOG\.md|PRE-1\.36\.2\.md):'
# в changelog'ах deb и rpm сверху должна стоять 1.36.2:
make -C docs ../build/unit.deb-changelog ../build/unit-php8.3.deb-changelog ../build/unit.rpm-changelog
```

- [ ] PR `pre-1.36.2` → `master`. Должны быть зелёными: build-test, sanitize,
      security-seccomp, security-syscalls, analysis-clang-ast, audit, unitctl,
      lint-whitespace, fuzzing, eol-check.
- [ ] Запустить вручную (workflow_dispatch) `build-distros.yml` (сам он идёт раз
      в неделю по расписанию) и `build-deb.yml` на этой ветке. Второй собирает
      .deb и прогоняет smoke-тесты, а публикует только по тегу.

В `freeunitx/freeunit` нет ни прогонов Actions, ни релизов: CI и публикация
идут в `freeunitorg/freeunit`.

## 7. Тег и публикация

- [ ] После merge: `git tag -a -m "FreeUnit 1.36.2" 1.36.2 <commit>`, затем
      `git push <upstream> 1.36.2`.
- Тег запускает:
  - `release-docker.yml`: образы `1.36.2-<variant>` для amd64 и arm64 в GHCR
    и Docker Hub;
  - `unitctl.yml`: релиз `unitctl/1.36.2` с бинарниками;
  - `build-deb.yml`: .deb и `SHA256SUMS` в релиз `1.36.2`.
- [ ] `make -C pkg dist` (работает только после тега) → `unit-1.36.2.tar.gz`
      и `.sha512`; выложить на freeunit.org/download.
- [ ] GitHub Release 1.36.2: текст взять из секции `CHANGES`.
- [ ] unit-docs: скопировать `CHANGES` в `source/CHANGES.txt`.

## 8. Не блокирует релиз

- `RELEASE-PROCESS.md`: шаг `mv build/CHANGES .` стоит заменить на склейку из
  §2.3.
- `pkg/docker/Makefile`: у `TODO(#milestone-1.35.8)` для цели `library`
  (Docker Hub Official Images) срок 2026-08-28 уже прошёл — перенести или
  убрать.
- `tools/unitctl/HomebrewFormula/unitctl.rb`: формула от nginxinc (версия
  0.3.0), не обновлялась с 2024 года и в процесс релиза не входит.
- EOL: на 2026-10 ни один вариант из `pkg/eol.json` не вышел за
  `supported_until`. go 1.25, node 20 и perl 5.38 помечены EOL, но ещё в
  пределах grace-периода.
