# 🎨 Frontend: Общие знания для разработки интерфейсов Битрикс24

## 📋 Обзор

Этот файл содержит **общую информацию по frontend-разработке** для приложений Битрикс24, не зависящую от выбранного backend. Для специфических инструкций обратитесь к соответствующим файлам в этой папке.

---

## 🌐 Frontend экосистема для Битрикс24

### Основные принципы дизайна

#### Битрикс24 Design System
- **Консистентность** с нативным интерфейсом портала
- **Адаптивность** для разных размеров экрана
- **Доступность** (accessibility) для пользователей с ограниченными возможностями
- **Производительность** для быстрой загрузки

#### Цветовая палитра
```css
:root {
  /* Основные цвета */
  --b24-primary-blue: #2fc6f6;
  --b24-success-green: #55d88a;
  --b24-warning-orange: #ffab00;
  --b24-danger-red: #ff5752;
  
  /* Нейтральные цвета */
  --b24-gray-50: #f8f9fa;
  --b24-gray-100: #e9ecef;
  --b24-gray-200: #dee2e6;
  --b24-gray-300: #ced4da;
  --b24-gray-400: #adb5bd;
  --b24-gray-500: #6c757d;
  --b24-gray-600: #495057;
  --b24-gray-700: #343a40;
  --b24-gray-800: #212529;
  --b24-gray-900: #000000;
  
  /* Фоновые цвета */
  --b24-bg-primary: #ffffff;
  --b24-bg-secondary: #f8f9fa;
  --b24-bg-accent: #e3f2fd;
}
```

### Технологический стек

#### Базовые технологии
- **HTML5** с семантической разметкой
- **CSS3** с Flexbox/Grid для компоновки
- **JavaScript (ES2022+)** или **TypeScript** для логики
- **Responsive Design** принципы

#### Рекомендуемые фреймворки/библиотеки
```json
{
  "dependencies": {
    "@bitrix24/b24ui-nuxt": "^2.0.0",
    "@bitrix24/b24icons-vue": "^1.0.0",
    "vue": "^3.3.0",
    "nuxt": "^3.8.0",
    "@nuxtjs/tailwindcss": "^6.8.0",
    "@vueuse/core": "^10.5.0"
  },
  "alternatives": {
    "react": "^18.2.0",
    "@emotion/react": "^11.11.0",
    "next": "^14.0.0"
  }
}
```

---

## 🎯 Базовая архитектура frontend

### Типичная структура проекта

```
frontend/
├── components/                # Переиспользуемые компоненты
│   ├── ui/                   # UI компоненты (кнопки, инпуты, модалы)
│   │   ├── B24Button.vue
│   │   ├── B24Card.vue
│   │   └── B24Table.vue
│   ├── business/             # Бизнес-компоненты
│   │   ├── DealCard.vue
│   │   ├── ContactList.vue
│   │   └── ActivityTimeline.vue
│   └── layout/               # Компоненты раскладки
│       ├── Header.vue
│       ├── Sidebar.vue
│       └── Footer.vue
├── pages/                    # Страницы приложения
│   ├── index.vue
│   ├── deals/
│   │   ├── index.vue
│   │   └── [id].vue
│   └── contacts/
├── composables/              # Переиспользуемая логика
│   ├── useBitrix24.ts
│   ├── useDeals.ts
│   └── useApi.ts
├── stores/                   # Управление состоянием
│   ├── deals.ts
│   ├── contacts.ts
│   └── auth.ts
├── types/                    # TypeScript типы
│   ├── deal.ts
│   ├── contact.ts
│   └── api.ts
├── utils/                    # Утилиты
│   ├── formatters.ts
│   ├── validators.ts
│   └── constants.ts
└── assets/                   # Статические ресурсы
    ├── css/
    ├── images/
    └── icons/
```

---

## 🧩 Основные UI компоненты

### 1. Bitrix24 UI Kit компоненты

> **⚠️ ВАЖНО**: Используйте компоненты с префиксом **B24*** из `@bitrix24/b24ui-nuxt`

