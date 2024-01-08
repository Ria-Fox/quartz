# 0. Ruff?

Rust로 작성된 Python linter 및 code formatter로, 굉장히 빠르다는 장점이 있음

![https://user-images.githubusercontent.com/1309177/232603514-c95e9b0f-6b31-43de-9a80-9e844173fd6a.svg](https://user-images.githubusercontent.com/1309177/232603514-c95e9b0f-6b31-43de-9a80-9e844173fd6a.svg)

  

Ruff의 장점으로 꼽히는 점:

  

- pip으로 설치

- pyproject.toml 지원

- Python 3.12까지 지원

- Flake8, isort, Black을 대체 가능

- Built-in 캐시(수정되지 않은 파일 재분석 방지)

- 자동 에러 수정 기능 지원

- Monorepo-friendly

- Et cetera

  

또한 Ruff는 다음과 같은 메이저한 프로젝트들에 대해서 많이 사용된다고 함

  

- Apache Airflow

- FastAPI

- Hugging Face(NLP 리서치 라이브러리)

- Pandas

- SciPy

  

심지어 다양한 유명 개발자들의 간증(?) 포함
![[Untitled.png]]
  

# 1. Getting Started

  

## 1.1. 설치

  

```bash
pip install ruff
brew install ruff
conda install ruff
```

  

## 1.2. Usage

  

### 1.2.1. 기본 사용법
```bash
# As a linter
ruff check . # 현재 디렉토리 및 하위 파일들에 대해 린팅
ruff check path/to/code/ # `/path/to/code` 안의 모든 하위 파일들에 대해 린팅
ruff check path/to/code/*.py # `/path/to/code` 안의 모든 `.py` 파일들에 대해 린팅
ruff check path/to/code/to/file.py # `file.py`에 대해 린팅
ruff check @arguments.txt # 입력한 모든 파일들에 대해 린팅

# As a formatter
ruff format . # 현재 디렉토리 및 하위 파일들 리포맷팅
ruff format path/to/code/ # `/path/to/code` 안의 모든 하위 파일들에 대해 리포맷팅
ruff format path/to/code/*.py # `/path/to/code` 안의 `.py` 파일들에 대해 리포맷팅
ruff format path/to/code/to/file.py # `file.py` 리포맷팅
ruff format @arguments.txt # 입력한 모든 파일들에 대해 리포맷팅
```

  

### 1.2.2. With pre-commit

  

```yaml
. - repo: https://github.com/astral-sh/ruff-pre-commit
	# Ruff version.
	rev: v0.1.5
	hooks:
	  # Run the linter.
	  - id: ruff
	    args: [ --fix ]
	  # Run the formatter.
	  - id: ruff-format
```

  

### 1.2.3. With VS Code extension
![[Untitled 1.png]]
### 1.2.4. With GitHub Action via `ruff-action`

  

```yaml
name: Ruff
on: [ push, pull_request ]
jobs:
	ruff:
		runs-on: ubuntu-latest
		steps:
			- uses: actions/checkout@v3
			- uses: chartboost/ruff-action@v1
```

  

# 2. Configuration

Ruff의 설정은 `pyproject.toml`, `ruff.toml`, `.ruff.toml` 파일로 관리할 수 있습니다.

자세한 설정은 [여기](https://docs.astral.sh/ruff/configuration/)를 참고해 주세요.

별다른 설정이 없는 경우 다음 디폴트 설정을 따릅니다.


```toml
[tool.ruff]
# Exclude a variety of commonly ignored directories.
exclude = [
    ".bzr",
    ".direnv",
    ".eggs",
    ".git",
    ".git-rewrite",
    ".hg",
    ".mypy_cache",
    ".nox",
    ".pants.d",
    ".pytype",
    ".ruff_cache",
    ".svn",
    ".tox",
    ".venv",
    "__pypackages__",
    "_build",
    "buck-out",
    "build",
    "dist",
    "node_modules",
    "venv",
]

# Same as Black.
line-length = 88
indent-width = 4

# Assume Python 3.8
target-version = "py38"

[tool.ruff.lint]
# Enable Pyflakes (`F`) and a subset of the pycodestyle (`E`)  codes by default.
# Unlike Flake8, Ruff doesn't enable pycodestyle warnings (`W`) or
# McCabe complexity (`C901`) by default.
select = ["E4", "E7", "E9", "F"]
ignore = []

# Allow fix for all enabled rules (when `--fix`) is provided.
fixable = ["ALL"]
unfixable = []

# Allow unused variables when underscore-prefixed.
dummy-variable-rgx = "^(_+|(_+[a-zA-Z0-9_]*[a-zA-Z0-9]+?))$"

[tool.ruff.format]
# Like Black, use double quotes for strings.
quote-style = "double"

# Like Black, indent with spaces, rather than tabs.
indent-style = "space"

# Like Black, respect magic trailing commas.
skip-magic-trailing-comma = false

# Like Black, automatically detect the appropriate line ending.
line-ending = "auto"

```

  

몇몇 설정은 커맨드라인으로 같이 줄 수 있으며 rule, 파일 관련 설정, 로깅 레벨 등의 옵션이 그렇습니다.

  

```bash
ruff check path/to/code/ --select F401 --select F403 --quiet
```

  

# 3. 직접 사용해보기

  

## 3.1. pyproject.toml 수정

  

Ruff를 사용해서 기존 `buzzni-ai-backend`의 `pylint`, `isort`, `black`을 대체하도록 함

  

- 기존 `pyproject.toml`

  

```toml
[tool.poetry.dev-dependencies]
pytest = "^7.0.1"
isort = "^5.10.1"
black = "^22.1.0"
pylint = "^2.12.2"
pytest-asyncio = "^0.18.1"
httpx = "^0.22.0"
locust = "^2.8.3"
pylint-mongoengine = "^0.4.0"
pre-commit = "^2.20.0"
pytest-postgresql = "^3.1.3"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"

[tool.black]
line-length = 120
exclude = '''
/(
    \.git
  | \.mypy_cache
  | \.tox
  | venv
  | \.venv
  | _build
  | buck-out
  | build
  | dist
)/
'''

[tool.isort]
line_length = 120
profile = "black"
```

  

- pylint, isort, black을 제거하고 ruff 관련 설정을 추가한 pyproject.toml

  

```toml
[tool.poetry.dev-dependencies]
pytest = "^7.0.1"
pytest-asyncio = "^0.18.1"
httpx = "^0.22.0"
locust = "^2.8.3"
pre-commit = "^2.20.0"
pytest-postgresql = "^3.1.3"
ruff = "^0.1.5"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"

[tool.ruff]
line-length = 120

[tool.ruff.lint]
ignore = ["F403"]

[tool.ruff.format]
exclude = [".git", ".mypy_cache", ".tox", "venv", ".venv", "_build", "buck-out", "build", "dist"]
```

  

변경점 :
- `[tool.black]`, `[tool.isort]` → `[tool.ruff]`, `[tool.ruff.format]`
- F403 에러 무시하도록 변경(기존에는 pylint disable 주석으로 각 파일의 맨 위에 작성됨)

| F403 | https://docs.astral.sh/ruff/rules/undefined-local-with-import-star/ | from {name} import * used; unable to detect undefined names |
| ---- | ------------------------------------------------------------------- | ----------------------------------------------------------- |

- exclude 형식 변경 → `List` 형식
- 하지만 exclude하는 것들이 모두 default 설정에 있는 거라서 지워도 될 거라고 판단 중

## 3.2. .pre-commit-config.yaml 수정
- 기존 pre-commit-config
```yaml
-   repo: local
    hooks:
    -   id: no-push-to-branch
        name: "🚨🚓🚨 master/develop 브랜치에는 푸시할 수 없습니다"
        entry: ./scripts/pre_commit_hooks/no_push_to_branch.py
        language: system
        stages: [push]
        require_serial: true
    -   id: black
        name: black
        entry: poetry run black
        language: system
        types: [python]
    -   id: isort
        name: isort
        entry: poetry run isort
        language: system
        types: [python]
    -   id: pylint
        name: pylint
        entry: poetry run pylint -sn
        language: system
        types: [python]
```

- 수정한 pre-commit-config
```yaml
    - repo: local
      hooks:
          - id: no-push-to-branch
            name: "🚨🚓🚨 master/develop 브랜치에는 푸시할 수 없습니다"
            entry: ./scripts/pre_commit_hooks/no_push_to_branch.py
            language: system
            stages: [push]
            require_serial: true
          - id: ruff-format
            name: ruff-format
            entry: ruff format .
            language: system
            types: [python]
          - id: ruff-lint
            name: ruff-lint
            entry: ruff check . --fix
            language: system
            types: [python]
```

  

## 3.3. 변경점

  

- 속도: 약 7초 → 0.1초(바로 나옴)
- 작은 시간이지만 기다리다 보면 금방 흐름이 깨짐
- Lint: 좀 더 깐깐해짐
- pylint disable을 주석으로 일일히 설정한 경우 Ruff로 옮기는 데 좀 귀찮을 수 있음
```yaml
sources/api/data_validation/repositories.py:213:70: E721 Do not compare types, use `isinstance()`
sources/api/recommend/middleware.py:24:26: E711 Comparison to `None` should be `cond is not None`
sources/common/connections/cassandra.py:88:9: E722 Do not use bare `except`
sources/common/connections/kafka.py:192:9: E722 Do not use bare `except`
sources/common/exceptions.py:31:1: E722 Do not use bare `except`
sources/common/models/data.py:4:1: F403 `from sources.common.connections.cassandra import *` used; unable to detect undefined names
sources/common/repositories/brand.py:248:85: E712 Comparison to `True` should be `cond is True` or `if cond:`
sources/common/repositories/brand.py:251:50: E712 Comparison to `False` should be `cond is False` or `if not cond:`
sources/common/repositories/category.py:67:5: F601 Dictionary key literal `"홈웨어/이지웨어/잠옷"` repeated
sources/common/repositories/storage.py:24:9: E722 Do not use bare `except`
sources/common/repositories/storage.py:37:17: E722 Do not use bare `except`
sources/common/repositories/storage.py:227:9: E722 Do not use bare `except`
sources/common/repositories/storage.py:288:13: E722 Do not use bare `except`
sources/common/utils/logger.py:11:1: E722 Do not use bare `except`
sources/job/broadcast_product_syncer.py:203:13: E722 Do not use bare `except`
sources/job/live_commerce_category_updater.py:36:103: E711 Comparison to `None` should be `cond is None`
sources/job/tvshop_catalog_builder.py:226:9: E722 Do not use bare `except`
sources/worker/broadcast_product_category_consumer.py:133:17: F841 Local variable `timestamp` is assigned to but never used
sources/worker/broadcast_product_category_consumer.py:133:17: F841 Local variable `timestamp` is assigned to but never used
sources/worker/hsmoa_log_consumer.py:97:13: E722 Do not use bare `except`
tests/end-to-end/test_reco_end2end.py:63:12: E721 Do not compare types, use `isinstance()`
tests/end-to-end/test_reco_end2end.py:83:12: E721 Do not compare types, use `isinstance()`
tests/end-to-end/test_reco_end2end.py:101:12: E721 Do not compare types, use `isinstance()`
tests/end-to-end/test_reco_end2end.py:138:12: E721 Do not compare types, use `isinstance()`
tests/end-to-end/test_reco_end2end.py:167:12: E721 Do not compare types, use `isinstance()`
tests/end-to-end/test_reco_end2end.py:200:9: F841 Local variable `res` is assigned to but never used
tests/end-to-end/test_reco_end2end.py:259:12: E721 Do not compare types, use `isinstance()`
tests/end-to-end/test_reco_end2end.py:280:12: E721 Do not compare types, use `isinstance()`
tests/unit/worker/test_broadcast_product_consumer.py:49:16: E721 Do not compare types, use `isinstance()`
tests/unit/worker/test_broadcast_product_consumer.py:50:16: E721 Do not compare types, use `isinstance()`
tests/unit/worker/test_broadcast_product_consumer.py:129:5: F841 Local variable `thumbnail_url` is assigned to but never used
tests/unit/worker/test_broadcast_product_consumer.py:263:5: F841 Local variable `thumbnail_url` is assigned to but never used
tests/unit/worker/test_broadcast_product_consumer.py:396:5: F841 Local variable `thumbnail_url` is assigned to but never used
Found 107 errors (74 fixed, 33 remaining).
No fixes available (10 hidden fixes can be enabled with the `--unsafe-fixes` option).
```
- Flake8을 사용중이었다면 좀 더 빠르고 쉽게 옮길 수 있겠지만, pylint는 일일히 고쳐줘야 될 부분이 있음