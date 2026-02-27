<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>移动端 AI 聊天</title>

<style>
body{
    margin:0;
    background:#111;
    display:flex;
    justify-content:center;
    align-items:center;
    height:100vh;
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto;
}

/* 手机壳 */
.phone{
    width:390px;
    height:844px;
    background:#000;
    border-radius:40px;
    padding:12px;
    box-shadow:0 0 40px rgba(0,0,0,0.8);
    display:flex;
}

/* 屏幕 */
.screen{
    background:#f2f2f7;
    border-radius:32px;
    flex:1;
    display:flex;
    flex-direction:column;
    overflow:hidden;
}

/* 顶部栏 */
.topbar{
    background:#fff;
    padding:8px 15px;
    text-align:center;
    font-weight:600;
    border-bottom:1px solid #ddd;
}

/* 设置区域 */
.settings{
    padding:10px;
    background:#fafafa;
    border-bottom:1px solid #ddd;
    font-size:12px;
}

.settings input, .settings select{
    width:100%;
    margin-bottom:6px;
    padding:6px;
    border-radius:6px;
    border:1px solid #ccc;
    font-size:12px;
}

.settings button{
    width:100%;
    padding:6px;
    border:none;
    border-radius:6px;
    background:#007aff;
    color:#fff;
    font-size:12px;
    margin-bottom:6px;
}

/* 聊天区域 */
.chat{
    flex:1;
    padding:10px;
    overflow-y:auto;
    display:flex;
    flex-direction:column;
    gap:8px;
}

.message{
    padding:8px 10px;
    border-radius:12px;
    max-width:75%;
    font-size:14px;
    line-height:1.4;
    word-wrap:break-word;
}

.user{
    background:#007aff;
    color:#fff;
    align-self:flex-end;
}

.assistant{
    background:#e5e5ea;
    color:#000;
    align-self:flex-start;
}

/* 输入栏 */
.inputbar{
    display:flex;
    padding:8px;
    background:#fff;
    border-top:1px solid #ddd;
}

.inputbar input{
    flex:1;
    padding:8px;
    border-radius:20px;
    border:1px solid #ccc;
    font-size:14px;
}

.inputbar button{
    margin-left:6px;
    padding:8px 12px;
    border:none;
    border-radius:20px;
    background:#007aff;
    color:#fff;
    font-size:14px;
}
</style>
</head>

<body>

<div class="phone">
<div class="screen">

<div class="topbar">AI 聊天</div>

<div class="settings">
<input id="baseUrl" placeholder="API 基础地址 (Base URL) 例如 https://api.openai.com/v1">
<input id="apiKey" placeholder="API 密钥 (API Key)">
<select id="modelSelect">
<option value="">请选择模型</option>
</select>
<button onclick="loadModels()">拉取模型列表</button>
</div>

<div class="chat" id="chat"></div>

<div class="inputbar">
<input id="userInput" placeholder="输入内容...">
<button onclick="sendMessage()">发送</button>
</div>

</div>
</div>

<script>
let messages = [];

/* 本地保存 */
window.onload = function(){
    document.getElementById("baseUrl").value = localStorage.getItem("baseUrl") || "";
    document.getElementById("apiKey").value = localStorage.getItem("apiKey") || "";
}

/* 拉取模型 */
async function loadModels(){
    const baseUrl = document.getElementById("baseUrl").value.trim();
    const apiKey = document.getElementById("apiKey").value.trim();

    if(!baseUrl || !apiKey){
        alert("请填写 Base URL 和 API Key");
        return;
    }

    localStorage.setItem("baseUrl", baseUrl);
    localStorage.setItem("apiKey", apiKey);

    try{
        const response = await fetch(baseUrl + "/models", {
            headers:{
                "Authorization":"Bearer " + apiKey
            }
        });

        const data = await response.json();

        const select = document.getElementById("modelSelect");
        select.innerHTML = "";

        if(data.data){
            data.data.forEach(model=>{
                const option = document.createElement("option");
                option.value = model.id;
                option.textContent = model.id;
                select.appendChild(option);
            });
        }else{
            alert("模型拉取失败");
        }
    }catch(e){
        alert("请求失败，请检查接口地址");
    }
}

/* 添加消息 */
function addMessage(content, role){
    const chat = document.getElementById("chat");
    const div = document.createElement("div");
    div.className = "message " + role;
    div.innerText = content;
    chat.appendChild(div);
    chat.scrollTop = chat.scrollHeight;
}

/* 发送消息 */
async function sendMessage(){
    const input = document.getElementById("userInput");
    const content = input.value.trim();
    if(!content) return;

    const baseUrl = document.getElementById("baseUrl").value.trim();
    const apiKey = document.getElementById("apiKey").value.trim();
    const model = document.getElementById("modelSelect").value;

    if(!model){
        alert("请选择模型");
        return;
    }

    addMessage(content,"user");
    messages.push({role:"user",content});
    input.value="";

    try{
        const response = await fetch(baseUrl + "/chat/completions",{
            method:"POST",
            headers:{
                "Content-Type":"application/json",
                "Authorization":"Bearer " + apiKey
            },
            body:JSON.stringify({
                model:model,
                messages:messages
            })
        });

        const data = await response.json();

        if(data.choices){
            const reply = data.choices[0].message.content;
            addMessage(reply,"assistant");
            messages.push({role:"assistant",content:reply});
        }else{
            addMessage("接口返回异常","assistant");
        }
    }catch(e){
        addMessage("请求失败，请检查接口","assistant");
    }
}

/* 回车发送 */
document.getElementById("userInput").addEventListener("keypress",function(e){
    if(e.key==="Enter"){
        sendMessage();
    }
});
</script>

</body>
</html>