#### Основные компоненты
```vue
<!-- Контейнеры -->
<B24App>          <!-- Корневой контейнер приложения -->
<B24Container>    <!-- Контейнер с отступами -->
<B24Card>         <!-- Карточка с тенью и границами -->

<!-- Навигация -->
<B24Breadcrumb>   <!-- Хлебные крошки -->
<B24Pagination>   <!-- Постраничная навигация -->
<B24Tabs>         <!-- Вкладки -->

<!-- Формы -->
<B24Input>        <!-- Поле ввода -->
<B24Textarea>     <!-- Многострочное поле -->
<B24Select>       <!-- Выпадающий список -->
<B24SelectMenu>   <!-- Продвинутый селектор -->
<B24Checkbox>     <!-- Чекбокс -->
<B24Radio>        <!-- Радиокнопка -->
<B24Toggle>       <!-- Переключатель -->
<B24Button>       <!-- Кнопка -->

<!-- Отображение данных -->
<B24Table>        <!-- Таблица -->
<B24Badge>        <!-- Бейдж -->
<B24Avatar>       <!-- Аватар пользователя -->
<B24Progress>     <!-- Индикатор прогресса -->
<B24Accordion>    <!-- Раскрывающийся список -->

<!-- Обратная связь -->
<B24Alert>        <!-- Уведомления -->
<B24Modal>        <!-- Модальные окна -->
<B24Tooltip>      <!-- Всплывающие подсказки -->
<B24Popover>      <!-- Поповеры -->

<!-- Макет -->
<B24Divider>      <!-- Разделитель -->
<B24Skeleton>     <!-- Скелетон для загрузки -->
```

### 2. Типичные паттерны использования

#### Список с фильтрацией
```vue
<template>
  <B24Container class="py-8">
    <!-- Заголовок и действия -->
    <div class="mb-6 flex items-center justify-between">
      <h1 class="text-2xl font-bold">Сделки</h1>
      <B24Button @click="openCreateModal">
        Создать сделку
      </B24Button>
    </div>

    <!-- Фильтры -->
    <B24Card class="mb-6">
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <B24SelectMenu
          v-model="filters.stage"
          :options="stageOptions"
          placeholder="Выберите стадию"
        />
        <B24Input
          v-model="filters.search"
          placeholder="Поиск по названию"
          icon="i-heroicons-magnifying-glass"
        />
        <B24Button
          variant="outline"
          @click="clearFilters"
        >
          Очистить фильтры
        </B24Button>
      </div>
    </B24Card>

    <!-- Список -->
    <B24Card>
      <B24Table
        :columns="columns"
        :rows="filteredDeals"
        :loading="isLoading"
      >
        <template #actions="{ row }">
          <div class="flex gap-2">
            <B24Button size="sm" @click="editDeal(row.id)">
              Редактировать
            </B24Button>
            <B24Button
              size="sm"
              color="red"
              variant="outline"
              @click="deleteDeal(row.id)"
            >
              Удалить
            </B24Button>
          </div>
        </template>
      </B24Table>

      <div class="mt-4">
        <B24Pagination
          v-model="page"
          :page-count="pageCount"
          :total="total"
        />
      </div>
    </B24Card>
  </B24Container>
</template>
```

