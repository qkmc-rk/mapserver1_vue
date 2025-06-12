<template>
  <div>
    <button class="chat-toggle" @click="show = !show">💬 专家咨询</button>
    <div v-if="show" class="chat-window">
      <div class="chat-header">
        <span>耕地粮食保护专家</span>
        <button class="close-btn" @click="show = false">×</button>
      </div>
      <div class="chat-messages">
        <div v-for="(msg, idx) in messages" :key="idx" class="chat-msg">
          <span>{{ msg }}</span>
        </div>
      </div>
      <div class="chat-input">
        <input v-model="input" @keyup.enter="send" placeholder="我是耕地保护专家，你有什么想咨询的吗?" />
        <button @click="send">发送</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Chat",
  data() {
    return {
      show: false,
      input: "",
      messages: [],
      loadingTimer: null,
      loadingStep: 0
    };
  },
  methods: {
    async send() {
      if (!this.input.trim()) return;
      const srname = localStorage.getItem('srname') || '';
      const prompt = srname
        ? `水系名称：${srname}。用户提问：${this.input.trim()}。记住你是一个河流方面的专家，回答要科学严谨且简介有力，要以专家的口吻和身份来回答，省略思考推理的流程.`
        : this.input.trim();
      this.messages.push(`我：${this.input.trim()}`);
      this.input = "";

      // 插入“请等待...”并循环动画
      let loadingMsg = "专家：请等待";
      this.messages.push(loadingMsg);
      const aiIdx = this.messages.length - 1;
      this.loadingStep = 0;
      this.loadingTimer = setInterval(() => {
        this.loadingStep = (this.loadingStep + 1) % 4;
        this.messages[aiIdx] = "专家：请等待" + ".".repeat(this.loadingStep);
      }, 500);

      try {
        const url = `http://localhost:8080/ai/generate?message=${encodeURIComponent(prompt)}`;
        const res = await fetch(url, { method: 'GET' });
        if (!res.ok) throw new Error('网络错误');
        const data = await res.json();
        clearInterval(this.loadingTimer);
        this.messages[aiIdx] = "专家：" + (data.generation || "抱歉，未获取到有效回复。");
      } catch (e) {
        clearInterval(this.loadingTimer);
        this.messages[aiIdx] = "专家：抱歉，服务暂时不可用。";
        console.error(e);
      }
    }
  }
};
</script>

<style scoped>
.chat-toggle {
  position: fixed;
  top: 30px;
  left: 20px;
  z-index: 9999;
  background: #409eff;
  color: #fff;
  border: none;
  border-radius: 20px;
  padding: 8px 18px;
  cursor: pointer;
  font-size: 16px;
}
.chat-window {
  position: fixed;
  top: 70px;
  left: 20px;
  width: 320px;
  background: #fff;
  border: 1px solid #eee;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.15);
  z-index: 10000;
  display: flex;
  flex-direction: column;
}
.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background: #409eff;
  color: #fff;
  border-radius: 8px 8px 0 0;
}
.close-btn {
  background: transparent;
  border: none;
  color: #fff;
  font-size: 20px;
  cursor: pointer;
}
.chat-messages {
  flex: 1;
  padding: 10px;
  max-height: 200px;
  overflow-y: auto;
  background: #f7f7f7;
}
.chat-msg {
  margin-bottom: 8px;
}
.chat-input {
  display: flex;
  border-top: 1px solid #eee;
  padding: 8px;
  background: #fafafa;
}
.chat-input input {
  flex: 1;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 6px;
  margin-right: 6px;
}
.chat-input button {
  background: #409eff;
  color: #fff;
  border: none;
  border-radius: 4px;
  padding: 6px 12px;
  cursor: pointer;
}
</style>