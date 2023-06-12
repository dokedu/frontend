<template>
  <div v-if="file" class="absolute left-0 top-0 z-50 h-screen w-full bg-stone-900/80" @click="emit('close')">
    <div class="flex h-full w-full items-center justify-between p-4">
      <img
        v-if="!file.name.includes('.pdf') && url"
        :src="url"
        alt=""
        class="mx-auto max-h-full rounded-lg bg-stone-900/10 object-contain backdrop-blur-sm"
      />
      <div v-if="file.name.includes('.pdf')" class="mx-auto h-fit max-h-full overflow-scroll rounded-lg shadow-xl">
        <canvas ref="canvas"></canvas>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { useMutation } from "@urql/vue";
import { graphql } from "../../gql";
import { File } from "../../gql/graphql";
import { ref, watch } from "vue";
import { onKeyStroke } from "@vueuse/core";

export interface Props {
  file: File | null;
}

const props = withDefaults(defineProps<Props>(), {
  file: null,
});

const emit = defineEmits(["close"]);

onKeyStroke("Escape", () => {
  emit("close");
});

const canvas = ref<HTMLCanvasElement | null>(null);
const url = ref<string | null>();

watch(
  () => props.file,
  async (file: File) => {
    if (file) {
      url.value = null;
      const { data } = await getFileURL({
        input: {
          id: file.id,
        },
      });
      url.value = data.generateFileURL.url;

      if (file.name.includes(".pdf")) {
        renderPDF(url.value);
      }
    }
  }
);

async function renderPDF(url) {
  const pdfjsLib = await import("pdfjs-dist/build/pdf");
  pdfjsLib.GlobalWorkerOptions.workerSrc = "/pdfjs-3.7.107-dist/build/pdf.worker.js";

  const loadingTask = pdfjsLib.getDocument(url);

  await loadingTask.promise.then(async (pdf) => {
    const page = await pdf.getPage(1);

    const viewport = page.getViewport({ scale: 1 });

    const context = canvas.value?.getContext("2d");

    if (context) {
      canvas.value!.height = viewport.height;
      canvas.value!.width = viewport.width;

      const renderContext = {
        canvasContext: context,
        viewport: viewport,
      };

      await page.render(renderContext);
    }
  });
}

const { executeMutation: getFileURL } = useMutation(
  graphql(`
    mutation generateFileURL($input: GenerateFileURLInput!) {
      generateFileURL(input: $input) {
        url
      }
    }
  `)
);
</script>