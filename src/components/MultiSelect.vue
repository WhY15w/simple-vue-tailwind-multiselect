<template>
  <div class="relative">
    <!-- Dropdown trigger -->
    <button
      type="button"
      class="flex w-full items-center justify-between rounded-md border border-gray-200 bg-white px-3 py-2 text-left text-sm font-normal text-gray-700 shadow-sm transition-all duration-200 hover:shadow-md disabled:cursor-not-allowed disabled:opacity-60"
      :disabled="disabled"
      @click="toggleDropdown"
      :id="id"
      data-dropdown-trigger
    >
      <span v-if="selectedCount <= 0" class="text-gray-400">
        {{ placeholder }}
      </span>
      <div v-else class="flex max-w-[250px] gap-1 truncate">
        <span
          v-for="selectedName in selectedNames"
          :key="selectedName"
          class="rounded-md bg-blue-500/80 px-2 py-0.5 text-xs text-white transition-colors hover:bg-blue-600"
        >
          {{ selectedName }}
        </span>
      </div>
      <ChevronDownIcon
        class="ml-2 h-4 w-4 shrink-0 opacity-60 transition-transform duration-300"
        :class="{ 'rotate-180': isDropdownOpen }"
      />
    </button>

    <!-- Dropdown content -->
    <transition name="fade-slide">
      <div
        v-show="isDropdownOpen"
        class="absolute z-10 mt-1 w-full"
        v-on-click-outside="[closeDropdown, { ignore: ['[data-dropdown-trigger]'] }]"
      >
        <div
          class="overflow-hidden rounded-xl border border-gray-200 bg-white shadow-md ring-1 ring-black/5 backdrop-blur-sm"
        >
          <!-- Action bar -->
          <div class="flex items-center justify-between border-b border-gray-100 bg-gray-50 p-1">
            <!-- Search bar -->
            <div class="border-b border-gray-100 p-2">
              <input
                v-model="searchQuery"
                type="text"
                placeholder="搜索选项..."
                class="w-full rounded-sm border border-gray-200 bg-gray-50 px-2 py-1 text-xs transition placeholder:text-gray-400 focus:border-blue-500 focus:bg-white focus:outline-none"
              />
            </div>
            <div class="flex items-center gap-1 px-2">
              <button
                type="button"
                class="rounded px-2 py-1 text-xs text-gray-600 transition hover:text-blue-600 disabled:cursor-not-allowed disabled:opacity-40 hover:cursor-pointer"
                @click.prevent="selectAll"
                :disabled="!isSelectAllEnabled"
              >
                全选
              </button>
              <button
                type="button"
                class="rounded px-2 py-1 text-xs text-gray-600 transition hover:text-red-500 disabled:cursor-not-allowed disabled:opacity-40 hover:cursor-pointer"
                @click.prevent="clearSelection"
                :disabled="!isClearSelectionEnabled"
              >
                清空
              </button>
            </div>
          </div>

          <!-- Options list -->
          <div
            class="scrollbar-thin scrollbar-thumb-gray-300 scrollbar-track-transparent max-h-80 overflow-y-auto"
          >
            <div
              v-for="option in filteredOptions"
              :key="option.value"
              @click="toggleOption(option.value)"
              :class="[
                'relative flex cursor-pointer items-center px-3 py-1.5 text-sm font-medium transition-colors duration-150',
                isSelected(option) ? 'bg-blue-50 text-blue-700' : 'text-gray-700 hover:bg-gray-50',
              ]"
            >
              <div
                class="mr-3 flex h-4 w-4 items-center justify-center rounded border border-gray-300 transition-colors duration-150"
                :class="{
                  'border-blue-500 bg-blue-500': isSelected(option),
                  'bg-white': !isSelected(option),
                }"
              >
                <CheckIcon v-if="isSelected(option)" class="h-3.5 w-3.5 text-white" />
              </div>

              <div class="truncate">{{ option.name }}</div>
            </div>

            <div
              v-if="filteredOptions.length === 0"
              class="px-4 py-3 text-center text-sm text-gray-400 select-none"
            >
              没有匹配的选项
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { vOnClickOutside } from '@vueuse/components'
import { ChevronDownIcon, CheckIcon } from 'lucide-vue-next'

interface OptionItem {
  name: string
  value: string | number
}

interface Props {
  options: OptionItem[]
  id: string
  name: string
  required: boolean
  disabled?: boolean
  placeholder: string
}

const props = defineProps<Props>()

const model = defineModel<(string | number)[]>({
  default: () => [],
})

const isDropdownOpen = ref(false)
const searchQuery = ref('')

const optionValueToObjectMap = computed<Record<string | number, OptionItem>>(() => {
  const map: Record<string | number, OptionItem> = {}
  for (const o of props.options) {
    map[o.value] = o
  }
  return map
})

const selectedCount = computed(() => model.value.length)

const selectedNames = computed(() =>
  model.value.map((v) => optionValueToObjectMap.value[v]?.name ?? ''),
)

const isSelectAllEnabled = computed(() => selectedCount.value < props.options.length)
const isClearSelectionEnabled = computed(() => selectedCount.value > 0)

const filteredOptions = computed(() => {
  const query = searchQuery.value.trim().toLowerCase()
  return query
    ? props.options.filter((option) => option.name.toLowerCase().includes(query))
    : props.options
})

function closeDropdown(): void {
  isDropdownOpen.value = false
}
function toggleDropdown(): void {
  isDropdownOpen.value = !isDropdownOpen.value
}

function toggleOption(value: string | number): void {
  if (!model.value.includes(value)) {
    model.value = [...model.value, value]
  } else {
    model.value = model.value.filter((v) => v !== value)
  }
}
function selectAll(): void {
  model.value = props.options.map((o) => o.value)
}
function clearSelection(): void {
  model.value = []
}

function isSelected(option: OptionItem): boolean {
  return model.value.includes(option.value)
}
</script>

<style scoped>
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.2s ease;
}
.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(-4px);
}
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

.scrollbar-thin::-webkit-scrollbar {
  width: 6px;
}
.scrollbar-thin::-webkit-scrollbar-thumb {
  border-radius: 4px;
  background-color: rgba(0, 0, 0, 0.15);
}
.scrollbar-thin::-webkit-scrollbar-thumb:hover {
  background-color: rgba(0, 0, 0, 0.3);
}
</style>
