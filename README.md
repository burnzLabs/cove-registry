# Cove Registry

> **[Cove](https://github.com/burnzLabs)** 의 패키지 매니페스트 레지스트리 — 지원 패키지의 **버전과 다운로드 방법**을 정의합니다.

Cove는 이 저장소를 **단일 진실원(source of truth)** 으로 삼아 각 패키지의 설치 가능한 **버전 목록**과 다운로드/실행 방법을 가져옵니다. 버전 매니페스트만 여기에 올리면 **앱 재배포 없이** 모든 사용자에게 반영됩니다.

---

## ⚠️ 지원 범위 (먼저 읽어주세요)

- Cove가 지원하는 패키지는 **아래 목록으로 고정**되어 있습니다. 설치·서비스 기동/종료·버전 전환·DB 설정·PHP 확장 같은 동작은 **Cove에 내장된 로직**이 처리하며, **이 패키지들만** 대상으로 합니다.
- **목록에 없는 새 패키지를 `*.toml` 로 추가해도 "지원"되지 않습니다.** (패키지 탭에 보이더라도 서비스/런타임/설정 처리가 연결돼 있지 않아 제대로 동작하지 않습니다.)
- 이 저장소로 할 수 있는 일은 **기존 지원 패키지의 새 버전 추가**(또는 기존 버전의 `url`/`hash`/`version` 수정)뿐입니다.
- 새 패키지 자체를 추가하려면 **Cove 앱 쪽 코드** 변경이 필요합니다(이 저장소만으로는 불가).

**지원 패키지**

| 분류 | 패키지 |
|------|--------|
| 런타임 | `php`, `nodejs`, `python`, `go`, `java` |
| 데이터베이스 | `postgresql`, `mariadb`, `mysql`, `mongodb` |
| 웹서버 | `caddy` |
| 서비스 | `redis`, `mailpit`, `meilisearch`, `minio` |
| 도구 | `composer`, `mkcert`, `pnpm`, `git`, `uv` |

---

## 어떻게 동작하나

```
cove-registry (이 저장소)
        │  main 브랜치 zip (archive/refs/heads/main.zip)
        ▼
Cove 앱  ──►  <cove_home>/registry/  ──►  설치/버전 전환
```

- **첫 실행**: Cove가 자동으로 1회 받아 옵니다(온라인).
- **이후**: 패키지 탭의 **"패키지 목록 업데이트"** 로 수동 갱신.
- Cove는 `*.toml` 을 **재귀로 모두 읽고**, 같은 `name` 의 파일들을 **한 패키지의 여러 버전**으로 묶습니다.

---

## 구조

```
runtimes/        # 언어 런타임 (php, nodejs, python, go, java)
databases/       # 데이터베이스 (postgresql, mariadb, mysql, mongodb)
webservers/      # 웹서버 (caddy)
services/        # 서비스 (redis, mailpit, meilisearch, minio)
tools/           # 도구 (composer, mkcert, pnpm, git, uv)
php-extensions/  # PHP 마이너 버전별 PECL 확장 핀 (아래 참고)
```

각 패키지는 **버전별 파일**로 둡니다 — 예: `runtimes/php/php-8.4.toml`, `databases/mysql/mysql-9.toml`.
같은 패키지의 새 버전은 같은 폴더에 파일을 하나 더 추가합니다(예: `runtimes/go/go-1.27.toml`).

---

## 매니페스트 형식 (TOML)

```toml
name = "php"            # 패키지 이름 (지원 목록의 이름과 정확히 일치해야 함)
version = "8.4.22"      # 버전
type = "runtime"        # runtime | database | webserver | service | tool

homepage = "https://www.php.net"
license = "PHP-3.01"

bin = ["php.exe", "php-cgi.exe"]   # PATH에 노출할 실행파일(설치 폴더 기준 상대경로)
env_add_path = ["."]               # PATH에 추가할 하위 경로
# persist = ["data"]               # (선택) 버전 업데이트 시 보존할 파일/폴더

[architecture.x64]
url  = "https://windows.php.net/downloads/releases/php-8.4.22-nts-Win32-vs17-x64.zip"
hash = "sha256:..."                # 다운로드 무결성 검증(없으면 검증 생략)
```

서비스/데이터베이스는 `[service]` 블록으로 포트 등을 정의합니다:

```toml
[service]
name = "cove-redis"
port = 6379
```

지원 형식: `.zip`, `.tar.gz`, 단일 `.exe`, `.phar`. 단일 루트 폴더는 자동으로 벗겨집니다.

> `name` 은 **지원 패키지 목록의 이름과 정확히 같아야** Cove가 해당 패키지의 버전으로 인식합니다.

---

## PHP 확장 (`php-extensions/`)

PHP의 PECL 확장은 **PHP 마이너 버전마다** 다운로드 경로가 다릅니다(ABI: `nts/vsXX/x64`). 그래서 매니페스트가 아니라 별도 폴더에 **PHP 마이너 버전별 핀 파일**로 둡니다.

```
php-extensions/php-8.3.toml
php-extensions/php-8.4.toml
php-extensions/php-8.5.toml
```

각 파일은 그 PHP 버전에 맞는 확장 DLL을 `url` + `hash` 로 고정합니다:

```toml
[[extension]]
name = "redis"
url  = "https://downloads.php.net/~windows/pecl/releases/redis/6.3.0/php_redis-6.3.0-8.4-nts-vs17-x64.zip"
hash = "sha256:..."

[[extension]]
name = "xdebug"
zend = true            # Zend 확장(zend_extension=)으로 로드
url  = "https://downloads.php.net/~windows/pecl/releases/xdebug/3.5.1/php_xdebug-3.5.1-8.4-nts-vs17-x64.zip"
hash = "sha256:..."
```

- 환경 탭의 각 PHP 버전에서 해당 `php-<major>.<minor>.toml` 의 확장 목록을 읽어 설치/활성화합니다.
- `url` 의 ABI(`-8.4-nts-vs17-x64`)는 **그 PHP 버전과 정확히 맞아야** 합니다.
- 이 파일들은 Cove 저장소의 `scripts/gen-php-extensions.mjs` 로 생성/갱신합니다(PHP 매니페스트의 ABI를 읽어 최신 DLL을 해결).

---

## 새 버전 추가 (지원 패키지에 한함)

1. 해당 패키지 폴더에 `<패키지>-<버전>.toml` 을 추가합니다(또는 기존 파일의 `version`/`url` 수정).
2. `hash` 는 비워두면 검증 생략. 채우려면 Cove 저장소의 `scripts/fill-hashes.mjs` 로 자동 계산.
3. 상류 최신 버전 자동 확인·반영은 Cove 저장소의 `scripts/update-registry.mjs`.
4. `main` 에 push → 사용자는 "패키지 목록 업데이트" 로 받습니다.

> `version = "latest"`(롤링) 항목은 해시를 고정하지 않습니다(상류 갱신 시 깨짐 방지).
> 지원 목록에 **없는 새 패키지**를 추가하는 것은 이 저장소만으로는 불가합니다(앱 코드 변경 필요).

---

## 라이선스

각 매니페스트가 가리키는 소프트웨어의 라이선스는 매니페스트의 `license` 필드를 따릅니다. 이 저장소(매니페스트 메타데이터) 자체는 자유롭게 사용하세요.
