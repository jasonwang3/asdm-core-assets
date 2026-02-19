# ASDM Core Assets Repository

## Overview

ASDM Core Assets is a repository containing essential resources for the [ASDM](https://asdm.ai) (AI-First System Development Methodology) system. It includes toolsets, specifications, and contexts that are used to manage and enhance software development processes.

This repository is used by [ASDM Platform](https://platform.asdm.ai) to provide a centralized location for managing and sharing these resources. The platform will sync the contents of this repository as shared resources and provide a UI for managing and utilizing them.

## Directory Structure

```
README.md                     # ASDM核心存储库 总说明文件，当前文件
.github/                      # GitHub 资源目录
│   ├── README.md             # GitHub Action 说明文件
│   └── workflows/
│       ├── asdm-workspace-init-pipeline.yml  # ASDM 工作区初始化workflow
│       ├── asdm-workspace-execution.yml      # ASDM 工作区任务执行通用workflow
│       └── asdm-context-space-sync.yml       # 用于同步上下文注册表的workflow
asdm-core-assets/
│   ├── README.md             # ASDM资产说明文件
│   ├── toolsets/             # 工具包目录
│   │   ├── toolsets-registry.json   # 工具包注册表文件
│   │   ├── sample-toolset/          # 示例工具包
│   │   │   ├── README.md
│   │   │   ├── INSTALL.md
│   │   │   ├── actions/
│   │   │   │   ├── action-tool-001.md
│   │   │   │   └── action-tool-002.md
│   │   │   ├── specs/
│   │   │   │   ├── specs4action001.md
│   │   │   │   └── specs4action002.md
│   │   │   └── tools/
│   │   ├── context-builder/         # Context Builder 工具包
│   │   └── spec-builder-agile-standard/  # Agile Spec Builder 工具包
│   ├── specs/                 # 规约目录
│   │   ├── specs-registry.json
│   │   └── reactjs/
│   │       ├── reactjs-coding-standard.md
│   │       ├── reactjs-performance-guidelines.md
│   │       └── reactjs-testing-guidelines.md
│   └── contexts/              # 上下文存储
│       ├── contexts-registry.json
│       └── {context-space-guid}/
│           ├── README.md
│           ├── manifest.json
│           └── {resource-guid}/
│               └── README.md
```

## Components

### Toolsets

Toolsets provide reusable functionality for AI-assisted development. Each toolset includes:

- **Actions**: Slash commands for AI coding tools
- **Specs**: Internal specifications and rules
- **Tools**: CLI utilities for various tasks

Available toolsets:
- `sample-toolset`: Example toolset demonstrating structure
- `context-builder`: Build context from workspaces
- `spec-builder-agile-standard`: Generate Agile documentation

### Specs

General specifications (rules) for various technology stacks. These specs are automatically added by `asdm-bootstrapper` as needed.

### Contexts

Storage for all context spaces processed by ASDM, creating a unified repository for AI context.

## Usage

1. Browse the toolsets directory for available tools
2. Check `toolsets-registry.json` for toolset information
3. Review specs for coding standards and guidelines
4. Use context spaces for AI-assisted development

## Integration

This repository integrates with:
- [ASDM Platform](https://platform.asdm.ai) - For managing and sharing resources
- AI coding assistants - For context injection and tool usage
- `asdm-bootstrapper` - For automatic installation and updates

## Contributing

Contributions are welcome! Please:
1. Follow the existing directory structure
2. Add appropriate documentation
3. Update registry files as needed
4. Submit pull requests for review

## License

See [LICENSE](LICENSE) for details.

## Support

For questions and support, visit [ASDM Platform](https://platform.asdm.ai).
