<script setup lang="ts">
import { ref, computed } from 'vue'
import type { IInterview } from '@/interfaces'
import { v4 as uuidv4 } from 'uuid'
import { getFirestore, setDoc, doc } from 'firebase/firestore'
import { useRouter } from 'vue-router'
import { useToast } from 'primevue/usetoast'
import { useUserStore } from '@/stores/user'

const toast = useToast()
const db = getFirestore()
const router = useRouter()
const userStore = useUserStore()

const company = ref<string>('')
const vacancyLink = ref<string>('')
const hrName = ref<string>('')
const contactTelegram = ref<string>('')
const contactWhatsApp = ref<string>('')
const contactPhone = ref<string>('')
const loading = ref<boolean>(false)

const addNewInterview = async (): Promise<void> => {
  try {
    loading.value = true
    const payload: IInterview = {
      id: uuidv4(),
      company: company.value,
      vacancyLink: vacancyLink.value,
      hrName: hrName.value,
      contactTelegram: contactTelegram.value,
      contactPhone: contactPhone.value,
      contactWhatsapp: contactWhatsApp.value,
      createdAt: new Date()
    }
    const userId = userStore.userId
    if (userId) {
      await setDoc(doc(db, `users/${userId}/interviews`, payload.id), payload)
    }
    router.push({ name: 'List' })
  } catch (error) {
    if (error instanceof Error) {
      toast.add({ severity: 'error', summary: 'Error', detail: error.message, life: 3000 })
    }
  } finally {
    loading.value = false
  }
}
const disabledSaveButton = computed<boolean>(() => {
  return !(company.value && vacancyLink && hrName)
})
</script>

<template>
  <div class="content-interview">
    <app-card>
      <template #title>Новое собеседование</template>
      <template #content>
        <app-input-text class="input mb-3" placeholder="Компания" v-model="company" />
        <app-input-text
          v-model="vacancyLink"
          class="input mb-3"
          placeholder="Описание вакансии (ссылка)"
        />
        <app-input-text v-model="hrName" class="input mb-3" placeholder="Контакт (имя)" />
        <app-input-text
          v-model="contactTelegram"
          class="input mb-3"
          placeholder="Telegram username HR"
        />
        <app-input-text
          v-model="contactWhatsApp"
          class="input mb-3"
          placeholder="WhatsApp телефон HR"
        />
        <app-input-text v-model="contactPhone" class="input mb-3" placeholder="Телефон HR" />
        <app-button
          @click="addNewInterview"
          label="Создать собеседование"
          :disabled="disabledSaveButton"
          :loading="loading"
        ></app-button>
      </template>
    </app-card>
  </div>
</template>

<style scoped>
.input {
  width: 100%;
}
.content-interview {
  max-width: 600px;
  margin: auto;
}
</style>
