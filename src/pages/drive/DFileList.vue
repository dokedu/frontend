<template>
  <d-table
    v-bind="$attrs"
    v-model:variables="pageVariables"
    v-model:selected="selected"
    :columns="columns"
    objectName="files"
    :query="filesQuery"
    defaultSort="createdAt"
    @row-click="clickFile"
  >
    <template #name-data="{ column, item }">
      <div class="flex items-center gap-3">
        <component
          :is="useFileIcon(item)"
          :size="18"
          class="stroke-colors-default"
          :class="{ 'fill-stone-700': isFolder(item) }"
        />
        <div class="text-default">{{ column }}</div>
      </div>
    </template>
    <template #createdAt-data="{ column }">
      <div class="text-default">{{ formatDate(new Date(Date.parse(column)), "DD.MM.YYYY hh:mm") }} Uhr</div>
    </template>
    <template #size-data="{ column, item }">
      <div class="text-default">{{ isBlob(item) ? prettyBytes(column) : "-" }}</div>
    </template>
    <template #id-data="{ item }">
      <div>
        <div class="flex w-[80px] justify-end gap-1 pr-4">
          <div
            v-if="isBlob(item)"
            class="group/icon w-fit rounded-lg p-1.5 hover:bg-blue-100"
            @click.stop="downloadFile(item)"
          >
            <Download class="stroke-colors-subtle group-hover/icon:stroke-blue-900" :size="16" />
          </div>
          <div class="group/icon w-fit rounded-lg p-1.5 hover:bg-blue-100" @click.stop="onRenameFile(item)">
            <Edit class="stroke-colors-subtle group-hover/icon:stroke-blue-900" :size="16" />
          </div>
          <div class="group/icon w-fit rounded-lg p-1.5 hover:bg-blue-100" @click.stop="onDeleteFile(item)">
            <Trash class="stroke-colors-subtle group-hover/icon:stroke-blue-900" :size="16" />
          </div>
        </div>
      </div>
    </template>
    <template #empty>
      <div class="flex h-[75vh] w-full flex-1 items-center justify-center gap-3 px-8 py-3">
        <div class="flex select-none flex-col items-center justify-center rounded-xl bg-muted p-8">
          <div class="mb-4 w-fit rounded-xl bg-strong p-2.5">
            <FileText class="stroke-colors-strong" :size="24" />
          </div>
          <div class="mb-2 text-center text-sm font-medium text-strong">Drop file here</div>
          <div class="text-center text-sm text-subtle">or use the "New" button.</div>
        </div>
      </div>
    </template>
  </d-table>

  <d-dialog-rename-file :open="showRenameDialog" v-model="renameDialogFile" @close="closeRenameDialog" />
</template>

<script lang="ts" setup>
import { FileText, Download, Edit, FileImage, Folder } from "lucide-vue-next";
import { File } from "../../gql/graphql";
import { graphql } from "../../gql";
import { formatDate } from "@vueuse/core";
import DTable from "@/components/d-table/d-table.vue";
import { PageVariables } from "@/types/types";
import { computed, ref } from "vue";
import useDownloadFile from "@/composables/useDownloadFile";
import { Archive } from "lucide-vue-next";
import { useRoute } from "vue-router";
import { File as FileFile, Trash } from "lucide-vue-next";
import DDialogRenameFile from "./folders/DDialogRenameFile.vue";
import { useMutation } from "@urql/vue";

const route = useRoute();

interface Variables extends PageVariables {}

const parentId = computed(() => route.params.id);

const showRenameDialog = ref(false);

const renameDialogFile = ref<File | null>(null);

function onRenameFile(file: File) {
  renameDialogFile.value = file;
  showRenameDialog.value = true;
}

function closeRenameDialog() {
  showRenameDialog.value = false;
}

async function onDeleteFile(file: File) {
  await deleteFile({ id: file.id });
}

const { executeMutation: deleteFile } = useMutation(
  graphql(`
    mutation deleteFile($id: ID!) {
      deleteFile(input: { id: $id }) {
        success
        file {
          id
        }
      }
    }
  `)
);

const pageVariables = ref<Variables[]>([
  {
    limit: 25,
    offset: 0,
    filter: {
      parentId: parentId,
      myBucket: true,
    },
  },
]);

const selected = ref<{ id: string }[]>([]);

function isFolder(file: File) {
  return file.fileType === "folder";
}

function useFileIcon(file: File) {
  if (isFolder(file)) {
    return Folder;
  }

  switch (file.MIMEType) {
    case "application/pdf":
      return FileText;
    case "image/png":
      return FileImage;
    case "image/jpeg":
      return FileImage;
    case "image/gif":
      return FileImage;
    case "image/webp":
      return FileImage;
    case "image/jpg":
      return FileImage;
    case "application/zip":
      return Archive;
    default:
      return FileFile;
  }
}

function isBlob(file: File) {
  return file.fileType === "blob";
}

const { downloadFile } = useDownloadFile();

const emit = defineEmits(["click"]);

const columns = [
  {
    label: "name",
    key: "name",
    width: 0.5,
  },
  {
    label: "created_at",
    key: "createdAt",
  },
  {
    label: "filesize",
    key: "size",
  },
  {
    label: "-",
    key: "id",
    width: 0.1,
  },
];

const filesQuery = graphql(`
  query files($offset: Int, $limit: Int, $filter: FilesFilterInput) {
    files(input: $filter, limit: $limit, offset: $offset) {
      pageInfo {
        hasNextPage
        hasPreviousPage
      }
      edges {
        id
        name
        fileType
        MIMEType
        size
        createdAt
      }
    }
  }
`);

function clickFile(file: any) {
  const isSelected = selected.value.find((f) => f.id === file.id);
  if (!isSelected) return;

  emit("click", file);
}

function prettyBytes(bytes: number) {
  const units = ["B", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"];

  if (Math.abs(bytes) < 1) {
    return bytes + "B";
  }

  const u = Math.min(Math.floor(Math.log10(bytes) / 3), units.length - 1);
  const n = Number((bytes / Math.pow(1000, u)).toFixed(2));
  return `${n} ${units[u]}`;
}
</script>
