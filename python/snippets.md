
* 批量更新库(注意--upgrade后面的空格)
``` python
import pip
from subprocess import call

for dist in pip.get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```
