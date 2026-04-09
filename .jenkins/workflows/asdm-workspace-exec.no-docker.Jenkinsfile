  // ASDM Workspace 执行流水线：clone → asdm workspace install → CodeBuddy（支持 /slash-command）
  // 行为对齐 deploy/jenkins/ASDM_WORKSPACE_EXECUTION_PLAN.md
  //
  // Host-agent variant（no Docker / no K8s）
  // Kubernetes-agent variant: asdm-workspace-exec.k8s.Jenkinsfile
  //
  // 前置：
  // 1) Jenkins 节点需预装 codebuddy、codebuddy-log-parser、asdm
  // 2) 可选 Jenkins Secret text：codebuddy-api-key、codebuddy-base-url
  // 3) GIT_CLONE_TOKEN：通过构建参数传入（私有库）；勿将 token 写入仓库
  // 4) E2E_MODE=install-only 时跳过 CodeBuddy，用于廉价端到端验证（clone + install）

  pipeline {
    agent any

    environment {
      LC_ALL = 'C.UTF-8'
      LANG = 'C.UTF-8'
      JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8'
      CODEBUDDY_INTERNET_ENVIRONMENT = 'internal'
    }

    options {
      timestamps()
    }

    parameters {
      string(
        name: 'REPO_URL',
        defaultValue: '',
        description: '目标仓库 Git HTTPS URL（必填；私有库需填 GIT_CLONE_TOKEN）'
      )
      string(name: 'BRANCH', defaultValue: 'main', description: '分支；会 fallback main/master')
      string(
        name: 'TARGET_BRANCH',
        defaultValue: '',
        description: 'PR 目标 head 分支名（可选；为空则自动生成 asdm-e2e-${BUILD_NUMBER}）'
      )
      string(
        name: 'WORKSPACE_ID',
        defaultValue: '',
        description: 'ASDM workspace id（UI 示例：…/workspaces/12）'
      )
      text(
        name: 'PROMPT_CONTENT',
        defaultValue: '',
        description: '传给 codebuddy -p（自然语言）；若以 / 开头则按 workspace 已安装的 slash command 解析'
      )
      string(name: 'GIT_CLONE_TOKEN', defaultValue: '', description: '克隆私有库用的 PAT；公有库留空')
      string(
        name: 'ASDM_BASE_URL',
        defaultValue: 'https://platform-sit01.asdm.ai',
        description: 'ASDM 平台根地址；会写入 asdm config 并 export BASE_URL 供 CLI 使用（no-docker 默认本机 127.0.0.1）'
      )
      string(
        name: 'ASDM_ARTIFACT_BASE_URL',
        defaultValue: 'https://platform-sit01.asdm.ai/_artifacts',
        description: '制品基地址；会写入 asdm config 并 export ARTIFACT_BASE_URL 供 CLI 下载资源'
      )
      password(
        name: 'ASDM_API_TOKEN',
        defaultValue: '',
        description: 'ASDM 平台 API token（可留空；优先使用 Jenkins 凭据：Secret text，id=asdm-api-token）'
      )
      choice(
        name: 'E2E_MODE',
        choices: ['full', 'install-only'],
        description: 'full=完整执行 CodeBuddy；install-only=仅验证 clone + workspace install（不调 CodeBuddy）'
      )
      choice(
        name: 'EXECUTION_MODE',
        choices: ['prompt', 'auto', 'slash-command'],
        description: '默认 prompt=按自然语言处理；auto 根据是否以 / 开头判断'
      )
      password(
        name: 'CODEBUDDY_API_KEY_PARAM',
        defaultValue: '',
        description: 'CodeBuddy API Key（留空则用 Jenkins 凭据 id=codebuddy-api-key）'
      )
    }

    stages {
      stage('Verify agent toolchain') {
        steps {
          sh '''bash -s <<'ASDM_SCRIPT'
            set -euo pipefail
            set -x
            echo "=== Workspace exec / host agent 自检 ==="
            node --version
            npm --version
            git --version
            python3 --version
            codebuddy --version
            command -v asdm >/dev/null 2>&1 || { echo "缺少 asdm 命令：请在 Jenkins 节点安装 asdm CLI" >&2; exit 1; }
            asdm --version
ASDM_SCRIPT
          '''
        }
      }

      stage('Build log parser') {
        steps {
          script {
            def skip = sh(script: '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              if [ -f /opt/codebuddy-log-parser/dist/index.js ]; then exit 0; fi
              if [ -n "${CODEBUDDY_LOG_PARSER:-}" ] && [ -f "${CODEBUDDY_LOG_PARSER}" ]; then exit 0; fi
              exit 1
ASDM_SCRIPT
            ''', returnStatus: true) == 0
            if (skip) {
              echo '使用镜像内/节点内预装的 codebuddy-log-parser，跳过本阶段'
              return
            }
            if (!fileExists('tools/codebuddy-log-parser/package.json')) {
              error('缺少 tools/codebuddy-log-parser，且未预装 /opt/codebuddy-log-parser')
            }
            dir('tools/codebuddy-log-parser') {
              sh 'bash -euo pipefail -c "npm install && npm run build"'
            }
          }
        }
      }

      stage('Clone target repository') {
        steps {
          script {
            if (!params.REPO_URL?.trim()) {
              error('REPO_URL is required')
            }
            if (!params.WORKSPACE_ID?.trim()) {
              error('WORKSPACE_ID is required')
            }
          }
          withEnv([
            "REPO_URL=${params.REPO_URL.trim()}",
            "BRANCH=${params.BRANCH.trim()}",
            "GIT_CLONE_TOKEN=${params.GIT_CLONE_TOKEN}"
          ]) {
            sh '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              CLONE_URL=$(REPO_URL="$REPO_URL" GIT_CLONE_TOKEN="${GIT_CLONE_TOKEN:-}" \
                python3 << 'PY'
import os
from urllib.parse import urlparse, urlunparse, quote

raw = os.environ.get("REPO_URL", "").strip()
token = (os.environ.get("GIT_CLONE_TOKEN") or "").strip()

if not raw:
  raise SystemExit("REPO_URL is empty")

if "://" not in raw:
  raw = "https://" + raw

p = urlparse(raw)
if p.scheme not in ("http", "https"):
  raise SystemExit("Only http(s) repository URLs are supported")

host = (p.hostname or "").lower()
hostport = f"{p.hostname}:{p.port}" if p.port else (p.hostname or "")
path = p.path.rstrip("/") or "/"

if token:
  if "github.com" in host:
      netloc = f"x-access-token:{quote(token, safe='')}@{hostport}"
  elif "gitlab" in host:
      netloc = f"oauth2:{quote(token, safe='')}@{hostport}"
  else:
      netloc = f"oauth2:{quote(token, safe='')}@{hostport}"
else:
  netloc = p.netloc

if not path.endswith(".git"):
  path = path + ".git"

print(urlunparse((p.scheme, netloc, path, "", "", "")))
PY
              )

              REPO_NAME=$(echo "$REPO_URL" | sed 's/.*\\///' | sed 's/\\.git$//')
              {
                printf 'export REPO_URL=%q\n' "$REPO_URL"
                printf 'export BRANCH=%q\n' "$BRANCH"
                printf 'export REPO_NAME=%q\n' "$REPO_NAME"
                printf 'export CLONE_URL=%q\n' "$CLONE_URL"
              } > "${WORKSPACE}/.asdm_clone_env"

              try_clone() {
                local b="$1"
                rm -rf "$REPO_NAME"
                if git clone --branch "$b" --single-branch "$CLONE_URL" "$REPO_NAME" 2> "${WORKSPACE}/.git_clone_err.txt"; then
                  return 0
                fi
                echo "git clone 失败（branch=$b）。错误如下：" >&2
                cat "${WORKSPACE}/.git_clone_err.txt" >&2 || true
                return 1
              }

              if try_clone "$BRANCH"; then
                printf 'export ACTUAL_BRANCH=%q\n' "$BRANCH" >> "${WORKSPACE}/.asdm_clone_env"
              elif [ "$BRANCH" = "main" ] && try_clone "master"; then
                printf 'export ACTUAL_BRANCH=%q\n' "master" >> "${WORKSPACE}/.asdm_clone_env"
              elif try_clone "main"; then
                printf 'export ACTUAL_BRANCH=%q\n' "main" >> "${WORKSPACE}/.asdm_clone_env"
              elif try_clone "master"; then
                printf 'export ACTUAL_BRANCH=%q\n' "master" >> "${WORKSPACE}/.asdm_clone_env"
              else
                exit 1
              fi

              set -a
              . "${WORKSPACE}/.asdm_clone_env"
              set +a
              cd "${WORKSPACE}/${REPO_NAME}"
              git log -1 --oneline > "${WORKSPACE}/.asdm_git_head.txt"
ASDM_SCRIPT
            '''
          }
          script {
            echo '当前提交：' + readFile(encoding: 'UTF-8', file: "${env.WORKSPACE}/.asdm_git_head.txt").trim()
          }
        }
      }

      stage('Prepare workspace exec dirs') {
        steps {
          sh '''bash -s <<'ASDM_SCRIPT'
            set -euo pipefail
            set -a
            . "${WORKSPACE}/.asdm_clone_env"
            set +a
            cd "${WORKSPACE}/${REPO_NAME}"
            RESULT_DIR="$(pwd)/asdm-workspace-exec-result"
            mkdir -p "$RESULT_DIR"
            printf 'export RESULT_DIR=%q\n' "$RESULT_DIR" >> "${WORKSPACE}/.asdm_clone_env"
ASDM_SCRIPT
          '''
        }
      }

      stage('Configure ASDM and install workspace') {
        steps {
          script {
            def bu = params.ASDM_BASE_URL?.trim() ?: 'http://127.0.0.1:8880'
            def au = params.ASDM_ARTIFACT_BASE_URL?.trim() ?: 'https://platform-dt01.asdm.ai/_artifacts'
            def atParam = (params.ASDM_API_TOKEN ?: '').toString().trim()
            def atCred = ''
            try {
              withCredentials([string(credentialsId: 'asdm-api-token', variable: 'ASDM_API_TOKEN')]) {
                atCred = (env.ASDM_API_TOKEN ?: '').toString().trim()
              }
            } catch (err) {
              echo "Jenkins credential 'asdm-api-token' not found, skipping."
            }
            def at = atParam ? atParam : atCred
            withEnv([
              "ASDM_BASE_URL_PARAM=${bu}",
              "ASDM_ARTIFACT_BASE_URL_PARAM=${au}",
              "BASE_URL=${bu}",
              "ARTIFACT_BASE_URL=${au}",
              "ASDM_API_TOKEN=${at}",
              "WS_ID=${params.WORKSPACE_ID.trim()}"
            ]) {
              sh '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              set -a
              . "${WORKSPACE}/.asdm_clone_env"
              set +a
              cd "${WORKSPACE}/${REPO_NAME}"
              echo "=== asdm 环境（CLI 优先读 BASE_URL / ARTIFACT_BASE_URL）==="
              echo "BASE_URL=${BASE_URL}"
              echo "ARTIFACT_BASE_URL=${ARTIFACT_BASE_URL}"
              export BASE_URL ARTIFACT_BASE_URL
              if [ -n "${ASDM_API_TOKEN:-}" ]; then
                export ASDM_API_TOKEN
                echo "ASDM_API_TOKEN 已设置（已隐藏）"
              else
                echo "未设置 ASDM_API_TOKEN：如 workspace install 报 401，请在 Jenkins 添加 Secret text 凭据 id=asdm-api-token，或在参数 ASDM_API_TOKEN 传入。"
              fi
              asdm config --set-base-url "${ASDM_BASE_URL_PARAM}"
              asdm config --set-artifact-base-url "${ASDM_ARTIFACT_BASE_URL_PARAM}"
              asdm config || true
              asdm workspace install "${WS_ID}"
              asdm workspace switch "${WS_ID}"
              asdm workspace codebuddy only "${WS_ID}"
              echo "=== .codebuddy/commands (sample) ==="
              find .codebuddy/commands -maxdepth 1 -type f 2>/dev/null | head -50 || true
              if [ -d "$RESULT_DIR" ]; then
                asdm workspace current > "${RESULT_DIR}/workspace-current.txt" 2>&1 || true
                cp -a .asdm/workspace-installs.json "${RESULT_DIR}/workspace-installs.json.copy" 2>/dev/null || true
              fi
ASDM_SCRIPT
              '''
            }
          }
        }
      }

      stage('Resolve prompt') {
        when {
          expression { return params.E2E_MODE == 'full' }
        }
        steps {
          writeFile encoding: 'UTF-8', file: 'workspace-exec-prompt.txt', text: "${params.PROMPT_CONTENT ?: ''}"
          withEnv([
            "EXEC_MODE=${params.EXECUTION_MODE}",
            "E2E_MODE_SHELL=${params.E2E_MODE}"
          ]) {
            sh '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              set -a
              . "${WORKSPACE}/.asdm_clone_env"
              set +a
              cd "${WORKSPACE}/${REPO_NAME}"
              RAW=$(cat "${WORKSPACE}/workspace-exec-prompt.txt")
              printf '%s' "$RAW" > "${RESULT_DIR}/prompt.raw.txt"
              TRIMMED=$(printf '%s' "$RAW" | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//')
              if [ -z "$TRIMMED" ]; then
                echo "PROMPT_CONTENT is required when E2E_MODE=full" >&2
                exit 1
              fi
              printf '%s' "$TRIMMED" > "${RESULT_DIR}/prompt.resolved.txt"
              MODE="${EXEC_MODE:-auto}"
              if [ "$MODE" = "auto" ]; then
                case "$TRIMMED" in
                  /*) MODE="slash-command" ;;
                  *) MODE="prompt" ;;
                esac
              fi
              printf '{"execution_mode":"%s","e2e_mode":"%s"}\n' "$MODE" "${E2E_MODE_SHELL}" > "${RESULT_DIR}/prompt-meta.json"
ASDM_SCRIPT
            '''
          }
        }
      }

      stage('Execute CodeBuddy') {
        when {
          expression { return params.E2E_MODE == 'full' }
        }
        steps {
          script {
            def runCb = {
              sh '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              set -a
              . "${WORKSPACE}/.asdm_clone_env"
              set +a
              cd "${WORKSPACE}/${REPO_NAME}"
              RESOLVED=$(cat "${RESULT_DIR}/prompt.resolved.txt")
              RESOLVED="${RESOLVED//\\{RESULT_DIR\\}/$RESULT_DIR}"
              printf '%s' "$RESOLVED" > "${WORKSPACE}/.workspace_exec_prompt_final.txt"
ASDM_SCRIPT
              '''
              echo '=== 最终提示词 ==='
              echo readFile(encoding: 'UTF-8', file: "${env.WORKSPACE}/.workspace_exec_prompt_final.txt")
              echo '=== 结束 ==='
              sh '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              set -a
              . "${WORKSPACE}/.asdm_clone_env"
              set +a
              cd "${WORKSPACE}/${REPO_NAME}"
              LOG_FILE="${WORKSPACE}/workspace-exec.log"
              if [ -n "${CODEBUDDY_LOG_PARSER:-}" ] && [ -f "${CODEBUDDY_LOG_PARSER}" ]; then
                PARSER="${CODEBUDDY_LOG_PARSER}"
              elif [ -f /opt/codebuddy-log-parser/dist/index.js ]; then
                PARSER=/opt/codebuddy-log-parser/dist/index.js
              else
                PARSER="${WORKSPACE}/tools/codebuddy-log-parser/dist/index.js"
              fi
              if [ ! -f "$PARSER" ]; then
                echo "缺少 CodeBuddy 日志解析器: $PARSER" >&2
                exit 1
              fi
              FINAL_PROMPT=$(cat "${WORKSPACE}/.workspace_exec_prompt_final.txt")
              export CODEBUDDY_API_KEY
              export CODEBUDDY_BASE_URL="${CODEBUDDY_BASE_URL:-}"
              export CODEBUDDY_INTERNET_ENVIRONMENT="${CODEBUDDY_INTERNET_ENVIRONMENT:-}"
              mkdir -p "$RESULT_DIR"
              set +e
              codebuddy -p "$FINAL_PROMPT" -y --output-format stream-json 2>&1 | tee "$LOG_FILE" | node "$PARSER" --stdin --stream -f human-chat --no-color
              CB=${PIPESTATUS[0]}
              set -e
              exit "$CB"
ASDM_SCRIPT
              '''
            }
            try {
              withCredentials([string(credentialsId: 'codebuddy-api-key', variable: 'CODEBUDDY_API_KEY')]) {
                env.CODEBUDDY_API_KEY_CRED = env.CODEBUDDY_API_KEY
              }
            } catch (err) {
              echo "Jenkins credential 'codebuddy-api-key' not found, skipping."
              env.CODEBUDDY_API_KEY_CRED = ''
            }
            try {
              withCredentials([string(credentialsId: 'codebuddy-base-url', variable: 'CODEBUDDY_BASE_URL_CRED')]) {
                env.CODEBUDDY_BASE_URL_CRED = env.CODEBUDDY_BASE_URL_CRED
              }
            } catch (err) {
              echo "Jenkins credential 'codebuddy-base-url' not found, skipping."
              env.CODEBUDDY_BASE_URL_CRED = ''
            }
            def paramKey = (params.CODEBUDDY_API_KEY_PARAM ?: '').toString().trim()
            // 优先级：构建参数 > 已存在环境变量（用户已在 Jenkins 全局/节点里配置） > Jenkins 凭据
            def envKey = (env.CODEBUDDY_API_KEY ?: '').toString().trim()
            def credKey = (env.CODEBUDDY_API_KEY_CRED ?: '').toString().trim()
            def effectiveKey = paramKey ? paramKey : (envKey ? envKey : credKey)
            withEnv([
              "CODEBUDDY_API_KEY=${effectiveKey}",
              "CODEBUDDY_BASE_URL=${env.CODEBUDDY_BASE_URL_CRED ?: ''}",
              "CODEBUDDY_INTERNET_ENVIRONMENT=internal"
            ]) {
              runCb()
            }
            def logPath = "${env.WORKSPACE}/workspace-exec.log"
            if (fileExists(logPath)) {
              def logText = readFile(encoding: 'UTF-8', file: logPath)
              if (logText.contains('401 Unauthorized')) {
                error('CodeBuddy 返回 401 Unauthorized：请核对 API Key。')
              }
            }
          }
        }
      }

      stage('Create PR') {
        when {
          expression { return params.E2E_MODE == 'full' }
        }
        steps {
          script {
            def token = (params.GIT_CLONE_TOKEN ?: '').toString().trim()
            if (!token) {
              error('未提供 GIT_CLONE_TOKEN，无法执行 push/创建 PR。')
            }
            withEnv([
              "GIT_CLONE_TOKEN=${token}",
              "TARGET_BRANCH=${params.TARGET_BRANCH?.trim() ?: ''}"
            ]) {
              sh '''bash -s <<'ASDM_SCRIPT'
                set -euo pipefail
                set -a
                . "${WORKSPACE}/.asdm_clone_env"
                set +a
                cd "${WORKSPACE}/${REPO_NAME}"

                if git diff --quiet && git diff --cached --quiet; then
                  echo "仓库无改动，跳过 PR 创建。"
                  exit 0
                fi

                git config user.email "jenkins@local"
                git config user.name "jenkins-bot"

                TARGET_BRANCH_TRIMMED="$(echo "${TARGET_BRANCH:-}" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')"
                BR="${TARGET_BRANCH_TRIMMED:-asdm-e2e-${BUILD_NUMBER}}"
                export BR
                git checkout -b "$BR"
                git add -A
                # 这些是执行时生成的工作区文件，不应进入目标仓库 PR
                git reset -q .asdm .codebuddy asdm-workspace-exec-result || true
                if git diff --cached --quiet; then
                  echo "仅有工作区生成文件变更，跳过 PR 创建。"
                  exit 0
                fi
                git commit -m "chore: asdm workspace e2e output"

                # push：使用 token 注入 https remote（不回显 token）
                CLEAN_URL="${REPO_URL}"
                if echo "$CLEAN_URL" | grep -qv '://'; then CLEAN_URL="https://$CLEAN_URL"; fi
                export CLEAN_URL
                CLEAN_URL=$(python3 - <<'PY'
import os
from urllib.parse import urlparse, urlunparse
u=os.environ["CLEAN_URL"]
p=urlparse(u)
print(urlunparse((p.scheme,p.netloc,p.path,"","","")))
PY
                )
                PUSH_URL=$(python3 - <<'PY'
import os
from urllib.parse import urlparse, urlunparse, quote
u=os.environ["CLEAN_URL"]
t=os.environ["GIT_CLONE_TOKEN"]
p=urlparse(u)
netloc=f"x-access-token:{quote(t, safe='')}@{p.netloc}"
print(urlunparse((p.scheme,netloc,p.path,"","","")))
PY
                )
                git push "$PUSH_URL" "HEAD:$BR"

                # 创建 PR：GitHub REST API
                OWNER_REPO=$(echo "$CLEAN_URL" | sed -E 's#https?://[^/]+/##' | sed 's/\\.git$//')
                TITLE="ASDM workspace E2E changes"
                BODY="自动化 E2E（workspace install + codebuddy）产物提交。Build #${BUILD_NUMBER}。"
                export TITLE BODY
                JSON=$(python3 - <<'PY'
import json, os
print(json.dumps({
  "title": os.environ["TITLE"],
  "head": os.environ["BR"],
  "base": os.environ["ACTUAL_BRANCH"],
  "body": os.environ["BODY"],
}, ensure_ascii=False))
PY
                )
                echo "创建 PR: ${OWNER_REPO} head=${BR} base=${ACTUAL_BRANCH}"
                curl -sS -X POST \
                  -H "Authorization: Bearer ${GIT_CLONE_TOKEN}" \
                  -H "Accept: application/vnd.github+json" \
                  "https://api.github.com/repos/${OWNER_REPO}/pulls" \
                  -d "$JSON" > "${RESULT_DIR}/pr.json"
                cat "${RESULT_DIR}/pr.json" | head -c 800 || true
ASDM_SCRIPT
              '''
            }
          }
        }
      }

      stage('Parse final log') {
        when {
          expression { return params.E2E_MODE == 'full' }
        }
        steps {
          sh '''bash -s <<'ASDM_SCRIPT'
            set -euo pipefail
            set -a
            . "${WORKSPACE}/.asdm_clone_env"
            set +a
            LOG_FILE="${WORKSPACE}/workspace-exec.log"
            if [ -n "${CODEBUDDY_LOG_PARSER:-}" ] && [ -f "${CODEBUDDY_LOG_PARSER}" ]; then
              PARSER="${CODEBUDDY_LOG_PARSER}"
            elif [ -f /opt/codebuddy-log-parser/dist/index.js ]; then
              PARSER=/opt/codebuddy-log-parser/dist/index.js
            else
              PARSER="${WORKSPACE}/tools/codebuddy-log-parser/dist/index.js"
            fi
            if [ -f "$LOG_FILE" ]; then
              cat "$LOG_FILE" | node "$PARSER" --stdin -f human-chat --no-color 2>/dev/null || true
              cat "$LOG_FILE" | node "$PARSER" --stdin -f json > "$RESULT_DIR/analysis-log.json" 2>/dev/null || true
            fi
ASDM_SCRIPT
          '''
        }
      }

      stage('Metadata') {
        steps {
          withEnv([
            "WS_ID_META=${params.WORKSPACE_ID.trim()}",
            "E2E_MODE_META=${params.E2E_MODE}",
            "EXEC_MODE_META=${params.EXECUTION_MODE}"
          ]) {
            sh '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              set -a
              . "${WORKSPACE}/.asdm_clone_env"
              set +a
              cat > "$RESULT_DIR/metadata.json" << EOF2
  {
    "pipeline": "asdm-workspace-exec.no-docker",
    "e2e_mode": "${E2E_MODE_META}",
    "execution_mode_param": "${EXEC_MODE_META}",
    "workspace_id": "${WS_ID_META}",
    "repository": {
      "url": "${REPO_URL}",
      "name": "${REPO_NAME}",
      "branch": "${ACTUAL_BRANCH}",
      "requested_branch": "${BRANCH}"
    },
    "run": {
      "timestamp": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")",
      "jenkins_build": "${BUILD_NUMBER}",
      "result_directory": "${RESULT_DIR}"
    }
  }
EOF2
              cat "$RESULT_DIR/metadata.json"
ASDM_SCRIPT
            '''
          }
        }
      }
    }

    post {
      always {
        script {
          if (!env.WORKSPACE?.trim()) {
            echo '未获取到 WORKSPACE，跳过归档。'
            return
          }
          // install-only 模式下不会生成 workspace-exec.log；避免 archiveArtifacts 报“Configuration error?”
          if (fileExists('workspace-exec.log')) {
            archiveArtifacts artifacts: 'workspace-exec.log', fingerprint: true, allowEmptyArchive: true
          } else {
            echo '未生成 workspace-exec.log（通常是 install-only 模式），跳过该日志归档。'
          }
        }
        sh '''bash -s <<'ASDM_SCRIPT'
          set -euo pipefail
          if [ -f "${WORKSPACE}/.asdm_clone_env" ]; then
            set -a
            . "${WORKSPACE}/.asdm_clone_env"
            set +a
            cd "${WORKSPACE}"
            tar czf asdm-workspace-exec-result.tgz -C "${WORKSPACE}/${REPO_NAME}" asdm-workspace-exec-result 2>/dev/null || true
          fi
ASDM_SCRIPT
        '''
        archiveArtifacts artifacts: 'asdm-workspace-exec-result.tgz', fingerprint: true, allowEmptyArchive: true
      }
    }
  }
