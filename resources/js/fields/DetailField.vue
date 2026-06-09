<script setup lang="ts">
import { Entity } from '__types__'
import { computed, onMounted, ref } from 'vue'
import FieldCard from '@/components/Cards/FieldCard.vue'
import PreviewModal from '@/components/Modals/PreviewModal.vue'
import { useClipboard } from '@/hooks'
import useBrowserStore from '@/stores/browser'

interface Props {
  field: any
  index: number
  resourceName?: string
  resourceId: string | number
}

const props = defineProps<Props>()

const selected = ref(undefined as Entity | undefined)
const store = useBrowserStore()
const { copy: clipboardCopy } = useClipboard()

const dark = computed(() => store.dark)
const preview = computed(() => store.preview)

const colsClasses = computed(() =>
  Object.entries(props.field.cols || {}).map(([breakpoint, cols]) =>
    breakpoint === 'default' ? `grid-cols-${cols}` : `${breakpoint}:grid-cols-${cols}`
  )
)

const component = computed(() => (props.field.showCard ? 'Card' : 'div'))

const isEmpty = computed(() => !props.field.value?.length)

const copy = (file: Entity) => {
  selected.value = file
  clipboardCopy(file.url)

  setTimeout(() => {
    selected.value = undefined
  }, 1000)
}

onMounted(() => {
  store.syncDarkMode()
})
</script>

<template>
  <component
    :is="component"
    :class="{
      'flex flex-1 flex-col gap-4 overflow-hidden bg-popover p-4 pr-0': field.showCard,
      hidden: isEmpty,
    }"
  >
    <div class="max-h-80 3xl:max-h-40 overflow-y-auto pr-4">
      <h3 v-if="field.showCard" class="text-xl font-bold mb-2">{{ field.name }}</h3>
      <PanelItem :field="field" :index="index" class="flex-1 !border-t-0 px-0">
        <template v-if="field.value" v-slot:value>
          <div class="nova-file-manager">
            <div :class="{ dark }">
              <ul
                :class="['grid w-full', { [`gap-${field.gap || 2}`]: field.gap }, ...colsClasses]"
                role="group"
              >
                <template v-for="file in field.value" :key="file.id">
                  <FieldCard
                    :field="field"
                    :file="file"
                    :attribute="field.attribute"
                    :resource-name="resourceName"
                    :resource-id="resourceId"
                    :detail="true"
                    :on-copy="copy"
                  />

                  <PreviewModal :file="file" v-if="!!preview && preview.id === file.id" :read-only="true" />
                </template>
              </ul>
            </div>
          </div>
        </template>
      </PanelItem>
    </div>
  </component>
</template>
