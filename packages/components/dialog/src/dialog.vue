<template>
  <teleport to="body" :disabled="!appendToBody">
    <transition
      name="dialog-fade"
      @after-enter="afterEnter"
      @after-leave="afterLeave"
      @before-leave="beforeLeave"
    >
      <el-overlay
        v-show="visible"
        custom-mask-event
        :mask="modal"
        :overlay-class="modalClass"
        :z-index="zIndex"
      >
        <div
          role="dialog"
          aria-modal="true"
          :aria-label="title || undefined"
          :aria-labelledby="!title ? titleId : undefined"
          :aria-describedby="bodyId"
          :class="`${ns.namespace.value}-overlay-dialog`"
          @click="overlayEvent.onClick"
          @mousedown="overlayEvent.onMousedown"
          @mouseup="overlayEvent.onMouseup"
        >
          <el-focus-trap
            loop
            :trapped="visible"
            focus-start-el="container"
            @focus-after-trapped="onOpenAutoFocus"
            @focus-after-released="onCloseAutoFocus"
            @release-requested="onCloseRequested"
          >
            <el-dialog-content
              v-if="rendered"
              ref="dialogContentRef"
              :custom-class="customClass"
              :center="center"
              :close-icon="closeIcon"
              :draggable="draggable"
              :fullscreen="fullscreen"
              :show-close="showClose"
              :style="style"
              :title="title"
              @close="handleClose"
            >
              <template #header>
                <slot
                  v-if="!$slots.title"
                  name="header"
                  :close="handleClose"
                  :title-id="titleId"
                  :title-class="ns.e('title')"
                />
                <slot v-else name="title" />
              </template>
              <slot />
              <template v-if="$slots.footer" #footer>
                <slot name="footer" />
              </template>
            </el-dialog-content>
          </el-focus-trap>
        </div>
      </el-overlay>
    </transition>
  </teleport>
</template>

<script lang="ts" setup>
import { computed, provide, ref, useSlots } from 'vue'
import { ElOverlay } from '@element-plus/components/overlay'
import {
  useDeprecated,
  useDraggable,
  useNamespace,
  useSameTarget,
} from '@element-plus/hooks'
import { dialogInjectionKey } from '@element-plus/tokens'
import ElFocusTrap from '@element-plus/components/focus-trap'
import ElDialogContent from './dialog-content.vue'
import { dialogEmits, dialogProps } from './dialog'
import { useDialog } from './use-dialog'

defineOptions({
  name: 'ElDialog',
})

const props = defineProps(dialogProps)
defineEmits(dialogEmits)
const slots = useSlots()

useDeprecated(
  {
    scope: 'el-dialog',
    from: 'the title slot',
    replacement: 'the header slot',
    version: '3.0.0',
    ref: 'https://element-plus.org/en-US/component/dialog.html#slots',
  },
  computed(() => !!slots.title)
)

const ns = useNamespace('dialog')
const dialogRef = ref<HTMLElement>()
const headerRef = ref<HTMLElement>()
const dialogContentRef = ref()

const {
  visible,
  titleId,
  bodyId,
  style,
  rendered,
  zIndex,
  afterEnter,
  afterLeave,
  beforeLeave,
  handleClose,
  onModalClick,
  onOpenAutoFocus,
  onCloseAutoFocus,
  onCloseRequested,
} = useDialog(props, dialogRef)

provide(dialogInjectionKey, {
  dialogRef,
  headerRef,
  bodyId,
  ns,
  rendered,
  style,
})

const overlayEvent = useSameTarget(onModalClick)

const draggable = computed(() => props.draggable && !props.fullscreen)

useDraggable(dialogRef, headerRef, draggable)

defineExpose({
  /** @description whether the dialog is visible */
  visible,
  dialogContentRef,
})
</script>
