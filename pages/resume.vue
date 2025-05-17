<template>
  <UCard>
    <div v-if="loading" class="flex justify-center p-8">
      <USpin class="w-8 h-8" />
    </div>

    <div v-else-if="error" class="text-red-500 p-4">
      Ошибка загрузки данных: {{ error }}
    </div>

    <div v-else class="flex flex-col gap-y-4">
      <!-- Иконки файлов -->
      <div class="flex gap-2">
        <UIcon
            v-for="(icon, index) in fileIcons"
            :key="index"
            :class="icon.class"
            :name="icon.name"
        />
      </div>

      <!-- Основная информация -->
      <div class="flex flex-col md:flex-row gap-6">
        <!-- Аватар -->
        <div class="flex-shrink-0">
          <NuxtImg
              class="h-50 w-50 border border-gray-200"
              src="/logo.svg"
              alt="Аватар пользователя"
          />
        </div>

        <!-- Информация -->
        <div class="flex-1 flex items-center">
          <div class="mb-4">
            <h1 class="text-2xl font-bold text-white-900">{{ resumeData.name || 'Имя не указано' }}</h1>
            <div class="flex flex-col gap-2 mt-2 text-sm text-gray-500">
              <div class="w-full flex gap-10">
                <span>Кандидат на вакансию: <span class="text-blue-600">Жадный благотворитель</span></span>
                <span class="text-blue-600">({{ formatDate(resumeData.date) || 'Не указано' }})</span>
              </div>
              <span>{{ translateStatus(resumeData.status) || 'Не указано' }}</span>
              <span>{{ getAge(resumeData.birth_date || '') || 'Возраст не указан' }}</span>
              <div class="flex items-center">
                <div class="flex items-center">
                  <UIcon :name="contacts.phone.icon" :class="contacts.phone.class" />
                  <span class="text-blue-600">{{ resumeData.phone || 'Не указано' }}</span>
                </div>
                <div class="flex items-center gap-2 ml-6">
                  <UIcon
                      v-for="(messenger, key) in contacts.messengers"
                      :key="key"
                      :class="messenger.class"
                      :name="messenger.name"
                  />
                </div>
              </div>
              <div class="flex items-center">
                <UIcon :name="contacts.email.icon" :class="contacts.email.class" />
                <span class="text-blue-300">{{ resumeData.email || 'Не указано' }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Дела -->
      <div class="space-y-3 sm:space-y-4">
        <h3 class="text-lg font-semibold text-gray-900">Дела:</h3>
        <div class="flex flex-wrap gap-5">
          <UButton
              v-for="(action, index) in actions"
              :key="index"
              :icon="action.icon"
              :label="action.label"
              :color="action.color"
              :variant="action.variant"
              :class="action.class"
          />
        </div>
      </div>

      <!-- Статусы -->
      <div class="space-y-4">
        <h2 class="text-lg font-semibold">Статус рассмотрения:</h2>
        <div class="flex flex-wrap gap-2">
          <div
              v-for="(status, index) in statuses.firstRow"
              :key="index"
              :class="status.class"
          >
            {{ status.text }}
          </div>
        </div>
        <div class="flex flex-wrap gap-2">
          <div
              v-for="(status, index) in statuses.secondRow"
              :key="index"
              :class="status.class"
          >
            {{ status.text }}
          </div>
        </div>
      </div>

      <!-- Pikabu отклик -->
      <div class="space-y-6">
        <div>
          <h2 class="text-xl font-semibold">Pikabu отклик</h2>
          <p class="text-gray-500 text-sm mt-1">Отклик с портала pikabu.</p>
        </div>

        <div class="flex flex-col gap-y-5">
          <div class="flex gap-2">
            <p class="text-sm text-gray-500">Дата рождения:</p>
            <p class="text-sm text-gray-500">{{ resumeData.birth_date || 'Не указано' }}</p>
          </div>
          <div class="flex gap-2">
            <p class="text-sm text-gray-500">Гражданство:</p>
            <p class="text-sm text-gray-500">{{ resumeData.citizenship || 'Не указано' }}</p>
          </div>
        </div>

        <div>
          <h3 class="font-medium mb-2">Сопроводительное письмо</h3>
          <div class="p-4 bg-gray-50 rounded-lg border border-gray-200 text-gray-700">
            <p class="text-gray-500 italic">{{ extractDescription(resumeData.description) || 'Не указано' }}</p>
          </div>
        </div>

        <div>
          <h3 class="font-medium mb-2">Файлы портфолио:</h3>
          <div class="flex flex-wrap gap-3">
            <UButton
                icon="i-heroicons-document-text"
                label="Резюме"
                variant="outline"
                color="gray"
                class="cursor-pointer"
            />
          </div>
        </div>
      </div>
    </div>
  </UCard>
</template>

<script setup lang="ts">

const config = useRuntimeConfig()

interface ResumeData {
  id?: number
  name?: string
  description?: string
  date?: string
  status?: string
  portfolios?: null
  town?: string
  phone?: string
  age?: number
  birth_date?: string
  email?: string
  listing_id?: number
  photo?: string
  citizenship?: string
  cover_letter?: string
  resume_file?: string
}

const loading = ref<boolean>(true)
const error = ref<string | null>(null)
const resumeData = ref<ResumeData>({})

const formatDate = (dateString?: string): string => {
  if (!dateString) return ''
  try {
    const date = new Date(dateString)
    return date.toLocaleDateString('ru-RU')
  } catch {
    return dateString
  }
}

const translateStatus = (status?: string): string => {
  if (!status) return 'Не указано';

  const statusMap: Record<string, string> = {
    viewed: 'Просмотрено',
    new: 'Новое',
    rejected: 'Отклонено',
    accepted: 'Принято',
    interview: 'Собеседование',
    pending: 'На рассмотрении',


  };

  return statusMap[status.toLowerCase()] || status;
};

function getAge(birthDate: string): number {
  const today = new Date();
  const birth = new Date(birthDate);

  let age = today.getFullYear() - birth.getFullYear();

  const hasBirthdayPassed =
      today.getMonth() > birth.getMonth() ||
      (today.getMonth() === birth.getMonth() && today.getDate() >= birth.getDate());

  if (!hasBirthdayPassed) {
    age--;
  }

  return age;
}

const fetchResumeData = async (): Promise<void> => {
  try {
    loading.value = true
    const response = await fetch(`${config.public.apiBase}/test/v2/app`)

    const data: ResumeData = await response.json()
    resumeData.value = data

    if (data.description) {
      const citizenshipMatch = data.description.match(/Гражданство: (.*?)\r\n/)
      if (citizenshipMatch) {
        resumeData.value.citizenship = citizenshipMatch[1]
      }
    }
  } catch (err) {
    console.error('Ошибка загрузки данных:', err)
  } finally {
    loading.value = false
  }
}

const extractDescription = (text?: string): string => {
  if (!text) return '';

  const lines = text.split('\r\n');

  for (const line of lines) {
    if (line.startsWith('Описание:')) {
      return line.substring('Описание:'.length).trim();
    }
  }

  return '';
};

onMounted(() => {
  fetchResumeData()
})

// Статические данные
const fileIcons = [
  { name: 'vscode-icons:default-file', class: 'text-red-500 text-3xl cursor-pointer' },
  { name: 'vscode-icons:file-type-pdf2', class: 'text-red-500 text-3xl cursor-pointer' },
  { name: 'vscode-icons:file-type-word2', class: 'text-red-500 text-3xl cursor-pointer' },
  { name: 'vscode-icons:file-type-aspx', class: 'text-red-500 text-3xl cursor-pointer' },
  { name: 'heroicons-solid:archive-box-x-mark', class: 'text-red-500 text-3xl cursor-pointer' },
  { name: 'mingcute:send-fill', class: 'text-white-500 text-3xl cursor-pointer' },
  { name: 'icon-park-solid:like', class: 'text-white-500 text-3xl cursor-pointer' }
]

const contacts = {
  phone: { icon: 'i-heroicons-phone', class: 'text-yellow-400 mr-2 w-5 h-5' },
  email: { icon: 'i-heroicons-envelope', class: 'text-yellow-400 mr-2 w-5 h-5' },
  messengers: [
    { name: 'ic:baseline-telegram', class: 'text-blue-500 w-6 h-6' },
    { name: 'basil:viber-solid', class: 'text-purple-500 w-6 h-6' },
    { name: 'ic:baseline-whatsapp', class: 'text-green-500 w-6 h-6' }
  ]
}

const actions = [
  {
    icon: 'i-heroicons-chat-bubble-left-right',
    label: 'Собеседование запланировано',
    color: 'gray',
    variant: 'soft',
    class: 'min-h-12 sm:min-h-15 border cursor-pointer hover:bg-green-500 w-60 transition-colors duration-300'
  },
  {
    icon: 'i-heroicons-video-camera',
    label: 'Создать видеозвонок',
    color: 'gray',
    variant: 'soft',
    class: 'min-h-12 sm:min-h-15 border cursor-pointer hover:bg-green-500 w-60 transition-colors duration-300'
  },
  {
    icon: 'i-heroicons-calendar',
    label: 'Запланировать событие',
    color: 'gray',
    variant: 'soft',
    class: 'min-h-12 sm:min-h-15 border cursor-pointer hover:bg-green-500 w-60 transition-colors duration-300'
  },
  {
    icon: 'i-heroicons-paper-airplane',
    label: 'Отправить запрос',
    color: 'gray',
    variant: 'soft',
    class: 'min-h-12 sm:min-h-15 border cursor-pointer hover:bg-green-500 w-60 transition-colors duration-300'
  }
]

const statuses = {
  firstRow: [
    { text: 'Новое', class: 'px-3 py-1.5 cursor-pointer rounded-full border font-medium text-blue-700 text-sm' },
    { text: 'Просмотрено', class: 'px-3 cursor-pointer py-1.5 rounded-full border font-medium text-blue-700 text-sm' },
    { text: 'Проведено собеседование', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' },
    { text: 'Отправлено приглашение', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' },
    { text: 'Назначено собеседование', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' },
    { text: 'Не дошел', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' }
  ],
  secondRow: [
    { text: 'Ожидание ответа соискателя', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' },
    { text: 'Принятие решения', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' },
    { text: 'Принят', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' },
    { text: 'Отклонено/Отказ', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' },
    { text: 'Архивировано', class: 'px-3 cursor-pointer py-1.5 rounded-full border text-sm' }
  ]
}
</script>