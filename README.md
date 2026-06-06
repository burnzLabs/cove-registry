# Cove Registry

> **[Cove](https://github.com/burnzLabs)** 의 패키지 매니페스트 레지스트리 — Windows 개발 도구·런타임·데이터베이스를 한 곳에서 정의합니다.

Cove는 이 저장소를 **단일 진실원(source of truth)** 으로 삼아, 설치 가능한 패키지 목록과 다운로드/실행 방법을 가져옵니다. 매니페스트만 여기에 올리면 **앱 재배포 없이** 모든 사용자에게 반영됩니다.

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
- 디렉터리 구조는 자유 — Cove는 `*.toml` 을 **재귀로 모두 읽고**, 분류는 파일 위치가 아니라 매니페스트의 `type` 으로 정합니다.

---

## 구조

```
runtimes/     # 언어 런타임 (php, nodejs, python, go, java)
databases/    # 데이터베이스 (postgresql, mariadb, mysql, mongodb)
webservers/   # 웹서버 (caddy)
services/     # 서비스 (redis, mailpit, meilisearch, minio)
tools/        # 도구 (composer, mkcert, pnpm, git, uv)
```

각 패키지는 버전별 파일로 둡니다 — 예: `runtimes/php/php-8.4.toml`, `databases/mysql/mysql-9.toml`.

---

## 매니페스트 형식 (TOML)

```toml
name = "php"            # 패키지 이름 (같은 이름 = 같은 패키지의 버전들)
version = "8.4.22"      # 버전
type = "runtime"        # runtime | database | webserver | service | tool

homepage = "https://www.php.net"
license = "PHP-3.01"

bin = ["php.exe", "php-cgi.exe"]   # PATH에 노출할 실행파일(설치 폴더 기준 상대경로)
env_add_path = ["."]               # PATH에 추가할 하위 경로
persist = ["php.ini"]              # 버전 업데이트 시 보존할 파일

[architecture.x64]
url  = "https://windows.php.net/downloads/releases/php-8.4.22-nts-Win32-vs17-x64.zip"
hash = "sha256:..."                # 다운로드 무결성 검증(없으면 검증 생략)
```

서비스/데이터베이스는 `[service]` 블록으로 포트·기동/종료를 정의합니다:

```toml
[service]
name  = "cove-redis"
port  = 6379
```

지원 형식: `.zip`, `.tar.gz`, 단일 `.exe`, `.phar`. 단일 루트 폴더는 자동으로 벗겨집니다.

---

## 기여 / 새 버전 추가

1. 해당 카테고리 폴더에 `<패키지>/<패키지>-<버전>.toml` 추가(또는 기존 파일의 `version`/`url` 수정).
2. `hash` 는 비워두면 검증 생략. 채우려면 Cove 저장소의 `scripts/fill-hashes.mjs` 로 자동 계산.
3. `main` 에 push → 사용자는 "패키지 목록 업데이트" 로 받습니다.

> `version = "latest"`(롤링) 항목은 해시를 고정하지 않습니다(상류 갱신 시 깨짐 방지).

---

## 라이선스

각 매니페스트가 가리키는 소프트웨어의 라이선스는 매니페스트의 `license` 필드를 따릅니다. 이 저장소(매니페스트 메타데이터) 자체는 자유롭게 사용하세요.
