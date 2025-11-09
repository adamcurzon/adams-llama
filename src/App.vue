<template>
  <nav>
    Adam's Llama
    <div :class="['status-loader', status]"></div>
  </nav>
  <main>
    <div v-for="response in responses" :key="response">
      <MarkdownRenderer :source="response" />
    </div>
  </main>
  <textarea v-model="prompt" placeholder="Enter your prompt..." :disabled="status != 'done'"></textarea>
  <button @click="sendPrompt" :disabled="status != 'done'">Send</button>
</template>

<script>
import MarkdownRenderer from './components/ResponseMessage.vue';

const STATUS = {
  THINKING: 'thinking',
  ANSWERING: 'answering',
  DONE: 'done'
}

const HOST = "http://localhost";
const PORT = 11434;
const MODEL = "qwen3:4b";

export default {
  name: 'App',
  components: {
    MarkdownRenderer
  },
  data() {
    return {
      prompt: "",
      status: STATUS.DONE,
      responses: [],
    }
  },
  methods: {
    async sendPrompt() {
      const response = await fetch(HOST + ":" + PORT + "/api/generate", {
        method: "POST",
        body: JSON.stringify({
          "model": MODEL,
          "prompt": this.prompt
        })
      });

      if (!response.ok) {
        throw new Error(`HTTP error: ${response.status}`);
      }

      this.responses.push("");

      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      var reading = true;
      let buffer = '';

      while (reading) {
        const { value, done } = await reader.read();
        if (done) break;
        buffer += decoder.decode(value, { stream: true });
        const lines = buffer.split('\n');
        buffer = lines.pop();

        for (const line of lines) {
          console.log(line);
          const chunk = JSON.parse(line);
          if (chunk.thinking) {
            this.status = STATUS.THINKING;
          } else {
            this.status = STATUS.ANSWERING;
            this.responses[this.responses.length - 1] += chunk.response;
          }
        }
      }
      this.status = STATUS.DONE;
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  font-family: sans-serif;
}

body {
  margin: 0 auto;
  background-color: #34495e;
  color: white;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}

nav {
  width: 100%;
  height: 50px;
  background-color: #2c3e50;
  line-height: 50px;
  text-align: left;
  padding: 0px 20px;
  font-weight: 700;
}

main {
  height: calc(100vh - 50px - 80px);
  overflow-y: scroll;
}

main::-webkit-scrollbar {
  width: 6px;
}

main::-webkit-scrollbar-track {
  background: transparent;
}

main::-webkit-scrollbar-thumb {
  background-color: rgba(100, 100, 100, 0.5);
  border-radius: 3px;
}

main::-webkit-scrollbar-thumb:hover {
  background-color: rgba(100, 100, 100, 0.8);
}

textarea {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 80px;
  resize: none;
  border: 0px;
  padding: 10px;
  outline: 0px;
}

button {
  position: fixed;
  width: 160px;
  height: 80px;
  right: 0;
  bottom: 0;
  border-radius: 0;
  border: 0;
}

.status-loader {
  position: fixed;
  right: 20px;
  top: 15px;
  width: 20px;
  height: 20px;
  border-radius: 100%;
}

.status-loader.done {
  animation: none;
  background: #27ae60;
  box-shadow: 0px 0px 3px #27ae60;
  animation: pulse 1s infinite linear;
}

.status-loader.thinking {
  animation: none;
  background: #e67e22;
  box-shadow: 0px 0px 3px #e67e22;
  animation: pulse-thinking 1s infinite linear;
}

@keyframes pulse-thinking {
  0% {
    box-shadow: 0px 0px 3px #e67e22;
  }

  100% {
    box-shadow: 0px 0px 10px #e67e22;
  }
}

.status-loader.answering {
  animation: none;
  background: #3498db;
  box-shadow: 0px 0px 3px #3498db;
  animation: pulse-answering 1s infinite linear;
}

@keyframes pulse-answering {
  0% {
    box-shadow: 0px 0px 3px #3498db;
  }

  100% {
    box-shadow: 0px 0px 10px #3498db;
  }
}
</style>
