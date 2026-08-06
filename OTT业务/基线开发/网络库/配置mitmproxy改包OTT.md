
### 配置方式

https://iwiki.woa.com/p/4010225093
1. 按照iwiki端口映射：
	1. adb reverse tcp:8888 tcp:8080
	2. app彩蛋页配置代理：127.0.0.1:8888
	3. app彩蛋页需要把TQuic关掉
	4. 然后启动： mitmweb --listen-port 8080 --web-port 8081 -s /Users/yuyaobo/project/mitmproxy/main.py

> 基线本身是绕过https，不需要配置https证书，如果需要可以配置证书到APP里，之前针对线上digicert、globalsign旧证书过期 预埋了这个能力https_cert_config

### 脚本编写

```python
import json
from mitmproxy import http, ctx

TARGET_PATH = "/trpc.kt_st_activity.ferocious_tiger.TigerAccessHTTP/TigerAct"


MOCK_RESPONSE = {
    "ret": 0,
    "msg": "ok",
    "data": {
        "scene": 0,
        "template": 4,
        "tiger_act_result": 0,
        "tiger_act_msg": "领取成功"
    }
}


class TigerAct244Mock:
    def request(self, flow: http.HTTPFlow) -> None:
        if TARGET_PATH not in flow.request.path:
            return

        body = json.dumps(MOCK_RESPONSE, ensure_ascii=False).encode("utf-8")

        flow.response = http.Response.make(
            200,
            body,
            {
                "Content-Type": "application/json; charset=utf-8",
                "Cache-Control": "no-cache",
                "X-Mitm-Mock": "TigerAct244Mock"
            }
        )

        ctx.log.info(
            f"[TigerAct244Mock] mocked TigerAct, act_id={act_id}, url={flow.request.pretty_url}"
        )


addons = [
    TigerAct244Mock()
]
```


搭建了一套自己的本地mock回包工作流：
```text
mitmproxy/
├── main.py
├── jce.py
├── mock_tiger_act_244.py
└── mock_free_welfare_half_screen.py
```

+ main.py
	+ Mock 统一入口，配置并加载各个 Mock 模块，同时负责请求分发和热加载。
+ jce.py
	+ JCE 二进制协议编码工具，用于构造结构体、列表、Map、字符串等响应数据。
+ mock_tiger_act_244.py
	+ 活动接口 Mock，匹配指定请求后返回 JSON 数据。
+ mock_free_welfare_half_screen.py
	+ 半屏接口 Mock，匹配指定请求后返回 JCE 二进制数据，并支持将请求中的 cid、vid 写入回包。