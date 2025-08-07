<template>
  <div class="chat-container">
    <div class="messages" v-for="message in messages" :key="message.id">
      <p :class="message.type">{{ message.text }}</p>
    </div>
    <input
      v-model="userInput"
      @keyup.enter="sendMessage"
      placeholder="Type your message..."
    />
  </div>
</template>
<script setup lang="ts">
import { ref } from "vue";

const nextId = ref(1);
const userInput = ref("");
const messages = ref<{ id: number; text: string; type: "user" | "bot" }[]>([]);

const sendMessage = async () => {
  if (!userInput.value) return;

  messages.value.push({
    id: nextId.value++,
    text: userInput.value,
    type: "user",
  });

  try {
    const apiKey = import.meta.env.VITE_MISTRAL_API_KEY;
    if (!apiKey) {
      throw new Error("API key is not set");
    }

    const response = await fetch("https://api.mistral.ai/v1/chat/completions", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${apiKey.trim()}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        model: "mistral-tiny",
        messages: [{ role: "user", content: userInput.value }],
      }),
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(
        `API Error: ${data.error?.message || response.statusText}`
      );
    }

    messages.value.push({
      id: nextId.value++,
      text: data.choices[0].message.content,
      type: "bot",
    });
  } catch (error: { message: string } | any) {
    console.error("API Error:", error);
    messages.value.push({
      id: nextId.value++,
      text: `Error: ${error.message}`,
      type: "bot",
    });
  }

  userInput.value = "";
};
</script>
<style scoped>
.chat-container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}
.messages p {
  padding: 10px;
  margin: 5px 0;
  border-radius: 5px;
  color: black;
}
.user {
  background: #d1e7ff;
  text-align: right;
}
.bot {
  background: #f1f1f1;
  text-align: left;
}
input {
  width: 100%;
  padding: 12px;
  margin-top: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  box-sizing: border-box;
}
input:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 0 2px rgba(74, 144, 226, 0.2);
}
</style>
