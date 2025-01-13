<template>
  <div style="display: flex; flex-direction: column;
  justify-content: center; gap: 18px">
    <textarea v-model="respVal" style="overflow-y: auto; width: 500px; min-height: 300px"></textarea>

    <div>
      <input v-model="chatSession" style="width: 420px; height: 35px" placeholder="请输入sessionId(字母或者数字)"/>
    </div>
    <div style="display: flex; justify-content: space-between; width: 500px">
      <input v-model="inputVal" style="width: 420px; height: 35px"/>
      <button style="height: 35px" :disabled="disabled" @click="btnClick()">发送</button>
      <button style="height: 35px"  @click="closeClick()">关闭</button>
    </div>
    <div style="border-bottom: solid 1px #aaaaaa; width: 500px"></div>
    <div style="color: #aaaaaa">
      <h2>流水API说明（原一次性返回大模型结果接口不变）</h2>
      <ul>
        <li>1. npm install @microsoft/fetch-event-source</li>
        <li>2. 每次用户输入完成问题，点击提交按钮后，调用例子中：btnClick</li>
        <li>3. 需要携带header:
          "Content-Type": 'application/json',  // 固定值
          "Accept": 'text/event-stream',       //固定值
          "Ap": 'base',                        // 固定值
          "Authorization": 'Bearer ff61d3e5afd0f0f0bfa947325afe9ea6ba43393fc343782aeee3cba5441d67b47f00eb753062f95e0d68cb19c675bd4727f1a809c64bd814d4fd785e0a2668b114c24910cff57c04c27a3d314997a075008f3731a6868db2e6eccc9b882bae7fe2b320f7981c8b7302c66e2b90788d9239ecad3d6e9f3ca3b2e7a05ddf036903045ec73c09d93afb403661296ed3a1e23639f826ddd7c358e689191a88812ed004b26b4e30377a716f844e5fe7bb14a099973b5d6b02c9cfb07e8a082260daea40eda0976b30cf1c3cc1bf355fa1dbf8550790466eec74eddd20d65623cced97720e8bf5bcf86ece3d0ed82048daf89cd5556ded906ec9718c68ed39dae1d12968e11b2415e4a4d845c4b5950100a68166f52e90ba8969b13216c929f79e0b77f3b1177989fdd58cf26861bce778e16f93cd8253eb1bd29d502b7e71bc309301649bcb9640a5df4d6f971e205c041dfc7fa69d5a0c06b2869572a0ddfdf30c5d',
          "Cid": '1030560',                    //固定值
          "Slt": 'GHf__NKHt-e1W-bTyCxZj'</li>
        <li>4.收到后端消息后才能允许停止，消息开始标志：data:[start]</li>
        <li>5.消息结束标志：data:[end]</li>
        <li>6.消息结束标志：data:[end]</li>
      </ul>
    </div>

  </div>
<!--    <div class="container">
        <Header @send-todo="getList"></Header>
        <List v-bind="{list}"  :deleteFun="deleteFun" :checkTodo="checkTodo"></List>
        <Footer :list="list" :clearAll="clearAll" ></Footer>
    </div>-->
</template>

