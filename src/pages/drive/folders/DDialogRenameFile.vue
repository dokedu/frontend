<template>
  <d-dialog :open="open" @close="onClose">
    <template #header>
      <div class="font-medium text-strong">Rename file</div>
    </template>
    <template #main>
      <d-input v-if="file" v-model="file.name" name="name"></d-input>
    </template>
    <template #footer>
      <div class="flex justify-between">
        <d-button size="sm" type="outline" @click="onClose">Cancel</d-button>
        <d-button size="sm" @click="onSave">Save</d-button>
      </div>
    </template>
  </d-dialog>
</template>

<script lang="ts" setup>
import DDialog from "@/components/d-dialog/d-dialog.vue";
import DInput from "@/components/d-input/d-input.vue";
import DButton from "@/components/d-button/d-button.vue";
import { toRef } from "vue";
import { useVModel } from "@vueuse/core";
import { useMutation } from "@urql/vue";
import { graphql } from "@/gql";
import { File } from "@/gql/graphql";

const props = defineProps<{
  modelValue: File | null;
  open: boolean;
}>();
const emit = defineEmits(["update:modelValue", "close"]);

const open = toRef(props, "open");

const file = useVModel(props, "modelValue", emit);

function onClose() {
  emit("close");
}

const { executeMutation: renameFile } = useMutation(
  graphql(`
    mutation renameFile($input: RenameFileInput!) {
      renameFile(input: $input) {
        id
        name
      }
    }
  `)
);

function onSave() {
  if (!file.value) return;
  renameFile({
    input: {
      id: file.value.id,
      name: file.value.name,
    },
  });
  emit("close");
}
</script>
