---
tags:
  - Tools/Python
type: page
---
---

## Profile python memory

```
# https://github.com/pythonprofilers/memory_profilerpip 
install memory_profiler psutil
# psutil нужен для повышения производительности 
memory_profiler python -m memory_profiler some-code.py
```
В some-code.py использовать @profile