# Init dự án mới sử dụng poetry

### Project setup  

Tạo mới project sử dụng lênh sau
```
$ poetry new poetry-demo
```

Project mới sẽ có cấu trúc như sau
```
poetry-demo
├── pyproject.toml
├── README.md
├── poetry_demo
│   └── __init__.py
└── tests
    └── __init__.py
```

File `pyproject.toml` chứa các phần phụ thuộc của dự án
```
[tool.poetry]
name = "poetry-demo"
version = "0.1.0"
description = ""
authors = ["Sébastien Eustace <sebastien@eustace.io>"]
readme = "README.md"
packages = [{include = "poetry_demo"}]

[tool.poetry.dependencies]
python = "^3.7"


[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

**Setting a Python Version**    
Setting version python phù hợp trong file `pyproject.toml`. Lệnh dưới đây yêu cầu dự án chạy với python > 3.7.0
```
[tool.poetry.dependencies]
python = "^3.7.0"
```

**Initialising a pre-existing project**    
Đối với những dự án có sẵn, init poetry bằng lệnh sau
```
cd pre-existing-project
poetry init
```

**Specifying dependencies**  
Thêm dependencies bằng lệnh sau
```
$ poetry add pendulum
```
hoặc thêm trực tiếp vào `tool.poetry.dependencies` trong file `pyproject.toml`
```
[tool.poetry.dependencies]
pendulum = "^2.1"
```

### Using your virtual environment

**Activating the virtual environment**  
Sử dụng lệnh sau để activate virtual environment
```
$ poetry shell
```

hoặc activate trực tiếp virtual environment có sẵn, sử dụng lệnh sau để liệt kê danh sách các env có sẵn và activate env
```
$ poetry env info --path
$ source {path_to_venv}/bin/activate
```

**Installing dependencies**  
Sử dụng lệnh install dependency từ file lock
```
$ poetry install
```

Để install dependency only, sử dụng lệnh sau
```
$ poetry install --no-root
```
