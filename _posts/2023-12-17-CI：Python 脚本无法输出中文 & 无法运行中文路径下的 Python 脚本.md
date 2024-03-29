---
layout: post
title: "CI：Python 脚本无法输出中文 & 无法运行中文路径下的 Python 脚本"
date: 2023-12-17-17:49:28 +0800
categories: CI Python
---

## Python 脚本输出中文为乱码

### Reproduce

test.py

```
print("Test - 中文测试")
```

.gitlab-ci.yml

```
stages:
    - test

test_jog:
    stage: test
    tags:
        - your_runner_tag
    script:
        - echo 中文输出测试
        - python --version
        - python test.py
```

output

![Chinese Garbled](https://raw.githubusercontent.com/zhous1028/zhous1028.github.io/main/imgs/2023-12-17/ci_chinese_garbled.PNG)

### Fix

#### Check console's encoding

test.py

```
import sys
print(sys.stdin.encoding)
print(sys.stdout.encoding)
print("Test - 中文测试")
```

output

![console_encoding](https://raw.githubusercontent.com/zhous1028/zhous1028.github.io/main/imgs/2023-12-17/console_encoding.PNG)

#### 设置Runner服务器环境变量

Environment Variables -> System variables -> New ->  PYTHONIOENCODING = UTF-8

![pythonencoding](https://raw.githubusercontent.com/zhous1028/zhous1028.github.io/main/imgs/2023-12-17/pythonencoding.PNG)

#### 重启 Runner

```
gitlab-runner.exe restart
```

output

![output_utf8](https://raw.githubusercontent.com/zhous1028/zhous1028.github.io/main/imgs/2023-12-17/output_utf8.PNG)



## 无法运行中文路径下的 Python 脚本

### Reproduce

Test_中文路径/test.py

```
print("测试运行中文路径下的Python脚本")
```

.gitlab-ci.yml

```
stages:
    - test

test_jog:
    stage: test
    tags:
        - your_runner_tag
    script:
        - echo 测试中文路径
        - python --version
        - python Test_中文路径/test.py
```

output

![chinese_path_err](https://raw.githubusercontent.com/zhous1028/zhous1028.github.io/main/imgs/2023-12-17/chinese_path_err.PNG)

### Fix

Add before "stages:" command "- CHCP 65001" in "before_script:"

.gitlab-ci.yml

```
before_script:
    - chcp 65001

stages:
    - test

test_jog:
    stage: test
    tags:
        - your_runner_tag
    script:
        - echo 测试中文路径
        - python --version
        - python Test_中文路径/test.py
```

output

![chcp_65001](https://raw.githubusercontent.com/zhous1028/zhous1028.github.io/main/imgs/2023-12-17/65001.PNG)
