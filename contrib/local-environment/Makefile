# 你好! 作为一名长期在生产环境中使用 Make 自动化工具的高级开发人员, 我非常欣赏这份 Makefile 的设计思路.
# 它通过 Make 强大的通配符和变量替换机制, 成功地将复杂的 docker compose 命令抽象成了简洁的开发指令.
# 这种写法本质上是把 Make 当作了一个任务运行器(Task Runner), 而不是传统的编译器.

# ==============================================================================
# Docker Compose & Kubernetes 自动化管理脚本
# ==============================================================================
# 设计哲学:
# 1. 抽象化: 隐藏复杂的 -f 文件路径参数.
# 2. 一致性: 所有服务都遵循 {service}-up 和 {service}-{command} 的逻辑.
# 3. 灵活性: 通过 % 通配符支持任意 docker compose 原生子命令.
# ==============================================================================

# .PHONY 用于声明伪目标.
# 作用: 防止目录下如果恰好存在同名文件(如名为 'up' 的文件)时, 导致 Make 命令失效.
# 高级习惯: 所有不产出具体文件的指令都应该声明为 .PHONY.

# ------------------------------------------------------------------------------
# 1. 默认基础服务 (基于当前目录默认的 docker-compose.yaml)
# ------------------------------------------------------------------------------
.PHONY: up
up:
	# 启动默认配置下的所有容器. 使用 -d 以后台模式运行是生产/开发环境的通用做法.
	docker compose up -d
# $ make -n --dry-run up
# # 启动默认配置下的所有容器. 使用 -d 以后台模式运行是生产/开发环境的通用做法.
# docker compose up -d
# $

.PHONY: %
%:
	# 通配符目标. 允许执行类似 `make ps`, `make logs`, `make stop` 等指令.
	# $* 会匹配 % 所代表的内容, 并将其作为参数传递给 docker compose.
	docker compose $*
# $ make -n --dry-run ps
# # 通配符目标. 允许执行类似 `make ps`, `make logs`, `make stop` 等指令.
# # ps 会匹配 % 所代表的内容, 并将其作为参数传递给 docker compose.
# docker compose ps
# $
# $ make -n --dry-run logs
# # 通配符目标. 允许执行类似 `make ps`, `make logs`, `make stop` 等指令.
# # logs 会匹配 % 所代表的内容, 并将其作为参数传递给 docker compose.
# docker compose logs
# $
# $ make -n --dry-run stop
# # 通配符目标. 允许执行类似 `make ps`, `make logs`, `make stop` 等指令.
# # stop 会匹配 % 所代表的内容, 并将其作为参数传递给 docker compose.
# docker compose stop
# $

# ------------------------------------------------------------------------------
# 2. 复合配置服务 (使用 -f 组合多个 YAML 文件)
# ------------------------------------------------------------------------------
# 这里的重点在于"配置叠加". docker compose -f base.yaml -f override.yaml 会合并配置.

# Alpha Config 服务
.PHONY: alpha-config-up
alpha-config-up:
	docker compose -f docker-compose.yaml -f docker-compose-alpha-config.yaml up -d
# $ make -n --dry-run alpha-config-up
# docker compose -f docker-compose.yaml -f docker-compose-alpha-config.yaml up -d
# $

.PHONY: alpha-config-%
alpha-config-%:
	# 允许对该特定组合执行任意指令. 例如: make alpha-config-ps
	docker compose -f docker-compose.yaml -f docker-compose-alpha-config.yaml $*
# $ make -n --dry-run alpha-config-ps
# # 允许对该特定组合执行任意指令. 例如: make alpha-config-ps
# docker compose -f docker-compose.yaml -f docker-compose-alpha-config.yaml ps
# $

# Nginx 服务 (基础配置 + Nginx 特定配置)
.PHONY: nginx-up
nginx-up:
	docker compose -f docker-compose.yaml -f docker-compose-nginx.yaml up -d

.PHONY: nginx-%
nginx-%:
	docker compose -f docker-compose.yaml -f docker-compose-nginx.yaml $*

# ------------------------------------------------------------------------------
# 3. 独立组件服务 (单文件模式)
# ------------------------------------------------------------------------------
# 适用于不需要与基础 docker-compose.yaml 共享网络或卷的独立中间件.

# Keycloak 身份认证服务
.PHONY: keycloak-up
keycloak-up:
	docker compose -f docker-compose-keycloak.yaml up -d

.PHONY: keycloak-%
keycloak-%:
	docker compose -f docker-compose-keycloak.yaml $*

.PHONY: gitea-up
gitea-up:
	docker compose -f docker-compose-gitea.yaml up -d

.PHONY: gitea-%
gitea-%:
	docker compose -f docker-compose-gitea.yaml $*

# ------------------------------------------------------------------------------
# 4. Kubernetes 集群管理
# ------------------------------------------------------------------------------
# 这里体现了 Makefile 的递归调用能力.
.PHONY: kubernetes-up
kubernetes-up:
	# -C 参数表示切换到指定目录执行那里的 Makefile.
	# 这种分层设计(Recursive Make)能有效降低根目录 Makefile 的复杂度.
	make -C kubernetes create-cluster
	make -C kubernetes deploy
# $ make -n --dry-run kubernetes-up
# # -C 参数表示切换到指定目录执行那里的 Makefile.
# # 这种分层设计(Recursive Make)能有效降低根目录 Makefile 的复杂度.
# make -C kubernetes create-cluster
# make -C kubernetes deploy
# $

.PHONY: kubernetes-down
kubernetes-down:
	make -C kubernetes delete-cluster

.PHONY: traefik-up
traefik-up:
	docker compose -f docker-compose.yaml -f docker-compose-traefik.yaml up -d

.PHONY: traefik-%
traefik-%:
	docker compose -f docker-compose.yaml -f docker-compose-traefik.yaml $*
