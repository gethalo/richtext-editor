<script lang="ts" setup>
import type { Node as ProseMirrorNode } from "prosemirror-model";
import type { Decoration } from "prosemirror-view";
import { Editor, NodeViewWrapper, Node } from "@tiptap/vue-3";
import { computed } from "vue";
import BlockCard from "@/components/block/BlockCard.vue";
import BlockActionButton from "@/components/block/BlockActionButton.vue";
import BlockActionInput from "@/components/block/BlockActionInput.vue";
import BlockActionSeparator from "@/components/block/BlockActionSeparator.vue";
import MdiLinkVariant from "~icons/mdi/link-variant";
import MdiImageSizeSelectActual from "~icons/mdi/image-size-select-actual";
import MdiImageSizeSelectSmall from "~icons/mdi/image-size-select-small";
import MdiImageSizeSelectLarge from "~icons/mdi/image-size-select-large";
import MdiCogPlay from "~icons/mdi/cog-play";
import MdiCogPlayOutline from "~icons/mdi/cog-play-outline";
import MdiPlayCircle from "~icons/mdi/play-circle";
import MdiPlayCircleOutline from "~icons/mdi/play-circle-outline";
import MdiMotionPlayOutline from "~icons/mdi/motion-play-outline";
import MdiMotionPlay from "~icons/mdi/motion-play";
import { i18n } from "@/locales";

const props = defineProps<{
  editor: Editor;
  node: ProseMirrorNode;
  decorations: Decoration[];
  selected: boolean;
  extension: Node<any, any>;
  getPos: () => number;
  updateAttributes: (attributes: Record<string, any>) => void;
  deleteNode: () => void;
}>();

const src = computed({
  get: () => {
    return props.node?.attrs.src;
  },
  set: (src: string) => {
    props.updateAttributes({ src: src });
  },
});

const width = computed({
  get: () => {
    return props.node.attrs.width;
  },
  set: (value: string) => {
    handleSetSize(value, height.value);
  },
});

const height = computed({
  get: () => {
    return props.node.attrs.height;
  },
  set: (value: string) => {
    handleSetSize(width.value, value);
  },
});

const controls = computed(() => {
  return props.node.attrs.controls;
});

const autoplay = computed(() => {
  return props.node.attrs.autoplay;
});

const loop = computed(() => {
  return props.node.attrs.loop;
});

function handleSetFocus() {
  props.editor.commands.setNodeSelection(props.getPos());
}

function handleSetSize(width: string, height: string) {
  props.updateAttributes({ width, height });
  props.editor.chain().focus().setNodeSelection(props.getPos()).run();
}

function handleToggleControls() {
  props.updateAttributes({
    controls: props.node.attrs.controls ? null : true,
  });
  props.editor.chain().focus().setNodeSelection(props.getPos()).run();
}

function handleToggleAutoplay() {
  props.updateAttributes({
    autoplay: props.node.attrs.autoplay ? null : true,
  });
  props.editor.chain().focus().setNodeSelection(props.getPos()).run();
}

function handleToggleLoop() {
  props.updateAttributes({
    loop: props.node.attrs.loop ? null : true,
  });
  props.editor.chain().focus().setNodeSelection(props.getPos()).run();
}

function handleOpenLink() {
  window.open(src.value, "_blank");
}
</script>

<template>
  <node-view-wrapper as="div">
    <block-card
      :editor="editor"
      :get-pos="getPos"
      :delete-node="deleteNode"
      :selected="selected"
    >
      <template #content>
        <div
          class="inline-block overflow-hidden transition-all text-center relative h-full"
          :style="{
            width: node.attrs.width,
          }"
        >
          <div class="py-1.5">
            <input
              v-model.lazy="src"
              class="block px-2 w-full py-1.5 text-sm text-gray-900 border border-gray-300 rounded-md bg-gray-50 focus:ring-blue-500 focus:border-blue-500"
              :placeholder="
                i18n.global.t('editor.common.placeholder.link_input')
              "
              tabindex="-1"
              @focus="handleSetFocus"
            />
          </div>
          <video
            v-if="src"
            :controls="controls"
            :autoplay="autoplay"
            :loop="loop"
            class="rounded-md m-0"
            :src="node!.attrs.src"
            :style="{
              width: node.attrs.width,
              height: node.attrs.height,
            }"
            @mouseenter="handleSetFocus"
          ></video>
        </div>
      </template>
      <template #actions>
        <BlockActionButton
          :selected="controls"
          :tooltip="`${
            controls
              ? i18n.global.t('editor.extensions.video.disable_controls')
              : i18n.global.t('editor.extensions.video.enable_controls')
          }`"
          @click="handleToggleControls"
        >
          <template #icon>
            <MdiCogPlay v-if="controls" />
            <MdiCogPlayOutline v-else />
          </template>
        </BlockActionButton>

        <BlockActionButton
          :selected="autoplay"
          :tooltip="`${
            autoplay
              ? i18n.global.t('editor.extensions.video.disable_autoplay')
              : i18n.global.t('editor.extensions.video.enable_autoplay')
          }`"
          @click="handleToggleAutoplay"
        >
          <template #icon>
            <MdiPlayCircle v-if="autoplay" />
            <MdiPlayCircleOutline v-else />
          </template>
        </BlockActionButton>

        <BlockActionButton
          :selected="loop"
          :tooltip="`${
            loop
              ? i18n.global.t('editor.extensions.video.disable_loop')
              : i18n.global.t('editor.extensions.video.enable_loop')
          }`"
          @click="handleToggleLoop"
        >
          <template #icon>
            <MdiMotionPlay v-if="loop" />
            <MdiMotionPlayOutline v-else />
          </template>
        </BlockActionButton>

        <BlockActionSeparator />

        <BlockActionInput
          v-model.lazy.trim="width"
          :tooltip="i18n.global.t('editor.common.tooltip.custom_width_input')"
        />

        <BlockActionInput
          v-model.lazy.trim="height"
          :tooltip="i18n.global.t('editor.common.tooltip.custom_height_input')"
        />

        <BlockActionSeparator />

        <BlockActionButton
          :tooltip="i18n.global.t('editor.extensions.video.small_size')"
          :selected="node.attrs.width === '25%'"
          @click="handleSetSize('25%', 'auto')"
        >
          <template #icon>
            <MdiImageSizeSelectSmall />
          </template>
        </BlockActionButton>

        <BlockActionButton
          :tooltip="i18n.global.t('editor.extensions.video.medium_size')"
          :selected="node.attrs.width === '50%'"
          @click="handleSetSize('50%', 'auto')"
        >
          <template #icon>
            <MdiImageSizeSelectLarge />
          </template>
        </BlockActionButton>

        <BlockActionButton
          :tooltip="i18n.global.t('editor.extensions.video.large_size')"
          :selected="node.attrs.width === '100%'"
          @click="handleSetSize('100%', 'auto')"
        >
          <template #icon>
            <MdiImageSizeSelectActual />
          </template>
        </BlockActionButton>

        <BlockActionSeparator />

        <BlockActionButton
          :tooltip="i18n.global.t('editor.common.tooltip.open_link')"
          @click="handleOpenLink"
        >
          <template #icon>
            <MdiLinkVariant />
          </template>
        </BlockActionButton>
      </template>
    </block-card>
  </node-view-wrapper>
</template>