#### Детальная карточка
```vue
<template>
  <B24Container class="py-8">
    <B24Breadcrumb :links="breadcrumbLinks" />
    
    <div class="mt-6 grid grid-cols-1 lg:grid-cols-3 gap-6">
      <!-- Основная информация -->
      <div class="lg:col-span-2">
        <B24Card>
          <template #header>
            <div class="flex items-center justify-between">
              <h2 class="text-xl font-semibold">{{ deal.title }}</h2>
              <B24Badge :color="stageColor">{{ stageName }}</B24Badge>
            </div>
          </template>

          <!-- Форма редактирования -->
          <div class="space-y-4">
            <B24Input
              v-model="form.title"
              label="Название сделки"
              required
            />
            <B24Input
              v-model="form.opportunity"
              label="Сумма"
              type="number"
              step="0.01"
            />
            <B24SelectMenu
              v-model="form.stageId"
              :options="stageOptions"
              label="Стадия сделки"
            />
          </div>

          <template #footer>
            <div class="flex gap-2">
              <B24Button @click="saveDeal" :loading="isSaving">
                Сохранить
              </B24Button>
              <B24Button variant="outline" @click="resetForm">
                Отменить
              </B24Button>
            </div>
          </template>
        </B24Card>
      </div>

      <!-- Боковая панель -->
      <div>
        <B24Card>
          <template #header>
            <h3 class="font-medium">Дополнительная информация</h3>
          </template>

          <div class="space-y-3">
            <div>
              <span class="text-sm text-gray-500">Дата создания:</span>
              <p>{{ formatDate(deal.dateCreate) }}</p>
            </div>
            <div>
              <span class="text-sm text-gray-500">Последнее изменение:</span>
              <p>{{ formatDate(deal.dateModify) }}</p>
            </div>
          </div>
        </B24Card>
      </div>
    </div>
  </B24Container>
</template>
```

---

## 💾 Управление состоянием

### 1. Pinia Store (рекомендуется для Vue/Nuxt)

```typescript
// stores/deals.ts
import { defineStore } from 'pinia';
import type { Deal, DealCreateData, DealFilters } from '~/types/deal';

export const useDealsStore = defineStore('deals', () => {
  // Состояние
  const deals = ref<Deal[]>([]);
  const currentDeal = ref<Deal | null>(null);
  const isLoading = ref(false);
  const error = ref<string | null>(null);
  
  // Фильтры
  const filters = ref<DealFilters>({
    stage: null,
    search: '',
    dateFrom: null,
    dateTo: null
  });

  // Геттеры
  const filteredDeals = computed(() => {
    let result = deals.value;
    
    if (filters.value.stage) {
      result = result.filter(deal => deal.stageId === filters.value.stage);
    }
    
    if (filters.value.search) {
      const search = filters.value.search.toLowerCase();
      result = result.filter(deal => 
        deal.title.toLowerCase().includes(search)
      );
    }
    
    return result;
  });

  const dealsByStage = computed(() => {
    return filteredDeals.value.reduce((acc, deal) => {
      if (!acc[deal.stageId]) {
        acc[deal.stageId] = [];
      }
      acc[deal.stageId].push(deal);
      return acc;
    }, {} as Record<string, Deal[]>);
  });

  // Действия
  async function fetchDeals() {
    isLoading.value = true;
    error.value = null;
    
    try {
      const { data } = await $fetch<{data: Deal[]}>('/api/deals');
      deals.value = data;
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Failed to fetch deals';
    } finally {
      isLoading.value = false;
    }
  }

  async function fetchDealById(id: string) {
    isLoading.value = true;
    error.value = null;
    
    try {
      const { data } = await $fetch<{data: Deal}>(`/api/deals/${id}`);
      currentDeal.value = data;
      return data;
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Deal not found';
      return null;
    } finally {
      isLoading.value = false;
    }
  }

  async function createDeal(dealData: DealCreateData) {
    isLoading.value = true;
    error.value = null;
    
    try {
      const { data } = await $fetch<{data: {id: string}}>('/api/deals', {
        method: 'POST',
        body: dealData
      });
      
      // Обновляем список
      await fetchDeals();
      
      return data.id;
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Failed to create deal';
      throw err;
    } finally {
      isLoading.value = false;
    }
  }

  async function updateDeal(id: string, updateData: Partial<Deal>) {
    isLoading.value = true;
    error.value = null;
    
    try {
      await $fetch(`/api/deals/${id}`, {
        method: 'PATCH',
        body: updateData
      });
      
      // Обновляем локальное состояние
      const index = deals.value.findIndex(deal => deal.id === id);
      if (index !== -1) {
        deals.value[index] = { ...deals.value[index], ...updateData };
      }
      
      if (currentDeal.value?.id === id) {
        currentDeal.value = { ...currentDeal.value, ...updateData };
      }
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Failed to update deal';
      throw err;
    } finally {
      isLoading.value = false;
    }
  }

  function setFilters(newFilters: Partial<DealFilters>) {
    filters.value = { ...filters.value, ...newFilters };
  }

  function clearFilters() {
    filters.value = {
      stage: null,
      search: '',
      dateFrom: null,
      dateTo: null
    };
  }

  return {
    // State
    deals: readonly(deals),
    currentDeal: readonly(currentDeal),
    isLoading: readonly(isLoading),
    error: readonly(error),
    filters,
    
    // Getters
    filteredDeals,
    dealsByStage,
    
    // Actions
    fetchDeals,
    fetchDealById,
    createDeal,
    updateDeal,
    setFilters,
    clearFilters
  };
});
```

