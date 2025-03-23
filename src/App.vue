<script setup lang="ts">
import { ref, useTemplateRef, watch } from "vue";
import { useRouter, useRoute } from "vue-router";
import { X } from "lucide-vue-next";
import { totp } from "native-browser-otp";

const router = useRouter();
const route = useRoute();

const copyButtonRef = useTemplateRef("copyButton");
const secretInputRef = useTemplateRef("secretInputRef");

const errorMessage = ref<string>("");

const secretInput = ref<string>("");
const code = ref<string>("000000");
const timeRemaining = ref<number>(0);
let timerId = ref<number | undefined>(undefined);

const copyCode = async () => {
  try {
    await navigator.clipboard.writeText(code.value);
    if (copyButtonRef.value) {
      copyButtonRef.value.textContent = "Copied";
    }

    setTimeout(() => {
      if (copyButtonRef.value) {
        copyButtonRef.value.textContent = "Copy Code";
      }
    }, 700);
  } catch (err) {
    console.log(err);
  }
};

// Функция для обновления кода
const updateCode = async (secret: string) => {
  secretInput.value = secret;

  try {
    code.value = await totp(secret);
    errorMessage.value = "";
  } catch (err) {
    errorMessage.value = "Invalid secret";
    code.value = "000000";
    clearInterval(timerId.value);
    timerId.value = undefined;
    timeRemaining.value = 0;
  }
};

const clearSecret = () => {
  secretInput.value = "";
  errorMessage.value = "";
  code.value = "000000";
  router.push("/");
  secretInputRef.value?.focus();

  if (timerId) {
    clearInterval(timerId.value);
    timerId.value = undefined;
    timeRemaining.value = 0;
  }
};

// Функция для обновления времени
const updateTimeRemaining = () => {
  const currentTime = Math.floor(Date.now() / 1000);
  timeRemaining.value = 30 - (currentTime % 30);

  updateCode(secretInput.value);
};

const handlePath = (path: string) => {
  const secret = path.replace("/", "").toUpperCase().trim();

  if (/[\w]/.test(secret)) {
    router.push(`/${secret}`);

    if (!timerId.value) {
      timerId.value = setInterval(updateTimeRemaining, 1000);
    }

    updateCode(secret);
    updateTimeRemaining();
  }
};

// Функция для обработки ввода
const handleInput = (input: string) => {
  secretInput.value = input.toUpperCase().replace(/([^\w])/g, "");

  if (secretInput.value) {
    updateCode(secretInput.value);
    router.push(`/${secretInput.value}`);
  } else {
    clearSecret();
  }
};

// Слушаем изменения в route.path
watch(
  () => route.path,
  (newPath) => handlePath(newPath),
);

// Слушаем изменения в secretInput
watch(secretInput, (newSecretInput) => handleInput(newSecretInput));
</script>

<template>
  <div class="mx-4 flex h-dvh items-center justify-center select-none">
    <div class="relative flex w-full max-w-md flex-col gap-4">
      <h1 class="text-center text-4xl font-bold text-slate-600">
        Code Generator 2FA
      </h1>
      <div class="relative flex w-full items-center">
        <input
          ref="secretInputRef"
          v-model="secretInput"
          type="text"
          class="w-full rounded-md border-2 py-2 pr-9 pl-2 text-slate-600 outline-0"
          :class="
            errorMessage
              ? 'border-red-600 focus:border-red-600'
              : 'border-slate-600 focus:border-sky-600'
          "
          placeholder="S2HV2DTHMDVZ6WXRZ6AP35JBBAHSQPFD"
          spellcheck="false"
          data-1p-ignore
        />
        <X
          :size="24"
          @click="clearSecret"
          v-if="secretInput"
          class="absolute top-1/2 right-2 -translate-y-1/2 rounded-md bg-slate-600 text-white hover:bg-slate-500"
        />
        <p
          v-if="errorMessage"
          class="absolute -top-1/3 left-1/2 w-full -translate-x-1/2 rounded-t-md bg-red-600 px-2 text-center text-sm text-white"
        >
          {{ errorMessage }}
        </p>
      </div>

      <div class="flex items-center justify-between gap-4">
        <div
          class="flex w-full items-center justify-between gap-2 rounded-md border-2 border-slate-600 px-2 py-1 text-center text-2xl text-slate-600"
        >
          <div class="font-bold">
            {{ code }}
          </div>
          <div :class="timeRemaining <= 5 ? 'text-red-600' : 'text-green-600'">
            {{ timeRemaining }}s
          </div>
        </div>

        <div
          ref="copyButton"
          @click="copyCode"
          class="w-full rounded-md border-2 border-slate-700 bg-slate-700 px-2 py-1 text-center text-2xl font-bold text-white hover:border-slate-600 hover:bg-slate-600"
        >
          Copy Code
        </div>
      </div>

      <p class="flex items-center justify-center gap-1">
        Created by

        <a
          href="https://t.me/doroved_stories"
          target="_blank"
          class="flex items-center gap-1 font-bold text-slate-900 hover:text-slate-600"
        >
          <img src="/doroved.jpeg" class="size-6 rounded-full shadow-md" />
          doroved</a
        >
      </p>
    </div>
  </div>
</template>
