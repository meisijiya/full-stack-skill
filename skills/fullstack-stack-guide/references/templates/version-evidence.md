# 版本查证记录 · <项目名>

查证日期：YYYY-MM-DD

命令与输出逐层记录。结论里写清「选了什么版本」与「冲突如何消解」。

---

### <层> · <包名>

- 日期：YYYY-MM-DD
- 命令：
  ```
  npm view <pkg> dist-tags --json
  npm view <pkg> engines --json
  npm view <pkg> peerDependencies --json
  ```
- 输出：
  ```json
  {
    "dist-tags": {},
    "engines": {},
    "peerDependencies": {}
  }
  ```
- 冲突检查：与已选 <其它包> <版本> 的 peer 范围比对 →
- 结论：<版本号> / <换版本或换包的理由> / 未查得官方声明

---

### <层> · <Python 包名>

- 日期：YYYY-MM-DD
- 命令：
  ```
  curl -s https://pypi.org/pypi/<pkg>/json | jq '{version: .info.version, requires_python: .info.requires_python, yanked: .info.yanked}'
  pip index versions <pkg>
  ```
- 输出：
  ```json
  {}
  ```
- 冲突检查：
- 结论：