### 2. Composables для переиспользования логики

```typescript
// composables/useApi.ts
export function useApi() {
  const isLoading = ref(false);
  const error = ref<string | null>(null);

  async function apiCall<T>(
    url: string, 
    options?: RequestInit
  ): Promise<T | null> {
    isLoading.value = true;
    error.value = null;
    
    try {
      const response = await $fetch<T>(url, options);
      return response;
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'API call failed';
      return null;
    } finally {
      isLoading.value = false;
    }
  }

  return {
    isLoading: readonly(isLoading),
    error: readonly(error),
    apiCall
  };
}

// composables/useDeals.ts  
export function useDeals() {
  const dealsStore = useDealsStore();
  const toast = useToast();

  // Действия с уведомлениями
  async function createDealWithNotification(dealData: DealCreateData) {
    try {
      const dealId = await dealsStore.createDeal(dealData);
      toast.add({
        title: 'Успешно',
        description: 'Сделка создана',
        color: 'green'
      });
      return dealId;
    } catch (error) {
      toast.add({
        title: 'Ошибка',
        description: 'Не удалось создать сделку',
        color: 'red'
      });
      throw error;
    }
  }

  async function updateDealWithNotification(
    id: string, 
    updateData: Partial<Deal>
  ) {
    try {
      await dealsStore.updateDeal(id, updateData);
      toast.add({
        title: 'Успешно',
        description: 'Сделка обновлена',
        color: 'green'
      });
    } catch (error) {
      toast.add({
        title: 'Ошибка',
        description: 'Не удалось обновить сделку',
        color: 'red'
      });
      throw error;
    }
  }

  // Валидация
  function validateDealData(data: Partial<DealCreateData>) {
    const errors: string[] = [];
    
    if (!data.title?.trim()) {
      errors.push('Название сделки обязательно');
    }
    
    if (data.opportunity !== undefined && data.opportunity < 0) {
      errors.push('Сумма не может быть отрицательной');
    }
    
    return errors;
  }

  return {
    // Store state
    ...storeToRefs(dealsStore),
    
    // Store actions
    ...dealsStore,
    
    // Enhanced actions
    createDealWithNotification,
    updateDealWithNotification,
    
    // Validation
    validateDealData
  };
}
```

---

## 🎨 Стилизация и темизация

