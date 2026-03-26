pipeline {
  agent any

  environment {
    LC_ALL = 'C.UTF-8'
    LANG = 'C.UTF-8'
    JAVA_TOOL_OPTIONS = '-Dfile.encoding=UTF-8'
    // 全局设置国内互联网环境
    CODEBUDDY_INTERNET_ENVIRONMENT = 'internal'
  }

  options {
    timestamps()
  }

  parameters {
    string(name: 'REPO_URL', defaultValue: '', description: '目标 Git HTTPS 地址')
    string(name: 'BRANCH', defaultValue: 'main', description: '分支名称 (main/master)')
    text(name: 'PROMPT_CONTENT', defaultValue: '', description: 'CodeBuddy 分析 Prompt; 可使用 {RESULT_DIR} 变量')
    string(name: 'GIT_CLONE_TOKEN', defaultValue: '', description: '私有仓库访问 Token (PAT)；公开仓库请留空')
  }

  stages {
    stage('Check Environment') {
      steps {
        script {
          ['git', 'node', 'npm', 'codebuddy'].each { c ->
            if (sh(script: "command -v ${c} >/dev/null 2>&1", returnStatus: true) != 0) {
              error("节点缺失关键工具: ${c}")
            }
          }
          def py = sh(script: 'command -v python3 || command -v python', returnStdout: true).trim()
          sh "echo \"export PY=${py}\" > \"${env.WORKSPACE}/.python_cmd\""
        }
      }
    }

    stage('Clone Repository') {
      steps {
        script {
          if (!params.REPO_URL?.trim()) error('必须填写 REPO_URL')
        }
        withEnv([
          "REPO_URL=${params.REPO_URL.trim()}",
          "BRANCH=${params.BRANCH.trim()}",
          "GIT_CLONE_TOKEN=${params.GIT_CLONE_TOKEN}"
        ]) {
          sh '''bash -s <<'ASDM_SCRIPT'
            set -euo pipefail
            . "${WORKSPACE}/.python_cmd"
            
            CLONE_URL=$("${PY}" << 'PY'
import os
from urllib.parse import urlparse, urlunparse, quote
raw = os.environ.get("REPO_URL", "").strip()
token = (os.environ.get("GIT_CLONE_TOKEN") or "").strip()
p = urlparse(raw if "://" in raw else "https://" + raw)
host = (p.hostname or "").lower()
hostport = f"{p.hostname}:{p.port}" if p.port else (p.hostname or "")
if token:
    user = "x-access-token" if "github.com" in host else "oauth2"
    netloc = f"{user}:{quote(token, safe='')}@{hostport}"
else:
    netloc = p.netloc
path = p.path.rstrip("/") or "/"
if not path.endswith(".git"): path += ".git"
print(urlunparse((p.scheme, netloc, path, "", "", "")))
PY
            )

            REPO_NAME=$(echo "$REPO_URL" | sed 's/.*\\///' | sed 's/\\.git$//')
            echo "export REPO_NAME=\"$REPO_NAME\"" > "${WORKSPACE}/.asdm_clone_env"
            echo "export CLONE_URL=\"$CLONE_URL\"" >> "${WORKSPACE}/.asdm_clone_env"

            rm -rf "$REPO_NAME"
            git clone --branch "$BRANCH" --single-branch "$CLONE_URL" "$REPO_NAME" || git clone "$CLONE_URL" "$REPO_NAME"
            cd "$REPO_NAME" && git log -1 --oneline > "${WORKSPACE}/.asdm_git_head.txt"
ASDM_SCRIPT
          '''
          script {
            env.REPO_NAME = params.REPO_URL.trim().replaceAll(/.*\//, '').replaceAll(/\.git$/, '')
          }
        }
      }
    }

    stage('Analyze with CodeBuddy') {
      steps {
        script {
          writeFile encoding: 'UTF-8', file: 'analysis-prompt.txt', text: params.PROMPT_CONTENT ?: ''
          
          withCredentials([string(credentialsId: 'codebuddy-api-key', variable: 'CODEBUDDY_API_KEY')]) {
            sh '''bash -s <<'ASDM_SCRIPT'
              set -euo pipefail
              [ -f "${WORKSPACE}/.asdm_clone_env" ] && . "${WORKSPACE}/.asdm_clone_env"
              
              cd "${WORKSPACE}/${REPO_NAME}"
              RESULT_DIR="$(pwd)/asdm-context-space-result"
              mkdir -p "$RESULT_DIR"
              
              # 替换变量
              cat "${WORKSPACE}/analysis-prompt.txt" | sed "s@{RESULT_DIR}@${RESULT_DIR}@g" > "${WORKSPACE}/.resolved_prompt.txt"
              
              # 解析器定位
              if [ -f /opt/codebuddy-log-parser/dist/index.js ]; then
                PARSER=/opt/codebuddy-log-parser/dist/index.js
              elif [ -f "${WORKSPACE}/tools/codebuddy-log-parser/dist/index.js" ]; then
                PARSER="${WORKSPACE}/tools/codebuddy-log-parser/dist/index.js"
              else
                echo "Log Parser Not Found" && exit 1
              fi

              # 执行分析，环境变量已经由流水线 environment 块注入
              echo "Starting CodeBuddy analysis (Environment: $CODEBUDDY_INTERNET_ENVIRONMENT)..."
              set +e
              codebuddy -p "$(cat "${WORKSPACE}/.resolved_prompt.txt")" -y --output-format stream-json 2>&1 \
                | tee "$RESULT_DIR/codebuddy-raw.log" \
                | node "$PARSER" --stdin --stream -f human-chat --no-color | tee "$RESULT_DIR/codebuddy-analysis.txt"
              
              CB_EXIT=${PIPESTATUS[0]}
              
              # 将原始日志转换为JSON格式保存
              cat "$RESULT_DIR/codebuddy-raw.log" | node "$PARSER" --stdin -f json > "$RESULT_DIR/codebuddy-analysis.json" 2>/dev/null || true
              
              # 保存分析配置和统计信息
              echo "=== CodeBuddy Analysis Summary ===" > "$RESULT_DIR/summary.txt"
              echo "Timestamp: $(date -u +'%Y-%m-%dT%H:%M:%SZ')" >> "$RESULT_DIR/summary.txt"
              echo "Repository: ${REPO_URL}" >> "$RESULT_DIR/summary.txt"
              echo "Branch: ${BRANCH}" >> "$RESULT_DIR/summary.txt"
              echo "Analysis Exit Code: ${CB_EXIT}" >> "$RESULT_DIR/summary.txt"
              echo "Working Directory: ${RESULT_DIR}" >> "$RESULT_DIR/summary.txt"
              echo "" >> "$RESULT_DIR/summary.txt"
              echo "=== Generated Files ===" >> "$RESULT_DIR/summary.txt"
              ls -lh "$RESULT_DIR"/ | grep -v "^total" | awk '{print $9, "(" $5 ")"}' >> "$RESULT_DIR/summary.txt" 2>/dev/null || true
              
              exit $CB_EXIT
ASDM_SCRIPT
            '''
          }
        }
      }
    }

    stage('Archive') {
      steps {
        sh '''bash -s <<'ASDM_SCRIPT'
          set -euo pipefail
          [ -f "${WORKSPACE}/.asdm_clone_env" ] && . "${WORKSPACE}/.asdm_clone_env"
          cd "${WORKSPACE}/${REPO_NAME}"
          RESULT_DIR="$(pwd)/asdm-context-space-result"
          
          # 生成元数据
          printf '{"build":"%s","timestamp":"%s","repo":"%s","branch":"%s"}' \
            "${BUILD_ID}" \
            "$(date -u +'%Y-%m-%dT%H:%M:%SZ')" \
            "${REPO_URL}" \
            "${BRANCH}" > "$RESULT_DIR/metadata.json"
          
          # 列出RESULT_DIR中的所有文件
          echo "=== Files in RESULT_DIR ===" 
          find "$RESULT_DIR" -type f | while read file; do
            size=$(ls -lh "$file" | awk '{print $5}')
            echo "$(basename "$file") ($size)"
          done
          
          # 创建汇总索引文件
          {
            echo "# CodeBuddy Analysis Results"
            echo ""
            echo "## Metadata"
            cat "$RESULT_DIR/metadata.json" | python3 -m json.tool 2>/dev/null || cat "$RESULT_DIR/metadata.json"
            echo ""
            echo "## Generated Files"
            find "$RESULT_DIR" -type f -not -name 'index.md' | sort | while read file; do
              size=$(ls -lh "$file" | awk '{print $5}')
              echo "- $(basename "$file") ($size)"
            done
            echo ""
            echo "## Analysis Summary"
            [ -f "$RESULT_DIR/summary.txt" ] && cat "$RESULT_DIR/summary.txt" || echo "No summary available"
          } > "$RESULT_DIR/index.md"
          
          cd "${WORKSPACE}"
          tar czf asdm-result.tgz -C "${WORKSPACE}/${REPO_NAME}" asdm-context-space-result
          
          echo "Archive created successfully: asdm-result.tgz"
          ls -lh asdm-result.tgz
ASDM_SCRIPT
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: "asdm-result.tgz, ${env.REPO_NAME}/asdm-context-space-result/**/*", allowEmptyArchive: true
    }
  }
}