<template>
  <PageWrapper>
    <PageHeaderDrive @upload="upload" />
    <PageContent>
      <DFileDropZone @upload="upload">
        <DFileList @click="clickFile" />
      </DFileDropZone>
    </PageContent>
  </PageWrapper>
  <DFilePreview :file="previewFile" @close="previewFile = null" />
</template>
<script setup lang="ts">
import PageWrapper from "@/components/PageWrapper.vue";
import PageContent from "@/components/PageContent.vue";
import { useMutation } from "@urql/vue";
import { graphql } from "@/gql";
import { computed, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import { File } from "@/gql/graphql";
import DFileList from "./../DFileList.vue";
import DFileDropZone from "./../DFileDropZone.vue";
import PageHeaderDrive from "./../PageHeaderDrive.vue";
import DFilePreview from "./../DFilePreview.vue";

const router = useRouter();
const route = useRoute();

const folderId = computed(() => {
  return route.params.id;
});

const previewFile = ref<File | null>(null);

// @ts-expect-error
async function upload({ file, parentId = folderId.value }) {
  console.log(parentId);
  await uploadFile({
    input: {
      file: file,
      parentId: parentId as string,
    },
  });
}

const { executeMutation: uploadFile } = useMutation(
  graphql(`
    mutation uploadFile($input: FileUploadInput!) {
      uploadFile(input: $input) {
        id
      }
    }
  `)
);

async function clickFile(file: File) {
  if (file.fileType === "folder") {
    await router.push({
      name: "drive-my-drive-folders-folder",
      params: {
        id: file.id,
      },
    });
  } else {
    previewFile.value = file;
  }
}
</script>