<script lang="ts" name="App" setup>
// 先npm install @microsoft/fetch-event-source
import { reactive,onBeforeMount, ref } from 'vue';
// import { EventSourcePolyfill } from 'event-source-polyfill'
import {EventStreamContentType, fetchEventSource} from '@microsoft/fetch-event-source';
let signal = null
var ctrlAbout
let source = Promise<void>;
let list=reactive<object[]>([])
const inputVal = ref("")
const chatSession = ref('')
const respVal = ref('')
const disabled = ref(false)
const btnClick = async () => {
  console.log(">>> input ", inputVal.value)
  console.log(">> btnClick")

  if(!inputVal.value) {
    alert("输入不能为空");
  }
  if(!chatSession.value) {
    alert("sessionId 不能为空");
  }
  const data = {
    "chatId": chatSession.value,
    "stream": true,
    "deatail":false,
    "fastType":"optical",
    "messages":[
      {"role":"user","content": inputVal.value }
    ]
  }

  //source = new EventSourcePolyfill('/api/xcg/sse/subscribe?id=1234', {
  // 场景1：get方法可以添加Header
  /* source = new EventSourcePolyfill('/api/xcg/sse/subscribe?id=1234', {
    heartbeatTimeout: 30000,
    headers: {

      "Accept": 'text/event-stream',
      "Ap": 'base',
      "Authorization": 'Bearer ff61d3e5afd0f0f0bfa947325afe9ea6ba43393fc343782aeee3cba5441d67b47f00eb753062f95e0d68cb19c675bd4727f1a809c64bd814d4fd785e0a2668b114c24910cff57c04c27a3d314997a075008f3731a6868db2e6eccc9b882bae7fe2b320f7981c8b7302c66e2b90788d9239ecad3d6e9f3ca3b2e7a05ddf036903045ec73c09d93afb403661296ed3a1e23639f826ddd7c358e689191a88812ed004b26b4e30377a716f844e5fe7bb14a099973b5d6b02c9cfb07e8a082260daea40eda0976b30cf1c3cc1bf355fa1dbf8550790466eec74eddd20d65623cced97720e8bf5bcf86ece3d0ed82048daf89cd5556ded906ec9718c68ed39dae1d12968e11b2415e4a4d845c4b5950100a68166f52e90ba8969b13216c929f79e0b77f3b1177989fdd58cf26861bce778e16f93cd8253eb1bd29d502b7e71bc309301649bcb9640a5df4d6f971e205c041dfc7fa69d5a0c06b2869572a0ddfdf30c5d',
      "Cid": '1030560',
      "Slt": 'GHf__NKHt-e1W-bTyCxZj'
    }
  });
  source.addEventListener('open', () => {
    console.log('SSE连接成功')
  })

  // 接收消息回调
  source.addEventListener('message', (event: any) => {
    try {
      console.log(data)
    } catch (error) {
      console.error('解析消息失败:', error)
    }
  })

  // 错误处理
  source.addEventListener('error', (error: any) => {
    console.error('SSE连接错误:', error)

  })
  */
  // 场景2： 普通get方法，
  /*var source = new EventSource('/api/xcg/sse/subscribe?id=1234');
  if (source) {
    source.onmessage = function (event) {
      console.log(">>> resp: ", event.data)
    };
    source.close = function () {
      console.log(">>> close")
    };
    source.onerror = function (e) {
      console.log(">>> error, ", e)
    }
    //连接上服务端回调
    source.onopen = function (event) {
      console.log('>>> open success, ', event);
    };
  }*/


  // 场景3：POST方法
  ctrlAbout = new AbortController();
  var { signal } = ctrlAbout;
  // fetchEventSource('/api/xcg/sse/chat', {
  // 调试
   var source = await fetchEventSource('/api/xcg/sse/chat', {
   //source =  await fetchEventSource('/xcg/sse/chat', {
    method: 'POST',
    signal: signal,
    openWhenHidden: true,
    headers: {
      "Content-Type": 'application/json',
      "Accept": 'text/event-stream',
      "Ap": 'base',
      "chatSession": '', //每次打开会话id，（从后台获取的）
      "Authorization": 'Bearer ff61d3e5afd0f0f0bfa947325afe9ea6ba43393fc343782aeee3cba5441d67b47f00eb753062f95e0d68cb19c675bd4727f1a809c64bd814d4fd785e0a2668b114c24910cff57c04c27a3d314997a075008f3731a6868db2e6eccc9b882bae7fe2b320f7981c8b7302c66e2b90788d9239ecad3d6e9f3ca3b2e7a05ddf036903045ec73c09d93afb403661296ed3a1e23639f826ddd7c358e689191a88812ed004b26b4e30377a716f844e5fe7bb14a099973b5d6b02c9cfb07e8a082260daea40eda0976b30cf1c3cc1bf355fa1dbf8550790466eec74eddd20d65623cced97720e8bf5bcf86ece3d0ed82048daf89cd5556ded906ec9718c68ed39dae1d12968e11b2415e4a4d845c4b5950100a68166f52e90ba8969b13216c929f79e0b77f3b1177989fdd58cf26861bce778e16f93cd8253eb1bd29d502b7e71bc309301649bcb9640a5df4d6f971e205c041dfc7fa69d5a0c06b2869572a0ddfdf30c5d',
      "Cid": '1030560',
      "Slt": 'GHf__NKHt-e1W-bTyCxZj'
    },
    body: JSON.stringify(data),
    onmessage(event) {
      respVal.value += event.data;
      console.info(event.data);
      // 在这里操作流式数据
    },
    async onopen(response) {
      if (response.ok && response.headers.get('content-type') === EventStreamContentType) {
        disabled.value = true;
        console.log(">>> openSuccess")
        return; // everything's good
      } else if (response.status >= 400 && response.status < 500 && response.status !== 429) {
        console.log(">>> openErr")
        // client-side errors are usually non-retriable:
        // throw new FatalError();
      } else {
        console.log(">>> open fail")
        // throw new RetriableError();
      }
    },
    onclose() {
      // 服务端关闭
      console.log("closed")
      // 关闭流
    },
    onerror(error) {
      console.info(error);
      //返回流报错
    }
  })
}

// 关闭
const closeClick = () => {
  ctrlAbout.abort()

}
onBeforeMount(()=>{


})
// 添加任务

</script>
    
<style>
    .container{
        display: inline-block;
        border: 1px solid #aaa;
        padding: 15px;
    }
</style>