### 1. Tailwind CSS конфигурация

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './components/**/*.{js,vue,ts}',
    './layouts/**/*.vue',
    './pages/**/*.vue',
    './plugins/**/*.{js,ts}',
    './nuxt.config.{js,ts}',
    './app.vue'
  ],
  theme: {
    extend: {
      colors: {
        // Bitrix24 цветовая схема
        primary: {
          50: '#e3f2fd',
          100: '#bbdefb',
          200: '#90caf9',
          300: '#64b5f6',
          400: '#42a5f5',
          500: '#2fc6f6', // основной синий
          600: '#1e88e5',
          700: '#1976d2',
          800: '#1565c0',
          900: '#0d47a1'
        },
        success: {
          50: '#e8f5e8',
          100: '#c8e6c9',
          200: '#a5d6a7',
          300: '#81c784',
          400: '#66bb6a',
          500: '#55d88a', // основной зеленый
          600: '#4caf50',
          700: '#43a047',
          800: '#388e3c',
          900: '#2e7d32'
        },
        warning: {
          50: '#fff8e1',
          100: '#ffecb3',
          200: '#ffe082',
          300: '#ffd54f',
          400: '#ffca28',
          500: '#ffab00', // основной оранжевый
          600: '#ffb300',
          700: '#ffa000',
          800: '#ff8f00',
          900: '#ff6f00'
        },
        danger: {
          50: '#ffebee',
          100: '#ffcdd2',
          200: '#ef9a9a',
          300: '#e57373',
          400: '#ef5350',
          500: '#ff5752', // основной красный
          600: '#e53935',
          700: '#d32f2f',
          800: '#c62828',
          900: '#b71c1c'
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif']
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem'
      }
    }
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography')
  ]
};
```

### 2. CSS Custom Properties для темизации

```css
/* assets/css/themes.css */
:root {
  /* Основная цветовая схема */
  --b24-primary: theme('colors.primary.500');
  --b24-primary-hover: theme('colors.primary.600');
  --b24-success: theme('colors.success.500');
  --b24-warning: theme('colors.warning.500');
  --b24-danger: theme('colors.danger.500');
  
  /* Текст */
  --b24-text-primary: theme('colors.gray.900');
  --b24-text-secondary: theme('colors.gray.600');
  --b24-text-muted: theme('colors.gray.500');
  
  /* Фоны */
  --b24-bg-primary: theme('colors.white');
  --b24-bg-secondary: theme('colors.gray.50');
  --b24-bg-accent: theme('colors.primary.50');
  
  /* Границы */
  --b24-border-color: theme('colors.gray.200');
  --b24-border-hover: theme('colors.gray.300');
  
  /* Тени */
  --b24-shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --b24-shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --b24-shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
}

/* Темная тема */
[data-theme="dark"] {
  --b24-text-primary: theme('colors.gray.100');
  --b24-text-secondary: theme('colors.gray.300');
  --b24-text-muted: theme('colors.gray.400');
  
  --b24-bg-primary: theme('colors.gray.900');
  --b24-bg-secondary: theme('colors.gray.800');
  --b24-bg-accent: theme('colors.gray.800');
  
  --b24-border-color: theme('colors.gray.700');
  --b24-border-hover: theme('colors.gray.600');
}

/* Утилитарные классы */
.bg-b24-primary { background-color: var(--b24-bg-primary); }
.bg-b24-secondary { background-color: var(--b24-bg-secondary); }
.text-b24-primary { color: var(--b24-text-primary); }
.text-b24-secondary { color: var(--b24-text-secondary); }
.border-b24 { border-color: var(--b24-border-color); }
```

---

## 📱 Адаптивный дизайн

### 1. Breakpoints и сетки

```css
/* Стандартные breakpoints */
@media (min-width: 640px) { /* sm */ }
@media (min-width: 768px) { /* md */ }
@media (min-width: 1024px) { /* lg */ }
@media (min-width: 1280px) { /* xl */ }
@media (min-width: 1536px) { /* 2xl */ }
```

```vue
<!-- Адаптивная сетка -->
<template>
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
    <!-- Карточки сделок -->
    <DealCard
      v-for="deal in deals"
      :key="deal.id"
      :deal="deal"
    />
  </div>
</template>
```

### 2. Мобильная навигация

```vue
<!-- components/layout/MobileNavigation.vue -->
<template>
  <div class="lg:hidden">
    <!-- Мобильное меню -->
    <B24Button
      variant="ghost"
      @click="isOpen = !isOpen"
      class="p-2"
    >
      <Icon name="i-heroicons-bars-3" />
    </B24Button>

    <!-- Выдвижная панель -->
    <USlideover v-model="isOpen" side="left">
      <div class="p-4">
        <nav class="space-y-2">
          <NuxtLink
            v-for="item in navigation"
            :key="item.to"
            :to="item.to"
            class="block px-4 py-2 rounded-lg hover:bg-gray-100"
            @click="isOpen = false"
          >
            {{ item.label }}
          </NuxtLink>
        </nav>
      </div>
    </USlideover>
  </div>
