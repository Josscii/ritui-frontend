<script setup lang="ts">
import { ref, onMounted } from "vue";

const daily = ref<{
  id: number;
  text: string;
  nickName: string;
  avatarUrl: string;
} | null>(null);

onMounted(async () => {
  await fetchDaily();
});

async function fetchDaily() {
  try {
    const res = await fetch("http://localhost:3001/wapi/daily");
    const json = await res.json();
    if ("data" in json) {
      daily.value = json.data;
    } else {
      daily.value = null;
    }
  } catch (error) {
    console.log(error);
  }
}

async function refuse() {
  await audit(2);
  await fetchDaily();
}

async function approve() {
  await audit(1);
  await fetchDaily();
}

async function audit(status: number) {
  if (!daily.value) {
    return;
  }
  try {
    await fetch("http://localhost:3001/wapi/audit", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        id: daily.value.id,
        status,
      }),
    });
  } catch (error) {
    console.log(error);
  }
}
</script>

<template>
  <div class="container h-full mx-auto pt-10 flex justify-center items-start">
    <div v-if="daily == null">没有要审核的内容</div>
    <div v-else class="w-80 border-2 rounded-md p-2">
      <h1 class="text-lg">审核</h1>
      <div class="my-2 border-y py-2">
        <div class="flex items-center">
          <img
            :src="daily.avatarUrl"
            alt="avatar"
            class="w-8 h-8 rounded-full"
          />
          <p class="ml-2">{{ daily.nickName }}</p>
        </div>
        <p class="rounded-md mt-2">{{ daily.text }}</p>
      </div>
      <div class="flex justify-end gap-2">
        <button class="text-red-500" @click="refuse">拒绝</button>
        <button class="text-green-500" @click="approve">通过</button>
      </div>
    </div>
  </div>
</template>

<style></style>
