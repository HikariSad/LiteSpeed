# node_LiteSpeedTest
修改自 github LiteSpeedTest
node_LiteSpeedTest is a simple tool for batch test ss/ssr/v2ray/trojan/clash servers.   
Feature
- 支持ss/ssr/v2ray/trojan/vmess/clash订阅链接\节点链接\文件

 配合另一个节点获取项目free-vpn-nodes，初级筛选节点，再调用该项目进行测速排序过滤


### Usage
```
Run as a speed test tool:
    # run this command then open http://127.0.0.1:10888/ in your browser to start speed test
    ./lite
    # start with another port
    ./lite -p 10889
    
    # test in command line only mode
    ./lite --test https://raw.githubusercontent.com/freefq/free/master/v2
    # test in command line only mode with custom config.
    ./lite --config config.json --test https://raw.githubusercontent.com/freefq/free/master/v2
    # details can find here  ./config.json
    # all config options:
    #       "group":"job",   // group name
	#       "speedtestMode":"pingonly", // speedonly pingonly all
	#       "pingMethod":"googleping",  // googleping tcpping
	#       "sortMethod":"rspeed",      // speed rspeed ping rping
	#       "concurrency":1,  // concurrency number
	#       "testMode":2,   // 2: ALLTEST 3: RETEST
	#       "subscription":"subscription url",
	#       "timeout":16,  // timeout in seconds
	#       "language":"en", // en cn
	#       "fontSize":24,
	#       "unique": true,  // remove duplicated value
	#       "theme":"rainbow", 
	#       "outputMode": 1  // 0: base64 1: pic path 2: no pic 3: json 4: txt
```
### 针对返回结果格式 测速格式"info": "gotspeed",
    [{
        "id": 998,
        "info": "gotspeed",
        "speed": "1.2MB",
        "maxspeed": "1.2MB",
        "traffic": 1277730,
        "link": "vmess://ewogICAgImFkZCI6ICIxNzIuNjQuMjMzLjMyIiwKICAgICJhaWQiOiAwLAogICAgImhvc3QiOiAiaXAyLjE0NTcyMzAueHl6IiwKICAgICJpZCI6ICJlOWUzY2MxMy1kYjQ4LTRjYzEtOGMyNC03NjI2NDM5YTUzMzkiLAogICAgIm5ldCI6ICJ3cyIsCiAgICAicGF0aCI6ICJnaXRodWIuY29tL0FsdmluOTk5OSIsCiAgICAicG9ydCI6IDIwODYsCiAgICAicHMiOiAi8J+PgVJFTEFZLTE3Mi42NC4yMzMuMzItMDEzMiIsCiAgICAidGxzIjogIiIsCiAgICAidHlwZSI6ICJhdXRvIiwKICAgICJzZWN1cml0eSI6ICJhdXRvIiwKICAgICJza2lwLWNlcnQtdmVyaWZ5IjogZmFsc2UsCiAgICAic25pIjogImlwMi4xNDU3MjMwLnh5eiIKfQ=="
    },
    ......
]
调用协议： websocket 可以网页、后端调用
举例python
``````

# 示例用法
async def connect_websocket():
    global best_nodes_detail, best_nodes_urls
    file_path = './o/allnode.txt'
    #file_path = './o/clash.yaml'
    clash_yaml_string = read_clash_yaml_as_string(file_path)
    if len(clash_yaml_string) < 10:
    print("未从节点列表获取节点信息无法进一步测试节点")
    return
    #suburls = base64.b64decode(clash_yaml_string).decode('utf-8')
    #nodes = clash_yaml_string.split('\n')

    uri = "ws://192.168.2.182:10888/test"
    # uri = "ws://127.0.0.1:10888/test"
    params = {
        "concurrency": 7,
        "fontSize": 24,
        "group": "?empty?",
        "language": "en",
        "pingMethod": "googleping",
        "sortMethod": "rspeed",
        "speedtestMode": "speedonly",
        "testMode": 2,
        "theme": "rainbow",
        "timeout": 18,
        "unique": True,
        # "subscription": "trojan://518a6ff8-7233-4f28-8a40-d3fa82a5875d@5gzdx.233235.xyz:706?allowInsecure=1&sni=5gzdx.233235.xyz#0%7C-https%3A//t.me/MrXbin-107",
        "subscription": clash_yaml_string,
        "outputMode": 3

    }
    #  "subscription": base64.b64decode(clash_yaml_string).decode('utf-8')

    async with websockets.connect(uri) as websocket:
        # 发送参数
        await websocket.send(json.dumps(params))
        print(f"发送参数: {params}")
        # 在主循环外部初始化一个字典
        best_nodes_dict = {}
        servers_list = []
        # 持续接收消息
        while True:
            try:
                response = await asyncio.wait_for(websocket.recv(), timeout=9999)
                data = json.loads(response)
                if data["info"] == "eof":
                    break
                if data.get("info", "") == "gotservers":
                    servers_list.extend(data.get("servers", []))

                if data.get("id", 0) not in unique_ids and data.get("info", "N/A") == "gotspeed" \
                        and data.get("speed", "N/A") != "N/A" and "MB" in data.get("speed", "") \
                        and len(data.get("link", "")) > 10:

                    unique_ids.add(data.get("id", 0))
                    best_nodes_urls.append(data.get("link", ""))
                    best_nodes_detail.append(data)

                    print(f'收到第{len(unique_ids)}个消息: \033[1;32m....{data}.\033[0m')
                else:
                    print(f"------------不满足条件的消息: {data}")
                # 在这里处理接收到的数据
            except websockets.exceptions.ConnectionClosed:
                print("连接已关闭")
                break




