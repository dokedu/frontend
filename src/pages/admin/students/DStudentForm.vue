<template>
  <DSidebar :title="title" :delete="deletable" @cancel="onCancel" @delete="onDelete">
    <template #main>
      <div class="flex flex-col gap-2">
        <DInput name="firstName" :label="$t('first_name')" v-model="student.firstName"></DInput>
        <DInput name="lastName" :label="$t('last_name')" v-model="student.lastName"></DInput>
        <DInput
          type="number"
          v-if="student.student"
          :label="$t('grade')"
          :min="0"
          name="grade"
          v-model="student.student.grade"
        ></DInput>
        <DInput type="date" :label="$t('birthday')" name="birthday" v-model="birthday"></DInput>
        <DInput type="date" :label="$t('joined_at')" name="birthday" v-model="joinedAt"></DInput>
        <DInput type="date" :label="$t('left_at')" name="birthday" v-model="leftAt"></DInput>
      </div>
    </template>
    <template #footer>
      <div class="flex w-full justify-between">
        <DButton type="outline" size="md" @click="onCancel">{{ $t("cancel") }}</DButton>
        <DButton v-if="student.id" type="primary" size="md" @click="onSave">{{ $t("save") }}</DButton>
        <DButton v-else type="primary" size="md" @click="onSave">{{ $t("create") }}</DButton>
      </div>
    </template>
  </DSidebar>
</template>

<script lang="ts" setup>
import DSidebar from "@/components/d-sidebar/d-sidebar.vue";
import { User } from "@/gql/graphql";
import { useRouter } from "vue-router";
import DInput from "@/components/d-input/d-input.vue";
import DButton from "@/components/d-button/d-button.vue";
import { computed, toRef } from "vue";

const router = useRouter();

export interface Props {
  student: User;
  title: string;
  deletable?: boolean;
}
const props = defineProps<Props>();
const emit = defineEmits(["save", "delete"]);
const student = toRef(props, "student");

const birthday = computed({
  get: () => {
    if (student.value.student?.birthday) {
      return student.value.student.birthday.slice(0, 10);
    }
    return new Date(0);
  },
  set: (value) => {
    if (student.value.student) {
      student.value.student.birthday = new Date(value).toISOString();
    }
  },
});

const joinedAt = computed({
  get: () => {
    if (student.value.student?.joinedAt) {
      return student.value.student.joinedAt.slice(0, 10);
    }
    return new Date(0);
  },
  set: (value) => {
    if (student.value.student) {
      student.value.student.joinedAt = new Date(value).toISOString();
    }
  },
});

const leftAt = computed({
  get: () => {
    if (student.value.student?.leftAt) {
      return student.value.student.leftAt.slice(0, 10);
    }
    return new Date(0);
  },
  set: (value) => {
    if (student.value.student) {
      student.value.student.leftAt = new Date(value).toISOString();
    }
  },
});

const onCancel = () => {
  router.push({ name: "admin-students" });
};

const onDelete = () => {
  emit("delete");
};

const onSave = () => {
  emit("save");
};
</script>
