# ai-code-improvement-skill

**作者：** DuLane

**注意：建议苍穹开发团队内部使用迭代**

针对金蝶云苍穹 Java 开发的 AI 代码自我提升的技能。

在 AI 生成或修改代码后自动检查常见错误，修复发现的问题，并将错误记录到经验库中。

当 AI 写完 Java 代码、用户要求代码审查、自检、或提及代码质量时触发。


---

## 📌 概览

`ai-code-improvement-skill` 是一个面向金蝶云苍穹开发的 AI Skill

它的主要作用是解决AI在开发代码中出现的一些常见的错误修复及自我记录

比如AI在做删除前会缺少状态字段的查询,在动态表单执行itemClick事件而非Click事件等。当AI发现自己有犯类似的错误时候会自己进行修复

在AI发现错误未在errors-log里面时候,自己会将当前错误修复信息追加到记录里面,避免下次再犯类似的错误


## 🧠 核心能力

- `🧩` 第1步：读取经验库
- `🔎` 第2步：按所有检查分类扫描代码
- `🗂️` 第3步：修复所有发现的问题
- `📝` 第4步：将新错误记录到经验库
- `🐯` 第5步：向用户汇报结果

## 🌲 使用 Skill


### 团队使用

1、建议团队使用,团队成员共同贡献AI错误记录库来帮助AI去成长。

2、开发团队将该Skill上传到Git仓库,团队成员拉取Skill信息到本地,每个人的AI都会将错误记录进行积累,每次提交前去拉取下最新的AI经验库。
3、众人拾柴火焰高,这样可以积累大量的AI代码错误问题,帮助AI迅速成长为更加好用的助手。

### 个人使用

1、个人使用可以自己去积累AI的经验库
2、可以去GitHub上去拉取最新的,自己手动把一些经验积累添加到本地的skill下相应的文件中

## 🍃 安装 Skill


目前很多人都是有安装多个AI Agent,比如Qoder、CodeBuddy、Trae、Cursor、Claude Code等,如果你有用多个AI工具开发,将他们指向同一个Skill目录比较好,既方面更新,又方面自己记录。

通过一份 skill 源目录，不同AI工具 skill 安装目录通过软链接指向该目录。


### AI工具Skill默认目录结构

```text
/Users/yourname/skills/ai-code-improvement-skill          # skill 源目录
~/.qoder/skills/ai-code-improvement-skill                 # Qoder
~/.workbuddy/skills/ai-code-improvement-skill 			  # WorkBuddy
~/.codebuddy/skills/ai-code-improvement-skill             # Codebuddy

```

### MacOS / Linux

1、 准备统一维护目录

```bash
mkdir -p ~/skills
cp -R ai-code-improvement-skill ~/skills/
```

2、 为不同 AI工具 创建软链接

```bash
mkdir -p ~/.qoder/skills ~/.workbuddy/skills ~/.codebuddy/skills 
ln -sfn ~/skills/ai-code-improvement-skill ~/.qoder/skills/ai-code-improvement-skill
ln -sfn ~/skills/ai-code-improvement-skill ~/.workbuddy/skills/ai-code-improvement-skill
ln -sfn ~/skills/ai-code-improvement-skill ~/.codebuddy/skills/ai-code-improvement-skill

```

### Windows

1、 准备统一维护目录

```powershell
mkdir $env:USERPROFILE\skills -ErrorAction SilentlyContinue
Copy-Item -Path D:\path\to\ai-code-improvement-skill -Destination $env:USERPROFILE\skills\ai-code-improvement-skill -Recurse
```

2、 为不同 AI工具 创建目录联接（推荐管理员 PowerShell）

```powershell
mkdir $env:USERPROFILE\.qoder\skills -ErrorAction SilentlyContinue
mkdir $env:USERPROFILE\.workbuddy\skills -ErrorAction SilentlyContinue
mkdir $env:USERPROFILE\.codebuddy\skills -ErrorAction SilentlyContinue

cmd /c mklink /J "$env:USERPROFILE\.qoder\skills\ai-code-improvement-skill" "$env:USERPROFILE\skills\ai-code-improvement-skill"
cmd /c mklink /J "$env:USERPROFILE\.workbuddy\skills\ai-code-improvement-skill" "$env:USERPROFILE\skills\ai-code-improvement-skill"
cmd /c mklink /J "$env:USERPROFILE\.codebuddy\skills\ai-code-improvement-skill" "$env:USERPROFILE\skills\ai-code-improvement-skill"
```

### 维护

- 如果只有一个AI工具使用,把 skill 源文件放到对应目录也行,不需要维护多个添加超链接
- 如果只是想省事,很多AI工具也支持手动导入skill的zip包进行安装
- 团队建议共同维护,这样AI的能力提升更快

---

## ✈️ 加载 Skill 


### 加载 Skill

在代码开发过程中,可以选择 / 呼出对应的skill,让AI通过该skill去进行自查.也可以手动让AI去使用这个skill

```
使用 ai-code-improvement-skill 技能
```

即使不手动选择该skill,该skill也会在有类似让AI自查这样的提示词时触发.

```
你检查下代码
```

## 🚗 版本更新

### GitHub 地址 

https://github.com/LaneDu/ai-code-improvement-skill.git
非常欢迎fork PR

### 20260407 v1.0 

初始化skill信息,列出常用的检查点,以及经验记录样式


