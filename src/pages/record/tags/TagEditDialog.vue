<template>
  <DDialog :open="modalOpen">
    <template #header>
      <div class="flex items-center justify-between">
        <div class="font-medium text-strong">Create tag</div>
        <button @click="onClose"><X class="h-4 w-4"></X></button>
      </div>
    </template>
    <template #main>
      <div class="pb-4">
        <div class="flex items-center gap-4">
          <div class="min-w-16 text-stone-400">Name</div>
          <DInput v-if="tag" name="name" v-model="tag.name" />
        </div>
        <div class="relative mt-4 flex items-center gap-4">
          <div class="min-w-16 text-stone-400">Color</div>
          <div class="relative w-full">
            <DContextMenu
              :show="contextMenuOpen"
              @close="contextMenuOpen = false"
              :alignment="ContextMenuAlignment.Overlay"
              class="max-h-[150px] w-full overflow-y-auto p-1"
            >
              <div class="flex w-full flex-col items-start rounded-md">
                <div
                  class="w-full cursor-pointer p-1 hover:bg-stone-100"
                  v-for="color in colors"
                  @click="onSelectColor(color)"
                >
                  <DTag :color="color">{{ color }}</DTag>
                </div>
              </div>
            </DContextMenu>
            <div
              class="flex w-full flex-wrap items-start gap-2 rounded-md p-2 hover:bg-stone-50"
              @click="contextMenuOpen = true"
            >
              <DTag v-if="tag && tag.color" :color="tag.color" class="w-1/4 p-2">{{ tag.color }}</DTag>
              <div v-else class="text-stone-400">Set color</div>
            </div>
          </div>
        </div>
      </div>
    </template>
    <template #footer>
      <div class="flex justify-between">
        <DButton type="outline" size="md" @click="onClose">Cancel</DButton>
        <div class="flex gap-2">
          <DButton type="outline" size="md" @click="onArchive">Archive</DButton>
          <DButton type="primary" size="md" @click="onUpdate">Update</DButton>
        </div>
      </div>
    </template>
  </DDialog>
</template>

<script setup lang="ts">
import DDialog from "../../../components/d-dialog/d-dialog.vue";
import DButton from "../../../components/d-button/d-button.vue";
import { X } from "lucide-vue-next";
import { toRef, ref } from "vue";
import DInput from "../../../components/d-input/d-input.vue";
import DContextMenu from "../../../components/d-context-menu/d-context-menu.vue";
import { ContextMenuAlignment } from "../../../components/d-context-menu/d-context-menu.vue";
import DTag from "../../../components/d-tag/d-tag.vue";
import { useMutation } from "@urql/vue";
import { graphql } from "../../../gql";
import { Tag } from "../../../gql/graphql";

const colors = ["red", "orange", "yellow", "green", "blue", "indigo", "purple", "pink", "gray"];

const props = defineProps({
  open: Boolean,
  tag: {
    type: Object as () => Tag,
    required: true,
  },
});
const emit = defineEmits(["close", "updated"]);

const modalOpen = toRef(props, "open");
const contextMenuOpen = ref(false);
const tag = toRef(props, "tag");

const { executeMutation: updateTag } = useMutation(
  graphql(`
    mutation UpdateTag($id: ID!, $input: CreateTagInput!) {
      updateTag(id: $id, input: $input) {
        id
        name
        color
        deletedAt
        createdAt
      }
    }
  `)
);

const { executeMutation: archiveTag } = useMutation(
  graphql(`
    mutation ArchiveTag($id: ID!) {
      archiveTag(id: $id) {
        id
        name
        color
        deletedAt
        createdAt
      }
    }
  `)
);

const onSelectColor = (color: string) => {
  if (tag.value) {
    tag.value.color = color;
  }
  contextMenuOpen.value = false;
};

const onClose = () => {
  emit("close", false);
};

const onUpdate = async () => {
  await updateTag({
    id: tag.value.id,
    input: {
      name: tag.value.name,
      color: tag.value.color,
    },
  });
  emit("updated");
};

const onArchive = async () => {
  await archiveTag({
    id: tag.value.id,
  });
  emit("updated");
};
</script>