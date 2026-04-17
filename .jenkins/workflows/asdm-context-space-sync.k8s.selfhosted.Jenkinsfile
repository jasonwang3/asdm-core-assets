// Same behavior as deploy/.github/workflows/asdm-context-space-sync.yml (Kubernetes agent).
// Docker-agent variant: asdm-context-space-sync.Jenkinsfile
// Host-agent variant: asdm-context-space-sync.no-docker.Jenkinsfile
//
// Setup:
// 1) SCM checkout of a repo that contains this Jenkinsfile and tools/codebuddy-log-parser (fallback only).
// 2) Agent 镜像名写死在下方 Pod yaml 的 image；需换镜像时直接改 Jenkinsfile 并提交到 Git。
// 3) Secret text credentials (all optional): codebuddy-api-key, codebuddy-base-url.
// 4) GIT_CLONE_TOKEN: pass via build parameter only; no Jenkins credential id required. Use empty for public repos.

pipeline {
  agent {
    node {
      label 'asdm-agent'
    }
  }

  environment {
    LC_ALL = 'C.UTF-8'
    LANG = 'C.UTF-8'
    JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8'
    // Self-hosted mode (per customer env). Fill placeholders below as needed.
    CODEBUDDY_INTERNET_ENVIRONMENT = 'selfhosted'
    CODEBUDDY_ENTERPRISE_ENDPOINT = 'https://copilot.your-company.com'
    CODEBUDDY_API_KEY_MANUAL = 'REPLACE_WITH_YOUR_API_KEY'
  }

  options {
    timestamps()
    // Requires AnsiColor plugin: ansiColor('xterm')
  }

  parameters {
    string(name: 'REPO_URL', defaultValue: '', description: 'Target Git HTTPS URL')
    string(name: 'BRANCH', defaultValue: 'master', description: 'Branch; falls back main/master per workflow')
    text(name: 'PROMPT_CONTENT', defaultValue: '分析这个代码仓库的整体架构和技术栈，输出一份 markdown 分析报告到 {RESULT_DIR}/project-overview.md，包含以下内容：1. 项目概述 2. 技术栈 3. 模块结构 4. 关键依赖。用中文输出。', description: 'Prompt for codebuddy -p; {RESULT_DIR} may be used')
    string(name: 'GIT_CLONE_TOKEN', defaultValue: '', description: 'Optional PAT for private repo; empty for public')
  }

  stages {
    stage('Verify agent toolchain') {
      steps {
        sh '''bash -s <<'ASDM_SCRIPT'
          set -euo pipefail
          set -x
          echo "=== Runtime Debug (inside k8s agent container) ==="
          echo "SHELL=${SHELL:-unknown}"
          echo "PWD=$(pwd)"
          echo "USER=$(id -un) UID=$(id -u) GID=$(id -g)"
          id
          command -v sh
          command -v bash
          command -v docker || true
          docker --version || true
          node --version
          npm --version
          git --version
          python3 --version
          codebuddy --version
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
            echo '使用镜像内预装的 codebuddy-log-parser，跳过本阶段'
            return
          }
          if (!fileExists('tools/codebuddy-log-parser/package.json')) {
            error('缺少 tools/codebuddy-log-parser，且镜像内未预装 /opt/codebuddy-log-parser')
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
              git clone --branch "$b" --single-branch "$CLONE_URL" "$REPO_NAME"
            }

            if try_clone "$BRANCH" 2>/dev/null; then
              printf 'export ACTUAL_BRANCH=%q\n' "$BRANCH" >> "${WORKSPACE}/.asdm_clone_env"
            elif [ "$BRANCH" = "main" ] && try_clone "master" 2>/dev/null; then
              printf 'export ACTUAL_BRANCH=%q\n' "master" >> "${WORKSPACE}/.asdm_clone_env"
            elif try_clone "main" 2>/dev/null; then
              printf 'export ACTUAL_BRANCH=%q\n' "main" >> "${WORKSPACE}/.asdm_clone_env"
            elif try_clone "master" 2>/dev/null; then
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

    stage('Prepare result dir') {
      steps {
        sh '''bash -s <<'ASDM_SCRIPT'
          set -euo pipefail
          set -a
          . "${WORKSPACE}/.asdm_clone_env"
          set +a
          cd "${WORKSPACE}/${REPO_NAME}"
          RESULT_DIR="$(pwd)/asdm-context-space-result"
          mkdir -p "$RESULT_DIR"
          printf 'export RESULT_DIR=%q\n' "$RESULT_DIR" >> "${WORKSPACE}/.asdm_clone_env"
ASDM_SCRIPT
        '''
      }
    }

    stage('Analyze with CodeBuddy') {
      steps {
        writeFile encoding: 'UTF-8', file: 'analysis-prompt.txt', text: "${params.PROMPT_CONTENT ?: ''}"
        script {
          def runAnalyze = {
            sh '''bash -s <<'ASDM_SCRIPT'
            set -euo pipefail
            set -a
            . "${WORKSPACE}/.asdm_clone_env"
            set +a
            cd "${WORKSPACE}/${REPO_NAME}"
            ANALYSIS_PROMPT=$(cat "${WORKSPACE}/analysis-prompt.txt")
            ANALYSIS_PROMPT="${ANALYSIS_PROMPT//\\{RESULT_DIR\\}/$RESULT_DIR}"
            printf '%s' "$ANALYSIS_PROMPT" > "${WORKSPACE}/.analysis_prompt_resolved.txt"
ASDM_SCRIPT
            '''
            echo '=== 提示内容 ==='
            echo readFile(encoding: 'UTF-8', file: "${env.WORKSPACE}/.analysis_prompt_resolved.txt")
            echo '=== 提示结束 ==='
            sh '''bash -s <<'ASDM_SCRIPT'
            set -euo pipefail
            set -a
            . "${WORKSPACE}/.asdm_clone_env"
            set +a
            cd "${WORKSPACE}/${REPO_NAME}"
            LOG_FILE="${WORKSPACE}/analysis.log"
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
            ANALYSIS_PROMPT=$(cat "${WORKSPACE}/.analysis_prompt_resolved.txt")
            export CODEBUDDY_API_KEY
            export CODEBUDDY_INTERNET_ENVIRONMENT="${CODEBUDDY_INTERNET_ENVIRONMENT:-}"
            export CODEBUDDY_MODEL="glm-5"
            mkdir -p "${HOME}/.codebuddy"
            cat > "${HOME}/.codebuddy/settings.json" << EOF2
{
  "enterpriseEndpoint": "${CODEBUDDY_ENTERPRISE_ENDPOINT:-}",
  "env": {
    "CODEBUDDY_INTERNET_ENVIRONMENT": "${CODEBUDDY_INTERNET_ENVIRONMENT:-selfhosted}",
    "CODEBUDDY_API_KEY": "${CODEBUDDY_API_KEY:-}"
  }
}
EOF2
            mkdir -p "$RESULT_DIR"
            echo "=== codebuddy version ===" && codebuddy --version || true
            set +e
            echo "=== settings.json ===" && cat "${HOME}/.codebuddy/settings.json" && echo "=== end ==="
            codebuddy --model "$CODEBUDDY_MODEL" -p "$ANALYSIS_PROMPT" -y --output-format stream-json 2>&1 | tee "$LOG_FILE" | node "$PARSER" --stdin --stream -f human-chat --no-color
            CB=${PIPESTATUS[0]}
            set -e
            exit "$CB"
ASDM_SCRIPT
            '''
          
          def manualApiKey = (env.CODEBUDDY_API_KEY_MANUAL ?: '').trim()
          def manualEndpoint = (env.CODEBUDDY_ENTERPRISE_ENDPOINT ?: '').trim()
          def effectiveApiKey = (!manualApiKey.isEmpty() && manualApiKey != 'REPLACE_WITH_YOUR_API_KEY')
                  ? manualApiKey
                  : ''

          if (effectiveApiKey) {
            echo "Using manual CodeBuddy API key from Jenkinsfile environment."
          } else {
            error("CODEBUDDY_API_KEY_MANUAL is not set. Please update Jenkinsfile environment.")
          }
          if (!manualEndpoint.isEmpty() && manualEndpoint != 'https://copilot.your-company.com') {
            echo "Using manual CodeBuddy endpoint: ${manualEndpoint}"
          } else {
            error("CODEBUDDY_ENTERPRISE_ENDPOINT is not set. Please update Jenkinsfile environment.")
          }
          withEnv([
            "CODEBUDDY_API_KEY=${effectiveApiKey}",
            "CODEBUDDY_INTERNET_ENVIRONMENT=${env.CODEBUDDY_INTERNET_ENVIRONMENT ?: 'selfhosted'}"
            "CODEBUDDY_CODE_SUBAGENT_MODEL=glm-5"
          ]) {
            runAnalyze()
          }
          def analysisLogPath = "${env.WORKSPACE}/analysis.log"
          if (fileExists(analysisLogPath)) {
            def logText = readFile(encoding: 'UTF-8', file: analysisLogPath)
            if (logText.contains('401 Unauthorized')) {
              error('CodeBuddy 返回 401 Unauthorized：请核对 Jenkinsfile 中手动配置的 API Key 是否有效、未过期、无多余空格。')
            }
          }
        }
      }
    }

    stage('Parse final log') {
      steps {
        sh '''bash -s <<'ASDM_SCRIPT'
          set -euo pipefail
          set -a
          . "${WORKSPACE}/.asdm_clone_env"
          set +a
          LOG_FILE="${WORKSPACE}/analysis.log"
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
        sh '''bash -s <<'ASDM_SCRIPT'
          set -euo pipefail
          set -a
          . "${WORKSPACE}/.asdm_clone_env"
          set +a
          cat > "$RESULT_DIR/metadata.json" << EOF2
{
  "repository": {
    "url": "${REPO_URL}",
    "name": "${REPO_NAME}",
    "branch": "${ACTUAL_BRANCH}",
    "requested_branch": "${BRANCH}"
  },
  "analysis": {
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

  post {
    always {
      script {
        if (!env.WORKSPACE?.trim()) {
          echo '未获取到 WORKSPACE（通常是 agent 未成功启动），跳过产物归档。'
          return
        }
      }
      archiveArtifacts artifacts: 'analysis.log', fingerprint: true, allowEmptyArchive: true
      sh '''bash -s <<'ASDM_SCRIPT'
        set -euo pipefail
        if [ -f "${WORKSPACE}/.asdm_clone_env" ]; then
          set -a
          . "${WORKSPACE}/.asdm_clone_env"
          set +a
          cd "${WORKSPACE}"
          tar czf asdm-context-space-result.tgz -C "${WORKSPACE}/${REPO_NAME}" asdm-context-space-result 2>/dev/null || true
        fi
ASDM_SCRIPT
      '''
      archiveArtifacts artifacts: 'asdm-context-space-result.tgz', fingerprint: true, allowEmptyArchive: true
    }
  }
}

