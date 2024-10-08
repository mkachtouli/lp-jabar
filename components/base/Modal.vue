<script setup lang="ts">
import { setHtmlStyle } from '~/utils/document';

const { to, size, inside, modelValue } = defineProps({
  to: {
    type: String,
    default: 'body',
  },

  size: {
    type: String,
    default: '300px',
  },
  modelValue: {
    type: Boolean,
    required: true,
  },
  inside: {
    type: Boolean,
    default: false,
  },
  closeButton: {
    type: Boolean,
    default: true,
  },
});

const emit = defineEmits(['update:modelValue']);

const bodySizeStyle = computed(() => {
  return {
    width: '100%',
    height: size,
    maxHeight: '100%',
  };
});

const closeModal = () => {
  document.body.style.overflow = '';
  emit('update:modelValue', false);
};

const showBody = ref(false);
const showModal = ref(false);

const beforeEnter = () => {
  showModal.value = true;
  showBody.value = false;
};

const enter = (el: Element, done: () => void) => {
  if (!inside) {
    console.log(inside);

    setHtmlStyle({
      overflow: 'hidden',
    });
  }

  setTimeout(() => {
    showBody.value = true;
  }, 200);
  setTimeout(done, 500); // Total animation duration
};

const leave = (el: Element, done: () => void) => {
  setHtmlStyle({
    overflow: '',
  });
  showBody.value = false;
  setTimeout(() => {
    showModal.value = false;
    done();
  }, 300);
};

onMounted(() => {
  if (modelValue) {
    showModal.value = modelValue;
    if (!inside) {
      setHtmlStyle({
        overflow: 'hidden',
      });
    }
    setTimeout(() => {
      showBody.value = modelValue;
    }, 200);
  }
});
watch(
  () => modelValue,
  (newValue) => {
    if (newValue) {
      beforeEnter();
      enter({} as Element, () => {});
    } else {
      leave({} as Element, () => {});
    }
  }
);
</script>

<template>
  <ClientOnly>
    <Teleport :to="to">
      <div class="modal" :class="{ inside: inside }" v-show="showModal">
        <Transition
          name="modal-bg"
          @before-enter="beforeEnter"
          @enter="enter"
          @leave="leave"
        >
          <div v-if="modelValue" class="modal-bg" @click="closeModal"></div>
        </Transition>

        <Transition name="modal-body">
          <div v-if="showBody" class="modal-body" :style="[bodySizeStyle]">
            <div v-if="$slots.header" class="modal-header">
              <slot name="header" />
            </div>
            <div class="modal-content">
              <BaseButton
                v-if="closeButton"
                class="modal-closebtn"
                @click="closeModal"
              >
                <IconX />
              </BaseButton>
              <slot />
            </div>
            <div v-if="$slots.footer" class="modal-footer">
              <slot name="footer" />
            </div>
          </div>
        </Transition>
      </div>
    </Teleport>
  </ClientOnly>
</template>
<style lang="scss">
$modal: '.modal';

#{$modal} {
  @apply fixed h-dvh w-screen z-[1000] top-0 left-0;

  &.inside {
    @apply absolute h-full w-full z-[10] top-0 left-0;

    .modal-bg {
      @apply bg-white;
    }
  }

  &-bg {
    @apply relative bg-black opacity-40 h-full w-full;
  }

  &-closebtn {
    @apply absolute right-1 top-1.5 z-20;
  }

  &-body {
    @apply absolute top-0 left-0 bg-white z-10 h-full w-full duration-300
    flex flex-col border border-black;

    &.top {
      @apply top-0 left-0;
    }
    &.bottom {
      @apply bottom-0 left-0;
    }
    &.right {
      @apply top-0 right-0;
    }
    &.left {
      @apply top-0 left-0;
    }
    &.center {
      @apply top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2;
    }
  }

  &-header {
    @apply min-h-14 flex-col flex justify-center px-5 py-2 border-b border-black;
  }
  &-content {
    @apply p-5 flex-1 overflow-auto;
  }

  &-footer {
    @apply min-h-14 flex items-center px-5 py-2 border-t border-black;
  }
}
</style>

<!-- for animation -->
<style lang="scss">
.modal-bg-enter-active,
.modal-bg-leave-active {
  transition: opacity 0.3s ease;
}

.modal-bg-enter-from,
.modal-bg-leave-to {
  opacity: 0;
}

.modal-body-enter-active,
.modal-body-leave-active {
  // transition: transform 0.3s ease, opacity 0.3s ease, scale 0.3s ease;
  transition: all 0.3s ease;
}

// .modal-body-enter-from,
// .modal-body-leave-to {
//   transform: translateX(-100%);
// }

// .modal-body.right.modal-body-enter-from,
// .modal-body.right.modal-body-leave-to {
//   transform: translateX(100%);
// }

// .modal-body.top.modal-body-enter-from,
// .modal-body.top.modal-body-leave-to {
//   transform: translateY(-100%);
// }

// .modal-body.bottom.modal-body-enter-from,
// .modal-body.bottom.modal-body-leave-to {
//   transform: translateY(100%);
// }

.modal-body.modal-body-enter-from,
.modal-body.modal-body-leave-to {
  opacity: 0;
  filter: blur(10px);
}
</style>