</template>
```

---

## ⚡ Производительность

### 1. Ленивая загрузка компонентов

```vue
<script setup>
// Ленивая загрузка тяжелых компонентов
const LazyDealChart = defineAsyncComponent(() => import('~/components/DealChart.vue'));
const LazyDataTable = defineAsyncComponent(() => import('~/components/DataTable.vue'));

const showChart = ref(false);
</script>

<template>
  <div>
    <!-- Основной контент загружается сразу -->
    <DealSummary :deals="deals" />
    
    <!-- Тяжелые компоненты загружаются по требованию -->
    <B24Button @click="showChart = true" v-if="!showChart">
      Показать график
    </B24Button>
    
    <LazyDealChart v-if="showChart" :data="chartData" />
  </div>
</template>
```

### 2. Виртуализация списков

```vue
<!-- components/VirtualizedDealList.vue -->
<template>
  <div class="h-96 overflow-auto">
    <RecycleScroller
      class="scroller"
      :items="deals"
      :item-size="80"
      key-field="id"
      v-slot="{ item }"
    >
      <DealListItem :deal="item" />
    </RecycleScroller>
  </div>
</template>

<script setup>
import { RecycleScroller } from 'vue-virtual-scroller';

defineProps<{
  deals: Deal[]
}>();
</script>
```

### 3. Оптимизация изображений

```vue
<template>
  <!-- Использование Nuxt Image для оптимизации -->
  <NuxtImg
    :src="user.avatar"
    :alt="user.name"
    width="40"
    height="40"
    class="rounded-full"
    loading="lazy"
    placeholder
  />
</template>
```

---

## 🔧 Утилиты и помощники

### 1. Форматтеры данных

```typescript
// utils/formatters.ts
export const formatters = {
  // Форматирование валюты
  currency(amount: number, currency: string = 'RUB'): string {
    return new Intl.NumberFormat('ru-RU', {
      style: 'currency',
      currency: currency,
      minimumFractionDigits: 0,
      maximumFractionDigits: 2
    }).format(amount);
  },

  // Форматирование даты
  date(date: string | Date, format: 'short' | 'long' = 'short'): string {
    const d = typeof date === 'string' ? new Date(date) : date;
    
    if (format === 'short') {
      return d.toLocaleDateString('ru-RU');
    }
    
    return d.toLocaleDateString('ru-RU', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    });
  },

  // Относительное время
  timeAgo(date: string | Date): string {
    const d = typeof date === 'string' ? new Date(date) : date;
    const now = new Date();
    const diff = now.getTime() - d.getTime();
    
    const minutes = Math.floor(diff / 60000);
    const hours = Math.floor(diff / 3600000);
    const days = Math.floor(diff / 86400000);
    
    if (minutes < 1) return 'только что';
    if (minutes < 60) return `${minutes} мин. назад`;
    if (hours < 24) return `${hours} ч. назад`;
    if (days < 7) return `${days} дн. назад`;
    
    return d.toLocaleDateString('ru-RU');
  },

  // Сокращение текста
  truncate(text: string, length: number = 100): string {
    if (text.length <= length) return text;
    return text.slice(0, length) + '...';
  },

  // Форматирование номера телефона
  phone(phone: string): string {
    const cleaned = phone.replace(/\D/g, '');
    
    if (cleaned.length === 11 && cleaned.startsWith('7')) {
      return `+7 (${cleaned.slice(1, 4)}) ${cleaned.slice(4, 7)}-${cleaned.slice(7, 9)}-${cleaned.slice(9)}`;
    }
    
    return phone;
  }
};
```

### 2. Валидаторы

```typescript
// utils/validators.ts
export const validators = {
  required: (value: any) => {
    if (Array.isArray(value)) return value.length > 0;
    return !!value && value.toString().trim().length > 0;
  },

  email: (value: string) => {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(value);
  },

  phone: (value: string) => {
    const regex = /^[\+]?[1-9][\d]{0,15}$/;
    return regex.test(value.replace(/\s/g, ''));
  },

  minLength: (min: number) => (value: string) => {
    return value && value.length >= min;
  },

  maxLength: (max: number) => (value: string) => {
    return !value || value.length <= max;
  },

  positiveNumber: (value: number) => {
    return !isNaN(value) && value > 0;
  },

  url: (value: string) => {
    try {
      new URL(value);
      return true;
    } catch {
      return false;
    }
  }
};

