<template>
  <div class="select-none text-sm">
    <form @submit.prevent="onSubmit" class="mx-auto flex max-w-xs flex-col gap-2 py-24 text-strong">
      <div class="flex flex-col">
        <img height="67" width="100" class="mx-auto mb-8 w-2/5" src="/dokedu-logo.svg" alt="dokedu logo" />
        <div v-if="successBanner" class="mb-4 rounded-lg bg-green-100 p-2 text-green-800">
          <div><span class="text-green-950">Success!</span> Check your email for a link to reset your password.</div>
        </div>
        <label class="mb-1 text-xs text-stone-500" for="email">Email</label>
        <input
          v-model="email"
          type="email"
          name="email"
          id="email"
          required
          class="block w-full rounded-md border-0 py-2 text-stone-900 shadow-sm ring-1 ring-inset ring-stone-300 placeholder:text-stone-400 focus:ring-2 focus:ring-inset focus:ring-black sm:text-sm sm:leading-6"
          placeholder="Your email address"
        />
      </div>
      <button
        class="block rounded-md bg-black px-2.5 py-2.5 text-sm font-medium leading-none text-white shadow-sm hover:bg-stone-900 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-black"
        type="submit"
      >
        Send password reset email
      </button>
      <router-link
        class="mx-auto mt-2 block w-fit rounded-md text-center text-xs font-medium leading-none text-muted hover:text-default focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-black"
        to="/login"
      >
        Back to sign in
      </router-link>
    </form>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import { useMutation } from "@urql/vue";
import { graphql } from "@/gql";

const email = ref("");
const successBanner = ref(false);

const { executeMutation: forgotPassword } = useMutation(
  graphql(`
    mutation forgotPassword($input: ForgotPasswordInput!) {
      forgotPassword(input: $input) {
        success
      }
    }
  `)
);

async function onSubmit() {
  const { data } = await forgotPassword({
    input: {
      email: email.value,
    },
  });

  if (data?.forgotPassword.success) {
    successBanner.value = true;
    email.value = "";
  } else {
    alert("Something went wrong");
  }
}
</script>
