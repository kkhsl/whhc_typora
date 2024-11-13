# 1.命名规范

### 1. **分支命名的基本结构**

一个好的分支命名结构通常包括以下几个元素：

- **类型（Type）**: 分支的目的，比如功能开发（feature）、bug 修复（bugfix）、热修复（hotfix）等。
- **工作项编号（Ticket ID）**: 如果你使用了任务管理工具（如 Jira、Trello、GitHub Issues），可以在分支中加入任务编号，帮助快速关联分支与任务。
- **简短描述（Description）**: 对分支的简短说明，通常是功能、修复或变更的描述。

### 2. **常见分支类型命名规范**

- **功能分支（Feature Branches）**：用于开发新功能。
  - 格式：`feature/<ticket-id>-<short-description>`
  - 示例：`feature/1234-add-login-page`
  - **说明**：`feature` 用来标识这个分支是开发新功能，`<ticket-id>` 是与此功能相关的任务编号，`<short-description>` 是功能的简短描述。
- **修复分支（Bugfix Branches）**：用于修复 bug。
  - 格式：`bugfix/<ticket-id>-<short-description>`
  - 示例：`bugfix/5678-fix-login-error`
  - **说明**：`bugfix` 用来标识这个分支是解决一个 bug，`<ticket-id>` 是该 bug 所在的任务编号，`<short-description>` 是修复的简短描述。
- **热修复分支（Hotfix Branches）**：用于快速修复生产环境中的问题。
  - 格式：`hotfix/<ticket-id>-<short-description>`
  - 示例：`hotfix/9012-fix-crash-on-login`
  - **说明**：`hotfix` 用于修复急需解决的生产环境问题，通常会直接从 `main` 或 `master` 分支创建。
- **发布分支（Release Branches）**：用于版本发布的准备。
  - 格式：`release/<version>`
  - 示例：`release/1.2.0`
  - **说明**：`release` 用于版本发布的分支，`<version>` 表示发布的版本号。
- **开发分支（Develop Branch）**：用于日常开发，通常是所有开发工作汇聚的地方。
  - 格式：`develop`
  - 示例：`develop`
  - **说明**：`develop` 是常见的主开发分支，所有的功能分支通常会基于它创建，所有的开发工作也会最终合并回 `develop`。
- **主分支（Main/Master Branch）**：用于存储已发布或稳定的版本代码。
  - 格式：`main` 或 `master`（推荐使用 `main`）
  - 示例：`main` 或 `master`
  - **说明**：`main`（或 `master`）是主分支，用于存储已发布和稳定的版本代码。

### 3. **分支命名规范的实践和示例**

根据项目的需求，确保每个分支名称都能准确反映出它的目的。以下是一些常见场景和分支命名示例：

#### 3.1 **功能分支命名（Feature Branch）**

- 用于开发新功能。

- 格式：`feature/<ticket-id>-<short-description>`

- 示例：

  - `feature/101-add-user-authentication`
  - `feature/234-integrate-payment-gateway`

  描述

  ：

  - `101` 是与功能开发相关的任务编号。
  - `add-user-authentication` 是该功能的简短描述。

#### 3.2 **修复分支命名（Bugfix Branch）**

- 用于修复 bug 或错误。

- 格式：`bugfix/<ticket-id>-<short-description>`

- 示例：

  - `bugfix/567-fix-login-error`
  - `bugfix/890-remove-footer-margin`

  描述

  ：

  - `567` 是与修复相关的任务编号。
  - `fix-login-error` 是对修复问题的简短描述。

#### 3.3 **热修复分支命名（Hotfix Branch）**

- 用于修复生产环境中的问题。

- 格式：`hotfix/<ticket-id>-<short-description>`

- 示例：

  - `hotfix/123-fix-crash-on-login`
  - `hotfix/456-fix-security-vulnerability`

  描述

  ：

  - `123` 是修复问题的任务编号。
  - `fix-crash-on-login` 是修复内容的简短描述。

#### 3.4 **发布分支命名（Release Branch）**

- 用于准备发布新版本。

- 格式：`release/<version>`

- 示例：

  - `release/1.0.0`
  - `release/2.1.3`

  描述

  ：

  - `release/1.0.0` 表示准备发布版本 1.0.0。

#### 3.5 **开发分支命名（Develop Branch）**

- 开发过程中所有工作汇总的地方。

- 格式：`develop`

- 示例：

  - `develop`

  描述

  ：这个分支通常作为所有功能分支的基础，所有的功能都应该先合并到 

  ```
  develop
  ```

  ，然后再合并到 

  ```
  main
  ```

  。

#### 3.6 **主分支命名（Main Branch）**

- 主分支，存放稳定版本。

- 格式：`main` 或 `master`

- 示例：

  - `main` 或 `master`

  描述

  ：通常在项目上线后，

  ```
  main
  ```

   用于存放生产环境的代码。

### 4. **其他命名最佳实践**

- **使用小写字母和连字符**：分支名称应该使用小写字母，并且多个词之间用连字符 `-` 连接。避免使用空格和下划线。

  示例：

  - 推荐：`feature/add-login-page`
  - 不推荐：`Feature/AddLoginPage`

- **描述清晰简洁**：分支的名称应简洁、直接地反映其目的，避免过长的名称。

- **避免特殊字符**：避免在分支名称中使用如 `!`, `@`, `#`, `%` 等特殊字符，避免可能的编码或操作系统兼容问题。

- **统一的命名约定**：确保团队中的每个人都遵循相同的命名规范，可以通过文档或 Git Hook 来强制执行。

### 5. **GitHub 分支管理工具**

为了确保规范化的分支命名，你可以使用 GitHub 的 **Pull Request 模板** 来提醒开发人员在创建分支时遵循命名规则，或者使用 **GitHub Actions** 来验证分支命名是否符合规范。

### 6. **总结**

规范的分支命名有助于提高团队协作效率，确保每个分支的目的和内容都能被明确识别。一个好的命名规则能够帮助团队成员快速了解每个分支的用途，并且为代码管理带来更多的可控性和规范性。常见的分支类型如功能分支（feature）、修复分支（bugfix）、热修复分支（hotfix）等，都应该遵循统一的命名格式。
