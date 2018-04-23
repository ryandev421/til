---
layout: post
title: Makefile에서 help target 추가하는 법
date: 2018-04-24
excerpt: ""
tags: [Makefile, make]
comments: true
category: development
---

## 1. Makefile에 help 타겟 추가

```
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

```

## 2. help 에 노출하고 싶은 곳에 설명추가

`##`으로 해당 타겟에 대한 설명추가

```
prod-deploy:    ## Deploy Production
	dep deploy production -v -p

```

## 완료된 파일

- Makefile

```
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

prod-deploy:    ## Deploy Production
	dep deploy production -v -p

prod-rollback:  ## Rollback Production
	dep rollback production -v -p

prod-unlock:    ## Unlock Production
	dep deploy:unlock production -v

```

- 결과

```
ryan@macbook:~/deployer$ make help
prod-deploy                    Deploy Production
prod-rollback                  Rollback Production
prod-unlock                    Unlock Production
```
