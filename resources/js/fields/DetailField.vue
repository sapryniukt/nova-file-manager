<template>
  <component
    :is="component"
    :class="{
      'flex flex-1 flex-col gap-4 overflow-hidden bg-popover p-4 pr-0': field.showCard,
      hidden: field.value.length === 0,
    }"
  >
    <ScrollArea class="max-h-80 pr-4">
      <h3 v-if="field.showCard" class="text-xl font-bold">{{ field.name }}</h3>
      <PanelItem :field="field" :index="index" class="flex-1 !border-t-0 px-0">
        <template v-if="field.value" v-slot:value>
          <div class="nova-file-manager">
            <div :class="darkMode && 'dark'">
              <ul
                :class="['grid w-full', { [`gap-${field.gap || 2}`]: field.gap }, ...colsClasses]"
                role="group"
              >
                <template v-for="file in field.value" :key="file.id">
                  <FieldCard
                    :field="field"
                    :file="mapEntity(file)"
                    :attribute="field.attribute"
                    :detail="true"
                    :on-copy="copy"
                  />
                  <PreviewModal
                    :file="mapEntity(file)"
                    v-if="!!preview && preview?.id === mapEntity(file)?.id"
                    :read-only="true"
                  />
                </template>
              </ul>
            </div>
          </div>
        </template>
      </PanelItem>
    </ScrollArea>
  </component>
</template>

<script>
import { CopiesToClipboard } from 'laravel-nova'
import { mapMutations, mapState } from 'vuex'
import FieldCard from '@/components/Cards/FieldCard'
import Entity from '@/types/Entity'
import PreviewModal from '@/components/Modals/PreviewModal'

export default {
    mixins: [CopiesToClipboard],

    components: {
        PreviewModal,
        FieldCard,
    },

    props: ['field', 'index'],

    computed: {
        ...mapState('nova-file-manager', ['darkMode', 'preview']),

        colsClasses() {
            return Object.entries(this.field.cols || {}).map(([breakpoint, cols]) =>
                breakpoint === 'default' ? `grid-cols-${cols}` : `${breakpoint}:grid-cols-${cols}`
            )
        },

        component() {
            return this.field.showCard ? 'Card' : 'div'
        },
    },

    mounted() {
        this.detectDarkMode()
    },

    data: () => ({
        selected: null,
    }),

    methods: {
        ...mapMutations('nova-file-manager', ['init', 'detectDarkMode', 'previewFile']),

        copy(file) {
            this.selected = file
            this.copyValueToClipboard(file.url)

            setTimeout(() => {
                this.selected = null
            }, 1000)
        },

        openPreview(file) {
            this.previewFile(file)
        },

        mapEntity: file =>
            new Entity(
                file.id,
                file.name,
                file.path,
                file.size,
                file.extension,
                file.mime,
                file.url,
                file.lastModifiedAt,
                file.type,
                file.exists,
                file.disk
            ),
    },
}
</script>
