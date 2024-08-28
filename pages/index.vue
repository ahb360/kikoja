<script setup lang="ts">
import { z } from 'zod'
import type { FormSubmitEvent } from '#ui/types'
import {reactive, type Ref, ref} from 'vue'

const name: Ref<string[]> = ref([])
const withWhom: Ref<string[]> = ref([])
const where: Ref<string[]> = ref([])
const when: Ref<string[]> = ref([])
const does: Ref<string[]> = ref([])
const result: Ref<string[]> = ref([])
const steps = [name, withWhom, where, when, does]
const stepsNames = ['کی', 'با کی', 'کجا', 'چه وقت', 'چه کار']

const step = ref(0)

const schema = z.object({
  inputString: z.string().min(2, 'حداقل دوتا حرف وارد کن!')
})

type Schema = z.output<typeof schema>

const state = reactive({
  inputString: undefined
})

async function onSubmit (event: FormSubmitEvent<Schema>) {
  // Do something with data
  steps[step.value].value.push(event.data.inputString)
  state.inputString = undefined
}

function isAddDisable (): boolean {
  if (step.value === 0) return false
  return steps[step.value].value.length > name.value.length - 1
}

const shuffleArray = (array) => {
  const shuffled = array.slice(); // Copy array to avoid mutating the original
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  return shuffled;
};


function generateSentences (): void {
  // Shuffle each array
  const shuffledName = shuffleArray(name.value);
  const shuffledWithWhom = shuffleArray(withWhom.value);
  const shuffledWhere = shuffleArray(where.value);
  const shuffledWhen = shuffleArray(when.value);
  const shuffledDoes = shuffleArray(does.value);

// Concatenate the values
  result.value = shuffledName.map((value, index) => {
    return `${value} و ${shuffledWithWhom[index]} ${shuffledWhen[index]} ${shuffledWhere[index]} ${shuffledDoes[index]}`;
  });
}

function reset() {
  step.value = 0
  name.value = []
  withWhom.value = []
  where.value = []
  when.value = []
  does.value = []
  result.value = []
}
</script>

<template>
<UForm
    v-if="result.length === 0"
    :schema="schema"
    :state="state"
    class="space-y-4"
    @submit="onSubmit"
>
  <div class="flex flex-col w-full pt-6">
    <h1 class="text-2xl font-bold mb-4">
      {{ stepsNames[step] }}
    </h1>
    <div
      class="flex flex-row gap-2"
    >
      <UInput
          v-if="!isAddDisable()"
          v-model="state.inputString"
      />
      <UButton
          v-if="!isAddDisable()"
          type="submit"
          :disabled="isAddDisable()"
      >
          اضافه کن
      </UButton>
      <div
          v-if="isAddDisable()"
      >
        دیگه وقتشه بری مرحله بعدی!
      </div>
    </div>
    <div
        class="my-5"
    >
      <ul>
        <li v-for="item in steps[step].value" :key="item.toString()">
          {{ item }}
        </li>
      </ul>
    </div>
  </div>

  <div class="mt-6">
    <UButton
        v-if="step < 4"
        block
        @click="(step < 4) ? step++ : console.log('error')"
    >
      مرحله بعد
    </UButton>

    <UButton
        v-if="step === 4"
        block
        @click="generateSentences()"
    >
      پایان
    </UButton>
  </div>
</UForm>
<div
    v-if="result.length > 0"
    class="py-12 px-4"
>
  <ul>
    <li v-for="item in result" :key="item.toString()">
      {{ item }}
    </li>
  </ul>
  <UButton
      block
      @click="reset()"
      class="mt-6"
  >
    از اول
  </UButton>
</div>
</template>