```
Run as a grpc server:
    # start the grpc server  
    ./lite -grpc -p 10999
    # grpc go client example in ./api/rpc/liteclient/client.go 
    # grpc python client example in ./api/rpc/liteclientpy/client.py

Run as a http/socks5 proxy:
    # use default port 8090
    ./lite vmess://aHR0cHM6Ly9naXRodWIuY29tL3h4ZjA5OC9MaXRlU3BlZWRUZXN0
    ./lite ssr://aHR0cHM6Ly9naXRodWIuY29tL3h4ZjA5OC9MaXRlU3BlZWRUZXN0
    # use another port
    ./lite -p 8091 vmess://aHR0cHM6Ly9naXRodWIuY29tL3h4ZjA5OC9MaXRlU3BlZWRUZXN0
```

### Build 最好在linux下进行编译 防止win下编译的文件在linux下运行不了
```linux bash
    # require go>=1.18.1, nodejs >= 14
    # build frontend
    cp $(go env GOROOT)/misc/wasm/wasm_exec.js ./web/gui/wasm_exec.js
    npm install --prefix web/gui build
    npm run --prefix web/gui build
    GOOS=js GOARCH=wasm go get -u ./...
    GOOS=js GOARCH=wasm go build -o ./web/gui/dist/main.wasm ./wasm
    go build -o lite
    
```win
    cp  %go_home%/misc/wasm/wasm_exec.js ./web/gui/wasm_exec.js
    npm install --prefix web/gui build
    npm run --prefix web/gui build
    
    set GOOS=js
    set GOARCH=wasm
    go get -u ./...
    go build -o ./web/gui/dist/main.wasm ./wasm
    go build -o lite

```

### Docker
```bash
 docker build --network=host  -t lite:$(git describe --tags --abbrev=0) -f ./docker/Dockerfile ./
 docker run -p 10888:10888/tcp lite:$(git describe --tags --abbrev=0)
```

## Credits

- [clash](https://github.com/Dreamacro/clash)
- [stairspeedtest-reborn](https://github.com/tindy2013/stairspeedtest-reborn)
- [gg](https://github.com/fogleman/gg)

## Developer
```golang
import (
    "context"
    "fmt"
	"time"
    "github.com/xxf098/lite-proxy/web"
)
// see more details in ./examples
func testPing() error {
    ctx := context.Background()
    link := "https://www.example.com/subscription/link"
    opts := web.ProfileTestOptions{
		GroupName:     "Default", 
		SpeedTestMode: "pingonly",   //  pingonly speedonly all
		PingMethod:    "googleping", // googleping
		SortMethod:    "rspeed", // speed rspeed ping rping
		Concurrency:   2,
		TestMode:      2,
		Subscription:  link,
		Language:      "en",  // en cn
		FontSize:      24,
		Theme:         "rainbow",
        Unique:        true,
		Timeout:       10 * time.Second,
		OutputMode:  0,
	}
    nodes, err := web.TestContext(ctx, opts, &web.EmptyMessageWriter{})
    if err != nil {
        return err
    }
    // get all ok profile
    for _, node := range nodes {
        if node.IsOk {
			fmt.Println(node.Remarks)
		}
	}
    return nil
}
```