// Composable для валидации форм
export function useFormValidation<T extends Record<string, any>>(
  initialData: T,
  rules: Record<keyof T, Array<(value: any) => boolean | string>>
) {
  const form = reactive({ ...initialData });
  const errors = reactive<Record<keyof T, string[]>>({} as Record<keyof T, string[]>);
  const isValid = computed(() => 
    Object.values(errors).every(fieldErrors => fieldErrors.length === 0)
  );

  function validateField(field: keyof T) {
    const fieldRules = rules[field] || [];
    const fieldErrors: string[] = [];

    for (const rule of fieldRules) {
      const result = rule(form[field]);
      if (result !== true && typeof result === 'string') {
        fieldErrors.push(result);
      } else if (result === false) {
        fieldErrors.push(`Поле ${String(field)} заполнено некорректно`);
      }
    }

    errors[field] = fieldErrors;
  }

  function validateAll() {
    Object.keys(rules).forEach(field => validateField(field as keyof T));
  }

  function resetForm() {
    Object.assign(form, initialData);
    Object.keys(errors).forEach(key => {
      errors[key as keyof T] = [];
    });
  }

  return {
    form,
    errors: readonly(errors),
    isValid,
    validateField,
    validateAll,
    resetForm
  };
}
```

---

## 📚 Специфические инструкции

### Детальные руководства в этой папке:

**➡️ UI Kit и компоненты:** [`bitrix24-ui-kit.md`](bitrix24-ui-kit.md)

**➡️ JavaScript SDK:** [`bitrix24-js-sdk.md`](bitrix24-js-sdk.md)

**➡️ Аккордеон компонент:** [`accordion.md`](accordion.md)

**➡️ Календарь:** [`calendar.md`](calendar.md)

**➡️ Формы:** [`form.md`](form.md)

**➡️ Селекторы:** [`selector.md`](selector.md)

**➡️ Настройки:** [`settings-page.md`](settings-page.md)

**➡️ Таблицы и гриды:** [`table-and-grid.md`](table-and-grid.md)

---

## 🎯 Best practices

### 1. Компонентная архитектура

- **Разделяйте** презентационные и контейнерные компоненты
- **Используйте** композицию вместо наследования
- **Создавайте** переиспользуемые UI компоненты
- **Документируйте** API компонентов через props и emits

### 2. Управление состоянием

- **Локальное состояние** для UI логики компонента
- **Store** для глобального состояния приложения
- **Composables** для переиспользуемой логики
- **Избегайте** prop drilling, используйте provide/inject

### 3. Производительность

- **Ленивая загрузка** компонентов и маршрутов
- **Виртуализация** для больших списков
- **Мемоизация** вычислений через computed
- **Дебаунс** для пользовательского ввода

### 4. Доступность

- **Семантические** HTML элементы
- **ARIA** атрибуты для сложных компонентов
- **Клавиатурная** навигация
- **Контрастность** цветов

---

## ⚠️ Часто встречающиеся проблемы

### 1. Гидратация (hydration mismatch)

**Проблема:** Различия между серверным и клиентским рендерингом
**Решение:** Использовать `<ClientOnly>` для клиент-специфичного кода

### 2. Memory leaks в SPA

**Проблема:** Утечки памяти при навигации между страницами
**Решение:** Правильная очистка слушателей событий и таймеров

### 3. SEO оптимизация

**Проблема:** Плохая индексация динамического контента
**Решение:** SSR/SSG, правильные мета-теги, структурированные данные

---

*Обновлено: 25 ноября 2025*
*Версия: 2.0 - Модульная архитектура знаний*